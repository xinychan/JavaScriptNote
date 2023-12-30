[TOC]

# class

## 简介

类（class）是面向对象编程的基本构件，封装了属性和方法，TypeScript 给予了全面支持。

### 属性的类型

```ts
// 类的属性可以在顶层声明，也可以在构造方法内部声明。
class Point {
  x: number;
  y: number;
}
```

```ts
// 打开配置项 strictPropertyInitialization（默认是打开的）
// 会检查属性是否设置了初值，如果没有就报错
class Point {
  x: number; // 报错
  y: number; // 报错
}
```

### readonly 修饰符

```ts
// 属性名前面加上 readonly 修饰符，就表示该属性是只读的。实例对象不能修改这个属性。
class A {
  readonly id = "foo";
}

const a = new A();
a.id = "bar"; // 报错
```

```ts
// readonly 属性的初始值，可以写在顶层属性，也可以写在构造方法里面。
class A {
  readonly id: string;

  constructor() {
    // 构造方法内部设置只读属性的初值
    this.id = "bar"; // 正确
  }
}
```

```ts
// 如果顶层属性和构造方法都设置了只读属性的值，以构造方法为准。
// 在其他方法修改只读属性都会报错。
class A {
  readonly id: string = "foo";

  constructor() {
    this.id = "bar"; // 正确
  }
}
```

### 方法的类型

```ts
// 类的方法就是普通函数，类型声明方式与函数一致。
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }

  add(point: Point) {
    return new Point(this.x + point.x, this.y + point.y);
  }
}
```

### 存取器方法

```ts
// 存取器（accessor）是特殊的类方法，包括取值器（getter）和存值器（setter）两种方法。
// 它们用于读写某个属性，取值器用来读取属性，存值器用来写入属性。
class C {
  _name = "";
  get name() {
    return this._name;
  }
  set name(value) {
    this._name = value;
  }
}
```

TypeScript 对存取器有以下 3 个规则。

**如果某个属性只有 get 方法，没有 set 方法，那么该属性自动成为只读属性**

```ts
// 如果某个属性只有get方法，没有set方法，那么该属性自动成为只读属性。
class C {
  _name = "foo";

  get name() {
    return this._name;
  }
}

const c = new C();
c.name = "bar"; // 报错
```

**TypeScript 5.1 版之前，set 方法的参数类型，必须兼容 get 方法的返回值类型，否则报错**

```ts
// TypeScript 5.1 版之前，set方法的参数类型，必须兼容get方法的返回值类型，否则报错。
// TypeScript 5.1 版做出了改变，现在两者可以不兼容
class C {
  _name = "";
  get name(): string {
    // 报错
    return this._name;
  }
  set name(value: number) {
    this._name = String(value);
  }
}
```

**get 方法与 set 方法的可访问性必须一致，要么都为公开方法，要么都为私有方法**

### 属性索引

```ts
// 类允许定义属性索引
class MyClass {
  // [s:string]表示所有属性名类型为字符串的属性，它们的属性值是布尔值
  [s: string]: boolean;

  get(s: string) {
    return this[s] as boolean;
  }
}
```

## 类的 interface 接口

### implements 关键字

```ts
// interface 接口或 type 别名，可以用对象的形式，为 class 指定一组检查条件。
// 然后，类使用 implements 关键字，表示当前类满足这些外部类型条件的限制。
interface Country {
  name: string;
  capital: string;
}
// 或者
type Country = {
  name: string;
  capital: string;
};

class MyCountry implements Country {
  name = "";
  capital = "";
}
```

```ts
// interface 只是指定检查条件，如果不满足这些条件就会报错。它并不能代替 class 自身的类型声明。
interface A {
  x: number;
  y?: number;
}

class B implements A {
  x = 0;
}

// B类还是需要声明自身的可选属性y
const b = new B();
b.y = 10; // 报错
```

