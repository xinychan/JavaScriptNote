[TOC]

# Module 的语法

## 概述

```js
// 在 ES6 之前，社区制定了一些模块加载方案，比如 CommonJS
// CommonJS模块
let { stat, exists, readfile } = require("fs");

// 等同于
// 代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。
// 这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。
let _fs = require("fs");
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

```js
// ES6模块
// ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。
import { stat, exists, readFile } from "fs";
// 代码的实质是从fs模块加载 3 个方法，其他方法不加载。
// 这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。
// 当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。
```

## 严格模式

ES6 的模块自动采用严格模式，不管你有没有在模块头部加上"use strict"

严格模式主要有以下限制：

- 变量必须声明后再使用
- 函数的参数不能有同名属性，否则报错
- 不能使用 with 语句
- 不能对只读属性赋值，否则报错
- 不能使用前缀 0 表示八进制数，否则报错
- 不能删除不可删除的属性，否则报错
- 不能删除变量 `delete prop`，会报错，只能删除属性 `delete global[prop]`
- eval 不会在它的外层作用域引入变量
- eval 和 arguments 不能被重新赋值
- arguments 不会自动反映函数参数的变化
- 不能使用 arguments.callee
- 不能使用 arguments.caller
- 禁止 this 指向全局对象
- 不能使用 fn.caller 和 fn.arguments 获取函数调用的堆栈
- 增加了保留字（比如 protected、static 和 interface）

## export 命令

```js
// export 命令用于规定模块的对外接口
// export 命令能够对外输出的就是三种接口：函数（Functions）， 类（Classes），var、let、const 声明的变量（Variables）

// export命令写法1
// profile.js
export var firstName = "Michael";
export var lastName = "Jackson";
export var year = 1958;

// export命令写法2
// profile.js
var firstName = "Michael";
var lastName = "Jackson";
var year = 1958;
// 优先考虑使用这种写法。因为这样就可以在脚本尾部，一眼看清楚输出了哪些变量
export { firstName, lastName, year };
```

```js
// 使用as关键字对导出的变量重命名
function v1() {}
function v2() {}

export { v1 as streamV1, v2 as streamV2, v2 as streamLatestVersion };
```

## import 命令

```js
// 使用export命令定义了模块的对外接口以后，其他 JS 文件就可以通过import命令加载这个模块。

// main.js
import { firstName, lastName, year } from "./profile.js";

function setName(element) {
  element.textContent = firstName + " " + lastName;
}
```

```js
// 使用as关键字对输入的变量重命名
import { lastName as surname } from "./profile.js";
```

```js
// 由于import是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构

// 报错；使用表达式
import { 'f' + 'oo' } from 'my_module';

// 报错；使用变量
let module = 'my_module';
import { foo } from module;

// 报错；使用if结构
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

```js
// import 语句是 Singleton 模式
// 虽然foo和bar在两个语句中加载，但是它们对应的是同一个my_module模块
import { foo } from "my_module";
import { bar } from "my_module";

// 等同于
import { foo, bar } from "my_module";
```

## 模块的整体加载

```js
// 使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面

// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}
export function circumference(radius) {
  return 2 * Math.PI * radius;
}

// 逐一指定要加载的方法
// main.js
import { area, circumference } from "./circle";
console.log("圆面积：" + area(4));
console.log("圆周长：" + circumference(14));

// 整体加载整个模块
// main.js
import * as circle from "./circle";
console.log("圆面积：" + circle.area(4));
console.log("圆周长：" + circle.circumference(14));
```

## export default 命令

```js
// 本质上 export default 就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字
// 一个模块只能有一个默认输出，因此export default命令只能使用一次

// export-default.js
export default function () {
  console.log("foo");
}

// import-default.js
import customName from "./export-default";
customName(); // 'foo'
```

```js
// export-default.js
export default function foo() {
  console.log('foo');
}

// 等价于

function foo() {
  console.log('foo');
}
export default foo;
```

```js
// modules.js
function add(x, y) {
  return x * y;
}
export { add as default };
// 等同于
// export default add;

// app.js
import { default as foo } from "modules";
// 等同于
// import foo from 'modules';
```

