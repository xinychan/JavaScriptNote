[TOC]

# async 函数

## 含义

ES2017 标准引入了 async 函数，使得异步操作变得更加方便。

async 函数是什么？一句话，它就是 Generator 函数的语法糖。

```js
const fs = require("fs");

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function (error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

// 使用 Generator 函数，依次读取两个文件
const gen = function* () {
  const f1 = yield readFile("/etc/fstab");
  const f2 = yield readFile("/etc/shells");
  console.log(f1.toString());
  console.log(f2.toString());
};

// Generator 函数改成 async，yield 改成 await
const asyncReadFile = async function () {
  const f1 = await readFile("/etc/fstab");
  const f2 = await readFile("/etc/shells");
  console.log(f1.toString());
  console.log(f2.toString());
};
```

## 基本用法

```js
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

// 等待 50 毫秒以后，输出hello world
asyncPrint("hello world", 50);
```

## 语法

### 返回 Promise 对象

```js
// async函数返回一个 Promise 对象
async function f() {
  return "hello world";
}

f().then((v) => console.log(v));
// "hello world"
```

### Promise 对象的状态变化

```js
// async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
// getTitle 函数中三个操作全部完成，才会执行then方法里面的console.log
getTitle("https://tc39.github.io/ecma262/").then(console.log);
// "ECMAScript 2017 Language Specification"
```

### await 命令

```js
// 正常情况下，await命令后面是一个 Promise 对象，返回该对象的结果。
// 如果不是 Promise 对象，就直接返回对应的值。
async function f() {
  // 等同于
  // return 123;
  return await 123;
}

f().then((v) => console.log(v));
// 123
```

```js
// 如果await命令后面是一个thenable对象（即定义了then方法的对象），那么await会将其等同于 Promise 对象
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(() => resolve(Date.now() - startTime), this.timeout);
  }
}

(async () => {
  // await会将Sleep返回值视为Promise处理
  const sleepTime = await new Sleep(1000);
  console.log(sleepTime);
})();
// 1000
```

### 错误处理

```js
// 如果await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error("出错了");
  });
}

f()
  .then((v) => console.log(v))
  .catch((e) => console.log(e));
// Error：出错了
```

### 使用注意点

```js
// await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在try...catch代码块中
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一种写法；通过 Promise 对象的 catch 方法来捕获

async function myFunction() {
  await somethingThatReturnsAPromise().catch(function (err) {
    console.log(err);
  });
}
```

```js
// 多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发

// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

```js
// await命令只能用在async函数之中，如果用在普通函数，就会报错
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```

```js
// async 函数可以保留运行堆栈
// b()运行的时候，a()是暂停执行，上下文环境都保存着。一旦b()或c()报错，错误堆栈将包括a()
const a = async () => {
  await b();
  c();
};
```

## async 函数的实现原理

```js
// async 函数的实现原理，就是将 Generator 函数和自动执行器，包装在一个函数里

async function fn(args) {
  // dosomething
}

// 等同于如下代码

// 将 Generator 函数和自动执行器，包装在一个函数fn()
function fn(args) {
  return spawn(function* () {
    // dosomething
  });
}

// spawn函数，自动执行器的实现
function spawn(genF) {
  return new Promise(function (resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch (e) {
        return reject(e);
      }
      if (next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(
        function (v) {
          step(function () {
            return gen.next(v);
          });
        },
        function (e) {
          step(function () {
            return gen.throw(e);
          });
        }
      );
    }
    step(function () {
      return gen.next(undefined);
    });
  });
}
```

## 顶层 await

```js
// 早期的语法规定是，await命令只能出现在 async 函数内部，否则都会报错
const data = await fetch("https://api.example.com");
// 报错
```

```js
// 从 ES2022 开始，允许在模块的顶层独立使用await命令，它的主要目的是使用await解决模块异步加载的问题。

// 模块导出方
// awaiting.js
const dynamic = import(someMission);
const data = fetch(url);
// 只有等到dynamic/data这2个异步操作都完成，这个模块才会输出值
export const output = someProcess((await dynamic).default, await data);

// 模块导入方
// usage.js
import { output } from "./awaiting.js";
// 通过顶层 await，异步模块的加载与普通的模块加载写法完全一样
function outputPlusValue(value) {
  return output + value;
}

console.log(outputPlusValue(100));
setTimeout(() => console.log(outputPlusValue(100)), 1000);
```

```js
// 如果加载多个包含顶层await命令的模块，加载命令是同步执行的

// x.js
console.log("X1");
await new Promise((r) => setTimeout(r, 1000));
console.log("X2");

// y.js
console.log("Y");

// z.js
import "./x.js";
import "./y.js";
// x.js和y.js模块是同步加载的；z.js不会等x.js加载完成后再加载y.js
console.log("Z");
```
