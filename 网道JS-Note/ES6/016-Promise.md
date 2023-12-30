[TOC]

# Promise

## Promise 的含义

Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。
它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了 Promise 对象。
所谓 Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。
从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。
Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

**Promise 对象有以下两个特点。**

- 对象的状态不受外界影响。

Promise 对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和 rejected（已失败）。
只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。

- 一旦状态改变，就不会再变，任何时候都可以得到这个结果。

Promise 对象的状态改变，只有两种可能：从 pending 变为 fulfilled 和从 pending 变为 rejected。

**Promise 也有一些缺点。**

- 无法取消 Promise，一旦新建它就会立即执行，无法中途取消。
- 如果不设置回调函数，Promise 内部抛出的错误，不会反应到外部。
- 当处于 pending 状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。

## 基本用法

```js
// ES6 规定，Promise对象是一个构造函数，用来生成Promise实例
const promise = new Promise(function (resolve, reject) {
  // ... some code
  if (true) {
    // resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”
    resolve(value);
  } else {
    // reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”
    reject(error);
  }
});
```

```js
// Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数
promise.then(
  function (value) {
    // success
  },
  function (error) {
    // failure
  }
);
```

## Promise.prototype.then()

```js
// then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。
// 因此可以采用链式写法，即then方法后面再调用另一个then方法。
getJSON("/posts.json")
  .then(function (json) {
    return json.post;
  })
  .then(function (post) {
    // ...
  });
```

## Promise.prototype.catch()

```js
// Promise.prototype.catch()方法是.then(null, rejection)或.then(undefined, rejection)的别名，用于指定发生错误时的回调函数
getJSON("/posts.json")
  .then(function (posts) {
    // ...
  })
  .catch(function (error) {
    // 处理 getJSON 和 前一个回调函数运行时发生的错误
    console.log("发生错误！", error);
  });
```

## Promise.prototype.finally()

```js
// finally()方法用于指定不管 Promise 对象最后状态如何，都会执行的操作。
// 该方法是 ES2018 引入标准的
promise
  .then((result) => {
    //···
  })
  .catch((error) => {
    //···
  })
  .finally(() => {
    //···
  });
```

## Promise.all()

```js
// Promise.all()方法用于将多个 Promise 实例，包装成一个新的 Promise 实例
// p的状态由p1、p2、p3决定，分成两种情况。
// （1）只有p1、p2、p3的状态都变成fulfilled，p的状态才会变成fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。
// （2）只要p1、p2、p3之中有一个被rejected，p的状态就变成rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。
const p = Promise.all([p1, p2, p3]);
```

## Promise.race()

```js
// Promise.race()方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例
// 只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。
// 那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。
const p = Promise.race([p1, p2, p3]);
```

## Promise.allSettled()

```js
// ES2020 引入了 Promise.allSettled()方法，用来确定一组异步操作是否都结束了（不管成功或失败）
// Promise.allSettled()方法接受一个数组作为参数，数组的每个成员都是一个 Promise 对象，并返回一个新的 Promise 对象。
// 只有等到参数数组的所有 Promise 对象都发生状态变更（不管是 fulfilled 还是 rejected），返回的 Promise 对象才会发生状态变更。
// 该方法返回的新的 Promise 实例，一旦发生状态变更，状态总是fulfilled，不会变成rejected

const promises = [fetch("/api-1"), fetch("/api-2"), fetch("/api-3")];
await Promise.allSettled(promises);
// 只有等三个请求都结束了（不管请求成功还是失败），removeLoadingIndicator()才会执行
removeLoadingIndicator();
```

## Promise.any()

```js
// ES2021 引入了Promise.any()方法。该方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例返回。
// 只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；
// 如果所有参数实例都变成rejected状态，包装实例就会变成rejected状态。

Promise.any([
  fetch("https://v8.dev/").then(() => "home"),
  fetch("https://v8.dev/blog").then(() => "blog"),
  fetch("https://v8.dev/docs").then(() => "docs"),
])
  .then((first) => {
    // 只要有一个 fetch() 请求成功
    console.log(first);
  })
  .catch((error) => {
    // 所有三个 fetch() 全部请求失败
    console.log(error);
  });
```

## Promise.resolve()

```js
// Promise.resolve()方法，返回一个resolved状态的 Promise 对象。
const p = Promise.resolve();

p.then(function () {
  // ...
});
```

## Promise.reject()

```js
// Promise.reject(reason)方法会返回一个新的 Promise 实例，该实例的状态为rejected
const p = Promise.reject("出错了");
// 等同于
const p = new Promise((resolve, reject) => reject("出错了"));

p.then(null, function (s) {
  console.log(s);
});
// 出错了
```
