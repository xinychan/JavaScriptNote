[TOC]

# interface

## 简介

```ts
// interface 是对象的模板，可以看作是一种类型约定，中文译为“接口”
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

// 实现该接口，指定它作为对象的类型
const p: Person = {
  firstName: "John",
  lastName: "Smith",
  age: 25,
};
```

```ts
// 方括号运算符可以取出 interface 某个属性的类型
interface Foo {
  a: string;
}

// Foo['a']返回属性a的类型
type A = Foo["a"]; // string
```

### interface 表示对象的各种语法

interface 可以表示对象的各种语法，它的成员有 5 种形式。

- 对象属性
- 对象的属性索引
- 对象方法
- 函数
- 构造函数

#### 对象属性

```ts
// x和y都是对象的属性，分别使用冒号指定每个属性的类型
interface Point {
  x: number;
  y: number;
}
```

#### 对象的属性索引

```ts
// [prop: string]就是属性的字符串索引，表示属性名只要是字符串，都符合类型要求
interface A {
  [prop: string]: number;
}
```

#### 对象方法

```ts
// 对象的方法共有三种写法

// 写法一
interface A {
  f(x: boolean): string;
}

// 写法二
interface B {
  f: (x: boolean) => string;
}

// 写法三
interface C {
  f: { (x: boolean): string };
}
```

#### 函数

```ts
// interface 也可以用来声明独立的函数
interface Add {
  (x: number, y: number): number;
}

const myAdd: Add = (x, y) => x + y;
```

#### 构造函数

```ts
// interface 内部可以使用new关键字，表示构造函数
interface ErrorConstructor {
  new (message?: string): Error;
}
```

## interface 的继承

### interface 继承 interface

```ts
// interface 可以使用extends关键字，继承其他 interface
interface Shape {
  name: string;
}

interface Circle extends Shape {
  radius: number;
}
```

```ts
// interface 允许多重继承
interface Style {
  color: string;
}

interface Shape {
  name: string;
}

interface Circle extends Style, Shape {
  radius: number;
}
```

```ts
// 如果子接口与父接口存在同名属性，那么子接口的属性会覆盖父接口的属性。
// 注意，子接口与父接口的同名属性必须是类型兼容的，不能有冲突，否则会报错。
interface Foo {
  id: string;
}

interface Bar extends Foo {
  id: number; // 报错
}
```

```ts
// 多重继承时，如果多个父接口存在同名属性，那么这些同名属性不能有类型冲突，否则会报错。
interface Foo {
  id: string;
}

interface Bar {
  id: number;
}

// 报错
interface Baz extends Foo, Bar {
  type: string;
}
```

### interface 继承 type

```ts
// interface 可以继承type命令定义的对象类型
// 注意，如果type命令定义的类型不是对象，interface 就无法继承
type Country = {
  name: string;
  capital: string;
};

interface CountryWithPop extends Country {
  population: number;
}
```

### interface 继承 class

```ts
// interface 还可以继承 class，即继承该类的所有成员
class A {
  x: string = "";

  y(): boolean {
    return true;
  }
}

interface B extends A {
  z: number;
}

// 实现B接口的对象就需要实现这些属性
const b: B = {
  x: "",
  y: function () {
    return true;
  },
  z: 123,
};
```

## 接口合并

多个同名接口会合并成一个接口

```ts
// 如下两个Box接口会合并成一个接口，同时有height、width和length三个属性
// 这样的设计主要是为了兼容 JavaScript 的行为。
// JavaScript 开发者常常对全局对象或者外部库，添加自己的属性和方法。
// 那么，只要使用 interface 给出这些自定义属性和方法的类型，就能自动跟原始的 interface 合并，使得扩展外部类型非常方便。
interface Box {
  height: number;
  width: number;
}

interface Box {
  length: number;
}
```

```ts
// 同名接口合并时，同一个属性如果有多个类型声明，彼此不能有类型冲突
interface A {
  a: number;
}

interface A {
  a: string; // 报错
}
```

```ts
// 同名接口合并时，如果同名方法有不同的类型声明，那么会发生函数重载。
// 而且，后面的定义比前面的定义具有更高的优先级。
interface Cloner {
  clone(animal: Animal): Animal;
}

interface Cloner {
  clone(animal: Sheep): Sheep;
}

interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
}

// 等同于
interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
  clone(animal: Sheep): Sheep;
  clone(animal: Animal): Animal;
}
```

```ts
// 这个规则有一个例外。同名方法之中，如果有一个参数是字面量类型，字面量类型有更高的优先级。
interface A {
  f(x: "foo"): boolean;
}

interface A {
  f(x: any): void;
}

// 等同于
interface A {
  f(x: "foo"): boolean;
  f(x: any): void;
}
```

```ts
// 如果两个 interface 组成的联合类型存在同名属性，那么该属性的类型也是联合类型。
interface Circle {
  area: bigint;
}

interface Rectangle {
  area: number;
}

declare const s: Circle | Rectangle;
s.area; // bigint | number
```

## interface 与 type 的异同

```ts
// interface命令与type命令作用类似，都可以表示对象类型
type Country = {
  name: string;
  capital: string;
};

interface Country {
  name: string;
  capital: string;
}

// class命令也有类似作用，通过定义一个类，同时定义一个对象类型。
// 但是，它会创造一个值，编译后依然存在。
// 如果只是单纯想要一个类型，应该使用type或interface。
class Country {
  name: string;
  capital: string;
}
```

interface 与 type 的区别有下面几点。

**type 能够表示非对象类型，而 interface 只能表示对象类型（包括数组、函数等）。**

```ts
// 正确
type MyStr = string;

// 报错
interface MyStr extends string {}
```

**interface 可以继承其他类型，type 不支持继承。**

```ts
// 继承的主要作用是添加属性，type定义的对象类型如果想要添加属性，只能使用&运算符，重新定义一个类型
type Animal = {
  name: string;
};

type Bear = Animal & {
  honey: boolean;
};
```

```ts
// interface添加属性，采用的是继承的写法
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}
```

**同名 interface 会自动合并，同名 type 则会报错。TypeScript 不允许使用 type 多次定义同一个类型。**

```ts
// 同名type会报错
type A = { foo: number }; // 报错
type A = { bar: number }; // 报错
```

```ts
// 同名interface会自动合并
interface A {
  foo: number;
}
interface A {
  bar: number;
}

const obj: A = {
  foo: 1,
  bar: 1,
};
```

**interface 不能包含属性映射（mapping），type 可以。**

```ts
interface Point {
  x: number;
  y: number;
}

// 正确
type PointCopy1 = {
  [Key in keyof Point]: Point[Key];
};

// 报错
interface PointCopy2 {
  [Key in keyof Point]: Point[Key];
};
```

**this 关键字只能用于 interface。**

```ts
// 正确
interface Foo {
  add(num: number): this;
}

// 报错
type Foo = {
  add(num: number): this;
};
```

**type 可以扩展原始数据类型，interface 不行。**

```ts
// 正确
type MyStr = string & {
  type: "new";
};

// 报错
interface MyStr extends string {
  type: "new";
}
```

**interface 无法表达某些复杂类型（比如交叉类型和联合类型），但是 type 可以。**

```ts
type A = {
  // todo
};
type B = {
  // todo
};

type AorB = A | B;
type AorBwithName = AorB & {
  name: string;
};
```

综上所述，如果有复杂的类型运算，那么没有其他选择只能使用 type；一般情况下，interface 灵活性比较高，便于扩充类型或自动合并，建议优先使用。
