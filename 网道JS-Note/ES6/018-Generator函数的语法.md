[TOC]

# Generator 函数的语法

## 简介

### 基本概念

Generator 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。
Generator 函数有多种理解角度。
语法上，首先可以把它理解成，Generator 函数是一个状态机，封装了多个内部状态。
执行 Generator 函数会返回一个遍历器对象，也就是说，Generator 函数除了状态机，还是一个遍历器对象生成函数。
返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

```js
// Generator 函数function关键字与函数名之间有一个星号
// 函数体内部使用yield表达式，定义不同的内部状态
function* helloWorldGenerator() {
  yield "hello";
  yield "world";
  return "ending";
}

// value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束。
var hw = helloWorldGenerator();
hw.next();
// { value: 'hello', done: false }
hw.next();
// { value: 'world', done: false }
hw.next();
// { value: 'ending', done: true }
hw.next();
// { value: undefined, done: true }
```

### yield 表达式

由于 Generator 函数返回的遍历器对象，只有调用 next 方法才会遍历下一个内部状态，所以其实提供了一种可以暂停执行的函数。
yield 表达式就是暂停标志。

```js
// yield表达式后面的表达式，只有当调用next方法、内部指针指向该语句时才会执行
// 因此等于为 JavaScript 提供了手动的“惰性求值”（Lazy Evaluation）的语法功能。
function* gen() {
  // 只有调用next方法才会执行
  yield 123 + 456;
}
```

```js
// Generator 函数可以不用yield表达式，这时就变成了一个单纯的暂缓执行函数。
function* f() {
  console.log("执行了！");
}

var generator = f();

setTimeout(function () {
  generator.next();
}, 2000);
```

### 与 Iterator 接口的关系

```js
// 任意一个对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。
// 由于 Generator 函数就是遍历器生成函数，因此可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口。
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable]; // [1, 2, 3]
```

```js
// Generator 函数执行后，返回一个遍历器对象。该对象本身也具有Symbol.iterator属性，执行后返回自身。
function* gen() {
  // some code
}

var g = gen();

g[Symbol.iterator]() === g;
// true
```

## next 方法的参数

```js
// yield表达式本身没有返回值，或者说总是返回undefined。
// next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。

function* foo(x) {
  var y = 2 * (yield x + 1);
  var z = yield y / 3;
  return x + y + z;
}

var b = foo(5);
// (yield x + 1) == (yield 5 + 1);表达式返回值==6
b.next(); // { value:6, done:false }
// 上一个yield表达式的返回值==12;即 (var y = 2 * 12);
// 当前yield表达式==(yield 24 / 3);表达式返回值==8
b.next(12); // { value:8, done:false }
// 上一个yield表达式的返回值==13;即 (var z = 13);当前执行 return 语句
// x=5,y=24,z=13;(x + y + z = 42)
b.next(13); // { value:42, done:true }
```

## for...of 循环

```js
// for...of循环可以自动遍历 Generator 函数运行时生成的Iterator对象，且此时不再需要调用next方法。
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

```js
// 利用for...of循环，可以写出遍历任意对象（object）的方法
// 原生的 JavaScript 对象没有遍历接口，无法使用for...of循环，通过 Generator 函数为它加上这个接口，就可以用了
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}

// 对象jane原生不具备 Iterator 接口，无法用for...of遍
let jane = { first: "Jane", last: "Doe" };
// 通过 Generator 函数objectEntries为它加上遍历器接口，就可以用for...of遍历
for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe

// 添加遍历器接口的另一种写法是，将 Generator 函数加到对象的Symbol.iterator属性上面
let jane = { first: "Jane", last: "Doe" };
jane[Symbol.iterator] = objectEntries;
for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

```js
// 使用遍历器接口的场景
function* numbers() {
  yield 1;
  yield 2;
  return 3;
  yield 4;
}

// 扩展运算符
[...numbers()]; // [1, 2]

// Array.from 方法
Array.from(numbers()); // [1, 2]

// 解构赋值
let [x, y] = numbers();
x; // 1
y; // 2

// for...of 循环
for (let n of numbers()) {
  console.log(n);
}
// 1
// 2
```