## export 与 import 的复合写法

```js
// 如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起。
export { foo, bar } from "my_module";

// 可以简单理解为
// 先输入模块
import { foo, bar } from "my_module";
// 再输出模块
export { foo, bar };
```

## 模块的继承

```js
// 假设有一个 circleplus 模块，继承了 circle 模块

// circleplus.js
export * from "circle";
export var e = 2.71828182846;
export default function (x) {
  return Math.exp(x);
}
```

```js
// main.js
// 将circleplus整个模块加载为math对象
import * as math from "circleplus";
// 将circleplus模块的默认方法加载为exp方法
import exp from "circleplus";
// 相当于执行 circleplus.default(circleplus.e)
console.log(exp(math.e));
```

## 跨模块常量

```js
// const声明的常量只在当前代码块有效。
// 如果想设置跨模块的常量（即跨多个文件），或者说一个值要被多个模块共享，可以采用下面的写法。

// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from "./constants";
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import { A, B } from "./constants";
console.log(A); // 1
console.log(B); // 3
```

```js
// 如果要使用的常量非常多，可以建一个专门的constants目录，将各种常量写在不同的文件里面，保存在该目录下。

// constants/db.js
export const db = {
  url: "http://my.couchdbserver.local:5984",
  admin_username: "admin",
  admin_password: "admin password",
};

// constants/user.js
export const users = ["root", "admin", "staff", "ceo", "chief", "moderator"];
```

```js
// 将这些文件输出的常量，合并在index.js里面

// constants/index.js
export { db } from "./db";
export { users } from "./users";
```

```js
// 使用的时候，直接加载index.js就可以了

// script.js
import { db, users } from "./constants/index";
```

## import() 动态加载

### 简介

```js
// 因为import命令是静态加载模块，require是运行时加载模块，所以import命令无法取代require的动态加载功能
// 在语法上，import命令的条件加载就不可能实现

// require()的动态加载
const path = "./" + fileName;
const myModual = require(path);
```

```js
// ES2020提案 引入import()函数，支持动态加载模块。
// import()返回一个 Promise 对象

async function renderWidget() {
  const container = document.getElementById("widget");
  if (container !== null) {
    // import()函数动态加载
    const widget = await import("./widget.js");
    widget.render(container);
  }
}

renderWidget();
```

### 适用场合

```js
// 按需加载
button.addEventListener("click", (event) => {
  // 点击时才会加载模块
  import("./dialogBox.js")
    .then((dialogBox) => {
      dialogBox.open();
    })
    .catch((error) => {
      /* Error handling */
    });
});
```

```js
// 条件加载
// 不同条件加载不同模块
if (condition) {
  import("moduleA").then();
} else {
  import("moduleB").then();
}
```

```js
// 动态的模块路径
// import()允许模块路径动态生成
import(f()).then();
```

### 注意点

```js
async function main() {
  // import()加载模块成功以后，这个模块会作为一个对象
  const myModule = await import("./myModule.js");
  // 因此，可以使用对象解构赋值的语法，获取输出接口
  const { export1, export2 } = await import("./myModule.js");
  // 同时加载多个模块
  const [module1, module2, module3] = await Promise.all([
    import("./module1.js"),
    import("./module2.js"),
    import("./module3.js"),
  ]);
}
main();
```

## import.meta

ES2020 为 import 命令添加了一个元属性 import.meta，返回当前模块的元信息。
import.meta 只能在模块内部使用，如果在模块外部使用会报错。
这个属性返回一个对象，该对象的各种属性就是当前运行的脚本的元信息。

```js
// import.meta.url 返回当前模块的 URL 路径

// 获取模块里面文件data.txt的路径
new URL("data.txt", import.meta.url);
```

```js
// import.meta.scriptElement 是浏览器特有的元属性，返回加载模块的那个<script>元素，相当于document.currentScript属性

// HTML 代码为
// <script type="module" src="my-module.js" data-foo="abc"></script>

// my-module.js 内部执行下面的代码
import.meta.scriptElement.dataset.foo;
// "abc"
```