```ts
// implements关键字后面，不仅可以是接口，也可以是另一个类。
// 这时，后面的类将被当作接口。
class Car {
  id: number = 1;
  move(): void {}
}

// 此时 Car 被当作接口，要求MyCar实现Car里面的每一个属性和方法，否则就会报错
class MyCar implements Car {
  id = 2; // 不可省略
  move(): void {} // 不可省略
}
```

### 实现多个接口

```ts
// 类可以实现多个接口（其实是接受多重限制），每个接口之间使用逗号分隔。
class Car implements MotorVehicle, Flyable, Swimmable {
  // todo
}
```

```ts
// 注意，发生多重实现时（即一个接口同时实现多个接口），不同接口不能有互相冲突的属性。
interface Flyable {
  foo: number;
}

// 属性foo在两个接口里面的类型不同，如果同时实现这两个接口，就会报错。
interface Swimmable {
  foo: string;
}
```

### 类与接口的合并

```ts
// TypeScript 不允许两个同名的类，但是如果一个类和一个接口同名，那么接口会被合并进类。
class A {
  x: number = 1;
}

interface A {
  y: number;
}

// interface A 属性被合并到 class A
let a = new A();
a.y = 10;

a.x; // 1
a.y; // 10
```

## Class 类型

### 实例类型

```ts
// TypeScript 的类本身就是一种类型，但是它代表该类的实例类型，而不是 class 的自身类型。
// 定义了一个类Color。它的类名就代表一种类型，实例对象green就属于该类型。
class Color {
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const green: Color = new Color("green");
```

```ts
// 作为类型使用时，类名只能表示实例的类型，不能表示类的自身类型。
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

// 错误
// 把参数的类型写成Point就会报错，因为Point描述的是实例类型，而不是 Class 的自身类型
function createPoint(PointClass: Point, x: number, y: number) {
  return new PointClass(x, y);
}
```

```ts
// 对于引用实例对象的变量来说，既可以声明类型为 Class，也可以声明类型为 Interface，因为两者都代表实例对象的类型。
interface MotorVehicle {}

class Car implements MotorVehicle {}

// 写法一
const c1: Car = new Car();
// 写法二
const c2: MotorVehicle = new Car();

// 事实上，TypeScript 有三种方法可以为对象类型起名：type、interface 和 class。
```

### 类的自身类型

```ts
// 要获得一个类的自身类型，一个简便的方法就是使用 typeof 运算符。
function createPoint(PointClass: typeof Point, x: number, y: number): Point {
  return new PointClass(x, y);
}
```

```ts
// JavaScript 语言中，类只是构造函数的一种语法糖，本质上是构造函数的另一种写法。
// 所以，类的自身类型可以写成构造函数的形式。
function createPoint(
  PointClass: new (x: number, y: number) => Point,
  x: number,
  y: number
): Point {
  return new PointClass(x, y);
}
```

```ts
// 构造函数也可以写成对象形式，所以参数PointClass的类型还有另一种写法。
function createPoint(
  PointClass: {
    new (x: number, y: number): Point;
  },
  x: number,
  y: number
): Point {
  return new PointClass(x, y);
}

// 根据上面的写法，可以把构造函数提取出来，单独定义一个接口（interface），这样可以大大提高代码的通用性。
interface PointConstructor {
  new (x: number, y: number): Point;
}

function createPoint(
  PointClass: PointConstructor,
  x: number,
  y: number
): Point {
  return new PointClass(x, y);
}
```

**总结一下，类的自身类型就是一个构造函数，可以单独定义一个接口来表示。**

### 结构类型原则

Class 也遵循“结构类型原则”。一个对象只要满足 Class 的实例结构，就跟该 Class 属于同一个类型。

```ts
class Foo {
  id: number;
}

function fn(arg: Foo) {
  // todo
}

// 对象bar满足类Foo的实例结构
const bar = {
  id: 10,
  amount: 100,
};

// bar可以当作参数，传入函数fn()
fn(bar); // 正确
```

