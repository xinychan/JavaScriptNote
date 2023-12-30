[TOC]

# namespace

namespace 是一种将相关代码组织在一起的方式，中文译为“命名空间”。
它出现在 ES 模块诞生之前，作为 TypeScript 自己的模块格式而发明的。
但是，自从有了 ES 模块，官方已经不推荐使用 namespace 了。

## 基本用法

```ts
// namespace 用来建立一个容器，内部的所有变量和函数，都必须在这个容器里面使用。
namespace Utils {
  function isString(value: any) {
    return typeof value === "string";
  }

  // 正确
  isString("yes");
}

Utils.isString("no"); // 报错
```

```ts
// 如果要在命名空间以外使用内部成员，就必须为该成员加上export前缀，表示对外输出该成员。
namespace Utility {
  export function log(msg: string) {
    console.log(msg);
  }
  export function error(msg: string) {
    console.error(msg);
  }
}

Utility.log("Call me");
Utility.error("maybe!");
```

```js
// 编译出来的 JavaScript 代码
// 命名空间Utility变成了 JavaScript 的一个对象，凡是export的内部成员，都成了该对象的属性。
// 意味着 namespace 会变成一个值，保留在编译后的代码中。这一点要小心，它不是纯的类型代码。
var Utility;

(function (Utility) {
  function log(msg) {
    console.log(msg);
  }
  Utility.log = log;
  function error(msg) {
    console.error(msg);
  }
  Utility.error = error;
})(Utility || (Utility = {}));
```

```ts
// namespace 内部还可以使用import命令输入外部成员，相当于为外部成员起别名。
namespace Utils {
  export function isString(value: any) {
    return typeof value === "string";
  }
}

namespace App {
  // import命令输入外部成员
  import isString = Utils.isString;

  isString("yes");
  // 等同于
  Utils.isString("yes");
}
```

```ts
// import命令也可以在 namespace 外部，指定别名。
namespace Shapes {
  export namespace Polygons {
    export class Triangle {}
    export class Square {}
  }
}

// 指定 Shapes.Polygons 的别名
import polygons = Shapes.Polygons;

// 等同于 new Shapes.Polygons.Square()
let sq = new polygons.Square();
```

```ts
// namespace 可以嵌套。
namespace Utils {
  export namespace Messaging {
    export function log(msg: string) {
      console.log(msg);
    }
  }
}

// 使用嵌套的命名空间，必须从最外层开始引用，比如Utils.Messaging.log()
Utils.Messaging.log("hello"); // "hello"
```

```ts
// 如果 namespace 代码放在一个单独的文件里，那么引入这个文件需要使用三斜杠的语法。
/// <reference path = "SomeFileName.ts" />
```

**注意事项**

namespace 与模块的作用是一致的，都是把相关代码组织在一起，对外输出接口。
区别是一个文件只能有一个模块，但可以有多个 namespace。
由于模块可以取代 namespace，而且是 JavaScript 的标准语法，还不需要编译转换，所以建议总是使用模块，替代 namespace。

## namespace 的输出

```ts
// namespace 本身也可以使用export命令输出，供其他文件使用。
// shapes.ts
export namespace Shapes {
  export class Triangle {
    // todo
  }
  export class Square {
    // todo
  }
}
```

```ts
// 其他脚本文件使用import命令，加载这个命名空间。
// 写法一
import { Shapes } from "./shapes";
let t = new Shapes.Triangle();

// 写法二
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle();
```

```ts
// 不过，更好的方法还是建议使用模块，采用模块的输出和输入。
// namespace 改写成模块的输出和输入
// shapes.ts
export class Triangle {
  // todo
}
export class Square {
  // todo
}

// shapeConsumer.ts
import * as shapes from "./shapes";
let t = new shapes.Triangle();
```

## namespace 的合并

```ts
// 多个同名的 namespace 会自动合并，这一点跟 interface 一样。
// 这样做的目的是，如果同名的命名空间分布在不同的文件中，TypeScript 最终会将它们合并在一起。
// 这样就比较方便扩展别人的代码。

namespace Animals {
  export class Cat {}
}
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Dog {}
}

// 等同于
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Cat {}
  export class Dog {}
}
```

```ts
// 合并命名空间时，命名空间中的非export的成员不会被合并，但是它们只能在各自的命名空间中使用。
namespace N {
  const a = 0;

  export function foo() {
    console.log(a); // 正确
  }
}

namespace N {
  export function bar() {
    foo(); // 正确
    // 变量a只在第一个名称空间可用
    console.log(a); // 报错
  }
}
```

```ts
// 命名空间还可以跟同名函数合并，但是要求同名函数必须在命名空间之前声明。
// 这样做是为了确保先创建出一个函数对象，然后同名的命名空间就相当于给这个函数对象添加额外的属性。
function f() {
  return f.version;
}

namespace f {
  export const version = "1.0";
}

f(); // '1.0'
f.version; // '1.0'
```

```ts
// 命名空间也能与同名 class 合并，同样要求class 必须在命名空间之前声明。
class C {
  foo = 1;
}

// 名称空间C为类C添加了一个静态属性bar
namespace C {
  export const bar = 2;
}

C.bar; // 2
```

```ts
// 命名空间还能与同名 Enum 合并。
enum E {
  A,
  B,
  C,
}

// 命名空间E为枚举E添加了一个foo()方法
namespace E {
  export function foo() {
    console.log(E.C);
  }
}

E.foo(); // 2
```

```ts
// 注意，Enum 成员与命名空间导出成员不允许同名。
enum E {
  A, // 报错
  B,
}

namespace E {
  export function A() {} // 报错
}
```

## 命名空间与模块

**从文件角度**

TypeScript 里的 namespace 是跨文件的。
JavaScript 里的 module 是以文件为单位的，一个文件一个 module。

**从解决问题角度**

TypeScript 里的 namespace 主要是解决命名冲突的问题。
会在全局生成一个对象，定义在 namespace 内部的类都要通过这个对象的属性访问。
JavaScript 里的 module 主要是解决加载依赖关系的。
跟文件绑定在一起，一个文件就是一个 module。在一个文件中访问另一个文件必须要加载另一个文件。

**注意事项**

- 命名空间是位于全局命名空间下的一个普通的带有名字的 JavaScript 对象，使用起来十分容易。但就像其它的全局命名空间污染一样，它很难去识别组件之间的依赖关系，尤其是在大型的应用中。
- 像命名空间一样，模块可以包含代码和声明。不同的是模块可以声明它的依赖。
- 在正常的 TypeScript 项目开发过程中并不建议用命名空间，但通常在通过 d.ts 文件标记 js 库类型的时候使用命名空间，主要作用是给编译器编写代码的时候参考使用。
