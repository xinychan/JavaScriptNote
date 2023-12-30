[TOC]

# Symbol

## 概述

ES5 的对象属性名都是字符串，这容易造成属性名的冲突。
比如，想为某个对象添加新的方法（mixin 模式），新方法的名字就有可能与现有方法产生冲突。
如果有一种机制，保证每个属性的名字都是独一无二的就好了，这样就从根本上防止属性名的冲突。这就是 ES6 引入 Symbol 的原因。

ES6 引入了一种新的原始数据类型 Symbol，表示独一无二的值。
它属于 JavaScript 语言的原生数据类型之一，其他数据类型是：
undefined、null、布尔值（Boolean）、字符串（String）、数值（Number）、大整数（BigInt）、对象（Object）

对象的属性名现在可以有两种类型，一种是原来就有的字符串，另一种就是新增的 Symbol 类型。
凡是属性名属于 Symbol 类型，就都是独一无二的，可以保证不会与其他属性名产生冲突。

```js
// Symbol 值通过Symbol()函数生成
let s = Symbol();

// 变量s是 Symbol 数据类型
typeof s;
// "symbol"
```

因为生成的 Symbol 是一个原始类型的值，不是对象，所以不能使用 new 命令来调用。
另外，由于 Symbol 值不是对象，所以也不能添加属性。基本上，它是一种类似于字符串的数据类型。

```js
// Symbol()函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述
let s1 = Symbol("foo");
let s2 = Symbol("bar");

s1; // Symbol(foo)
s2; // Symbol(bar)

s1.toString(); // "Symbol(foo)"
s2.toString(); // "Symbol(bar)"
```

```js
// 如果 Symbol 的参数是一个对象，就会调用该对象的toString()方法，将其转为字符串，然后才生成一个 Symbol 值
const obj = {
  toString() {
    return "abc";
  },
};
const sym = Symbol(obj);
sym; // Symbol(abc)
```

```js
// 注意，Symbol()函数的参数只是表示对当前 Symbol 值的描述，因此相同参数的Symbol函数的返回值是不相等的

// 没有参数的情况
let s1 = Symbol();
let s2 = Symbol();

s1 === s2; // false

// 有参数的情况
let s1 = Symbol("foo");
let s2 = Symbol("foo");

s1 === s2; // false
```

```js
// Symbol 值不能与其他类型的值进行运算，会报错。
let sym = Symbol("My symbol");

"your symbol is " + sym;
// TypeError: can't convert symbol to string
`your symbol is ${sym}`;
// TypeError: can't convert symbol to string
```

```js
// Symbol 值可以显式转为字符串。
let sym = Symbol("My symbol");

String(sym); // 'Symbol(My symbol)'
sym.toString(); // 'Symbol(My symbol)'
```

```js
// Symbol 值也可以转为布尔值，但是不能转为数值。
let sym = Symbol();
Boolean(sym); // true
!sym; // false

if (sym) {
  // ...
}

Number(sym); // TypeError
sym + 2; // TypeError
```

## Symbol.prototype.description

```js
// ES2019 提供了一个 Symbol 值的实例属性 description，直接返回 Symbol 值的描述
const sym = Symbol("foo");
sym.description; // "foo"
```

## 作为属性名的 Symbol

```js
// 由于每一个 Symbol 值都是不相等的，这意味着只要 Symbol 值作为标识符，用于对象的属性名，就能保证不会出现同名的属性。
// 这对于一个对象由多个模块构成的情况非常有用，能防止某一个键被不小心改写或覆盖。
// 需要注意，Symbol 值作为属性名时，该属性还是公开属性，不是私有属性。

let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = "Hello!";

// 第二种写法
let a = {
  [mySymbol]: "Hello!",
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: "Hello!" });

// 以上写法都得到同样结果
a[mySymbol]; // "Hello!"
```

```js
// Symbol 值作为对象属性名时，不能用点运算符。
const mySymbol = Symbol();
const a = {};

// 因为点运算符后面总是字符串，所以不会读取mySymbol作为标识名所指代的那个值
a.mySymbol = "Hello!";
a[mySymbol]; // undefined
a["mySymbol"]; // "Hello!"
```

```js
// 在对象的内部，使用 Symbol 值定义属性时，Symbol 值必须放在方括号之中
let s = Symbol();

// 如果s不放在方括号中，该属性的键名就是字符串s，而不是s所代表的那个 Symbol 值
let obj = {
  [s]: function (arg) {
    // ...
  },
};

obj[s](123);
```

