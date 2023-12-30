# 函数

## 函数的定义

    // 使用 function 定义一个函数
    function test(a, b) {
    	var sun = a + b;
    	document.write(sun);
        // 使用 return 给函数添加返回值；如果没有 return，默认返回 undefined
        return sun;
    }
    // 函数名重复则原来的函数会被覆盖；函数不支持重载
    function test() {
    	document.write("function test");
        return undefined;
    }
    // 函数名相同，调用的都是 test() 无参函数
    test();
    test(1, 2);

## 默认参数的实现

    // 通过逻辑或实现
    function test(x, y) {
    	x = x || 0;
    	y = y || 0;
    	return x + y;
    }
    // 通过 if 判断实现
    function test2(x, y) {
    	if (x === undefined) {
    		x = 0;
    	}
    	if (y === undefined) {
    		y = 0;
    	}
    	return x + y;
    }
    // 通过 arguments 实现
    function test3(x, y) {
    	x = arguments[0] ? arguments[0] : 0;
    	y = arguments[1] ? arguments[1] : 0;
    	return x + y;
    }

## 可变参数的实现

    // 通过 arguments 实现可变参数
    function test4() {
    	var argSize = arguments.length;
    	var sum = 0;
    	for (var i = 0; i < argSize; i++) {
    		sum = sum + arguments[i];
    	}
    	return sum;
    }
    document.write(test4(1,2,3,4,5));

## 变量作用域

    // 使用 var 关键字在函数外定义的变量，可以全局使用
    var x = 1;
    function test5() {
    	// 在函数内定义的变量，使用 var 关键字的，只可以函数内使用
    	var y = 2;
    	// 在函数内定义的变量，不使用 var 关键字的，可以全局使用
    	z = 3;
    	// 函数内可调用所有 x/y/z 变量
    	alert(x);
    	alert(y);
    	alert(z);
    }
    test5();
    // 函数外仅可以调用 x/z 变量
    alert(x);
    alert(z);

## 全局函数-数值型函数

    // 转成整型值
    document.write(parseInt("66"));
    document.write("<br />");
    // 转成浮点型值
    document.write(parseFloat("6.6"));
    document.write("<br />");
    // 某个值是否是无穷值
    // 如果 number 是有限数字（或可转换为有限数字），那么返回 true
    // 否则，如果 number 是 NaN（非数字），或者是正、负无穷大的数，则返回 false
    document.write(isFinite(66));
    document.write("<br />");
    // 判断其参数是否是 NaN
    // 如果参数是特殊的非数字值 NaN（或者能被转换为这样的值），返回的值就是 true
    document.write(isNaN(66));
    document.write("<br />");

## 全局函数-常用全局函数

    var url = "http://www.baidu.com?search= 6 6 6";
    // 把字符串编码为 URI
    var res = encodeURI(url);
    document.write(res);
    document.write("<br />");
    // 解码某个编码的 URI
    document.write(decodeURI(res))
    document.write("<br />");
    // 把字符串编码为 URI 组件
    var res2 = encodeURIComponent(url);
    document.write(res2);
    document.write("<br />");
    // 解码一个编码的 URI 组件
    document.write(decodeURIComponent(res2));
    document.write("<br />");
    // 对字符串进行编码
    var res3 = escape(url);
    document.write(res3);
    document.write("<br />");
    // 对由 escape() 编码的字符串进行解码
    document.write(unescape(res3));
    document.write("<br />");
    // 计算 JavaScript 字符串，并把它作为脚本代码来执行
    eval("var val = 666");
    document.write(val);
    document.write("<br />");

    var boo = Boolean(666);
    document.write(boo);
    document.write("<br />");
    // 把对象的值转换为数字
    document.write(Number(boo));
    document.write("<br />");
    // 把对象的值转换为字符串
    document.write(String(boo));
    document.write("<br />");

## 匿名函数的应用

    // 方法没有名称，直接赋值给变量
    var testfunc = function(x,y){
    	return x + y;
    }
    document.write(testfunc(5,5));
    document.write("<br />");

## 回调函数

    // 回调函数
    function func1(x, y) {
    	return x() + y();
    }
    function func2() {
    	return 5;
    }
    function func3() {
    	return 5;
    }
    // func2 和 func3 作为回调函数，当成参数给 func1 调用
    // func2/func3 只需要传递参数名，而不是带括号的方法；带括号的方法在 func1 中被调用
    document.write(func1(func2,func3));
    document.write("<br />");

    // 通过 call/apply 回调参数自身
    function func(x, y) {
    	return x * y;
    }
    document.write(func.call(func, 5, 5));
    document.write("<br />");
    var params = [6, 6];
    document.write(func.apply(func, params));
    document.write("<br />");

## 自调函数

    // 自调函数的方式：使用小括号包含函数内容，再用小括号做参数回调
    (
    	function() {
    		document.write("self call function");
    	}
    )();

    // 第二个小括号中填充参数
    (
    	function(x, y) {
    		var sum = x + y;
    		document.write("value = " + sum);
    	}
    )(6, 6);

## 内部私有函数

    // test2 为 test 函数中的私有函数
    function test(t1) {
    	var test2 = function(t2) {
    		return t2 * 2;
    	}
    	return test2(t1);
    }
    document.write(test(8));

## 返回函数的函数

    // show 函数返回给了 showInfo 变量，需要调用 show 里面的函数，需要再次调用 showInfo()
    function show() {
    	alert("show");
    	return function() {
    		alert("JavaScript");
    	}
    }
    var showInfo = show();
    showInfo();

## 重写自己的函数

	// 内部重写了自己的方法逻辑
	function doSomething() {
		alert("doSomething");
		// doSomething 函数被再次赋值，再次调用时使用新赋值逻辑
		doSomething = function() {
			alert("function");
		}
	}
	doSomething(); // doSomething
	doSomething(); // function

## 函数构造器

    // 下面两种写法是等价的
    // 常规方式声明函数
	var myFunc1 = function(a, b) {
		return a + b;
	}
    // 使用内置函数构造器创建函数
	var myFunc2 = new Function('a', 'b', 'return a + b');
