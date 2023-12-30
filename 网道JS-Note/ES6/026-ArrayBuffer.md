[TOC]

# ArrayBuffer

ArrayBuffer 对象、TypedArray 视图和 DataView 视图是 JavaScript 操作二进制数据的一个接口。
这些对象早就存在，属于独立的规格（2011 年 2 月发布），ES6 将它们纳入了 ECMAScript 规格，并且增加了新的方法。
它们都是以数组的语法处理二进制数据，所以统称为二进制数组。

- ArrayBuffer 对象：代表内存之中的一段二进制数据，可以通过“视图”进行操作。“视图”部署了数组接口，这意味着，可以用数组的方法操作内存。
- TypedArray 视图：共包括 9 种类型的视图，比如 Uint8Array（无符号 8 位整数）数组视图, Int16Array（16 位整数）数组视图, Float32Array（32 位浮点数）数组视图等等。
- DataView 视图：可以自定义复合格式的视图，比如第一个字节是 Uint8（无符号 8 位整数）、第二、三个字节是 Int16（16 位整数）、第四个字节开始是 Float32（32 位浮点数）等等，此外还可以自定义字节序。

简单说，ArrayBuffer 对象代表原始的二进制数据，TypedArray 视图用来读写简单类型的二进制数据，DataView 视图用来读写复杂类型的二进制数据。

注意，二进制数组并不是真正的数组，而是类似数组的对象。

## ArrayBuffer 对象

### 概述

```js
// ArrayBuffer对象代表储存二进制数据的一段内存，它不能直接读写
// 只能通过视图（TypedArray视图和DataView视图)来读写，视图的作用是以指定格式解读二进制数据。
const buf = new ArrayBuffer(32);
const dataView = new DataView(buf);
dataView.getUint8(0); // 0
```

```js
// TypedArray视图，它不是一个构造函数，而是一组构造函数，代表不同的数据格式
const buffer = new ArrayBuffer(12);

// 由于两个视图对应的是同一段内存，一个视图修改底层内存，会影响到另一个视图
const x1 = new Int32Array(buffer);
x1[0] = 1;
const x2 = new Uint8Array(buffer);
x2[0] = 2;

x1[0]; // 2
```

```js
// TypedArray视图的构造函数，除了接受ArrayBuffer实例作为参数，还可以接受普通数组作为参数
// 直接分配内存生成底层的ArrayBuffer实例，并同时完成对这段内存的赋值。
const typedArray = new Uint8Array([0, 1, 2]);
typedArray.length; // 3

typedArray[0] = 5;
typedArray; // [5, 1, 2]
```

### ArrayBuffer.prototype.byteLength

```js
// ArrayBuffer实例的byteLength属性，返回所分配的内存区域的字节长度
const buffer = new ArrayBuffer(32);
buffer.byteLength;
// 32
```

```js
// 如果要分配的内存区域很大，有可能分配失败（因为没有那么多的连续空余内存），所以有必要检查是否分配成功。
if (buffer.byteLength === n) {
  // 成功
} else {
  // 失败
}
```

### ArrayBuffer.prototype.slice()

```js
// ArrayBuffer实例slice方法，允许将内存区域的一部分，拷贝生成一个新的ArrayBuffer对象
const buffer = new ArrayBuffer(8);
const newBuffer = buffer.slice(0, 3);
// 除了slice方法，ArrayBuffer对象不提供任何直接读写内存的方法，只允许在其上方建立视图，然后通过视图读写。
```

### ArrayBuffer.isView()

```js
// ArrayBuffer静态方法isView，表示参数是否为ArrayBuffer的视图实例，即是否为TypedArray实例或DataView实例
const buffer = new ArrayBuffer(8);
ArrayBuffer.isView(buffer); // false

const v = new Int32Array(buffer);
ArrayBuffer.isView(v); // true
```

## TypedArray 视图

### 概述

ArrayBuffer 对象作为内存区域，可以存放多种类型的数据。
同一段内存，不同数据有不同的解读方式，这就叫做“视图”（view）。
ArrayBuffer 有两种视图，一种是 TypedArray 视图，另一种是 DataView 视图。
前者的数组成员都是同一个数据类型，后者的数组成员可以是不同的数据类型。

TypedArray 视图一共包括 9 种类型，每一种视图都是一种构造函数

