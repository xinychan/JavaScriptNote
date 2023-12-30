[TOC]

# Enum

## 简介

```ts
// 实际开发中，经常需要定义一组相关的常量。
const RED = 1;
const GREEN = 2;
const BLUE = 3;

let color = userInput();

if (color === RED) {
  // todo
}
if (color === GREEN) {
  // todo
}
if (color === BLUE) {
  // todo
}
```

```ts
// TypeScript 设计了 Enum 结构，用来将相关常量放在一个容器里面，方便使用。
enum Color {
  Red, // 0
  Green, // 1
  Blue, // 2
}

// 使用时，调用 Enum 的某个成员，与调用对象属性的写法一样，可以使用点运算符，也可以使用方括号运算符。
let c = Color.Green; // 1
// 等同于
let c = Color["Green"]; // 1

// Enum 结构本身也是一种类型。
// 变量c它的类型可以是 Color，也可以是number
let c: Color = Color.Green; // 正确
let c: number = Color.Green; // 正确
```

```ts
// Enum 结构的特别之处在于，它既是一种类型，也是一个值。
// 绝大多数 TypeScript 语法都是类型语法，编译后会全部去除。
// 但是 Enum 结构是一个值，编译后会变成 JavaScript 对象，留在代码中。

// 编译前
enum Color {
  Red, // 0
  Green, // 1
  Blue, // 2
}

// 编译后
let Color = {
  Red: 0,
  Green: 1,
  Blue: 2,
};
```

```ts
// 由于 Enum 结构编译后是一个对象，所以不能有与它同名的变量（包括对象、函数、类等）
enum Color {
  Red,
  Green,
  Blue,
}

const Color = "rgb"; // 报错
```

```ts
// 很大程度上，Enum 结构可以被对象的as const断言替代。
enum Foo {
  A,
  B,
  C,
}

// 对象Bar使用了as const断言，作用就是使得它的属性无法修改。
// 这样的话，Foo和Bar的行为就很类似了，前者完全可以用后者替代。
const Bar = {
  A: 0,
  B: 1,
  C: 2,
} as const;

if (x === Foo.A) {
}
// 等同于
if (x === Bar.A) {
}
```

## Enum 成员的值

```ts
// Enum 成员默认不必赋值，系统会从零开始逐一递增，按照顺序为每个成员赋值
enum Color {
  Red,
  Green,
  Blue,
}

// 等同于
enum Color {
  Red = 0,
  Green = 1,
  Blue = 2,
}
```

```ts
// 如果只设定第一个成员的值，后面成员的值就会从这个值开始递增。
enum Color {
  Red = 7,
  Green, // 8
  Blue, // 9
}

// 或者
enum Color {
  Red, // 0
  Green = 7,
  Blue, // 8
}
```

```ts
// 成员的值甚至可以相同
enum Color {
  Red = 0,
  Green = 0,
  Blue = 0,
}
```

```ts
// 成员的值可以是任意数值，但不能是大整数（Bigint）
enum Color {
  Red = 90,
  Green = 0.5,
  Blue = 7n, // 报错
}
```

```ts
// Enum 成员的值也可以使用计算式
enum Permission {
  UserRead = 1 << 8,
  UserWrite = 1 << 7,
  UserExecute = 1 << 6,
  GroupRead = 1 << 5,
  GroupWrite = 1 << 4,
  GroupExecute = 1 << 3,
  AllRead = 1 << 2,
  AllWrite = 1 << 1,
  AllExecute = 1 << 0,
}

// Enum 成员的值等于一个计算式，或者等于函数的返回值，都是正确的
enum Bool {
  No = 123,
  Yes = Math.random(),
}
```

```ts
// Enum 成员值都是只读的，不能重新赋值
enum Color {
  Red,
  Green,
  Blue,
}

Color.Red = 4; // 报错
```

```ts
// 常量枚举
// Enum 结构前面加了const关键字，所以编译产物里面就没有生成对应的对象
// 而是把所有 Enum 成员出现的场合，都替换成对应的常量
const enum Color {
  Red,
  Green,
  Blue,
}

const x = Color.Red;
const y = Color.Green;
const z = Color.Blue;

// 编译后
const x = 0; /* Color.Red */
const y = 1; /* Color.Green */
const z = 2; /* Color.Blue */
```

## 同名 Enum 的合并

```ts
// 多个同名的 Enum 结构会自动合并
enum Foo {
  A,
}

enum Foo {
  B = 1,
}

enum Foo {
  C = 2,
}

// 等同于
enum Foo {
  A,
  B = 1，
  C = 2
}
```

