# Функции

## Необязательные и оставшиеся параметры

Необязательные параметры помечаются знаком вопроса (`?`) и должны следовать за обязательными

```ts
function concatStrings(a: string, b: string, c?: string) {
  return a + b + c;
}
```

Вариант синтаксиса необязательного параметра позволяет указать значение параметра по умолчанию

```ts
function concatStringsDefault(a: string, b: string, c?: string = "c") {
  return a + b + c;
}

console.log(concatStringsDefault("a", "b"));  // abc
```

Чтобы вызывать функцию с переменным числом аргументов необходимо использовать синтаксис ES6 (`...rest`) оставшихся параметров. И типизировать `rest` параметр, как массив значений

```ts
function restArguments(...argArray: number[]) {
  if (argArray > 0) {
    for (let i = 0; i < argArray.length; i++) {
      console.log(argArray[i]);
      // использовать rest аргумент, как массив [arguments] JavaScript
      console.log(arguments[i]);
    }
  }
}

restArguments(5);  // 5 5
restArguments(1, 2, 3);   // 1 1 2 2 3 3
```

Можно комбинировать обычные параметры с оставшимися в определении функции, оставшиеся параметры должны быть определены последними в списке параметров

```ts
function multiArguments(arg1: string, arg2: number, ...argArray: number[]) {
  // ...
}
```

## Сигнатуры типов функций

Для описания типа функции используется синтаксис выражения типа функции (Function Type Expressions) основанный на стрелочных функциях (Arrow Function). Имя параметра в сигнатуре выражения типа функции является обязательным. Если тип параметра не определен, неявно присваивается тип `any`.

Пример сигнатуры выражения типа функции

```ts
(arg: string) => void
```

так выглядит сигнатура выражения типа функции, когда функция является аргументом другой функции. Обычный прием описания типа для функций обратного вызова (callback function)

```ts
function greeter(func: (arg: string) => void) {
  func('Hello, World!');
}

function printToConsole(txt: string) console.log(txt);

greeter(printToConsole);  // Hello, World!
```

или используя псевдоним типа

```ts
type GreetFunction = (arg: string) => void;
function greeter(func: GreetFunction) {
  // ...
}
```

а так, если необходимо описать тип свойства функции. Обратите внимание, что синтаксис изменился и вместо `=>` используется `:`

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```
