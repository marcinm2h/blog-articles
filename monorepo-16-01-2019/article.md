# Tworzymy monorepozytorium z yarn workspaces krok po kroku

Pracując w dużym projekcie javascriptowym z czasem jak aplikacja rośnie możesz dojść do punktu w którym będziesz chciał podzielić ją na mniejsze części.

Reactową aplikację oddzielić od backendu w express, podzielić duże api na mikroserwisy, czy wynieść reużywane funkcje pomocnicze jako bibliotekę.

Aby nie tworzyć wielkiego ciężkiego w utrzymaniu monolitu kodu z pomocą przychodzi nam (nomen omen) **monorepozytorium**.
Jest to połączenie wielu odseparowanych pakietów (aplikacji) w jednym repo.

Zacznijmy od przykłądu aby wiedzieć z czym mamy do czynienia.

## Przykład
Projekt zawiera Node'owy serwer wystawiający RESTowe api (pakiet **api**), aplikację kliencką (pakiet **front**), a oba korzystają z funkcji pomocniczych (pakiet **utils**).
W ramach projektu chcemy mieć jedną konfigurację do przeprowadzania testów.

`TODO: dodać diagram z rozrysowanym repo`


## Yarn workspaces
Na początku utwórzmy nowy projekt.
```bash
mkdir app && cd app
yarn init -y
```

Aby podzielić go na pakiety użyjemy [yarn workspaces](https://yarnpkg.com/lang/en/docs/workspaces/). Do package.json dodaj:
```json
  "private": true,
  "workspaces": ["packages/*"]
```
\- folder packages będzie katalogiem na pakiety aplikacji (workspace'y).

Następnie utwórzmy powyższe pakiety:
```bash
mkdir -p packages/api && cd packages/app
yarn init -y
```
oraz w ten sam sposób dla **front** i **utils**.

W tym momencie struktura plików powinna wyglądać tak:
```
|-- package.json      //główny package.json - tu będziemy umieszczać reużywalne dev depedency (np. eslint)
`-- packages      //folder pakietów (skonfigurowany w głównym package.json)
    |-- api
    |   `-- package.json      //package.json pakietu api
    |-- front
    |   `-- package.json      //package.json pakietu front
    `-- utils
        `-- package.json      //package.json pakietu utils
```
## Utils
Przetestujmy jak yarn radzi sobie z zależnościami między pakietami.
W pakiecie **utils** utwórzmy funkcję: 
```javascript
//packages/utils/index.js
function objectKeys(object) {
  return Object.keys(object);
}

module.exports = { objectKeys };
```
 z której skorzystamy w **api** i **front**. W tym celu musimy dodać **utils** jako ich depedency:
 
 ```json
 //w packages/api/package.json i packages/front/package.json
  "depedencies": {
    "utils": "1.0.0"
  }
 ```
Następnie wymuśmy aby yarn utworzył sobie odpowiednie linki w node_modules.

W głównym katalogu wywołaj komendę:
```bash
yarn install
```

Teraz przetestujmy czy z pakietu api mamy dostęp do utils:
```javascript
//packages/api/index.js
const { objectKeys } = require('utils');
console.log(objectKeys({ a: 'a' }));
```
Taki plik uruchamiam node i poprawnie loguje mi:
```bash
$ node index.js 
[ 'a' ]
``` 
Tego samego efektu spodziewam się uruchamiając taki plik w pakiecie **front**.

Udało nam się więc utworzyć lokalne pakiety wewnątrz jednego repozytorium i uzależnić je od siebie.

## Depedencies - zewnętrzne zależności
Gdzie teraz trzymać zewnętrzne zależności? W naszym przykładzie api będzie wystawione przez express, więc dodajmy tę zależność wyłącznie dla pakietu **api**.
```bash
cd packages/api
yarn add express
```
Teraz przetestujmy serwer przykładem z https://expressjs.com/en/starter/hello-world.html.
```bash
$ node server.js 
Example app listening on port 3000!
```
Działa! 

(Uwaga - jako, że moduły i tak zostaną [hoistowane do głównego folderu node_modules](https://yarnpkg.com/blog/2018/02/15/nohoist/) w innych pakietach [front, utils] możesz użyć express, ale zawsze dodawaj zależności jawnie w packages.json)

## DevDepedencies - wspólna konfiguracja eslint
Prawdopodobnie wszystkie nasze pakiety będą wymagały testowania. Z tego powodu zależności deweloperskie dobrze będzie trzymać na głównym poziomie repozytorium.
Zainstalujmy więc dla przykładu eslint.
W tym celu w głównym katalogu wywołaj:
```bash
yarn add --dev -W eslint
# flaga '-W' pozwala na instalację zależności w głownym katalogu  
```
Komenda ta doda nam do głównego package.json pole:
```json
  "devDependencies": {
    "eslint": "^5.12.0"
  }
```
Teraz dodajmy je także w katalogach podrzędnych, aby miały jawne zależności:
```bash
cd packages/api
yarn add --dev eslint
```
i to samo w pakietach **front** i **utils**.

Wróćmy do głównego katalogu i wygenerujmy konfigurację eslint przez:
```bash
node_modules/.bin/eslint --init
```
Z tej konfiguracji korzystać będą wszystkie pakiety repozytorium.

Teraz tylko dodajmy standardowo skrypt lintowania do package.json każdego z poszczególnych pakietów:
```json
// w packages/api/package.json,  packages/front/package.json,  i  packages/utils/package.json
"scripts": {
  "lint": "eslint *.js"
}
```

Uruchamiając go komendą:
```bash
yarn lint
```
powinieneś zobaczyć błędy (lub komunikat o ich braku) co świadczy o poprawnie odczytanej konfiguracji z katalogu głównego.

## Uruchamianie wszystkich skryptów -  yarn workspaces run
Kiedy będziesz chciał zbudować projekt przez narzędzie CI, przydatna będzie jeszcze jedna komenda yarna.

```bash
yarn workspaces run
```
Służy do uruchamiania wszystkich skryptów z pakietów repozytorium.

Jeśli w poprzednim punkcie dodałeś do "scripts" skrypt lintowania w packages/api/package.json,  packages/front/package.json,  i  packages/utils/package.json, to wywołanie:

```bash
yarn workspaces run lint
```
spowoduje uruchomienia każdego z nich.

Dla wygody warto dodać te polecenie do package.json na głównym poziomie aplikacji:

```json
// główny package.json
"scripts": {
  "lint": "yarn workspaces run lint"
}
```
teraz w narzędziu CI którego używasz możesz dodać skrypty lintowania / testów i budowania z poziomu całej aplikacji:
```bash
yarn lint #test, build etc.
```

---

## tl;dr
- monorepozytotium pozwala podzielić projekt na mniejsze pakiety
- do tworzenia monorepo wystarczy yarn z [yarn workspaces](https://yarnpkg.com/lang/en/docs/workspaces/)
- **package.json w głównym katalogu** służy do zarządzania zależnościami deweloperskimi (np. eslint) i tam też trzymana jest ich konfiguracja
- pakiety monorepozytorium są uzależniane od siebie przez **własne package.json**
- pakiety monorepozytorium są uzależniane od zewnętrznych pakietów node przez **własne package.json**
-  do uruchamiania skryptów we wszystkich pakietach równocześnie używamy **yarn workspaces run**

---
## Tagi //FIXME - przerobić na format gatsbyjs

- monorepo
- yarn workspaces

