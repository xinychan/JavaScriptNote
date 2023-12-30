[TOC]

# Reflect

## 概述

Reflect 对象与 Proxy 对象一样，也是 ES6 为了操作对象而提供的新 API。

Reflect 对象的设计目的有这样几个。

- 将 Object 对象的一些明显属于语言内部的方法（比如 Object.defineProperty），放到 Reflect 对象上。

现阶段，某些方法同时在 Object 和 Reflect 对象上部署，未来的新方法将只部署在 Reflect 对象上。
也就是说，从 Reflect 对象上可以拿到语言内部的方法。

```js
// 老写法
Object.defineProperty(target, property, attributes);

// 新写法
Reflect.defineProperty(target, property, attributes);
```

- 修改某些 Object 方法的返回结果，让其变得更合理。

比如，Object.defineProperty(obj, name, desc)在无法定义属性时，会抛出一个错误，而 Reflect.defineProperty(obj, name, desc)则会返回 false。

```js
// 老写法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新写法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```

- 让 Object 操作都变成函数行为。

某些 Object 操作是命令式，比如`name in obj`和`delete obj[name]`，而 Reflect.has(obj, name)和 Reflect.deleteProperty(obj, name)让它们变成了函数行为。

```js
// 老写法，命令式
"assign" in Object; // true

// 新写法，函数行为
Reflect.has(Object, "assign"); // true
```

- Reflect 对象的方法与 Proxy 对象的方法一一对应，只要是 Proxy 对象的方法，就能在 Reflect 对象上找到对应的方法。

这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。
也就是说，不管 Proxy 怎么修改默认行为，你总可以在 Reflect 上获取默认行为。

```js
// 每一个Proxy对象的拦截操作（get、delete、has），内部都调用对应的Reflect方法，保证原生行为能够正常执行。
// 添加的工作，就是将每一个操作输出一行日志。
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log("get", target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log("delete" + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log("has" + name);
    return Reflect.has(target, name);
  },
});
```

## 静态方法

Reflect 对象一共有 13 个静态方法。

```
Reflect.apply(target, thisArg, args):等同于Function.prototype.apply.call(func, thisArg, args)，用于绑定this对象后执行给定函数。
Reflect.construct(target, args):等同于new target(...args)，这提供了一种不使用new，来调用构造函数的方法。
Reflect.get(target, name, receiver):查找并返回target对象的name属性，如果没有该属性，则返回undefined。
Reflect.set(target, name, value, receiver):设置target对象的name属性等于value。
Reflect.defineProperty(target, name, desc):基本等同于Object.defineProperty，用来为对象定义属性。
Reflect.deleteProperty(target, name):等同于delete obj[name]，用于删除对象的属性。
Reflect.has(target, name):对应name in obj里面的in运算符。
Reflect.ownKeys(target):用于返回对象的所有属性，基本等同于Object.getOwnPropertyNames与Object.getOwnPropertySymbols之和。
Reflect.isExtensible(target):对应Object.isExtensible，返回一个布尔值，表示当前对象是否可扩展。
Reflect.preventExtensions(target):对应Object.preventExtensions方法，用于让一个对象变为不可扩展。它返回一个布尔值，表示是否操作成功。
Reflect.getOwnPropertyDescriptor(target, name):基本等同于Object.getOwnPropertyDescriptor，用于得到指定属性的描述对象
Reflect.getPrototypeOf(target):用于读取对象的__proto__属性，对应Object.getPrototypeOf(obj)。
Reflect.setPrototypeOf(target, prototype):用于设置目标对象的原型（prototype），对应Object.setPrototypeOf(obj, newProto)方法。它返回一个布尔值，表示是否设置成功。
```
