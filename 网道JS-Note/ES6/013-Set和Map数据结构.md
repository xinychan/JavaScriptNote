[TOC]

# Set 和 Map 数据结构

## Set

### 基本用法

```js
// ES6 提供了新的数据结构 Set。它类似于数组，但是成员的值都是唯一的，没有重复的值。
// Set本身是一个构造函数，用来生成 Set 数据结构。

const s = new Set();
[2, 3, 5, 4, 5, 2, 2].forEach((x) => s.add(x));
for (let i of s) {
  console.log(i);
}
// 2 3 5 4
```

```js
// Set函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化。
// 例一
const set = new Set([1, 2, 3, 4, 4]);
[...set];
// [1, 2, 3, 4]

// 例二
const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size; // 5
```

```js
// 去除数组的重复成员
[...new Set(array)]

// 去除字符串里面的重复字符
[...new Set('ababbc')].join('')
// "abc"
```

```js
// 向 Set 加入值的时候，不会发生类型转换
// Set 使用 “Same-value-zero equality” 算法判断两个值是否相同
// 算法类似于精确相等运算符（===），主要的区别是向 Set 加入值时认为NaN等于自身，而精确相等运算符认为NaN不等于自身

let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set; // Set {NaN}
```

```js
// 另外，两个对象总是不相等的。
let set = new Set();

set.add({});
set.size; // 1

set.add({});
set.size; // 2
```

### Set 实例的属性和方法

#### Set 结构的实例属性

- Set.prototype.constructor：构造函数，默认就是 Set 函数。
- Set.prototype.size：返回 Set 实例的成员总数。

#### Set 实例的操作方法

- Set.prototype.add(value)：添加某个值，返回 Set 结构本身。
- Set.prototype.delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
- Set.prototype.has(value)：返回一个布尔值，表示该值是否为 Set 的成员。
- Set.prototype.clear()：清除所有成员，没有返回值。

```js
s.add(1).add(2).add(2);
// 注意2被加入了两次

s.size; // 2

s.has(1); // true
s.has(2); // true
s.has(3); // false

s.delete(2); // true
s.has(2); // false
```

```js
// Array.from()方法可以将 Set 结构转为数组
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);

// 通过 Array.from() 去除数组重复成员
function dedupe(array) {
  return Array.from(new Set(array));
}
dedupe([1, 1, 2, 3]); // [1, 2, 3]
```

#### Set 实例的遍历方法

- Set.prototype.keys()：返回键名的遍历器
- Set.prototype.values()：返回键值的遍历器
- Set.prototype.entries()：返回键值对的遍历器
- Set.prototype.forEach()：使用回调函数遍历每个成员

```js
// keys方法、values方法、entries方法返回的都是遍历器对象
// 由于 Set 结构没有键名，只有键值（或者说键名和键值是同一个值），所以keys方法和values方法的行为完全一致

let set = new Set(["red", "green", "blue"]);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

```js
// Set 结构的实例默认可遍历，它的默认遍历器生成函数就是它的values方法
Set.prototype[Symbol.iterator] === Set.prototype.values;
// true

// 这意味着，可以省略values方法，直接用for...of循环遍历 Set
let set = new Set(["red", "green", "blue"]);
for (let x of set) {
  console.log(x);
}
// red
// green
// blue
```

```js
// Set 结构的实例与数组一样，也拥有forEach方法，用于对每个成员执行某种操作，没有返回值。
let set = new Set([1, 4, 9]);
set.forEach((value, key) => console.log(key + " : " + value));
// 1 : 1
// 4 : 4
// 9 : 9
```

**遍历的应用**

```js
// 扩展运算符（...）内部使用for...of循环，所以也可以用于 Set 结构
let set = new Set(["red", "green", "blue"]);
let arr = [...set];
// ['red', 'green', 'blue']
```

```js
// 扩展运算符和 Set 结构相结合，就可以去除数组的重复成员
let arr = [3, 5, 2, 2, 5, 5];
let unique = [...new Set(arr)];
// [3, 5, 2]
```

```js
// 数组的map和filter方法也可以间接用于 Set

