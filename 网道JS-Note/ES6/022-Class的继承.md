[TOC]

# Class 的继承

## 简介

```js
// Class 可以通过extends关键字实现继承，让子类继承父类的属性和方法
class Point {}

class ColorPoint extends Point {}
```

```js
class Point {
  // do something
}

class ColorPoint extends Point {
  // 子类必须在constructor()方法中调用super()，否则就会报错
  // 因为子类自己的this对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，添加子类自己的实例属性和方法
  // 如果不调用super()方法，子类就得不到自己的this对象
  constructor(x, y, color) {
    // 调用父类的constructor(x, y)
    super(x, y);
    this.color = color;
  }

  toString() {
    // 调用父类的toString()
    return this.color + " " + super.toString();
  }
}
```

```js
// ES5 的继承机制，是先创造一个独立的子类的实例对象，然后再将父类的方法添加到这个对象上面，即“实例在前，继承在后”。
// ES6 的继承机制，则是先将父类的属性和方法，加到一个空的对象上面，然后再将该对象作为子类的实例，即“继承在前，实例在后”。
// 所以 ES6 的继承必须先调用super()方法，因为这一步会生成一个继承父类的this对象，没有这一步就无法继承父类。
// 子类在调用super()之前，是没有this对象的，任何对this的操作都要放在super()的后面
class Foo {
  // 新建子类实例时，父类的构造函数必定会先运行一次
  constructor() {
    console.log(1);
  }
}

class Bar extends Foo {
  constructor() {
    super();
    console.log(2);
  }
}

const bar = new Bar();
// 1
// 2
```

```js
// 如果子类没有定义constructor()方法，这个方法会默认添加，并且里面会调用super()
class ColorPoint extends Point {}

// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```

## 私有属性和私有方法的继承

```js
// 父类所有的属性和方法，都会被子类继承，除了私有的属性和方法
// 私有属性只能在定义它的 class 里面使用
class Foo {
  #p = 1;
  #m() {
    console.log("hello");
  }
}

class Bar extends Foo {
  constructor() {
    super();
    console.log(this.#p); // 报错
    this.#m(); // 报错
  }
}
```

## 静态属性和静态方法的继承

```js
// 父类的静态属性和静态方法，也会被子类继承
class A {
  static hello() {
    console.log("hello world");
  }
}

class B extends A {}

B.hello(); // hello world
```

## Object.getPrototypeOf()

```js
// Object.getPrototypeOf()方法可以用来从子类上获取父类
class Point {}

class ColorPoint extends Point {}

Object.getPrototypeOf(ColorPoint) === Point;
// true
```

## super 关键字

super 这个关键字，既可以当作函数使用，也可以当作对象使用

```js
// super作为函数调用时，代表父类的构造函数
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

```js
// super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    // 在普通方法中，调用父类原型对象的 p()
    console.log(super.p()); // 2
  }
}

let b = new B();
```

```js
// super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    // p是父类A实例的属性，super.p引用不到它
    return super.p;
  }
}

let b = new B();
b.m; // undefined
```

```js
// 属性定义在父类的原型对象上，super可以取到
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x); // 2
  }
}

let b = new B();
```

```js
// 如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象
class Parent {
  static myMethod(msg) {
    console.log("static", msg);
  }

  myMethod(msg) {
    console.log("instance", msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    // 静态方法之中，super指向父类
    super.myMethod(msg);
  }

  myMethod(msg) {
    // 普通方法中，super指向父类原型对象
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

## 类的 prototype 属性和`__proto__`属性

```js
class A {}

class B extends A {}

// 子类的__proto__属性，表示构造函数的继承，总是指向父类。
B.__proto__ === A; // true
// 子类prototype属性的__proto__属性，表示方法的继承，总是指向父类的prototype属性。
B.prototype.__proto__ === A.prototype; // true
```

## 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。

- Boolean()
- Number()
- String()
- Array()
- Date()
- Function()
- RegExp()
- Error()
- Object()

```js
// ES5 无法继承原生构造函数，ES6 允许继承原生构造函数定义子类
// 因为 ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length; // 1

arr.length = 0;
arr[0]; // undefined
```

## Mixin 模式的实现

```js
// Mixin 指的是多个对象合成一个新的对象，新对象具有各个组成成员的接口
// Mixin 最简单实现如下
const a = {
  a: "a",
};
const b = {
  b: "b",
};
const c = { ...a, ...b };
// {a: 'a', b: 'b'}
```

```js
// Mixin 更完备的实现
// mix函数，可以将多个对象合成为一个类
function mix(...mixins) {
  class Mix {
    constructor() {
      for (let mixin of mixins) {
        copyProperties(this, new mixin()); // 拷贝实例属性
      }
    }
  }

  for (let mixin of mixins) {
    copyProperties(Mix, mixin); // 拷贝静态属性
    copyProperties(Mix.prototype, mixin.prototype); // 拷贝原型属性
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if (key !== "constructor" && key !== "prototype" && key !== "name") {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```

```js
// 通过 mix 函数，Loggable和Serializable合成为一个DistributedEdit类
class DistributedEdit extends mix(Loggable, Serializable) {
  // do something
}
```
