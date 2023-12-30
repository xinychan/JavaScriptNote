[TOC]

# Generator 函数的异步应用

## 传统方法

当前异步编程的方法，大概有下面四种。

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象

Generator 函数将 JavaScript 异步编程带入了一个全新的阶段。

## 基本概念

### 异步

所谓"异步"，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。

比如，有一个任务是读取文件进行处理，任务的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其他任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就叫做异步。

相应地，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。

### 回调函数

JavaScript 语言对异步编程的实现，就是回调函数。
所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数。
回调函数的英语名字 callback，直译过来就是"重新调用"。

```js
// readFile函数的第三个参数，就是回调函数，也就是任务的第二段
// 操作系统返回了/etc/passwd这个文件以后，回调函数才会执行
fs.readFile("/etc/passwd", "utf-8", function (err, data) {
  // Node 约定，回调函数的第一个参数，必须是错误对象err；如果没有错误，该参数就是null
  // 原因是执行分成两段，第一段执行完以后，任务所在的上下文环境就已经结束
  // 在这以后抛出的错误，原来的上下文环境已经无法捕捉，只能当作参数，传入第二段
  if (err) {
    throw err;
  }
  console.log(data);
});
```

### Promise

```js
// Promise 对象解决"回调函数地狱"（callback hell）问题
// Promise 允许将回调函数的嵌套，改成链式调用
var readFile = require("fs-readfile-promise");

readFile(fileA)
  .then(function (data) {
    console.log(data.toString());
  })
  .then(function () {
    return readFile(fileB);
  })
  .then(function (data) {
    console.log(data.toString());
  })
  .catch(function (err) {
    console.log(err);
  });
```

Promise 的写法只是回调函数的改进，使用 then 方法以后，异步任务的两段执行看得更清楚了，除此以外，并无新意。
Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆 then，原来的语义变得很不清楚。

## Generator 函数

### 协程

协程的运行流程大致如下。

- 第一步，协程 A 开始执行。
- 第二步，协程 A 执行到一半，进入暂停，执行权转移到协程 B。
- 第三步，（一段时间后）协程 B 交还执行权。
- 第四步，协程 A 恢复执行。

上面流程的协程 A，就是异步任务，因为它分成两段（或多段）执行。

```js
// 读取文件的协程写法如下
// 函数asyncJob是一个协程，它的奥妙就在其中的yield命令
// 它表示执行到此处，执行权将交给其他协程。也就是说，yield命令是异步两个阶段的分界线。
function* asyncJob() {
  // ...其他代码
  var f = yield readFile(fileA);
  // ...其他代码
}
```

协程遇到 yield 命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。
它的最大优点，就是代码的写法非常像同步操作，如果去除 yield 命令，简直一模一样。

### 协程的 Generator 函数实现

Generator 函数是协程在 ES6 的实现，最大特点就是可以交出函数的执行权（即暂停执行）。

```js
function* gen(x) {
  var y = yield x + 2;
  return y;
}

// next方法的作用是分阶段执行Generator函数
// 每此执行完next方法，就交出函数的执行权
var g = gen(1);
g.next(); // { value: 3, done: false }
g.next(); // { value: undefined, done: true }
```

### 异步任务的封装

```js
var fetch = require("node-fetch");
// Generator 函数封装了一个异步操作
function* gen() {
  var url = "https://api.github.com/users/github";
  var result = yield fetch(url);
  console.log(result.bio);
}

// 执行Generator函数的异步操作
var g = gen();
var result = g.next();

result.value
  .then(function (data) {
    return data.json();
  })
  .then(function (data) {
    g.next(data);
  });
```

可以看到，虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。

## Thunk 函数

### 参数的求值策略

```js
// 函数传入参数表达式(x + 5)；这个表达式应该何时求值？
var x = 1;
function f(m) {
  return m * 2;
}
f(x + 5);

// "传值调用"（call by value）
// 即在进入函数体之前，就计算x + 5的值（等于 6），再将这个值传入函数f。
// C 语言就采用这种策略。
f(x + 5);
// 传值调用时，等同于
f(6);

// “传名调用”（call by name）
// 即直接将表达式x + 5传入函数体，只在用到它的时候求值。
// Haskell 语言采用这种策略。
f(x + 5);
// 传名调用时，等同于
(x + 5) * 2;
```

