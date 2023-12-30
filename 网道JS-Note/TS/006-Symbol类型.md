[TOC]

# Symbol 类型

## 简介

Symbol 是 ES2015 新引入的一种原始类型的值。
它类似于字符串，但是每一个 Symbol 值都是独一无二的，与其他任何值都不相等。

```ts
// Symbol 值通过Symbol()函数生成
// 在 TypeScript 里面，Symbol 的类型使用symbol表示
let x: symbol = Symbol();
let y: symbol = Symbol();

x === y; // false
```

## unique symbol

symbol 类型包含所有的 Symbol 值，但是无法表示某一个具体的 Symbol 值。
为了解决这个问题，TypeScript 设计了 symbol 的一个子类型 unique symbol，它表示单个的、某个具体的 Symbol 值。

```ts
// 因为unique symbol表示单个值，所以这个类型的变量是不能修改值的，只能用const命令声明，不能用let声明
// 正确
const x: unique symbol = Symbol();
// 报错
let y: unique symbol = Symbol();
```

```ts
// const命令为变量赋值 Symbol 值时，变量类型默认就是unique symbol，所以类型可以省略不写
const x: unique symbol = Symbol();
// 等同于
const x = Symbol();
```

```ts
// 每个声明为unique symbol类型的变量，它们的值都是不一样的，其实属于两个值类型
const a: unique symbol = Symbol();
const b: unique symbol = Symbol();
a === b; // 报错

// 类似于字符串的值类型
const a: "hello" = "hello";
const b: "world" = "world";
a === b; // 报错
```

```ts
// 变量a和b是两个类型，就不能把一个赋值给另一个
const a: unique symbol = Symbol();
const b: unique symbol = a; // 报错

// 如果要写成与变量a同一个unique symbol值类型，只能写成类型为typeof a
const a: unique symbol = Symbol();
const b: typeof a = a; // 正确
```

```ts
// unique symbol 类型是 symbol 类型的子类型，所以可以将前者赋值给后者，但是反过来就不行
const a: unique symbol = Symbol();
const b: symbol = a; // 正确
const c: unique symbol = b; // 报错
```

```ts
// unique symbol 类型的一个作用，就是用作属性名，这可以保证不会跟其他属性名冲突。
// 如果要把某一个特定的 Symbol 值当作属性名，那么它的类型只能是 unique symbol，不能是 symbol。
const x: unique symbol = Symbol();
const y: symbol = Symbol();

interface Foo {
  [x]: string; // 正确
  [y]: string; // 报错
}
```

```ts
// unique symbol类型也可以用作类（class）的属性值，但只能赋值给类的readonly static属性
class C {
  static readonly foo: unique symbol = Symbol();
}
```

## 类型推断

```ts
// let命令声明的变量，推断类型为 symbol
let x = Symbol();

// const命令声明的变量，推断类型为 unique symbol
const x = Symbol();
```

```ts
// 但是 const 命令声明的变量，如果赋值为另一个 symbol 类型的变量，则推断类型为 symbol
let x = Symbol();

// 类型为 symbol
const y = x;
```

```ts
// let命令声明的变量，如果赋值为另一个 unique symbol 类型的变量，则推断类型还是 symbol
const x = Symbol();

// 类型为 symbol
let y = x;
```