## Generator.prototype.throw()

```js
// Generator 函数返回的遍历器对象，都有一个throw方法，可以在函数体外抛出错误，然后在 Generator 函数体内捕获
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log("内部捕获", e);
  }
};

var i = g();
i.next();

try {
  // 第一个错误被 Generator 函数体内的catch语句捕获
  i.throw("a");
  // 第二次抛出错误，Generator 函数内部的catch语句已经执行过了，不会再捕捉到这个错误了，所以被函数体外的catch语句捕获
  i.throw("b");
} catch (e) {
  console.log("外部捕获", e);
}
// 内部捕获 a
// 外部捕获 b
```

```js
// 不用遍历器对象抛出错误，而直接用throw命令抛出的错误，只能被 Generator 函数体外的catch语句捕获
var g = function* () {
  while (true) {
    try {
      yield;
    } catch (e) {
      if (e != "a") throw e;
      console.log("内部捕获", e);
    }
  }
};

var i = g();
i.next();

try {
  throw new Error("a");
  throw new Error("b");
} catch (e) {
  console.log("外部捕获", e);
}
// 外部捕获 [Error: a]
```

```js
// throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法
function* gen() {
  try {
    yield 1;
  } catch (e) {
    console.log("内部捕获");
  }
}

var g = gen();
g.throw(1);
// Uncaught 1
```

```js
// Generator 函数体外抛出的错误，可以在函数体内捕获；反过来，Generator 函数体内抛出的错误，也可以被函数体外的catch捕获
function* foo() {
  var x = yield 3;
  var y = x.toUpperCase();
  yield y;
}

var it = foo();

it.next(); // { value:3, done:false }

try {
  it.next(42);
} catch (err) {
  console.log(err);
}
```

```js
// 如果 Generator 函数内部和外部，都没有部署try...catch代码块，那么程序将报错，直接中断执行
var gen = function* gen() {
  yield console.log("hello");
  yield console.log("world");
};

var g = gen();
g.next();
g.throw();
// hello
// Uncaught undefined
```

## Generator.prototype.return()

```js
// Generator 函数返回的遍历器对象，有一个return()方法，可以返回给定的值，并且终结遍历 Generator 函数
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next(); // { value: 1, done: false }
g.return("foo"); // { value: "foo", done: true }
g.next(); // { value: undefined, done: true }
```

## next()、throw()、return() 的共同点

```js
// next()、throw()、return()这三个方法本质上是同一件事，可以放在一起理解。
// 它们的作用都是让 Generator 函数恢复执行，并且使用不同的语句替换 yield 表达式

const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

// next()是将yield表达式替换成一个值
const gen = g(1, 2);
gen.next(); // Object {value: 3, done: false}
gen.next(1); // Object {value: 1, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = 1;

// throw()是将yield表达式替换成一个throw语句
gen.throw(new Error("出错了")); // Uncaught Error: 出错了
// 相当于将 let result = yield x + y
// 替换成 let result = throw(new Error('出错了'));

// return()是将yield表达式替换成一个return语句
gen.return(2); // Object {value: 2, done: true}
// 相当于将 let result = yield x + y
// 替换成 let result = return 2;
```

## `yield*` 表达式

```js
// ES6 提供了yield*表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数

function* foo() {
  yield "a";
  yield "b";
}

function* bar() {
  yield "x";
  yield* foo();
  yield "y";
}

// 等同于
function* bar() {
  yield "x";
  yield "a";
  yield "b";
  yield "y";
}

// 等同于
function* bar() {
  yield "x";
  for (let v of foo()) {
    yield v;
  }
  yield "y";
}

for (let v of bar()) {
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```

```js
// yield表达式 和 yield*表达式对比
// 如果yield表达式后面跟的是一个遍历器对象，需要在yield表达式后面加上星号，表明它返回的是一个遍历器对象。即yield*表达式
function* inner() {
  yield "hello!";
}

function* outer1() {
  yield "open";
  yield inner();
  yield "close";
}

var gen = outer1();
gen.next().value; // "open"
gen.next().value; // 返回一个遍历器对象
gen.next().value; // "close"

function* outer2() {
  yield "open";
  yield* inner();
  yield "close";
}

var gen = outer2();
gen.next().value; // "open"
gen.next().value; // "hello!"
gen.next().value; // "close"
```