```ts
// 如果两个类的实例结构相同，那么这两个类就是兼容的，可以用在对方的使用场合。
class Person {
  name: string;
}

class Customer {
  name: string;
}

// 正确
const cust: Customer = new Person();
```

```ts
// 空类不包含任何成员，任何其他类都可以看作与空类结构相同。
// 因此，凡是类型为空类的地方，所有类（包括对象）都可以使用。
class Empty {}

function fn(x: Empty) {
  // todo
}

// 任何对象都可以用作fn()的参数
fn({});
fn(window);
fn(fn);
```

```ts
// 注意，确定两个类的兼容关系时，只检查实例成员，不考虑静态成员和构造方法。
class Point {
  x: number;
  y: number;
  static t: number;
  constructor(x: number) {}
}

class Position {
  x: number;
  y: number;
  z: number;
  constructor(x: string) {}
}

// Point与Position的静态属性和构造方法都不一样
// 但因为Point的实例成员与Position相同，所以Position兼容Point
const point: Point = new Position("");
```

```ts
// 如果类中存在私有成员（private）或保护成员（protected），那么确定兼容关系时。
// TypeScript 要求私有成员和保护成员来自同一个类，这意味着两个类需要存在继承关系。

// A和B都有私有成员（或保护成员）name，这时只有在B继承A的情况下（class B extends A），B才兼容A
// 情况一
class A {
  private name = "a";
}

class B extends A {}

const a: A = new B();

// 情况二
class A {
  protected name = "a";
}

class B extends A {
  protected name = "b";
}

const a: A = new B();
```

## 类的继承

```ts
// 类（“子类”）可以使用 extends 关键字继承另一个类（“基类”）的所有属性和方法。
class A {
  greet() {
    console.log("Hello, world!");
  }
}

// 子类B继承了基类A，因此就拥有了greet()方法
class B extends A {}

const b = new B();
b.greet(); // "Hello, world!"
```

```ts
// 子类可以覆盖基类的同名方法。
class B extends A {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name}`);
    }
  }
}
```

```ts
// 但是，子类的同名方法不能与基类的类型定义相冲突。
class A {
  greet() {
    console.log("Hello, world!");
  }
}

// 子类B的greet()有一个name参数，跟基类A的greet()定义不兼容，因此报错
class B extends A {
  // 报错
  greet(name: string) {
    console.log(`Hello, ${name}`);
  }
}
```

```ts
// 如果基类包括保护成员（protected修饰符）
// 子类可以将该成员的可访问性设置为公开（public修饰符），也可以保持保护成员不变
// 但是不能改用私有成员（private修饰符）
class A {
  protected x: string = "";
  protected y: string = "";
  protected z: string = "";
}

class B extends A {
  // 正确
  public x: string = "";

  // 正确
  protected y: string = "";

  // 报错
  private z: string = "";
}
```

```ts
// 对于那些只设置了类型、没有初值的顶层属性，有一个细节需要注意。
interface Animal {
  animalStuff: any;
}

interface Dog extends Animal {
  dogStuff: any;
}

class AnimalHouse {
  resident: Animal;
  constructor(animal: Animal) {
    this.resident = animal;
  }
}

// 类DogHouse的顶层成员resident只设置了类型（Dog），没有设置初值
class DogHouse extends AnimalHouse {
  resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}

// 如果编译设置的target设成大于等于ES2022，或者useDefineForClassFields设成true
// DogHouse实例的属性resident输出的是undefined，而不是预料的dog
// 原因在于 ES2022 标准的 Class Fields 部分，与早期的 TypeScript 实现不一致
// 导致子类的那些只设置类型、没有设置初值的顶层成员在基类中被赋值后，会在子类被重置为undefined
const dog = {
  animalStuff: "animal",
  dogStuff: "dog",
};
const dogHouse = new DogHouse(dog);
console.log(dogHouse.resident); // undefined

