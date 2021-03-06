# Расширенные типы

## Перечисления

Перечисления `enum` – это тип, который обеспечивает решение проблемы специальных чисел.

```ts
enum Numbers {
  One,
  Two
}

console.log(Numbers.One);     // 0
console.log(Numbers.Two);     // 1
console.log(Numbers['Two']);  // 1
console.log(Numbers[0]);      // One
```

можно принудительно установить значения

```ts
enum Numbers {
  One = 1,
  Two = 2
}
```

или сделать значения строковыми

```ts
enum Numbers {
  One = "one",
  Two = "two"
}
```

Облегченный вариант типа enum – это `const enum`, который преобразуется компилятором к простой переменной. `const enum` был введен в основном по соображениям производительности.

```ts
const enum Numbers {
  One,
  Two
}

let constEnum = Numbers.One;

// в итоге транспилируется в такой JavaScript код:
var constEnum = 0 /* One */;
```

## Объединения

Тип объединения (Union Type) - это тип, образованный из двух или более типов, которые называют членами объединения. В объявлении объединенного типа используется символ вертикальной черты `|`, чтобы перечислить все типы, которые будут составлять новый тип

```ts
let unionType: string | number;

unionType = 1;      // OK
unionType = 'one';  // OK
unionType = true;   // Ошибка! Типа 'boolean' нет в объединении
```

## Охранники типов

TypeScript позволит вам делать что-то с объединением, только если это допустимо для каждого члена объединения. Чтобы использовать объединения и обеспечить проверку типов, применяется техника программирования под названием охранник типов. Охранник типов - это выражение, которое выполняет проверку типа в вашем коде, а затем гарантирует, что этот тип соответствует вашим ожиданиям.
> Более полную информацию по техникам обеспечения безопасности типов можно получить на официальном сайте по языку. Где этой теме посвящен отдельный раздел (Narrowing).

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
  // Ошибка! У типа 'number' нет метода toUpperCase()
}
```

Решение состоит в том, чтобы использовать охранник типов и вывести более конкретный тип (сузить тип) для значения на основе структуры кода

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // Эта ветка используется, если id имеет тип 'string'
    console.log(id.toUpperCase());
  } else {
    // Здесь id имеет тип 'number'.
    console.log(id);
  }
}
```

еще пример, когда для сужения типа используется метод массива Array.isArray()

```ts
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Используем эту ветку, если x - это массив строк
    console.log("Hello, " + x.join(" and "));
  } else {
    // Здесь x простая строка
    console.log("Welcome lone traveler " + x);
  }
}
```

Иногда у вас будет объединение, в котором у всех членов есть что-то общее. Например, и у массивов, и у строк есть метод slice. Тогда охранник типов вам не понадобиться

```ts
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> Всегда продумывайте логику программы, которая будет работать с объединенными типами.

## Псевдонимы типов

Для удобства использования объединений и сложных типов в TypeScript была введена конструкция `type` - псевдоним типа.
> Фактически, вы можете использовать псевдоним типа, чтобы дать имя любому типу, в том числе, и примитивному.

Назначим псевдоним объекту

```ts
type Point = {
  x: number;
  y: number;
};

function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
```

или объединению

```ts
type ID = number | string;
```

## Интерфейсы

Интерфейс в TypeScript имеет двойственную природу. С одной стороны - это еще один способ назвать тип объекта, подобный псевдониму типа, с другой - механизм определения интерфейсов в соответствии с ООП парадигмой. Когда, если класс придерживается интерфейса, говорят, что он реализует данный интерфейс (кон­трак­т вза­и­мо­действия объектов).&#185;
> Подробнее об интерфейсах в отдельном брифинге.

Основное различие между псевдонимами типа и интерфейсами, состоит в том, что интерфейс можно расширить после его определения, а псевдоним типа - нет.
Интерфейсы могут использоваться только для объявления форм объектов, но не для переименования примитивов.

Объявление интерфейса, как определение формы объекта

```ts
interface IPoint {
  x: number;
  y: number;
}
 
function printCoord(pt: IPoint) {
  // ...
}
```

> Обратите внимание, что в сигнатуре объявления интерфейса, нет оператора `=` присваивания, как в синтаксисе псевдонима типа

## Литеральные типы

В дополнение к примитивным типам `string` и `number` можно ссылаться на конкретные строки и числа в их позициях. Которые именуются строковыми и числовыми литералами.

```ts
let x: "hello";
x = "hello";  // OK
x = "bye";  // Ошибка! Тип 'bye' не назначен типу 'hello'.
```

Объединяя литеральные типы в объединения, можно выразить гораздо более полезную концепцию.

Пример функции, которая принимает только определенный набор известных значений

```ts
function printText(alignment: "left" | "right" | "center") {
  // ...
}
printText("left");  // OK

