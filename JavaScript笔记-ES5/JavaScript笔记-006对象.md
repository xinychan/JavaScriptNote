# 对象

## 自定义对象

    // 对象字面量方式
    var obj1 = {}; // 无属性对象；空对象
    // 有属性对象
    var obj2 = {
    	"x": 1,
    	"y": 2,
    	"z": 3
    };
    document.write(obj2.x);
    document.write("<br />");
    document.write(obj2.y);
    document.write("<br />");

    // Object 创建对象
    var obj3 = new Object();

    // 构造函数创建对象
    function Test(num1, num2) {
    	this.n1 = num1;
    	this.n2 = num2;
    }
    var obj4 = new Test(5, 6);
    document.write(obj4.n1);
    document.write("<br />");
    document.write(obj4.n2);
    document.write("<br />");

    // Object.create() 创建对象
    var obj5 = Object.create({
    	"x": 7
    });
    document.write(obj5.x);
    document.write("<br />");

## 对象的属性操作

    // 获取属性
    var obj7 = {
    	name: "javascript",
    	feature: "prototypical",
    	user: "web"
    };
    document.write(obj7.name);
    document.write("<br />");
    document.write(obj7["name"]);
    document.write("<br />");
    // 属性名不确定，使用[]来获取属性
    var key = "name";
    document.write(obj7[key]);
    document.write("<br />");
    // 没有 name 这个变量，name 变量未被定义，返回 undefined
    document.write(obj7[name]);
    document.write("<br />");
    // 没有属性名为 key 的属性，返回 undefined
    document.write(obj7.key);
    document.write("<br />");

    // 添加属性
    var obj8 = {};
    obj8.name = "javascript";
    obj8.feature = "prototypical";
    obj8["user"] = "web";
    document.write(obj8.feature);
    document.write("<br />");
    document.write(obj8.user);
    document.write("<br />");
    // 修改属性
    obj8.feature = "dynamic";
    document.write(obj8.feature);
    document.write("<br />");
    // 删除属性
    delete obj8.user;
    document.write(obj8.user);
    document.write("<br />");

    // for/in 遍历属性
    var obj9 = {
    	name: "javascript",
    	feature: "prototypical",
    	user: "web"
    };
    for (var property in obj9) {
    	document.write(property);
    	document.write("<br />");
    }

    // 对象中有方法，对方法的调用
    var obj9 = {
    	name: "javascript",
    	feature: "prototypical",
    	user: "web",
    	action: function() {
    		return "javascript action";
    	},
    	getName: function() {
    		return "name:" + this.name;
    	}
    };
    document.write(obj9.action());
    document.write("<br />");
    document.write(obj9.getName());
    document.write("<br />");
    // 不带括号，返回定义函数的字符串
    document.write(obj9.getName);
    document.write("<br />");

## 对象的结构

Object 是 JavaScript 中所有对象的父对象，所有对象都继承自 Object；

JavaScript 中的对象继承是基于原型，子对象会继承父对象原型中的所有属性；

所有 JavaScript 中的对象都是位于原型链顶端的 Object 的实例；

所有 JavaScript 对象都会从一个 prototype（原型对象）中继承属性和方法；

JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

**原型属性与对象本身的属性区别**

	// 原型上定义属性
	function foo() {};
	foo.prototype.z = 3;
	// 对象上定义属性
	var obj = new foo();
	obj.x = 1;
	obj.y = 2;
	document.write(obj.x);
	document.write("<br />");
	document.write(obj.y);
	document.write("<br />");
	document.write(obj.z);
	document.write("<br />");
	// 对象上属性覆盖原型属性
	obj.z = 6;
	document.write(obj.z);
	document.write("<br />");
	// 删除对象上属性；如果原型上有这个属性，则获取原型的属性
	delete obj.z;
	document.write(obj.z);
	document.write("<br />");
	// 删除原型上的属性；原型上也没有属性，则为 undefined
	delete foo.prototype.z;
	document.write(obj.z);
	document.write("<br />");

**原型属性与对象本身的属性的判断**

	// 原型上定义属性
	function foo() {};
	foo.prototype.z = 3;
	// 对象上定义属性
	var obj = new foo();
	obj.x = 1;
	obj.y = 2;
	// 通过 in 检测对象上的属性；能检测对象的属性，以及原型上的继承属性
	document.write("x" in obj);
	document.write("<br />");
	document.write("y" in obj);
	document.write("<br />");
	document.write("z" in obj);
	document.write("<br />");
	// 通过 hasOwnProperty 检测对象上的属性；只能检测属于对象自己的属性，不计算原型上的属性
	document.write(obj.hasOwnProperty("x"));
	document.write("<br />");
	document.write(obj.hasOwnProperty("y"));
	document.write("<br />");
	document.write(obj.hasOwnProperty("z"));
	document.write("<br />");
	// 属性是对象自己的并且是可枚举的返回true
	document.write(obj.propertyIsEnumerable('x'));
	document.write("<br />");
	document.write(obj.propertyIsEnumerable('y'));
	document.write("<br />");
	document.write(obj.propertyIsEnumerable('z'));
	document.write("<br />");
	// 返回的是所有能够通过对象访问的、可枚举的属性，既包括自身属性，也包括原型中的属性
	for (var p in obj) {
		document.write(p);
		document.write("<br />");
	}
	// 返回所有自身属性的名称，不包括原型上的属性
	document.write(Object.getOwnPropertyNames(obj));
	document.write("<br />");
	// 返回所有可枚举的自身属性的名称，不包括原型上的属性
	document.write(Object.keys(obj));
	document.write("<br />");


**直接给对象定义原型**

	// 直接给对象定义原型：第一个参数为原型对象
	var obj2 = Object.create({x:1});
	document.write(obj2.x);
	document.write("<br />");
	// 返回 false；这里 x 是原型的属性
	document.write(obj2.hasOwnProperty("x"));
	document.write("<br />");
