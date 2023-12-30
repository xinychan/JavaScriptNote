[TOC]

# Module 的加载实现

## ES6 模块与 CommonJS 模块的差异

Node.js 加载 ES6 模块与 CommonJS 模块三个重大差异：

- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。
- CommonJS 模块的 require()是同步加载模块，ES6 模块的 import 命令是异步加载，有一个独立的模块依赖的解析阶段。

```js
// CommonJS 模块输出的是值的拷贝，也就是说，一旦输出一个值，模块内部的变化就影响不到这个值

// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};

// main.js
var mod = require("./lib");

// lib.js模块加载以后，它的内部变化就影响不到输出的mod.counter
console.log(mod.counter); // 3
mod.incCounter();
console.log(mod.counter); // 3
```

```js
// 因为mod.counter是一个原始类型的值，会被缓存。除非写成一个函数，才能得到内部变动后的值

// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter;
  },
  incCounter: incCounter,
};

// main.js
var mod = require("./lib");

console.log(mod.counter); // 3
mod.incCounter();
console.log(mod.counter); // 4
```

```js
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from "./lib";
// ES6 模块是动态引用，并且不会缓存值，模块里面的变量绑定其所在的模块
// 如果模块里面的原始值变了，外部import加载的值也会跟着变
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

## Node.js 的模块加载方法

JavaScript 现在有两种模块。一种是 ES6 模块，简称 ESM；另一种是 CommonJS 模块，简称 CJS。

CommonJS 模块是 Node.js 专用的，与 ES6 模块不兼容。

语法上面，两者最明显的差异是，CommonJS 模块使用 require()和 module.exports，ES6 模块使用 import 和 export。

Node.js 要求 ES6 模块采用.mjs 后缀文件名。也就是说，只要脚本文件里面使用 import 或者 export 命令，那么就必须采用.mjs 后缀名。

Node.js 遇到.mjs 文件，就认为它是 ES6 模块，默认启用严格模式，不必在每个模块文件顶部指定"use strict"。

如果不希望将后缀名改成.mjs，可以在项目的 package.json 文件中，指定 type 字段为 module。

```js
{
   "type": "module"
}
```

如果这时还要使用 CommonJS 模块，那么需要将 CommonJS 脚本的后缀名都改成.cjs。

如果没有 type 字段，或者 type 字段为 commonjs，则.js 脚本会被解释成 CommonJS 模块。

总结为一句话：.mjs 文件总是以 ES6 模块加载，.cjs 文件总是以 CommonJS 模块加载，.js 文件的加载取决于 package.json 里面 type 字段的设置。

### CommonJS 模块的加载原理

CommonJS 的一个模块，就是一个脚本文件。require 命令第一次加载该脚本，就会执行整个脚本，然后在内存生成一个对象。

以后需要用到这个模块的时候，就会到 exports 属性上面取值。即使再次执行 require 命令，也不会再次执行该模块，而是到缓存之中取值。

也就是说，CommonJS 模块无论加载多少次，都只会在第一次加载时运行一次，以后再加载，就返回第一次运行的结果，除非手动清除系统缓存。

```js
// 模块在内存生成的对象
{
  // id属性是模块名
  id: 'id',
  // exports属性是模块输出的各个接口
  exports: {},
  // 表示该模块的脚本是否执行完毕
  loaded: true,
  // 其他属性
}
```

### ES6 模块的加载原理

ES6 模块是动态引用，如果使用 import 从一个模块加载变量（即 import foo from 'foo'），那些变量不会被缓存，而是成为一个指向被加载模块的引用，需要开发者自己保证，真正取值的时候能够取到值。
