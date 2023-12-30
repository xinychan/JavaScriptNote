[TOC]

# Class 的基本语法

## 类的由来

```js
// JavaScript 语言中，生成实例对象的传统方法是通过构造函数
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return "(" + this.x + ", " + this.y + ")";
};

var p = new Point(1, 2);
```

```js
// ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板
// 基本上，ES6 的class可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return "(" + this.x + ", " + this.y + ")";
  }
}

var p = new Point(1, 2);
```

```js
// ES6 的类，完全可以看作构造函数的另一种写法
// 类的数据类型就是函数，类本身就指向构造函数
typeof Point; // "function"
Point === Point.prototype.constructor; // true
```

```js
// 类的所有方法都定义在类的prototype属性上面
class Point {
  constructor() {
    // dosomething
  }

  toString() {
    // dosomething
  }

  toValue() {
    // dosomething
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```

```js
// ES6 类的内部所有定义的方法，都是不可枚举的
class Point {
  constructor(x, y) {
    // dosomething
  }

  toString() {
    // dosomething
  }
}

Object.keys(Point.prototype);
// []
Object.getOwnPropertyNames(Point.prototype);
// ["constructor","toString"]
```

```js
// ES5 构造函数定义的方法，是可枚举的
var Point = function (x, y) {
  // dosomething
};

Point.prototype.toString = function () {
  // dosomething
};

Object.keys(Point.prototype);
// ["toString"]
Object.getOwnPropertyNames(Point.prototype);
// ["constructor","toString"]
```

## constructor() 方法

```js
// constructor()方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法
// 一个类必须有constructor()方法，如果没有显式定义，一个空的constructor()方法会被默认添加
class Point {}

// 等同于
class Point {
  constructor() {}
}
```

```js
// constructor()方法默认返回实例对象（即this），但也可以指定返回另外一个对象
class Foo {
  constructor() {
    // 指定返回null
    return Object.create(null);
  }
}

new Foo() instanceof Foo;
// false
```

## 类的实例

```js
// 类的属性和方法，除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）
class Point {
  constructor(x, y) {
    // 定义在this对象上
    this.x = x;
    this.y = y;
  }

  // 定义在原型上
  toString() {
    return "(" + this.x + ", " + this.y + ")";
  }
}

var point = new Point(2, 3);

point.toString(); // (2, 3)

// x和y是实例对象point自身的属性
point.hasOwnProperty("x"); // true
point.hasOwnProperty("y"); // true
// toString()是原型对象的属性
point.hasOwnProperty("toString"); // false
point.__proto__.hasOwnProperty("toString"); // true
```

## 实例属性的新写法

```js
// 实例属性原来的写法
class IncreasingCounter {
  // 实例属性定义在constructor()方法里面的this上面
  constructor() {
    this._count = 0;
  }
  get value() {
    console.log("Getting the current value!");
    return this._count;
  }
  increment() {
    this._count++;
  }
}
```

```js
// 实例属性新的写法
class IncreasingCounter {
  // 实例属性定义在类的最顶层
  _count = 0;
  get value() {
    console.log("Getting the current value!");
    return this._count;
  }
  increment() {
    this._count++;
  }
}
```

## 取值函数（getter）和存值函数（setter）

```js
// 在“类”的内部可以使用get和set关键字，对某个属性设置存值函数和取值函数，拦截该属性的存取行为
class MyClass {
  constructor() {
    // dosomething
  }
  get prop() {
    return "getter";
  }
  set prop(value) {
    console.log("setter: " + value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop;
// 'getter'
```

## 属性表达式

```js
// 类的属性名，可以采用表达式
let methodName = "getArea";

class Square {
  constructor(length) {
    // dosomething
  }

  [methodName]() {
    // dosomething
  }
}
```

## Class 表达式

```js
// 类也可以使用表达式的形式定义
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

// 需要注意的是，这个类的名字是Me，但是Me只在 Class 的内部可用，指代当前类。
// 在 Class 外部，这个类只能用MyClass引用
let inst = new MyClass();
inst.getClassName(); // Me
Me.name; // ReferenceError: Me is not defined
```

## 静态方法

```js
// static关键字，表示该方法不会被实例继承，而是直接通过类来调用
class Foo {
  static classMethod() {
    return "hello";
  }
}

Foo.classMethod(); // 'hello'

var foo = new Foo();
foo.classMethod();
// TypeError: foo.classMethod is not a function
```

## 静态属性

```js
// 静态属性指的是 Class 本身的属性，即 Class.propName

// 老写法
class Foo {
  // dosomething
}
Foo.prop = 1;

// 新写法
// 使用static关键字
class Foo {
  static prop = 1;
}
```

## 私有方法和私有属性

```js
// ES2022正式为class添加了私有属性和私有方法，方法是在属性名和方法名之前使用#表示
class Foo {
  // 私有属性，只能在类的内部使用
  #a;
  #b;
  constructor(a, b) {
    this.#a = a;
    this.#b = b;
  }
  // 私有方法，只能在类的内部使用
  #sum() {
    return this.#a + this.#b;
  }
  printSum() {
    console.log(this.#sum());
  }
}
```

```js
// in运算符，可以用来判断私有属性是否存在
class C {
  #brand;
  static isC(obj) {
    if (#brand in obj) {
      // 私有属性 #brand 存在
      return true;
    } else {
      // 私有属性 #brand 不存在
      return false;
    }
  }
}
```

## 静态块

```js
// ES2022 引入了静态块（static block），允许在类的内部设置一个代码块，在类生成时运行且只运行一次，主要作用是对静态属性进行初始化。
// 以后，新建类的实例时，这个块就不运行了。
class C {
  static x = doSomething();
  static y;
  static z;
  // 静态块（static block）
  static {
    try {
      const obj = doSomethingWith(this.x);
      this.y = obj.y;
      this.z = obj.z;
    } catch {
      this.y = doSomething();
      this.z = doSomething();
    }
  }
}
```

## new.target 属性

```js
// ES6 为new命令引入了一个new.target属性，该属性一般用在构造函数之中，返回new命令作用于的那个构造函数。
// 如果构造函数不是通过new命令或Reflect.construct()调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。

function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error("必须使用 new 命令生成实例");
  }
}

// 使用new命令
var person = new Person("张三"); // 正确
// 未使用new命令
var notAPerson = Person.call(person, "张三"); // 报错
```

```js
// Class 内部调用new.target，返回当前 Class
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 输出 true
```