// 解决方法就是使用declare命令，去声明顶层成员的类型，告诉 TypeScript 这些成员的赋值由基类实现。
class DogHouse extends AnimalHouse {
  declare resident: Dog;
  constructor(dog: Dog) {
    super(dog);
  }
}
```

## 可访问性修饰符

### public

```ts
// public修饰符表示这是公开成员，外部可以自由访问。
// public修饰符是默认修饰符，如果省略不写，实际上就带有该修饰符。
class Greeter {
  public greet() {
    console.log("hi!");
  }
}

const g = new Greeter();
g.greet();
```

### private

```ts
// private修饰符表示私有成员，只能用在当前类的内部，类的实例和子类都不能使用该成员。
class A {
  private x: number = 0;
}

// 类的实例不能使用该成员
const a = new A();
a.x; // 报错

// 子类不能使用该成员
class B extends A {
  showX() {
    console.log(this.x); // 报错
  }
}
```

```ts
// 注意，子类不能定义父类私有成员的同名成员。
class A {
  private x = 0;
}

class B extends A {
  x = 1; // 报错
}
```

**严格地说，private 定义的私有成员，并不是真正意义的私有成员。**
一方面，编译成 JavaScript 后，private 关键字就被剥离了，这时外部访问该成员就不会报错。
另一方面，TypeScript 对于访问 private 成员没有严格禁止，使用方括号写法 `[]` 或者 `in` 运算符，实例对象就能访问该成员。

```ts
class A {
  private x = 1;
}

const a = new A();
a["x"]; // 1

if ("x" in a) {
  // todo 正确
}
```

**使用`#propName`，获得真正意义的私有成员。**
由于 private 存在这些问题，加上它是 ES2022 标准发布前出台的，而 ES2022 引入了自己的私有成员写法`#propName`。
因此建议不使用 private，改用 ES2022 的写法，获得真正意义的私有成员。

```ts
class A {
  #x = 1;
}

const a = new A();
a["x"]; // 报错
```

### protected

```ts
// protected修饰符表示该成员是保护成员，只能在类的内部使用该成员，实例无法使用该成员，但是子类内部可以使用。
class A {
  protected x = 1;
}

class B extends A {
  getX() {
    return this.x;
  }
}

const a = new A();
const b = new B();

a.x; // 报错
b.getX(); // 1
```

```ts
// 子类不仅可以拿到父类的保护成员，还可以定义同名成员。
// 子类还可以扩大父类保护成员的可访问范围。
class A {
  protected x = 1;
}

// 可以定义 protected 同名成员，并修改成 public 范围
class B extends A {
  public x = 2;
}
```

### 实例属性的简写形式

```ts
// 实际开发中，很多实例属性的值，是通过构造方法传入的。
class Point {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
// TypeScript 提供了一种简写形式。
// 直接在构造方法传参时声明实例属性。
class Point {
  constructor(public x: number, public y: number) {}
}

const p = new Point(10, 10);
p.x; // 10
p.y; // 10
```

## 静态成员

```ts
// 类的内部可以使用static关键字，定义静态成员。
// 静态成员是只能通过类本身使用的成员，不能通过实例对象使用。
class MyClass {
  static x = 0;
  static printX() {
    console.log(MyClass.x);
  }
}

MyClass.x; // 0
MyClass.printX(); // 0
```

```ts
// static关键字前面可以使用 public、private、protected 修饰符。
class MyClass {
  private static x = 0;
}

MyClass.x; // 报错

// 静态私有属性也可以用 ES6 语法的#前缀表示，可以改写如下。
class MyClass {
  static #x = 0;
}
```

```ts
// public和protected的静态成员可以被继承。
class A {
  public static x = 1;
  protected static y = 1;
}

class B extends A {
  static getY() {
    return B.y;
  }
}

B.x; // 1
B.getY(); // 1
```

## 泛型类