```js
// Symbol 类型可以用于定义一组常量，保证这组常量的值都是不相等的
const log = {};
log.levels = {
  DEBUG: Symbol("debug"),
  INFO: Symbol("info"),
  WARN: Symbol("warn"),
};
console.log(log.levels.DEBUG, "debug message");
console.log(log.levels.INFO, "info message");
```

## 属性名的遍历

```js
// Symbol 值作为属性名，遍历对象的时候，该属性不会出现在for...in、for...of循环中
// 也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。

// 通过 Object.getOwnPropertySymbols() 方法，可以获取指定对象的所有 Symbol 属性名
const obj = {};
let a = Symbol("a");
let b = Symbol("b");

obj[a] = "Hello";
obj[b] = "World";

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols;
// [Symbol(a), Symbol(b)]
```

```js
// 通过 Reflect.ownKeys() 方法可以返回所有类型的键名，包括常规键名和 Symbol 键名
let obj = {
  [Symbol("my_key")]: 1,
  enum: 2,
  nonEnum: 3,
};

Reflect.ownKeys(obj);
//  ["enum", "nonEnum", Symbol(my_key)]
```

## Symbol.for()，Symbol.keyFor()

```js
// 有时，我们希望重新使用同一个 Symbol 值，Symbol.for()方法可以做到这一点
let s1 = Symbol.for("foo");
let s2 = Symbol.for("foo");

s1 === s2; // true
```

```js
// Symbol.for() 不会每次调用就返回一个新的 Symbol 类型的值，而是会先全局环境检查给定的key是否已经存在，如果不存在才会去全局环境新建一个值
// Symbol() 每次调用都会生成新的 Symbol
Symbol.for("bar") === Symbol.for("bar");
// true

Symbol("bar") === Symbol("bar");
// false
```

```js
// Symbol.keyFor() 方法返回一个已登记的 Symbol 类型值的key
let s1 = Symbol.for("foo");
Symbol.keyFor(s1); // "foo"

let s2 = Symbol("foo");
Symbol.keyFor(s2); // undefined
```

## 内置的 Symbol 值

ES6 提供了 11 个内置的 Symbol 值，指向语言内部使用的方法

### Symbol.hasInstance

```js
// 对象的Symbol.hasInstance属性，指向一个内部方法
// 当其他对象使用instanceof运算符，判断是否为该对象的实例时，会调用这个方法
class Even {
  static [Symbol.hasInstance](obj) {
    return Number(obj) % 2 === 0;
  }
}

// 等同于
const Even = {
  [Symbol.hasInstance](obj) {
    return Number(obj) % 2 === 0;
  },
};

1 instanceof Even; // false
2 instanceof Even; // true
```

### Symbol.isConcatSpreadable

```js
// 对象的Symbol.isConcatSpreadable属性等于一个布尔值，表示该对象用于Array.prototype.concat()时，是否可以展开
let arr1 = ["c", "d"];
["a", "b"].concat(arr1, "e"); // ['a', 'b', 'c', 'd', 'e']
// 数组的默认行为是可以展开，Symbol.isConcatSpreadable默认等于undefined。该属性等于true时，也有展开的效果
arr1[Symbol.isConcatSpreadable]; // undefined

// 设置成 false，不可以展开
let arr2 = ["c", "d"];
arr2[Symbol.isConcatSpreadable] = false;
["a", "b"].concat(arr2, "e"); // ['a', 'b', ['c','d'], 'e']
```

```js
// 类似数组的对象默认不展开。它的Symbol.isConcatSpreadable属性设为true，才可以展开。
let obj = { length: 2, 0: "c", 1: "d" };
["a", "b"].concat(obj, "e"); // ['a', 'b', obj, 'e']

// 设置成 true，可以展开
obj[Symbol.isConcatSpreadable] = true;
["a", "b"].concat(obj, "e"); // ['a', 'b', 'c', 'd', 'e']
```

### Symbol.species

```js
// 对象的Symbol.species属性，指向一个构造函数。创建衍生对象时，会使用该属性。
class MyArray extends Array {
  // 默认的Symbol.species属性，等同于下面的写法
  static get [Symbol.species]() {
    return this;
  }
}

const a = new MyArray(1, 2, 3);
const b = a.map((x) => x);
const c = a.filter((x) => x > 1);

// 生成的衍生对象是MyArray的实例
b instanceof MyArray; // true
c instanceof MyArray; // true
```

