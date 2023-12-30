[TOC]

# Iterator

## Iterator（遍历器）的概念

JavaScript 原有的表示“集合”的数据结构，主要是数组（Array）和对象（Object），ES6 又添加了 Map 和 Set。

```
Iterator 的作用有三个：
1. 为各种数据结构，提供一个统一的、简便的访问接口；
2. 使得数据结构的成员能够按某种次序排列；
3. ES6 创造了一种新的遍历命令 for...of 循环，Iterator 接口主要供 for...of 消费。
```

```
Iterator 的遍历过程是这样的。
1. 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
2. 第一次调用指针对象的 next 方法，可以将指针指向数据结构的第一个成员。
3. 第二次调用指针对象的 next 方法，指针就指向数据结构的第二个成员。
4. 不断调用指针对象的 next 方法，直到它指向数据结构的结束位置。
```

每一次调用 next 方法，都会返回数据结构的当前成员的信息。
具体来说，就是返回一个包含 value 和 done 两个属性的对象。其中，value 属性是当前成员的值，done 属性是一个布尔值，表示遍历是否结束。

```js
// 模拟next方法返回值
var it = makeIterator(["a", "b"]);

it.next(); // { value: "a", done: false }
it.next(); // { value: "b", done: false }
it.next(); // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function () {
      return nextIndex < array.length
        ? { value: array[nextIndex++], done: false }
        : { value: undefined, done: true };
    },
  };
}
```

## 默认 Iterator 接口

Iterator 接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即 for...of 循环。
ES6 规定，默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性，或者说，一个数据结构只要具有 Symbol.iterator 属性，就可以认为是“可遍历的”。

```
原生具备 Iterator 接口的数据结构如下。
Array
Map
Set
String
TypedArray
函数的 arguments 对象
NodeList 对象
```

```js
// 数组的Symbol.iterator属性
let arr = ["a", "b", "c"];
// 遍历器对象
let iter = arr[Symbol.iterator]();

iter.next(); // { value: 'a', done: false }
iter.next(); // { value: 'b', done: false }
iter.next(); // { value: 'c', done: false }
iter.next(); // { value: undefined, done: true }
```

## 调用 Iterator 接口的场合

### 解构赋值

```js
// 对数组和 Set 结构进行解构赋值时，会默认调用Symbol.iterator方法。
let set = new Set().add("a").add("b").add("c");

let [x, y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

### 扩展运算符

```js
// 扩展运算符（...）也会调用默认的 Iterator 接口。
// 例一
var str = "hello";
[...str]; //  ['h','e','l','l','o']

// 例二
let arr = ["b", "c"];
["a", ...arr, "d"];
// ['a', 'b', 'c', 'd']
```

### `yield*`

```js
// yield*后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。
let generator = function* () {
  yield 1;
  yield* [2, 3, 4];
  yield 5;
};

var iterator = generator();

iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }
iterator.next(); // { value: 4, done: false }
iterator.next(); // { value: 5, done: false }
iterator.next(); // { value: undefined, done: true }
```

### 其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。

```
示例：
for...of
Array.from()
Map(), Set(), WeakMap(), WeakSet()（比如new Map([['a',1],['b',2]])）
Promise.all()
Promise.race()
```

## 字符串的 Iterator 接口

```js
// 字符串是一个类似数组的对象，也原生具有 Iterator 接口。
var someString = "hi";
typeof someString[Symbol.iterator];
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next(); // { value: "h", done: false }
iterator.next(); // { value: "i", done: false }
iterator.next(); // { value: undefined, done: true }
```

## Iterator 接口与 Generator 函数

```js
// Symbol.iterator()方法的最简单实现，是通过 Generator 函数
let myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  },
};
[...myIterable]; // [1, 2, 3]

// 或者采用下面的简洁写法

let obj = {
  *[Symbol.iterator]() {
    yield "hello";
    yield "world";
  },
};

for (let x of obj) {
  console.log(x);
}
// "hello"
// "world"
```

## for...of 循环

ES6 借鉴 C++、Java、C# 和 Python 语言，引入了 for...of 循环，作为遍历所有数据结构的统一的方法。
一个数据结构只要部署了 Symbol.iterator 属性，就被视为具有 iterator 接口，就可以用 for...of 循环遍历它的成员。
也就是说，for...of 循环内部调用的是数据结构的 Symbol.iterator 方法。

```js
// 最原始的for循环
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}

// 数组提供内置的forEach方法
// 无法中途跳出forEach循环，break命令或return命令都不能奏效
myArray.forEach(function (value) {
  console.log(value);
});

// for...in循环可以遍历数组的键名
// 数组的键名是数字，但是for...in循环是以字符串作为键名“0”、“1”、“2”等等
// for...in循环不仅遍历数组数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
// 某些情况下，for...in循环会以任意顺序遍历键名
for (var index in myArray) {
  console.log(myArray[index]);
}

// for...of 循环与其他遍历语法的比较
// 有着同for...in一样的简洁语法，但是没有for...in那些缺点
// 不同于forEach方法，它可以与break、continue和return配合使用
// 提供了遍历所有数据结构的统一操作接口
for (let value of myArray) {
  console.log(value);
}
```
