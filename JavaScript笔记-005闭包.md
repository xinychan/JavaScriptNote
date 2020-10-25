# 闭包

## 变量的作用域

全局变量所有函数可以读取；

函数内部的局部变量，只有函数内部可以读取；

当需要得到函数内的局部变量时，可以通过闭包方式来实现；

## 闭包的定义

    // func2 函数就是闭包
    function func1() {
    	var n = 999;
    	function func2() {
    		alert(n);
    	}
    	return func2;
    }
    var result = func1();
    result();

## 闭包概念

闭包就是能够读取其他函数内部变量的函数。

由于在 Javascript 语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。

所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。

## 闭包的用途

    // 1-可以读取函数内部的变量
    // func2 作为闭包，提供外部访问 func1 内部变量的能力
    function func1() {
    	var n = 999;
    	function func2() {
    		alert(n);
    	}
    	return func2;
    }
    var result = func1();
    result();

    // 2-让这些变量的值始终保持在内存中
    // 这里 func2 和 nAdd 函数都是闭包
    function func1() {
    	var n = 999;
    	nAdd = function() {
    		n = n + 1;
    	}
    	function func2() {
    		alert(n);
    	}
    	return func2;
    }
    var result = func1();
    result(); // 999
    nAdd();
    result(); // 1000

## 闭包注意事项

- 由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在 IE 中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
- 闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当作对象（object）使用，把闭包当作它的公用方法（Public Method），把内部变量当作它的私有属性（private value），这时一定要小心，不要随便改变父函数内部变量的值。