```js
class MyArray extends Array {
  // 修改Symbol.species属性，指向Array
  static get [Symbol.species]() {
    return Array;
  }
}

const a = new MyArray();
const b = a.map((x) => x);

// 生成的衍生对象，就不是MyArray的实例，而直接就是Array的实例
b instanceof MyArray; // false
b instanceof Array; // true
```

### Symbol.match

```js
// 对象的Symbol.match属性，指向一个函数。
// 当执行str.match(myObject)时，如果该属性存在，会调用它，返回该方法的返回值。
String.prototype.match(regexp);
// 等同于
regexp[Symbol.match](this);
```

```js
class MyMatcher {
  [Symbol.match](string) {
    return "hello world".indexOf(string);
  }
}

"e".match(new MyMatcher()); // 1
```

### Symbol.replace

```js
// 对象的Symbol.replace属性，指向一个方法
// 当该对象被String.prototype.replace方法调用时，会返回该方法的返回值。
String.prototype.replace(searchValue, replaceValue);
// 等同于
searchValue[Symbol.replace](this, replaceValue);
```

```js
const x = {};
x[Symbol.replace] = (...s) => console.log(s);

"Hello".replace(x, "World"); // ["Hello", "World"]
```

### Symbol.search

```js
// 对象的Symbol.search属性，指向一个方法
// 当该对象被String.prototype.search方法调用时，会返回该方法的返回值。
String.prototype.search(regexp);
// 等同于
regexp[Symbol.search](this);
```

```js
class MySearch {
  constructor(value) {
    this.value = value;
  }
  [Symbol.search](string) {
    return string.indexOf(this.value);
  }
}
"foobar".search(new MySearch("foo")); // 0
```

### Symbol.split

```js
// 对象的Symbol.split属性，指向一个方法
// 当该对象被String.prototype.split方法调用时，会返回该方法的返回值。
String.prototype.split(separator, limit);
// 等同于
separator[Symbol.split](this, limit);
```

```js
class MySplitter {
  constructor(value) {
    this.value = value;
  }
  [Symbol.split](string) {
    let index = string.indexOf(this.value);
    if (index === -1) {
      return string;
    }
    return [string.substr(0, index), string.substr(index + this.value.length)];
  }
}

"foobar".split(new MySplitter("foo"));
// ['', 'bar']

"foobar".split(new MySplitter("bar"));
// ['foo', '']

"foobar".split(new MySplitter("baz"));
// 'foobar'
```

### Symbol.iterator

```js
// 对象的Symbol.iterator属性，指向该对象的默认遍历器方法。
const myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable]; // [1, 2, 3]
```

### Symbol.toPrimitive

```js
// 对象的Symbol.toPrimitive属性，指向一个方法。
// 该对象被转为原始类型的值时，会调用这个方法，返回该对象对应的原始类型值。
let obj = {
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case "number":
        return 123;
      case "string":
        return "str";
      case "default":
        return "default";
      default:
        throw new Error();
    }
  },
};

2 * obj; // 246
3 + obj; // '3default'
obj == "default"; // true
String(obj); // 'str'
```

### Symbol.toStringTag

```js
// 对象的Symbol.toStringTag属性，用来设定一个字符串（设为其他类型的值无效，但不报错）。
// 在目标对象上面调用Object.prototype.toString()方法时，如果Symbol.toStringTag属性存在，该属性设定的字符串会出现在toString()方法返回的字符串之中，表示对象的类型。

// 例一
({ [Symbol.toStringTag]: "Foo" }.toString());
// "[object Foo]"

// 例二
class Collection {
  get [Symbol.toStringTag]() {
    return "xxx";
  }
}
let x = new Collection();
Object.prototype.toString.call(x); // "[object xxx]"
```

### Symbol.unscopables

```js
// 对象的Symbol.unscopables属性，指向一个对象。
// 该对象指定了使用with关键字时，哪些属性会被with环境排除。

Array.prototype[Symbol.unscopables];
// {
//   copyWithin: true,
//   entries: true,
//   fill: true,
//   find: true,
//   findIndex: true,
//   includes: true,
//   keys: true
// }

Object.keys(Array.prototype[Symbol.unscopables]);
// ['copyWithin', 'entries', 'fill', 'find', 'findIndex', 'includes', 'keys']
```