## 作为对象属性的 Generator 函数

```js
// 如果一个对象的属性是 Generator 函数，可以简写成下面的形式
let obj = {
  *myGeneratorMethod() {
    // ···
  },
};
// 等价于
let obj = {
  myGeneratorMethod: function* () {
    // ···
  },
};
```

## Generator 函数的 this

```js
function* g() {}

g.prototype.hello = function () {
  return "hi!";
};

// obj 是 Generator 函数的实例
// 也继承了 Generator 函数的prototype对象上的方法
let obj = g();

obj instanceof g; // true
obj.hello(); // 'hi!'
```

```js
// Generator 函数g在this对象上面添加了一个属性a，但是obj对象拿不到这个属性
// 因为g返回的总是遍历器对象，而不是this对象
function* g() {
  this.a = 11;
}

let obj = g();
obj.next();
obj.a; // undefined
```

```js
// Generator 函数也不能跟new命令一起用
// 因为F不是构造函数
function* F() {
  yield (this.x = 2);
  yield (this.y = 3);
}

new F();
// TypeError: F is not a constructor
```

```js
// 让 Generator 函数返回一个正常的对象实例，既可以用next方法，又可以获得正常的this 写法
function* F() {
  this.a = 1;
  yield (this.b = 2);
  yield (this.c = 3);
}
var obj = {};
// F内部的this对象绑定obj对象
var f = F.call(obj);

f.next(); // Object {value: 2, done: false}
f.next(); // Object {value: 3, done: false}
f.next(); // Object {value: undefined, done: true}

obj.a; // 1
obj.b; // 2
obj.c; // 3
```

```js
// 让 Generator 函数返回遍历器对象f，和生成的对象实例obj统一成一个对象的写法
function* F() {
  this.a = 1;
  yield (this.b = 2);
  yield (this.c = 3);
}
var f = F.call(F.prototype);

f.next(); // Object {value: 2, done: false}
f.next(); // Object {value: 3, done: false}
f.next(); // Object {value: undefined, done: true}

f.a; // 1
f.b; // 2
f.c; // 3
```

```js
// 让 Generator 函数返回的对象，可以执行new命令的写法
function* gen() {
  this.a = 1;
  yield (this.b = 2);
  yield (this.c = 3);
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next(); // Object {value: 2, done: false}
f.next(); // Object {value: 3, done: false}
f.next(); // Object {value: undefined, done: true}

f.a; // 1
f.b; // 2
f.c; // 3
```

## 含义

### Generator 与状态机

```js
// Generator 是实现状态机的最佳结构

// clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态

// ES5 实现
var ticking = true;
var clock = function () {
  if (ticking) {
    console.log("Tick!");
  } else {
    console.log("Tock!");
  }
  ticking = !ticking;
};

// Generator 函数实现
var clock = function* () {
  while (true) {
    console.log("Tick!");
    yield;
    console.log("Tock!");
    yield;
  }
};
```

### Generator 与协程

由于 JavaScript 是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误的时候，可以找到原始的调用栈。不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。

Generator 函数是 ES6 对协程的实现，但属于不完全实现。Generator 函数被称为“半协程”（semi-coroutine），意思是只有 Generator 函数的调用者，才能将程序的执行权还给 Generator 函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。

如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用 yield 表达式交换控制权。

### Generator 与上下文

JavaScript 代码运行时，会产生一个全局的上下文环境（context，又称运行环境），包含了当前所有的变量和对象。然后，执行函数（或块级代码）的时候，又会在当前上下文环境的上层，产生一个函数运行的上下文，变成当前（active）的上下文，由此形成一个上下文环境的堆栈（context stack）。

这个堆栈是“后进先出”的数据结构，最后产生的上下文环境首先执行完成，退出堆栈，然后再执行完成它下层的上下文，直至所有代码执行完成，堆栈清空。

