[TOC]

# let 和 const 命令

## let 命令

### 基本用法

ES6 新增了 let 命令，用来声明变量。它的用法类似于 var，但是所声明的变量，只在 let 命令所在的代码块内有效。

```js
{
  let a = 10;
  var b = 1;
}

a; // ReferenceError: a is not defined.
b; // 1
```

### 不存在变量提升

var 命令会发生“变量提升”现象，即变量可以在声明之前使用，值为 undefined。
为了纠正这种现象，let 命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

```js
// var 的情况
// 变量foo用var命令声明，会发生变量提升，即脚本开始运行时，变量foo已经存在了，但是没有值，会输出undefined
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
// 变量bar用let命令声明，不会发生变量提升；在声明它之前，变量bar是不存在的
console.log(bar); // 报错ReferenceError
let bar = 2;
```

### 暂时性死区

在代码块内，使用 let 命令声明变量之前，该变量都是不可用的。
这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）

```js
if (true) {
  // TDZ开始
  tmp = "abc"; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

### 不允许重复声明

let 不允许在相同作用域内，重复声明同一个变量。

```js
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```

## 块级作用域

### ES5 的全局作用域和函数作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

```js
// 第一种场景，内层变量可能会覆盖外层变量。
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    // 函数作用域中变量提升，导致内层的tmp变量覆盖了外层的tmp变量
    var tmp = "hello world";
  }
}

f(); // undefined
```

```js
// 第二种场景，用来计数的循环变量泄露为全局变量。
var s = "hello";

// 变量i只用来控制循环，但是循环结束后，并没有消失，泄露成了全局变量
for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```

### ES6 的块级作用域

let 实际上为 JavaScript 新增了块级作用域。

```js
function f1() {
  let n = 5;
  if (true) {
    // 块级作用域中的n，不影响外部的n
    let n = 10;
  }
  console.log(n); // 5
}
```

## const 命令

### 基本用法

const 声明一个只读的常量。一旦声明，常量的值就不能改变。

```js
const PI = 3.1415;
PI; // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

const 声明的变量不得改变值，这意味着，const 一旦声明变量，就必须立即初始化，不能留到以后赋值。

```js
// 对于const来说，只声明不赋值，就会报错
const foo;
// SyntaxError: Missing initializer in const declaration
```

const 的作用域与 let 命令相同：只在声明所在的块级作用域内有效。

```js
if (true) {
  const MAX = 5;
}

MAX; // Uncaught ReferenceError: MAX is not defined
```

const 命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

```js
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
```

const 声明的常量，也与 let 一样不可重复声明。

```js
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```

### 本质

const 实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。
对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。
但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const 只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了。

```js
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop; // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```

如果真的想将对象冻结，应该使用 Object.freeze 方法。

```js
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

### ES6 声明变量的六种方法

ES5 只有两种声明变量的方法：var 命令和 function 命令。

ES6 除了添加 let 和 const 命令，还包括另外两种声明变量的方法：import 命令和 class 命令

## 顶层对象的属性

顶层对象，在浏览器环境指的是 window 对象，在 Node 指的是 global 对象。
ES5 之中，顶层对象的属性与全局变量是等价的。

```js
window.a = 1;
a; // 1

a = 2;
window.a; // 2
```

顶层对象的属性与全局变量挂钩，被认为是 JavaScript 语言最大的设计败笔之一。
这样的设计带来了几个很大的问题，首先是没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）；
其次，程序员很容易不知不觉地就创建了全局变量（比如打字出错）；最后，顶层对象的属性是到处可以读写的，这非常不利于模块化编程。
另一方面，window 对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的。

ES6 为了改变这一点，一方面规定，为了保持兼容性，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性；
另一方面规定，let 命令、const 命令、class 命令声明的全局变量，不属于顶层对象的属性。
也就是说，从 ES6 开始，全局变量将逐步与顶层对象的属性脱钩。

```js
// var 命令声明的全局变量，依旧是顶层对象的属性
var a = 1;
// 如果在 Node 的 REPL 环境，可以写成 global.a
// 或者采用通用方法，写成 this.a
window.a; // 1

// let 命令声明的全局变量，不属于顶层对象的属性
let b = 1;
window.b; // undefined
```

## globalThis 对象

JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。但是，顶层对象在各种实现里面是不统一的。

- 浏览器里面，顶层对象是 window，但 Node 和 Web Worker 没有 window。
- 浏览器和 Web Worker 里面，self 也指向顶层对象，但是 Node 没有 self。
- Node 里面，顶层对象是 global，但其他环境都不支持。

ES2020 在语言标准的层面，引入 globalThis 作为顶层对象。
也就是说，任何环境下，globalThis 都是存在的，都可以从它拿到顶层对象，指向全局环境下的 this。