printText("centre"); // Ошибка!
// В аргументе "center" написан с ошибкой
```

так же работают числовые литералы

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

можно комбинировать литеральные типы с другими типами

```ts
interface IOptions {
  width: number;
}
function configure(x: IOptions | "auto") {
  // ...
}
```

Еще один способ применения литеральных типов - это литеральные интерфейсы. Они применяются, когда значению свойства объекта, необходимо явно назначить литеральный тип.

В данном примере, значение свойства "method" явно приводится к строковому литералу "GET". В ином случае, если этого не сделать, TypeScript неявно назначит свойству тип `string`, что приведет к ошибке при работе с объектом

```ts
const req = { url: "https://example.com", method: "GET" as "GET" };
handleRequest(req.url, req.method);
```

или так

```ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method as "GET");
```

Можно использовать синтаксическую конструкцию `as const`, чтобы привести значения всех свойства объекта к литеральному типу

```ts
const req = { url: "https://example.com", method: "GET" } as const;
```

> Есть еще один тип литералов: логические литералы. Существует только два типа логических литералов:  true и false. Сам тип boolean, на самом деле, является просто псевдонимом для объединения true | false.

## Кортежи

Тип кортеж (Tuple) – это массив типов с заданным количеством и порядком элементов.

```ts
let tupleType: [string, number];

tupleType = ['rabbit', 1];     // OK
tupleType = [1, 'rabbit'];     // Ошибка! Нарушен порядок типов элементов
tupleType = ['rabbit'];        // Ошибка! У типа должно быть два элемента
tupleType = ['rabbit', true];  // Ошибка! Типа 'boolean' нет в кортеже
```

доступ к значению элементов кортежа по индексу, как к обычному массиву

```ts
console.log(tupleType[0]);     // rabbit
console.log(tupleType[2]);     // undefined
console.log(tupleType.length); // 2
```

с использованием синтаксиса деконструкции массива JavaScript

```ts
let [t1, t2] = tupleType;

console.log(t1); // rabbit
console.log(t2); // 1
```

у кортежа могут быть необязательные элементы. Они всегда следуют в конце определения и влияют на свойство length

```ts
let optionalTuple: [string, number?];

optionalTuple = ['rabbit'];         // ОК
console.log(optionalTuple.length);  // 1
```

пример использования кортежа с оператором `...` оставшихся параметров

```ts
function tupleAsRestArg (...args: [number, string, boolean]) {
  // ...
}
tupleAsRestArg(10, 'rabbits', true);
```

используя синтаксис `rest` в определении кортежа. В этом случае, оставшиеся параметры могут использоваться в любом месте определения, и несколько раз

```ts
type restTuple = [number, ...string[], boolean];
let rabbitsTuple: restTuple = [3, 'GeorgeI', 'GeorgeII', 'GeorgeIII', true];
```

> Такая сигнатура, скорее подходит под определение `spread` синтаксиса распространения, но так заявлено в официальной документации.

с модификатором доступа `readonly`, когда запрещена запись в любой элемент кортежа

```ts
function defArgs(args: readonly [string, number]) {
  args[0] = 'rabbit'; // Ошибка! Запись в элемент кортежа запрещена
}
```

## Never

Тип `never` введен для представления состояния, которое никогда не должно произойти. Ни один тип не может быть назначен `never`, кроме самого себя. Данный тип используется для контроля логики исполняемого кода. Компилятор присваивает и возвращает тип `never`, когда что-то никогда не может быть достигнуто (принцип достижимости).

Такая функция никогда, ничего не вернет

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

В этом примере, компилятор выдаст ошибку, когда не все возможные случаи были обработаны и выполнение программы дойдет до попытки присвоить `never` другой тип.

```ts
type ShapeType = 'Circle' | 'Square';

function getShape(shape: ShapeType): string {
  switch (shape) {
    case 'Circle': return 'This is Circle!';
  }
  const _checkOnNever: never = shape; // Ошибка!
  // Тип 'string' не может быть назначен типу 'never'
}
```

исправим ситуацию и обработаем все возможные случаи, так мы контролируем целостность кода и охраняем тип

```ts
function getShape(shape: ShapeType): string {
  switch (shape) {
    case 'Circle': return 'This is Circle!';
    case 'Square': return 'This is Square!';
    // Добавили вторую ветвь и обработали тип полностью
  }
  const _checkOnNever: never = shape;
  // теперь, до этой строки кода выполнение не дойдет, и ошибки не будет
}
```

## Unknown

Тип `unknown` можно рассматривать как безопасный эквивалент типа `any`. Прежде чем использовать переменную, с типом `unknown`, ее необходимо явно привести к известному типу.

```ts
let unknownType: unknown = 'rabbit';
unknownType = 1;  // Допустимо: тип ведет себя, как тип 'any'

let numberType: number;
numberType = unknownType; // Ошибка! Тип `unknown' нельзя назначить типу 'number'
```

приведем переменную явно к нужному типу (осмысленное, контролируемое действие)

```ts
numberType = <number>unknownType; // OK, так корректно
```

Данный тип эффективно использовать, когда тип данных заранее неизвестен и определяется динамически, во время исполнения. Например, функция которая принимает и обрабатывает строку в JSON формате, на выходе будет иметь объекты разного типа

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
```

или для определения формы сложных объектов, когда тип значения свойства может изменяться

```ts
interface IComplexObject {
  [key: string]: unknown
}

function safeStringify(obj: IComplexObject): string {
  return JSON.stringify(obj);
}
```

> Используйте тип `unknown`, когда возникает необходимость в `any`. Это позволяет сохранить контроль над типами данных и избежать ошибок времени исполнения. Либо используйте `interface` для определения сложных структур данных.