Generator 函数不是这样，它执行产生的上下文环境，一旦遇到 yield 命令，就会暂时退出堆栈，但是并不消失，里面的所有变量和对象会冻结在当前状态。等到对它执行 next 命令时，这个上下文环境又会重新加入调用栈，冻结的变量和对象恢复执行。

```js
function* gen() {
  yield 1;
  return 2;
}

let g = gen();

// 第一次执行g.next()时，Generator 函数gen的上下文会加入堆栈，即开始运行gen内部的代码
// 等遇到yield 1时，gen上下文退出堆栈，内部状态冻结
console.log(g.next().value);
// 第二次执行g.next()时，gen上下文重新加入堆栈，变成当前的上下文，重新恢复执行
console.log(g.next().value);
```

## 应用

Generator 可以暂停函数执行，返回任意表达式的值。
这种特点使得 Generator 有多种应用场景。

### 异步操作的同步化表达

Generator 函数的暂停执行的效果，意味着可以把异步操作写在 yield 表达式里面，等到调用 next 方法时再往后执行。
这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 yield 表达式下面，反正要等到调用 next 方法时再执行。
所以，Generator 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

```js
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
// 第一次调用loadUI函数时，该函数不会执行，仅返回一个遍历器
var loader = loadUI();
// 加载UI
// 调用next方法，则会显示Loading界面（showLoadingScreen），并且异步加载数据（loadUIDataAsynchronously）
loader.next();
// 卸载UI
// 等到数据加载完成，再一次使用next方法，则会隐藏Loading界面
loader.next();
```

### 控制流管理

```js
// 如果有一个多步操作非常耗时，采用回调函数，可能会写成下面这样
step1(function (value1) {
  step2(value1, function (value2) {
    step3(value2, function (value3) {
      step4(value3, function (value4) {
        // Do something with value4
      });
    });
  });
});
```

```js
// 采用 Promise 改写上面的代码
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(
    function (value4) {
      // Do something with value4
    },
    function (error) {
      // Handle any error from step1 through step4
    }
  )
  .done();
```

```js
// 采用 Generator 函数改善代码运行流程
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}

// 然后，使用一个函数，按次序自动执行所有步骤
scheduler(longRunningTask(initialValue));

function scheduler(task) {
  // 注意：这里的task只适合同步操作，即所有的task都必须是同步的，不能有异步操作
  // 因为这里的代码一得到返回值，就继续往下执行，没有判断异步操作何时完成
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value;
    scheduler(task);
  }
}
```

```js
// 利用for...of循环会自动依次执行yield命令的特性，提供一种更一般的控制流管理的方法
// 注意，这里的step只适合同步操作，step得到返回值，就继续往下执行下一个step
let steps = [step1Func, step2Func, step3Func];

function* iterateSteps(steps) {
  for (var i = 0; i < steps.length; i++) {
    var step = steps[i];
    yield step();
  }
}
```

### 部署 Iterator 接口

```js
// 利用 Generator 函数，可以在任意对象上部署 Iterator 接口
function* iterEntries(obj) {
  let keys = Object.keys(obj);
  for (let i = 0; i < keys.length; i++) {
    let key = keys[i];
    yield [key, obj[key]];
  }
}

let myObj = { foo: 3, bar: 7 };

for (let [key, value] of iterEntries(myObj)) {
  console.log(key, value);
}

// foo 3
// bar 7
```

### 作为数据结构

```js
// Generator 可以看作是一个数组结构
// 因为 Generator 函数可以返回一系列的值，这意味着它可以对任意表达式，提供类似数组的接口
function* doStuff() {
  yield fs.readFile.bind(null, "hello.txt");
  yield fs.readFile.bind(null, "world.txt");
  yield fs.readFile.bind(null, "and-such.txt");
}

for (task of doStuff()) {
  // task是一个函数，可以像回调函数那样使用它
}
```

```js
// ES5 写法，用数组模拟 Generator 的这种用法
function doStuff() {
  return [
    fs.readFile.bind(null, "hello.txt"),
    fs.readFile.bind(null, "world.txt"),
    fs.readFile.bind(null, "and-such.txt"),
  ];
}
```