### Thunk 函数的含义

```js
// 编译器的“传名调用”实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。
// 这个临时函数就叫做 Thunk 函数。
function f(m) {
  return m * 2;
}

f(x + 5);

// 等同于

// Thunk 函数是“传名调用”的一种实现策略，用来替换某个表达式。
var thunk = function () {
  return x + 5;
};

// 函数 f 的参数x + 5被一个Thunk函数替换
// 凡是用到原参数的地方，对Thunk函数求值即可
function f(thunk) {
  return thunk() * 2;
}
```

### JavaScript 语言的 Thunk 函数

JavaScript 语言是传值调用，它的 Thunk 函数含义有所不同。
在 JavaScript 语言中，Thunk 函数替换的不是表达式，而是多参数函数，将其替换成一个只接受回调函数作为参数的单参数函数。

```js
// 正常版本的readFile（多参数版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（单参数版本）
var Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

```js
// 任何函数，只要参数有回调函数，就能写成 Thunk 函数的形式

// ES5版本
var Thunk = function (fn) {
  return function () {
    var args = Array.prototype.slice.call(arguments);
    return function (callback) {
      args.push(callback);
      return fn.apply(this, args);
    };
  };
};

// ES6版本
const Thunk = function (fn) {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback);
    };
  };
};
```

```js
// 使用上面的转换器，生成fs.readFile的 Thunk 函数
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);

// 使用上面的转换器，完整的例子
function f(a, cb) {
  cb(a);
}
const ft = Thunk(f);
ft(1)(console.log); // 1
```

### Generator 函数的流程管理

```js
// 如下代码，Generator 函数gen会自动执行完所有步骤。
// 但是，这不适合异步操作。
// 如果必须保证前一步执行完，才能执行后一步，这样的自动执行就不可行。
function* gen() {
  // ...
}

var g = gen();
var res = g.next();

while (!res.done) {
  console.log(res.value);
  res = g.next();
}
```

### Thunk 函数的自动流程管理

```js
// Thunk 函数真正的威力，在于可以自动执行 Generator 函数。
// 如下是一个基于 Thunk 函数的 Generator 执行器实现。
function run(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) {
      return;
    }
    result.value(next);
  }

  next();
}

function* g() {
  // do something
}

run(g);
```

```js
// 有了这个执行器，执行 Generator 函数方便多了。
// 不管内部有多少个异步操作，直接把 Generator 函数传入run函数即可。
// 前提是每一个异步操作，都要是 Thunk 函数，即跟在yield命令后面的必须是 Thunk 函数。

var g = function* () {
  var f1 = yield readFileThunk("fileA");
  var f2 = yield readFileThunk("fileB");
  var fn = yield readFileThunk("fileN");
};
// 函数g封装了n个异步的读取文件操作，只要执行run函数，这些操作就会自动完成
// 这样异步操作不仅可以写得像同步操作，而且一行代码就可以执行
run(g);
```

Thunk 函数并不是 Generator 函数自动执行的唯一方案。
因为自动执行的关键是，必须有一种机制，自动控制 Generator 函数的流程，接收和交还程序的执行权。
回调函数可以做到这一点，Promise 对象也可以做到这一点。

### 基于 Promise 对象的自动流程管理

```js
// 如下是一个基于 Promise 对象的 Generator 执行器实现。
function run(gen) {
  var g = gen();

  function next(data) {
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function (data) {
      next(data);
    });
  }

  next();
}

function* gen() {
  // do something
}

run(gen);
```

```js
// 把fs模块的readFile方法改成 Promise 对象实现
var fs = require("fs");

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function (error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* () {
  var f1 = yield readFile("/etc/fstab");
  var f2 = yield readFile("/etc/shells");
  console.log(f1.toString());
  console.log(f2.toString());
};

// 有了这个执行器，可以通过 Promise 对象自动执行 Generator 函数
run(gen);
```