let set = new Set([1, 2, 3]);
set = new Set([...set].map((x) => x * 2));
// 返回Set结构：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter((x) => x % 2 == 0));
// 返回Set结构：{2, 4}
```

```js
// 因此使用 Set 可以很容易地实现并集（Union）、交集（Intersect）和差集（Difference）
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter((x) => b.has(x)));
// set {2, 3}

// （a 相对于 b 的）差集
let difference = new Set([...a].filter((x) => !b.has(x)));
// Set {1}
```

## WeakSet

### 含义

WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别。

```js
// 首先，WeakSet 的成员只能是对象和 Symbol 值，而不能是其他类型的值。
// 其次，WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用。
// 也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中
const ws = new WeakSet();
ws.add(1); // 报错
ws.add(Symbol()); // 不报错

// 由于上面这个特点，WeakSet 的成员是不适合引用的，因为它会随时消失
// 另外，由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的
// 因此 ES6 规定 WeakSet 不可遍历。
```

### 语法

```js
// WeakSet 是一个构造函数，可以使用new命令，创建 WeakSet 数据结构。
const ws = new WeakSet();
```

```js
// 作为构造函数，WeakSet 可以接受一个数组或类似数组的对象作为参数。（实际上，任何具有 Iterable 接口的对象，都可以作为 WeakSet 的参数。）
// 该数组的所有成员，都会自动成为 WeakSet 实例对象的成员。
const a = [
  [1, 2],
  [3, 4],
];
const ws = new WeakSet(a);
// WeakSet {[1, 2], [3, 4]}
```

```js
// 注意，是b数组的成员成为 WeakSet 的成员，而不是b数组本身
// 数组b的成员不是对象，加入 WeakSet 就会报错
const b = [3, 4];
const ws = new WeakSet(b);
// Uncaught TypeError: Invalid value used in weak set(…)
```

WeakSet 结构有以下三个方法。

- WeakSet.prototype.add(value)：向 WeakSet 实例添加一个新成员，返回 WeakSet 结构本身。
- WeakSet.prototype.delete(value)：清除 WeakSet 实例的指定成员，清除成功返回 true，如果在 WeakSet 中找不到该成员或该成员不是对象，返回 false。
- WeakSet.prototype.has(value)：返回一个布尔值，表示某个值是否在 WeakSet 实例之中。

```js
const ws = new WeakSet();
const obj = {};
const foo = {};

ws.add(window);
ws.add(obj);

ws.has(window); // true
ws.has(foo); // false

ws.delete(window); // true
ws.has(window); // false
```

```js
// WeakSet 没有size属性，没有办法遍历它的成员
ws.size; // undefined
ws.forEach; // undefined
ws.forEach(function (item) {
  console.log("WeakSet has " + item);
});
// TypeError: undefined is not a function
```

## Map

### 含义和基本用法

ES6 提供了 Map 数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。

```js
const m = new Map();
const o = { p: "Hello World" };

m.set(o, "content");
m.get(o); // "content"

m.has(o); // true
m.delete(o); // true
m.has(o); // false
```

```js
// 作为构造函数，Map 可以接受一个数组作为参数
const map = new Map([
  ["name", "张三"],
  ["title", "Author"],
]);

map.size; // 2
map.has("name"); // true
map.get("name"); // "张三"
map.has("title"); // true
map.get("title"); // "Author"

// 不仅仅是数组，任何具有 Iterator 接口、且每个成员都是一个双元素的数组的数据结构，都可以当作Map构造函数的参数
const set = new Set([
  ["foo", 1],
  ["bar", 2],
]);
const m1 = new Map(set);
m1.get("foo"); // 1

const m2 = new Map([["baz", 3]]);
const m3 = new Map(m2);
m3.get("baz"); // 3
```

```js
// 如果对同一个键多次赋值，后面的值将覆盖前面的值。
const map = new Map();
map.set(1, "aaa").set(1, "bbb");
map.get(1); // "bbb"
```

```js
// 如果读取一个未知的键，则返回undefined。
new Map().get("asfddfsasadf");
// undefined
```

```js
// 如果 Map 的键是一个简单类型的值（数字、字符串、布尔值），则只要两个值严格相等，Map 将其视为一个键
let map = new Map();

map.set(-0, 123);
map.get(+0); // 123

// 布尔值true和字符串true则是两个不同的键
map.set(true, 1);
map.set("true", 2);
map.get(true); // 1

