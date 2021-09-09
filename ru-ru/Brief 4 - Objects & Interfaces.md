# Объекты и Интерфейсы

* Необязательные свойства
* Слабые типы
* Модификатор readonly
* Индексированные свойства
* Расширение интерфейсов
* Пересечение типов

Интерфейс - это способ определения пользовательского типа. Если объект придерживается интерфейса, говорят, что объект реализует интерфейс. Интерфейсы являются синтаксическим сахаром TypeScript и удаляются из результирующего кода в процессе компиляции.

```ts
interface IUser {
  id: number,
  name: string
}

const user: IUser;  // применили интерфейс к переменной
user = { id: 1, name: 'John' }; // создали экземпляр объекта типа IUser
```

 если попытаться присвоить переменной объект не полностью реализующий интерфейс, получим ошибку

```ts
user = {id: 1}; // Ошибка!
// Объект не может быть назначен типу IUser, отсутствует свойство 'name'
```

## Необязательные свойства

Можно  определить часть или все свойства необязательными

```ts
interface IOptionalUser {
  id: number,
  name?: string
}

let id: IOptionalUser = { id: 1 };  // OK

let user: IOptionalUser = { id: 2, name: 'John' };
user = id;  // OK! Такой трюк тоже стал возможен
```

> Мы также можем читать из таких свойств, но если конфигурация компилятора установлена в strictNullChecks, то TypeScript сообщит нам, что они undefined.

## Слабые типы

Когда все свойства интерфейса определены как необязательные, говорят, что это слабый тип. И возникает вопрос: объект реализует такой интерфейс? Правильный ответ: да, реализует. Компилятор настаивает, что должна быть реализована хотя бы часть соглашения

```ts
interface IWeak {
  product?: string,
  productName?: string
}

const productList: IWeak;
productList = { type: 'vehicle' };  // Ошибка!
// Объект не может быть назначен типу 'IWeak'. У типа нет свойства 'type'
```

## Модификатор `readonly`

Свойства объекта могут быть помечены как `readonly`. Компилятор проверяет, чтобы в такие свойства не производилась запись, только чтение. Надо понимать, что такой модификатор доступа актуален только на этапе компиляции,  а не времени исполнения.

```ts
interface IStep {
  readonly counter: number
}

function doSomething(obj: IStep) {
  console.log(obj.counter); // OK! читать можем
  obj.counter++;  // Ошибка! Изменение и запись в свойство запрещены
}
```

но, если значением свойства является другой объект, то ограничения на запись в него уже не действуют. Хотя изменить само свойство мы по прежнему не можем

```ts
interface IStep {
  readonly step: { counter: number }
}

function doSomething(obj: IStep) {
  obj.step.counter++; // OK
  obj.step.counter = 5; // OK
  obj.step = { counter: 5 };  // Ошибка! Изменение и запись в свойство запрещены
}
```

TypeScript не учитывает, являются ли свойства двух типов доступными только для чтения при проверке совместимости этих типов, поэтому свойства, доступные только для чтения, также могут изменяться через псевдоним.

```ts
interface Person {
  name: string;
  age: number;
}
 
interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}
 
let writablePerson: Person = {
  name: 'John',
  age: 33,
};
 
let readonlyPerson: ReadonlyPerson = writablePerson;
// 'readonlyPerson' выступает как ссылка на 'writablePerson'
// по существу являясь его псевдонимом
 
console.log(readonlyPerson.age); // 33

writablePerson.age++;
console.log(readonlyPerson.age); // 34
```

## Индексированные свойства

Иногда вы не знаете заранее все имена свойств типа, но знаете форму значений. В этих случаях, можно использовать сигнатуру индекса, для описания типов возможных значений

```ts
interface INumberIndexType {
  [index: number]: string;
}

const someArray: INumberIndexType = { 1: 'oneRabbit', 2: 'twoRabbit' };
console.log(someArray[1]);  // oneRabbit
```

Типом в сигнатуре индексированного свойства могут быть `number` или `string`

```ts
interface IStringIndexType {
  [index: string]: boolean;
}

const someDict: IStringIndexType = { oneRabbit: true, twoRabbit: false };
console.log(someDict.oneRabbit);  // true
```

Индексированные типы хорошо подходят для описания таких структур данных как словари. Для этого, помимо индексных свойств, определяют дополнительные, обычные свойства

```ts
interface IDictionary {
  [index: string]: number | string;
  length: number;
  name: string;
}

const dictionary: IDictionary = { a: 15, b: 22, c: 38, name: 'abc', length: 3 };
```

> Обратите внимание, что возвращаемый тип в сигнатуре индекса, в данном примере, определен как объединение нескольких типов. Это важно! Все типы возвращаемых значений объекта, это касается, как индексированных, так и обычных свойств, - должны быть определены явно в сигнатуре индексного свойства. Либо использованы типы `any` и `unknown` или `обобщения`.

можно использовать модификатор `readonly` с индексированными свойствами

```ts
interface IReadonlyIndexType {
  readonly [index: number]: string;
}
 
const someArray: IReadonlyIndexType = { 1: 'oneRabbit', 2: 'twoRabbit' };
someArray[1] = 'Wolf'; // Ошибка!
// Сигнатура индекса в типе IReadonlyIndexType разрешает только чтение.
```

## Расширение интерфейсов

При необходимости можно расширить (наследовать) любой интерфейс и добавить в него новые члены используя ключевое слово `extends`

```ts
interface IUser {
  id: number;
  name: string;
}

interface IUserWithAdmin extends IUser {
  admin: boolean;
}

const user: IUserWithAdmin = {
  id: 1,
  name: 'John',
  admin: true
}
```

или объединить несколько интерфейсов в один, создав новый тип

```ts
interface IColor {
  nameColor: string
}

interface IColorHexDecimal {
  hexColor: string
}

interface IColorful extends IColor, IColorHexDecimal {}

const color: IColorful = {
  nameColor: 'Red',
  hexColor: '#ff0000'
};
```

## Пересечения типов

В дополнение к расширению интерфейсов, в TypeScript существует конструкция `&`, позволяющая объединить несколько типов в один производный

```ts
interface IColor {
  nameColor: string
}

interface IColorHexDecimal {
  hexColor: string
}

type Colorful = IColor & IColorHexDecimal;
```

> Основная разница между подходами, заключается в том, как обрабатываются конфликты (сообщения компилятора). И это различие, а также естественная логика кода, определяют выбор той или иной конструкции, будь то расширение интерфейсов или пересечение типов.