- Int8Array：8 位有符号整数，长度 1 个字节。
- Uint8Array：8 位无符号整数，长度 1 个字节。
- Uint8ClampedArray：8 位无符号整数，长度 1 个字节，溢出处理不同。
- Int16Array：16 位有符号整数，长度 2 个字节。
- Uint16Array：16 位无符号整数，长度 2 个字节。
- Int32Array：32 位有符号整数，长度 4 个字节。
- Uint32Array：32 位无符号整数，长度 4 个字节。
- Float32Array：32 位浮点数，长度 4 个字节。
- Float64Array：64 位浮点数，长度 8 个字节。

普通数组与 TypedArray 数组的差异主要在以下方面

- TypedArray 数组的所有成员，都是同一种类型。
- TypedArray 数组的成员是连续的，不会有空位。
- TypedArray 数组成员的默认值为 0。比如，new Array(10)返回一个普通数组，里面没有任何成员，只是 10 个空位；new Uint8Array(10)返回一个 TypedArray 数组，里面 10 个成员都是 0。
- TypedArray 数组只是一层视图，本身不储存数据，它的数据都储存在底层的 ArrayBuffer 对象之中，要获取底层对象必须使用 buffer 属性。

### 构造函数

TypedArray 数组提供 9 种构造函数，用来生成相应类型的数组实例。

```js
// TypedArray(buffer, byteOffset=0, length?)
// 同一个ArrayBuffer对象之上，可以根据不同的数据类型，建立多个视图。

// 创建一个8字节的ArrayBuffer
const b = new ArrayBuffer(8);

// 创建一个指向b的Int32视图，开始于字节0，直到缓冲区的末尾
const v1 = new Int32Array(b);

// 创建一个指向b的Uint8视图，开始于字节2，直到缓冲区的末尾
const v2 = new Uint8Array(b, 2);

// 创建一个指向b的Int16视图，开始于字节2，长度为2
const v3 = new Int16Array(b, 2, 2);

// v1[0]是一个 32 位整数，指向字节 0 ～字节 3；
// v2[0]是一个 8 位无符号整数，指向字节 2；
// v3[0]是一个 16 位整数，指向字节 2 ～字节 3。
// 只要任何一个视图对内存有所修改，就会在另外两个视图上反应出来。
```

```js
// TypedArray(length)
// 视图还可以不通过ArrayBuffer对象，直接分配内存而生成。
const f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```

```js
// TypedArray(typedArray)
// TypedArray 数组的构造函数，可以接受另一个TypedArray实例作为参数。
const typedArray = new Int8Array(new Uint8Array(4));

// 注意，此时生成的新数组，只是复制了参数数组的值，对应的底层内存是不一样的。
// 新数组会开辟一段新的内存储存数据，不会在原数组的内存之上建立视图。
```

```js
// TypedArray(arrayLikeObject)
// 构造函数的参数也可以是一个普通数组，然后直接生成TypedArray实例。
const typedArray = new Uint8Array([1, 2, 3, 4]);

// 注意，这时TypedArray视图会重新开辟内存，不会在原数组的内存上建立视图。

// TypedArray 数组也可以转换回普通数组。
const normalArray = [...typedArray];
// or
const normalArray = Array.from(typedArray);
// or
const normalArray = Array.prototype.slice.call(typedArray);
```

### 数组方法

普通数组的操作方法和属性，对 TypedArray 数组完全适用。

- TypedArray.prototype.copyWithin(target, start[, end = this.length])
- TypedArray.prototype.entries()
- TypedArray.prototype.every(callbackfn, thisArg?)
- TypedArray.prototype.fill(value, start=0, end=this.length)
- TypedArray.prototype.filter(callbackfn, thisArg?)
- TypedArray.prototype.find(predicate, thisArg?)
- TypedArray.prototype.findIndex(predicate, thisArg?)
- TypedArray.prototype.forEach(callbackfn, thisArg?)
- TypedArray.prototype.indexOf(searchElement, fromIndex=0)
- TypedArray.prototype.join(separator)
- TypedArray.prototype.keys()
- TypedArray.prototype.lastIndexOf(searchElement, fromIndex?)
- TypedArray.prototype.map(callbackfn, thisArg?)
- TypedArray.prototype.reduce(callbackfn, initialValue?)
- TypedArray.prototype.reduceRight(callbackfn, initialValue?)
- TypedArray.prototype.reverse()
- TypedArray.prototype.slice(start=0, end=this.length)
- TypedArray.prototype.some(callbackfn, thisArg?)
- TypedArray.prototype.sort(comparefn)
- TypedArray.prototype.toLocaleString(reserved1?, reserved2?)
- TypedArray.prototype.toString()
- TypedArray.prototype.values()