// undefined和null是两个不同的键
map.set(undefined, 3);
map.set(null, 4);
map.get(undefined); // 3

// 虽然NaN不严格相等于自身，但 Map 将其视为同一个键
map.set(NaN, 123);
map.get(NaN); // 123
```

### Map 实例的属性和方法

#### Map 结构的实例属性

- Map.prototype.constructor：构造函数，默认就是 Map 函数。
- Map.prototype.size：返回 Map 实例的成员总数。

#### Map 实例的操作方法

- Map.prototype.set(key, value)：设置键名 key 对应的键值为 value，然后返回整个 Map 结构。如果 key 已经有值，则键值会被更新。
- Map.prototype.get(key)：读取 key 对应的键值，如果找不到 key，返回 undefined。
- Map.prototype.has(key)：返回一个布尔值，表示某个键是否在当前 Map 对象之中。
- Map.prototype.delete(key)：删除某个键，返回 true。如果删除失败，返回 false。
- Map.prototype.clear()：清除所有成员，没有返回值。

```js
// set
const m = new Map();
m.set("edition", 6); // 键是字符串
m.set(262, "standard"); // 键是数值
m.set(undefined, "nah"); // 键是 undefined

// get
const m = new Map();
const hello = function () {
  console.log("hello");
};
m.set(hello, "Hello ES6!"); // 键是函数
m.get(hello); // Hello ES6!

// has
const m = new Map();
m.set("edition", 6);
m.set(262, "standard");
m.set(undefined, "nah");
m.has("edition"); // true
m.has("years"); // false
m.has(262); // true
m.has(undefined); // true

// delete
const m = new Map();
m.set(undefined, "nah");
m.has(undefined); // true
m.delete(undefined);
m.has(undefined); // false

