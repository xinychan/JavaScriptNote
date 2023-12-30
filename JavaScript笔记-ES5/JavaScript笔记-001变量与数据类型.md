# 变量与数据类型

## 使用 var 关键字声明变量

    var a; // 未赋值，默认为 undefined
    var b = 10;
    var c = undefined;
    var d = null; // 被赋值为 null

## 数据类型

### 原始数据类型

- 数值型：Number
  - 整数
  - 浮点数
  - 特殊值：Infinity（无穷大）
  - 特殊值：NaN（Not a Number）
- 字符串：String
  - 使用 " "/' ' 包裹字符串
- 布尔类型：Boolean
  - true
  - false

### 复合数据类型

- 对象（object）
- 数组（array）
- 函数（function）

### 特殊数据类型

- 无定义数据类型：undefined
- 空值：null

## 数据类型转换

### 隐式转换

- 转换成布尔类型
  - undefined = false
  - null = false
  - 数值（0/0.0/NaN） = false
  - 字符串长度为 0 = false
  - 其他对象 = true
- 转换为数值型类型
  - undefined = NaN
  - null = 0
  - true = 1
  - false = 0
  - 字符内容为数字 = 对应数字
  - 字符内容不为数字 = NaN
  - 其他对象 = NaN
- 转换为字符串类型
  - 数值型/布尔类型/undefined/null = 内容直接显式转成字符串
  - 其他对象调用 toString 方法转换成字符串

### 显式转换

转换成布尔类型

- 通过 Boolean 函数强制转换成布尔值
  - Boolean(undefined) = false
  - Boolean(null) = false
  - Boolean(0) = false
  - Boolean(NaN) = false
  - Boolean("") = false
  - 其他对象 = true

转换为数值型类型

- 通过 Number 函数强制转换成数值型
  - Number(undefined) = NaN
  - Number(null) = 0
  - Number(false) = 0
  - Number(true) = 1
  - Number("123") = 123
  - Number("3kkk") = NaN
  - Number("3 6 9") = NaN
- 通过 parseInt 函数强制转换成整型
  - parseInt(undefined) = NaN
  - parseInt(null) = NaN
  - parseInt(false) = NaN
  - parseInt(true) = NaN
  - parseInt("123") = 123
  - parseInt("3kkk") = 3
  - parseInt("3 6 9") = 3
- 通过 parseFloat 函数强制转换成浮点型
  - parseFloat(undefined) = NaN
  - parseFloat(null) = NaN
  - parseFloat(false) = NaN
  - parseFloat(true) = NaN
  - parseFloat("123") = 123
  - parseFloat("3.14kkk") = 3.14
  - parseFloat("3 6 9") = 3

转换为字符串类型

- 通过 String 函数强制转换成字符串
  - String(undefined) = "undefined"
  - String(null) = "null"
  - String(0) = "0"
  - String(NaN) = "NaN"
  - String(false) = "false"
  - String(true) = "true"
  - 其他对象调用 toString 方法转成字符串