```ts
// Enum 结构合并时，只允许其中一个的首成员省略初始值，否则报错
enum Foo {
  A,
}

enum Foo {
  B, // 报错
}
```

```ts
// 同名 Enum 合并时，不能有同名成员，否则报错
enum Foo {
  A,
  B,
}

enum Foo {
  B = 1, // 报错
  C,
}
```

```ts
// 同名 Enum 合并前提是，所有定义必须同为 const 枚举或者非 const 枚举，不允许混合使用

// 正确
enum E {
  A,
}
enum E {
  B = 1,
}

// 正确
const enum E {
  A,
}
const enum E {
  B = 1,
}

// 报错
enum E {
  A,
}
const enum E {
  B = 1,
}
```

## 字符串 Enum

```ts
// Enum 成员的值除了设为数值，还可以设为字符串
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```

```ts
// 注意，字符串枚举的所有成员值，都必须显式设置。
// 如果没有显式设置，成员值默认为数值，且位置必须在字符串成员之前。
enum Foo {
  A, // 0
  B = "hello",
  C, // 报错
}
```

```ts
// Enum 成员可以是字符串和数值混合赋值
// 除了数值和字符串，Enum 成员不允许使用其他值（比如 Symbol 值）
enum Enum {
  One = "One",
  Two = "Two",
  Three = 3,
  Four = 4,
}
```

```ts
// 变量类型如果是字符串 Enum，就不能再赋值为字符串，这跟数值 Enum 不一样
enum MyEnum {
  One = "One",
  Two = "Two",
}

// 变量s的类型是MyEnum，再赋值为字符串就报错
let s = MyEnum.One;
s = "One"; // 报错
```

```ts
// 字符串 Enum 可以使用联合类型（union）代替
// 函数参数where属于联合类型，效果跟指定为字符串 Enum 是一样的
function move(where: "Up" | "Down" | "Left" | "Right") {
  // todo
}
```

```ts
// 字符串 Enum 的成员值，不能使用表达式赋值
enum MyEnum {
  A = "one",
  B = ["T", "w", "o"].join(""), // 报错
}
```

## keyof 运算符

```ts
// keyof 运算符可以取出 Enum 结构的所有成员名，作为联合类型返回
enum MyEnum {
  A = "a",
  B = "b",
}

// 'A'|'B'
type Foo = keyof typeof MyEnum;

// 注意，这里的typeof是必需的，否则keyof MyEnum相当于keyof string
type Foo = keyof MyEnum; // keyof string
```

```ts
// 如果要返回 Enum 所有的成员值，可以使用in运算符
enum MyEnum {
  A = "a",
  B = "b",
}

// { a: any, b: any }
type Foo = { [key in MyEnum]: any };
```

## 反向映射

```ts
// 数值 Enum 存在反向映射，即可以通过成员值获得成员名
enum Weekdays {
  Monday = 1,
  Tuesday,
  Wednesday, // 3
  Thursday,
  Friday,
  Saturday,
  Sunday,
}

// 成员值3取到对应的成员名Wednesday
console.log(Weekdays[3]); // Wednesday
```

```ts
// 这是因为 TypeScript 会将上面的 Enum 结构，编译成下面的 JavaScript 代码
var Weekdays;
(function (Weekdays) {
  Weekdays[(Weekdays["Monday"] = 1)] = "Monday";
  Weekdays[(Weekdays["Tuesday"] = 2)] = "Tuesday";
  Weekdays[(Weekdays["Wednesday"] = 3)] = "Wednesday";
  Weekdays[(Weekdays["Thursday"] = 4)] = "Thursday";
  Weekdays[(Weekdays["Friday"] = 5)] = "Friday";
  Weekdays[(Weekdays["Saturday"] = 6)] = "Saturday";
  Weekdays[(Weekdays["Sunday"] = 7)] = "Sunday";
})(Weekdays || (Weekdays = {}));

// 编译后的代码实际进行了两组赋值
Weekdays[(Weekdays["Monday"] = 1)] = "Monday";
// 等同于下面的代码
Weekdays["Monday"] = 1;
Weekdays[1] = "Monday";
```

```ts
// 反向映射只发生在数值 Enum，对于字符串 Enum，不存在反向映射。
// 这是因为字符串 Enum 编译后只有一组赋值。
enum MyEnum {
  A = "a",
  B = "b",
}

// 编译后
var MyEnum;
(function (MyEnum) {
  MyEnum["A"] = "a";
  MyEnum["B"] = "b";
})(MyEnum || (MyEnum = {}));
```