### 字节序

```js
// 字节序指的是数值在内存中的表示方式。

const buffer = new ArrayBuffer(16);
const int32View = new Int32Array(buffer);

// 由于每个 32 位整数占据 4 个字节，所以一共可以写入 4 个整数，依次为 0，2，4，6。
for (let i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}

// 因为 x86 体系的计算机都采用小端字节序，相对重要的字节排在后面的内存地址，相对不重要字节排在前面的内存地址
// 如果在这段数据上接着建立一个 16 位整数的视图，则可以读出完全不一样的结果
const int16View = new Int16Array(buffer);

for (let i = 0; i < int16View.length; i++) {
  console.log("Entry " + i + ": " + int16View[i]);
}
// Entry 0: 0
// Entry 1: 0
// Entry 2: 2
// Entry 3: 0
// Entry 4: 4
// Entry 5: 0
// Entry 6: 6
// Entry 7: 0
```

### BYTES_PER_ELEMENT 属性

```js
// 每一种视图的构造函数，都有一个BYTES_PER_ELEMENT属性，表示这种数据类型占据的字节数。

Int8Array.BYTES_PER_ELEMENT; // 1
Uint8Array.BYTES_PER_ELEMENT; // 1
Uint8ClampedArray.BYTES_PER_ELEMENT; // 1
Int16Array.BYTES_PER_ELEMENT; // 2
Uint16Array.BYTES_PER_ELEMENT; // 2
Int32Array.BYTES_PER_ELEMENT; // 4
Uint32Array.BYTES_PER_ELEMENT; // 4
Float32Array.BYTES_PER_ELEMENT; // 4
Float64Array.BYTES_PER_ELEMENT; // 8

// 这个属性在TypedArray实例上也能获取，即有TypedArray.prototype.BYTES_PER_ELEMENT
```

### ArrayBuffer 与字符串的互相转换

ArrayBuffer 和字符串的相互转换，使用原生 TextEncoder 和 TextDecoder 方法。

```js
/**
 * Convert ArrayBuffer/TypedArray to String via TextDecoder
 *
 * @see https://developer.mozilla.org/en-US/docs/Web/API/TextDecoder
 */
function ab2str(
  input:
    | ArrayBuffer
    | Uint8Array
    | Int8Array
    | Uint16Array
    | Int16Array
    | Uint32Array
    | Int32Array,
  outputEncoding: string = "utf8"
): string {
  const decoder = new TextDecoder(outputEncoding);
  return decoder.decode(input);
}

/**
 * Convert String to ArrayBuffer via TextEncoder
 *
 * @see https://developer.mozilla.org/zh-CN/docs/Web/API/TextEncoder
 */
function str2ab(input: string): ArrayBuffer {
  const view = str2Uint8Array(input);
  return view.buffer;
}

/** Convert String to Uint8Array */
function str2Uint8Array(input: string): Uint8Array {
  const encoder = new TextEncoder();
  const view = encoder.encode(input);
  return view;
}
```

### 溢出

```js
// 不同的视图类型，所能容纳的数值范围是确定的。超出这个范围，就会出现溢出。
// TypedArray 数组的溢出处理规则，简单来说，就是抛弃溢出的位，然后按照视图类型进行解释。

const uint8 = new Uint8Array(1);

// uint8是一个 8 位视图，而 256 的二进制形式是一个 9 位的值100000000，这时就会发生溢出
// 根据规则，只会保留后 8 位，即00000000
uint8[0] = 256;
uint8[0]; // 0

// 负向溢出
uint8[0] = -1;
uint8[0]; // 255
```

### 主要的属性

```js
// TypedArray实例的buffer属性，返回整段内存区域对应的ArrayBuffer对象。
// 该属性为只读属性。
const a = new Float32Array(64);
const b = new Uint8Array(a.buffer);
```