```ts
// 类也可以写成泛型，使用类型参数。
// Box有类型参数Type，因此属于泛型类。
class Box<Type> {
  contents: Type;

  constructor(value: Type) {
    this.contents = value;
  }
}

const b: Box<string> = new Box("hello!");
```

```ts
// 注意，静态成员不能使用泛型的类型参数。
class Box<Type> {
  static defaultContents: Type; // 报错
}
```

## 抽象类，抽象成员

```ts
// 在类的定义前面，加上关键字abstract，表示该类不能被实例化，只能当作其他类的模板。
// 这种类就叫做“抽象类”（abstract class）。
abstract class A {
  id = 1;
}

const a = new A(); // 报错
```

```ts
// 抽象类只能当作基类使用，用来在它的基础上定义子类。
abstract class A {
  id = 1;
}

class B extends A {
  amount = 100;
}

const b = new B();

b.id; // 1
b.amount; // 100
```

```ts
// 抽象类的子类也可以是抽象类
abstract class A {
  foo: number;
}

abstract class B extends A {
  bar: string;
}
```

```ts
// 抽象类的内部可以有已经实现好的属性和方法，也可以有还未实现的属性和方法。
// 后者就叫做“抽象成员”（abstract member），即属性名和方法名有abstract关键字，表示该方法需要子类实现。
// 如果子类没有实现抽象成员，就会报错。
abstract class A {
  abstract foo: string;
  bar: string = "";
}

class B extends A {
  foo = "b";
}
```

**抽象类注意事项**

- 抽象成员只能存在于抽象类，不能存在于普通类。
- 抽象成员不能有具体实现的代码。也就是说，已经实现好的成员前面不能加 abstract 关键字。
- 抽象成员前也不能有 private 修饰符，否则无法在子类中实现该成员。
- 一个子类最多只能继承一个抽象类。

## this 问题

```ts
// 类的方法经常用到this关键字，它表示该方法当前所在的对象。
class A {
  name = "A";

  getName() {
    return this.name;
  }
}

const a = new A();
a.getName(); // 'A'

const b = {
  name: "b",
  // b 的 return this.name;指向 b 的 name
  getName: a.getName,
};
b.getName(); // 'b'
```

```ts
// 有些场合需要给出this类型，但是 JavaScript 函数通常不带有this参数
// 这时 TypeScript 允许函数增加一个名为this的参数，放在参数列表的第一位，用来描述函数内部的this关键字的类型。

// 编译前
// 函数fn()的第一个参数是this，用来声明函数内部的this的类型
function fn(this: SomeType, x: number) {
  // todo
}

// 编译后
// 编译时，TypeScript 一旦发现函数的第一个参数名为this，则会去除这个参数
function fn(x) {
  // todo
}
```

```ts
class A {
  name = "A";

  // getName()添加了this参数，如果直接调用这个方法，this的类型就会跟声明的类型不一致，从而报错
  getName(this: A) {
    return this.name;
  }
}

const a = new A();
const b = a.getName;

b(); // 报错
```

```ts
// this参数的类型可以声明为各种对象。
// 不符合这个条件的this都会报错。
function foo(this: { name: string }) {
  this.name = "Jack";
  this.name = 0; // 报错
}

foo.call({ name: 123 }); // 报错
```

```ts
// TypeScript 提供了一个noImplicitThis编译选项。
// 如果打开了这个设置项，如果this的值推断为any类型，就会报错。

// noImplicitThis 打开
class Rectangle {
  constructor(public width: number, public height: number) {}

  getAreaFunction() {
    return function () {
      // 这里的this跟Rectangle没关系，它的类型推断为any
      return this.width * this.height; // 报错
    };
  }
}
```

```ts
// 在类的内部，this本身也可以当作类型使用，表示当前类的实例对象。
class Box {
  contents: string = "";

  set(value: string): this {
    this.contents = value;
    return this;
  }
}
```

```ts
// 注意，this类型不允许应用于静态成员。
class A {
  static a: this; // 报错
}
```
