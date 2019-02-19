---
title: var, let czy const? - jak deklarować zmienne w JavaScript?
date: '2019-01-18'
---

Specyfikacja ES6 wprowadziła nowe sposoby deklarowania zmiennych - obok słowa kluczowego `var` pojawiły się dodatkowo `let` i `const`.

Po co nam aż dwa nowe słowa kluczowe i którego z nich powinniśmy używać?
O tym za chwilę. Najpierw zacznijmy od przypomnienia sobie jak działa `var`.

## Zasięg var
Jaki jest zasięg zmiennych w JavaScript?
Prześledźmy przykład:

```javascript
function increment(state = 0) {
  var answer = state; // 0

  if (true) {
    var answer = 42;
    console.log(answer); // 42
  }

  answer += 1;
  console.log(answer); // 43 (?!)
}
```
### Co się stało?

Zmienna `answer` wewnątrz bloku `if` nadpisała `answer` zadeklarowane na samym początku funkcji (zauważ, że w obu przypadkach użyte zostało słowo kluczowe `var`).

Dzieje się tak dlatego, że **zmienne deklarowane przez `var` mają zasięg funkcyjny**.

Wewnątrz funkcji `increment` zmienna `answer` została utworzona jedynie raz - na górze jej ciała.
W bloku if (mimo użycia `var`) odwołujemy się wciąż do tej samej zmiennej.
Jak widać nietrudno tu o błąd. 

## ES6 na ratunek
Nowe sposoby deklarowania zmiennych powstały nie bez powodu. Wielu programistów przyzwyczajona jest do zasięgu blokowego.
To zachowanie zostało zaimplementowane w ES6.

**Zmienne deklarowane przez `const` i `let` mają zasięg blokowy**.

Scope zmiennej wyznaczają wszyskie bloki między klamrami `{}` - w tym funkcja i blok `if`.

Zmieńmy w naszym przykładzie `var` na ich nowocześniejsze odpowiedniki:
```javascript
const debugging = true;

function increment(state = 0) {
  let answer = state; // 0

  if (debugging) {
    let answer = 42; 
    console.log(answer); // 42
  }

  answer += 1;
  console.log(answer); //1 (!)
}
```
Tym razem wynik zgodny jest z oczekiwaniem - w bloku `if` tworzona jest całkiem nowa zmienna o tej samej nazwie.

## Właściwości let i const
Poza zasięgiem blokowym, `let` i `const` mają jeszcze kilka właściwości odróżniających je od `var`:

- W bloku kodu nie można dwukrotnie zadeklarować zmiennej o tej samej nazwie:
```javascript
function testLet1() {
  let a = 'a';
  let a = 'b'; // SyntaxError: Identifier 'a' has already been declared
}
```
```javascript
function testConst1() {
  const a = 'a';
  const a = 'b'; // SyntaxError: Identifier 'a' has already been declared
}
```

- Zmienne te w odróżnieniu od `var` nie są inicjalizowane:
```javascript
function testVar1() {
  console.log(hello); // undefined - zmienna hello została zainicjalizowana
  var hello = 'hello';
}
```

Zmienna `hello` zainicjalizowana wartością `undefined`. Przykład jest równoznaczny z zapisem:
```javascript
function testVar2() {
  var hello = undefined;
  console.log(hello); // undefined
  hello = 'hello';
}
```

Tymczasem w ES6 analogiczny zapis spowoduje błąd:
```javascript
function testConst2() {
  console.log(hello);
  const hello = 'hello'; // ReferenceError: hello is not defined
}
```

## Czym let różni się od const?
OK, wiemy już w czym `let` i `const` są lepsze od `var`. Czym zatem różnią się od siebie?
Bardzo proste - `let` można redeklarować, a `const` nie (są stałymi):
```javascript
  function testLet2() {
    let counter = 0;
    counter += 1;
    console.log(counter); // 1
  }
  function testConst3() {
    const counter = 0;
    counter += 1; // TypeError: Assignment to constant variable
  }
```
Próba zmiany stałej throwuje błąd.

Pamiętaj jednak, że niemożliwość redeklaracji nie znaczy niemutowalność.
Jeśli do zmiennej `const` przypiszemy obiekt - jego pola wciąż możemy zmieniać:
```javascript
function testConst3() {
  const user = { login: 'Harry' };
  user.login = 'Ron';
  console.log(user.login); // Ron
}
```

## Więc w końcu: -var, let czy const?
Przede wszystkim pisząc kod ES6 porzuć całkowicie `var` - używanie go może jedynie powodować trudne do przewidzenia błędy (sprawdź popularne niegdyś zadanie rekrutacyjne: [setTimeout w pętli](https://wesbos.com/for-of-es6/)).

Dobrą praktyką jest zawsze zaczynanie deklaracji zmiennej od `const`.
Dopiero w momencie kiedy będziemy potrzebować ją zmienić - zmienić też jej deklarację na `let`
(z oczywistego powodu - licznik pętli zawsze ustawimy na let).

Jeśli chcesz bardziej doglębnie wejść w temat polecam 2gi rozdział książki [You Don't Know JS: ES6 & beyond](https://github.com/getify/You-Dont-Know-JS/blob/master/es6%20%26%20beyond/ch2.md#block-scoped-declarations).

---

## tl;dr
- pisząc kod ES6 nie używaj `var`
- `let` i `const` mają zasięg blokowy (block scope) - scope wyznaczają bloki kodu między klamrami: `{}`
- `const` nie może być redeklarowany (co nie oznacza, że nie można zmienić pól obiektu który zostanie mu przypisany)
- dobrą praktyką jest zaczynanie deklaracji zmiennej od `const`, a w momencie kiedy trzeba będzie ją zmienić (np. jeśli będzie ona licznikiem pętli) - użycie `let`