```js
// byteLength属性返回 TypedArray 数组占据的内存长度，单位为字节。
// byteOffset属性返回 TypedArray 数组从底层ArrayBuffer对象的哪个字节开始。
// 这两个属性都是只读属性。

const b = new ArrayBuffer(8);

const v1 = new Int32Array(b);
const v2 = new Uint8Array(b, 2);
const v3 = new Int16Array(b, 2, 2);

v1.byteLength; // 8
v2.byteLength; // 6
v3.byteLength; // 4

v1.byteOffset; // 0
v2.byteOffset; // 2
v3.byteOffset; // 2
```

```js
// length属性表示 TypedArray 数组含有多少个成员。
const a = new Int16Array(8);

a.length; // 8；成员长度
a.byteLength; // 16；字节长度
```

## 复合视图

```js
// 由于视图的构造函数可以指定起始位置和长度，所以在同一段内存之中，可以依次存放不同类型的数据，这叫做“复合视图”。

const buffer = new ArrayBuffer(24);

const idView = new Uint32Array(buffer, 0, 1);
const usernameView = new Uint8Array(buffer, 4, 16);
const amountDueView = new Float32Array(buffer, 20, 1);
```

## DataView 视图

### 概念

如果一段数据包括多种类型（比如服务器传来的 HTTP 数据），这时除了建立 ArrayBuffer 对象的复合视图以外，还可以通过 DataView 视图进行操作。

DataView 视图提供更多操作选项，而且支持设定字节序。

```js
// DataView视图本身也是构造函数，接受一个ArrayBuffer对象作为参数，生成视图。
const buffer = new ArrayBuffer(24);
const dv = new DataView(buffer);
```

### 属性

DataView 实例有以下属性。

```
DataView.prototype.buffer：返回对应的 ArrayBuffer 对象
DataView.prototype.byteLength：返回占据的内存字节长度
DataView.prototype.byteOffset：返回当前视图从对应的 ArrayBuffer 对象的哪个字节开始
```

### 读取内存

DataView 实例提供 10 个方法读取内存。

```
getInt8：读取 1 个字节，返回一个 8 位整数。
getUint8：读取 1 个字节，返回一个无符号的 8 位整数。
getInt16：读取 2 个字节，返回一个 16 位整数。
getUint16：读取 2 个字节，返回一个无符号的 16 位整数。
getInt32：读取 4 个字节，返回一个 32 位整数。
getUint32：读取 4 个字节，返回一个无符号的 32 位整数。
getBigInt64：读取 8 个字节，返回一个 64 位整数。
getBigUint64：读取 8 个字节，返回一个无符号的 64 位整数。
getFloat32：读取 4 个字节，返回一个 32 位浮点数。
getFloat64：读取 8 个字节，返回一个 64 位浮点数。
```

```js
// 这一系列get方法的参数都是一个字节序号（不能是负数，否则会报错），表示从哪个字节开始读取。
const buffer = new ArrayBuffer(24);
const dv = new DataView(buffer);

// 从第1个字节读取一个8位无符号整数
const v1 = dv.getUint8(0);

// 从第2个字节读取一个16位无符号整数
const v2 = dv.getUint16(1);

// 从第4个字节读取一个16位无符号整数
const v3 = dv.getUint16(3);
```

```js
// 如果一次读取两个或两个以上字节，就必须明确数据的存储方式，到底是小端字节序还是大端字节序。
// 默认情况下，DataView的get方法使用大端字节序解读数据
// 如果需要使用小端字节序解读，必须在get方法的第二个参数指定true

// 小端字节序
const v1 = dv.getUint16(1, true);

// 大端字节序
const v2 = dv.getUint16(3, false);

// 大端字节序
const v3 = dv.getUint16(3);
```

### 写入内存

DataView 视图提供 10 个方法写入内存。

```
setInt8：写入 1 个字节的 8 位整数。
setUint8：写入 1 个字节的 8 位无符号整数。
setInt16：写入 2 个字节的 16 位整数。
setUint16：写入 2 个字节的 16 位无符号整数。
setInt32：写入 4 个字节的 32 位整数。
setUint32：写入 4 个字节的 32 位无符号整数。
setBigInt64：写入 8 个字节的 64 位整数。
setBigUint64：写入 8 个字节的 64 位无符号整数。
setFloat32：写入 4 个字节的 32 位浮点数。
setFloat64：写入 8 个字节的 64 位浮点数。
```

