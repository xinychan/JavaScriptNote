[TOC]

# declare

## 简介

Declare 关键字的重要特点是，它只是通知编译器某个类型是存在的，不用给出具体实现。
比如，只描述函数的类型，不给出函数的实现，如果不使用 Declare，这是做不到的。
Declare 只能用来描述已经存在的变量和数据结构，不能用来声明新的变量和数据结构。
另外，所有 Declare 语句都不会出现在编译后的文件里面。
Declare 关键字通常与声明文件（`.d.ts`）一起使用，声明文件用于描述已存在的 JavaScript 代码的类型信息。

Declare 关键字可以描述以下类型。

- 变量（const、let、var 命令声明）
- type 或者 interface 命令声明的类型
- class
- enum
- 函数（function）
- 模块（module）
- 命名空间（namespace）

## declare variable

declare 关键字可以给出外部变量的类型描述。

```ts
// 使用 Declare 关键字声明了一个名为 GLOBAL_VARIABLE 的全局变量，并指定了它的类型为字符串。
// 这样，我们就可以在 TypeScript 中使用这个全局变量，而无需实际定义它。
declare const GLOBAL_VARIABLE: string;
```

## declare function

declare 关键字可以给出外部函数的类型描述。

```ts
// 使用 Declare 关键字来声明全局函数
// 在引用第三方库或外部 JavaScript 代码时非常有用
declare function globalFunction(arg1: number, arg2: string): void;
```

## declare class

declare 关键字可以给出外部类的类型描述。

```ts
// 使用 Declare 关键字来声明全局类
// 在引用第三方库或外部 JavaScript 代码时非常有用
declare class GlobalClass {
  constructor(arg1: number);
  method(arg2: string): void;
}
```

## declare module，declare namespace

如果想把变量、函数、类组织在一起，可以将 declare 与 module 或 namespace 一起使用。

```ts
// 使用 Declare 关键字声明模块，就可以在 TypeScript 中使用这个模块
// declare module 和 declare namespace 里面，加不加 export 关键字都可以
declare namespace AnimalLib {
  class Animal {
    constructor(name: string);
    eat(): void;
    sleep(): void;
  }

  type Animals = "Fish" | "Dog";
}

// 或者
declare module AnimalLib {
  class Animal {
    constructor(name: string);
    eat(): void;
    sleep(): void;
  }

  type Animals = "Fish" | "Dog";
}
```

## declare global

如果要为 JavaScript 引擎的原生对象添加属性和方法，可以使用 `declare global {}` 语法。

```ts
// 第一行的空导出语句export {}，作用是强制编译器将这个脚本当作模块处理。
// 这是因为declare global必须用在模块里面。
export {};

declare global {
  interface String {
    // 为 JavaScript 原生的String对象添加了toSmallString()方法。
    // declare global 给出这个新增方法的类型描述。
    toSmallString(): string;
  }
}

String.prototype.toSmallString = (): string => {
  // 具体实现
  return "";
};
```

```ts
export {};

declare global {
  // 为 window 对象（类型接口为Window）添加一个属性myAppConfig
  interface Window {
    myAppConfig: object;
  }
}

const config = window.myAppConfig;
```

## declare enum

declare 关键字可以给出外部枚举的类型描述。

```ts
// declare enum 示例
declare enum E1 {
  A,
  B,
}

declare enum E2 {
  A = 0,
  B = 1,
}

declare const enum E3 {
  A,
  B,
}

declare const enum E4 {
  A = 0,
  B = 1,
}
```

## declare module 用于类型声明文件

我们可以为每个模块脚本，定义一个 `.d.ts` 文件，把该脚本用到的类型定义都放在这个文件里面。
但是，更方便的做法是为整个项目，定义一个大的 `.d.ts` 文件，在这个文件里面使用 declare module 定义每个模块脚本的类型。

```ts
// 如下是 node.d.ts 文件的一部分
// url和path都是单独的模块脚本，但是它们的类型都定义在node.d.ts这个文件里面
declare module "url" {
  export interface Url {
    protocol?: string;
    hostname?: string;
    pathname?: string;
  }

  export function parse(
    urlStr: string,
    parseQueryString?,
    slashesDenoteHost?
  ): Url;
}

declare module "path" {
  export function normalize(p: string): string;
  export function join(...paths: any[]): string;
  export var sep: string;
}
```

```ts
// 使用时，自己的脚本使用三斜杠命令，加载这个类型声明文件。
// 如果没有这一行命令，自己的脚本使用外部模块时，就需要在脚本里面使用 declare 命令单独给出外部模块的类型
/// <reference path="node.d.ts"/>
```
