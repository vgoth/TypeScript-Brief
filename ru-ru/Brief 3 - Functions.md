# Функции

Функции - основной строительный блок любого приложения. Будь то локальные функции, библиотеки функций или методы класса. Этот брифинг посвящен правилам аннотации типов функций.

## Необязательные параметры

Необязательные параметры помечаются знаком вопроса `?` и должны следовать за обязательными

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

## Оставшиеся параметры

Чтобы вызывать функцию с переменным числом аргументов необходимо использовать синтаксис `...` оставшихся параметров. И типизировать такой параметр, как массив значений

```ts
function restArguments(...argArray: number[]) {
  if (argArray > 0) {
    for (let i = 0; i < argArray.length; i++) {
      console.log(argArray[i]);

      console.log(arguments[i]);
      // использовать 'rest' аргумент, как обычный массив [arguments] JavaScript
      // такая возможность тоже поддерживается
    }
  }
}

restArguments(5);  // 5 5
restArguments(1, 2, 3);   // 1 1 2 2 3 3
```

Можно комбинировать обычные параметры с оставшимися в определении функции, оставшиеся параметры должны быть всегда последними в списке параметров

```ts
function multiArguments(arg1: string, arg2: number, ...argArray: number[]) {
  // ...
}
```

Когда аргументы передаются в функцию в виде массива, часто используют синтаксис распространения `spread`, но такой синтаксис может вызвать ошибку компиляции

```ts
const args = [8, 5];
const angle = Math.atan2(...args);  // Ошибка!
// Аргумент распространения должен иметь тип кортежа
// или передаваться параметру rest.
```

Решение зависит от вашего кода, но можно пойти простым путем, и просто привести весь массив к литеральному типу, используя `as const`

```ts
const args = [8, 5] as const;
```

## Деструктуризация параметров

Для деструктуризации объекта в параметрах функции, используют сигнатуру анонимного типа объекта

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}

sum({ a: 10, b: 3, c: 9 });
```

чтобы "причесать" код, можно воспользоваться псевдонимом типа

```ts
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  // ...
}
```

## Сигнатуры типов функций

Для описания функции - используется выражение типа функции (Function Type Expressions) основанное на стрелочных функциях (Arrow Function)

Пример сигнатуры выражения типа функции

```ts
(arg: string) => void
```

> Произвольное имя параметра в сигнатуре выражения типа функции является обязательным. Если тип параметра не определен, неявно присваивается тип `any`.

так выглядит сигнатура выражения типа функции, когда функция является аргументом другой функции. Типичный прием описания типа для функций обратного вызова (callback function)

```ts
function executeCallback(callback: (text: string) => void) {
  callback('Hey! Hey!');
}

function printToConsole(log: string): void {
  console.log(log);
}

executeCallback(printToConsole);  // Hey! Hey!
```

или используя псевдоним типа

```ts
type GreetFunction = (text: string) => void;
function executeCallback(callback: GreetFunction) {
  // ...
}
```

> При описании типа для функции обратного вызова никогда не используйте необязательные параметры, если вы не собираетесь вызывать функцию без передачи этого аргумента. Это может привести к критическим ошибкам.

используем такую сигнатуру, если необходимо описать тип свойства функции. Обратите внимание, что синтаксис изменился и вместо `=>` используется `:`

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function numToBoolean(arg: number) {
  return arg > 0 ? true : false;
}

numToBoolean.description = 'test';

doSomething(numToBoolean); // test returned true
```

сигнатура функции-конструктора, для вызова с оператором `new`

```ts
type SomeConstructor = {
  new (s: string): SomeObject;
};

function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

встроенные функции JavaScrip, такие как `Data()` можно вызывать с оператором `new` или без него. Для таких случаев, необходимо описать комбинированный тип

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

## Перегрузка функций

В TypeScript, что бы вызывать функцию с разными аргументами, необходимо описать все сигнатуры вызова, которые именуются сигнатурами перегрузки, и сигнатуру реализации. Сигнатуры перегрузки не имеют тела функции. Сигнатура реализации, должна быть совместима со всеми сигнатурами перегрузки и не может быть вызвана напрямую. Подразумевают, что сигнатура реализации, скрыта.

```ts
function addMisc(a: string, b: string): string; // 1-я сигнатура перегрузки
function addMisc(a: number, b: number): number; // 2-я сигнатура перегрузки
function addMisc(a: any, b: any): any {         // сигнатура реализации
  // в сигнатуре реализации типам параметров и возвращаемого значения,
  // в этом примере, присвоен тип 'any', для совместимости с обоими
  // вариантами вызова функции
  return a + b;
}

addMisc(1, 1);  // 2
addMisc('1', '1');  // 11
addMisc(true, false); // Ошибка! Тип аргумента не совместим с параметрами
```

более сложный пример с сужением типа в сигнатуре реализации

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}

const d1 = makeDate(12345678);  // 1970-01-01T03:25:45.678Z
const d2 = makeDate(5, 5, 5);   // 1905-06-04T21:29:43.000Z
const d3 = makeDate(1, 3);      // Ошибка!
// никакая перегрузка не ожидает двух аргументов, только 1 или 3
```

> Предпочитайте параметры с типом объединения вместо перегруженных функций, когда это только возможно. Неперегруженные версии функций - всегда будут более эффективными и безопасными.