```js
// 这一系列set方法，接受两个参数，第一个参数是字节序号，表示从哪个字节开始写入，第二个参数为写入的数据。
// 对于那些写入两个或两个以上字节的方法，需要指定第三个参数，false或者undefined表示使用大端字节序写入，true表示使用小端字节序写入

// 在第1个字节，以大端字节序写入值为25的32位整数
dv.setInt32(0, 25, false);

// 在第5个字节，以大端字节序写入值为25的32位整数
dv.setInt32(4, 25);

// 在第9个字节，以小端字节序写入值为2.5的32位浮点数
dv.setFloat32(8, 2.5, true);
```

## SharedArrayBuffer

JavaScript 是单线程的，Web worker 引入了多线程：主线程用来与用户互动，Worker 线程用来承担计算任务。
每个线程的数据都是隔离的，通过 postMessage()通信。

```js
// 主线程
// 主线程新建了一个 Worker 线程
const w = new Worker("myworker.js");

// 主线程
// 主线程通过w.postMessage向 Worker 线程发消息，同时通过message事件监听 Worker 线程的回应
w.postMessage("hi");
w.onmessage = function (ev) {
  console.log(ev.data);
};

// Worker 线程
// Worker 线程也通过监听message事件，来获取主线程发来的消息，并回复消息
onmessage = function (ev) {
  console.log(ev.data);
  postMessage("ho");
};
```

ES2017 引入 SharedArrayBuffer，允许 Worker 线程与主线程共享同一块内存。
SharedArrayBuffer 的 API 与 ArrayBuffer 一模一样，唯一的区别是后者无法共享数据。

```js
// 主线程

// 新建 1KB 共享内存
const sharedBuffer = new SharedArrayBuffer(1024);
// 主线程将共享内存的地址发送出去
w.postMessage(sharedBuffer);
// 在共享内存上建立视图，供写入数据
const sharedArray = new Int32Array(sharedBuffer);
```

```js
// Worker 线程
onmessage = function (ev) {
  // 主线程共享的数据，就是 1KB 的共享内存
  const sharedBuffer = ev.data;
  // 在共享内存上建立视图，方便读写
  const sharedArray = new Int32Array(sharedBuffer);
};
// 共享内存也可以在 Worker 线程创建，发给主线程。
```

## Atomics 对象

SharedArrayBuffer API 提供 Atomics 对象，保证所有共享内存的操作都是“原子性”的，并且可以在所有线程内同步。

```js
// 主线程
const sab = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 100000);
const ia = new Int32Array(sab);

for (let i = 0; i < ia.length; i++) {
  ia[i] = primes.next(); // 将质数放入 ia
}

// worker 线程
// Worker 线程直接改写共享内存ia[112]++是不正确的。
// 因为这行语句会被编译成多条机器指令，这些指令之间无法保证不会插入其他进程的指令。
ia[112]++; // 错误
// Atomics对象可以保证一个操作所对应的多条机器指令，一定是作为一个整体运行的，中间不会被打断。
// Atomics所涉及的操作都可以看作是原子性的单操作，这可以避免线程竞争，提高多线程共享内存时的操作安全。
Atomics.add(ia, 112, 1); // 正确
```

### Atomics 的方法

```js
// Atomics.store()方法用来向共享内存写入数据，返回值为写入的值
Atomics.load(typedArray, index);

// Atomics.load()方法用来从共享内存读出数据
Atomics.store(typedArray, index, value);

// Atomics.exchange()方法也可以用来向共享内存写入数据，返回值为被替换的值
Atomics.exchange(typedArray, index, value);
```

```js
// Atomics.wait() 通知 Worker 线程进入休眠
Atomics.wait(sharedArray, index, value, timeout);

// Atomics.notify() 唤醒 Worker 线程
Atomics.notify(sharedArray, index, count);
```

```js
// Atomics.add用于将value加到sharedArray[index]，返回sharedArray[index]旧的值
Atomics.add(sharedArray, index, value);

// Atomics.sub用于将value从sharedArray[index]减去，返回sharedArray[index]旧的值
Atomics.sub(sharedArray, index, value);

// Atomics.and用于将value与sharedArray[index]进行位运算and，放入sharedArray[index]，并返回旧的值
Atomics.and(sharedArray, index, value);

// Atomics.or用于将value与sharedArray[index]进行位运算or，放入sharedArray[index]，并返回旧的值
Atomics.or(sharedArray, index, value);

// Atomic.xor用于将vaule与sharedArray[index]进行位运算xor，放入sharedArray[index]，并返回旧的值
Atomics.xor(sharedArray, index, value);
```