// clear
let map = new Map();
map.set("foo", true);
map.set("bar", false);
map.size; // 2
map.clear();
map.size; // 0
```

#### Map 实例的遍历方法

- Map.prototype.keys()：返回键名的遍历器。
- Map.prototype.values()：返回键值的遍历器。
- Map.prototype.entries()：返回所有成员的遍历器。
- Map.prototype.forEach()：遍历 Map 的所有成员。

```js
// 需要特别注意的是，Map 的遍历顺序就是插入顺序。
const map = new Map([
  ["F", "no"],
  ["T", "yes"],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"

// 等同于使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
// "F" "no"
// "T" "yes"
```

```js
// Map 结构转为数组结构，比较快速的方法是使用扩展运算符（...）
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```

```js
// 结合数组的map方法、filter方法，可以实现 Map 的遍历和过滤（Map 本身没有map和filter方法）。
const map0 = new Map().set(1, "a").set(2, "b").set(3, "c");

const map1 = new Map([...map0].filter(([k, v]) => k < 3));
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

const map2 = new Map([...map0].map(([k, v]) => [k * 2, "_" + v]));
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```

```js
// Map 还有一个forEach方法，可以实现遍历
map.forEach(function (value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```

### 与其他数据结构的互相转换

#### Map 转为数组

```js
// Map 转为数组最方便的方法，就是使用扩展运算符（...）
const myMap = new Map().set(true, 7).set({ foo: 3 }, ["abc"]);
[...myMap];
// [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

#### 数组 转为 Map

```js
// 将数组传入 Map 构造函数，就可以转为 Map
new Map([
  [true, 7],
  [{ foo: 3 }, ["abc"]],
]);
// Map {
//   true => 7,
//   Object {foo: 3} => ['abc']
// }
```

#### Map 转为对象

```js
// 如果所有 Map 的键都是字符串，它可以无损地转为对象
// 如果有非字符串的键名，那么这个键名会被转成字符串，再作为对象的键名
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k, v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

const myMap = new Map().set("yes", true).set("no", false);
strMapToObj(myMap);
// { yes: true, no: false }
```

#### 对象转为 Map

```js
// 对象转为 Map 可以通过Object.entries()
let obj = { a: 1, b: 2 };
let map = new Map(Object.entries(obj));
```

```js
// 也可以自己实现一个转换函数
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}
objToStrMap({ yes: true, no: false });
// Map {"yes" => true, "no" => false}
```

#### Map 转为 JSON

```js
// Map 的键名都是字符串，这时可以选择转为对象 JSON
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set("yes", true).set("no", false);
strMapToJson(myMap);
// '{"yes":true,"no":false}'
```

```js
// Map 的键名有非字符串，这时可以选择转为数组 JSON
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({ foo: 3 }, ["abc"]);
mapToArrayJson(myMap);
// '[[true,7],[{"foo":3},["abc"]]]'
```

#### JSON 转为 Map

```js
// JSON 转为 Map，正常情况下，所有键名都是字符串
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes": true, "no": false}');
// Map {'yes' => true, 'no' => false}
```

```js
// 如果整个 JSON 就是一个数组，且每个数组成员本身，又是一个有两个成员的数组。这时，它可以一一对应地转为 Map
// 这往往是 Map 转为数组 JSON 的逆操作
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]');
// Map {true => 7, Object {foo: 3} => ['abc']}
```

## WeakMap

### 含义

WeakMap 结构与 Map 结构类似，也是用于生成键值对的集合。但是，WeakMap 与 Map 的区别有两点。

```js
// 首先，WeakMap只接受对象（null除外）和 Symbol 值作为键名，不接受其他类型的值作为键名。
// 其次，WeakMap的键名所指向的对象，不计入垃圾回收机制。
// WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。

const map = new WeakMap();
map.set(1, 2); // 报错
map.set(null, 2); // 报错
map.set(Symbol(), 2); // 不报错
```

```js
// 注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。
const wm = new WeakMap();
let key = {};
let obj = { foo: 1 };

// 键值obj是正常引用。所以，即使在 WeakMap 外部消除了obj的引用，WeakMap 内部的引用依然存在。
wm.set(key, obj);
obj = null;
wm.get(key);
// Object {foo: 1}
```

### 语法

WeakMap 与 Map 在 API 上的区别主要是两个：
一是没有遍历操作（即没有 keys()、values()和 entries()方法），也没有 size 属性。
二是无法清空，即不支持 clear 方法。

```js
// WeakMap只有四个方法可用：get()、set()、has()、delete()
const wm = new WeakMap();

// size、forEach、clear 方法都不存在
wm.size; // undefined
wm.forEach; // undefined
wm.clear; // undefined
```

## WeakRef

```js
// WeakSet 和 WeakMap 是基于弱引用的数据结构，ES2021 更进一步，提供了 WeakRef 对象，用于直接创建对象的弱引用。
let target = {};
let wr = new WeakRef(target);
```

```js
// WeakRef 实例对象有一个deref()方法，如果原始对象存在，该方法返回原始对象
// 如果原始对象已经被垃圾回收机制清除，该方法返回undefined
let target = {};
let wr = new WeakRef(target);

let obj = wr.deref();
if (obj) {
  // target 未被垃圾回收机制清除
}
```

```js
// 弱引用对象的一大用处，就是作为缓存，未被清除时可以从缓存取值，一旦清除缓存就自动失效。
function makeWeakCached(f) {
  const cache = new Map();
  return (key) => {
    const ref = cache.get(key);
    if (ref) {
      const cached = ref.deref();
      if (cached !== undefined) return cached;
    }

    const fresh = f(key);
    cache.set(key, new WeakRef(fresh));
    return fresh;
  };
}

const getImageCached = makeWeakCached(getImage);
```

## FinalizationRegistry

ES2021 引入了清理器注册表功能 FinalizationRegistry，用来指定目标对象被垃圾回收机制清除以后，所要执行的回调函数。

```js
// 首先，新建一个注册表实例。
const registry = new FinalizationRegistry((heldValue) => {
  // do something
});
```

```js
// 然后，注册表实例的register()方法，用来注册所要观察的目标对象。
// 一旦该对象被垃圾回收机制清除，注册表就会在清除完成后，调用早前注册的回调函数 registry，并将some value作为参数（给heldValue）传入回调函数
registry.register(theObject, "some value");
```

```js
// 最后，如果以后还想取消已经注册的回调函数，则要向register()传入第三个参数，作为标记值。
// 这个标记值必须是对象，一般都用原始对象。接着，再使用注册表实例对象的unregister()方法取消注册。
registry.register(theObject, "some value", theObject);
// ...其他操作...
registry.unregister(theObject);
```
