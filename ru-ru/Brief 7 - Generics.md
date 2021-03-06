
# Обобщения

Что, если нам требуется написать блок кода, который работает с любым типом объекта. Мы могли бы использовать тип `any`, но тогда мы теряем контроль над типом. Можно использовать тип `unknown`, в этом случае, необходимо будет использовать технику охранника типов, чтобы обработать все возможные ошибки и варианты. Еще один подход - создать несколько типов для всех возможных вариантов, но тогда прийдется их все обработать и создавать отдельные функции или перегрузки функций для каждого типа.

К счастью, в TypeScript есть универсальное решение - это обобщения. Обобщения (Generics) - это способ написания кода, который будет работать с любым типом объекта, но при этом сохранит контроль над типом, и обеспечит безопасность и целостность данного типа.

Обобщения вносят в язык новую синтаксическую конструкцию `<type>` и понятие ***параметр типа***

```ts
interface IMagicBox<Type> { // Объявили, что готовы принять параметр типа
  contents: Type            // Место, где используем переданный тип
}                           // Переданный тип просто подменит собой 'Type'

// передали тип как аргумент
let a: IMagicBox<string>; // свойство a.contents приняло тип 'string'
a = { contents: 'text' };

let b: IMagicBox<number>;  // здесь тип 'number'
b = { contents: 1 };

// или так...
type SomeObj = { str: string; num: number; uni: string | number | boolean };
let c: IMagicBox<SomeObj> = { contents: { str: 'text', num: 10, uni: true } };

console.log(a.contents); // text
console.log(b.contents); // 1
console.log(c.contents.uni); // true
```

> Имя параметра типа может быть произвольным. По сложившейся традиции используют осмысленное название `<Type>` или аббревиатуру `<T>`. Последнее, распространенная практика.

## Универсальные типы

## Универсальные функции

Используя обобщения можно полностью избежать перегрузок функций, создавая универсальные функции

```ts
function setContents<T>(box: IMagicBox<T>, newContents: T) {
  box.contents = newContents;
}
```

незакончен...
