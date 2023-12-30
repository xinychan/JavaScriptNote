# 对象的属性

## 数据属性

> 数据属性特性有：value = 属性值，writable = 是否可写，enumerable = 是否可枚举，configurable = 是否可配置

    // 通过 Object.defineProperty 定义属性特性
    var obj = {};
    Object.defineProperty(obj, 'x', {
    	value: 1, // 属性的值，默认为 undefined
    	writable: true, // 属性是否可写，true 表示属性的 value 可以被再次赋值
    	enumerable: true, // 属性是否可枚举，true 表示属性是可枚举的
    	configurable: true // 属性是否可配置，true 表示属性的 writable/enumerable 特性可以被修改，同时属性可以被删除
    });
    document.write(obj.x);
    document.write("<br />");
    obj.x = 2;
    document.write(obj.x);
    document.write("<br />");
    delete obj.x;
    document.write(obj.x);
    document.write("<br />");

    // 通过字面量定义的属性，writable/enumerable/configurable 属性特性都默认为 true
    var obj2 = {};
    obj2.x = 1;
    document.write(obj2.x);
    document.write("<br />");
    obj2.x = 2;
    document.write(obj2.x);
    document.write("<br />");
    delete obj2.x;
    document.write(obj2.x);
    document.write("<br />");

    // 通过 Object.defineProperty 定义属性，writable/enumerable/configurable 属性特性都默认为 false
    var obj3 = {};
    Object.defineProperty(obj3, 'x', {
    	value: 1
    });
    document.write(obj3.x); // 值为 1
    document.write("<br />");
    obj3.x = 2;
    document.write(obj3.x); // 值为 1，writable 为 false，值不可修改
    document.write("<br />");
    delete obj3.x;
    document.write(obj3.x); // 值为 1，configurable 为 false，值不可删除
    document.write("<br />");

    var obj4 = {};
    // 设置 writable 属性
    Object.defineProperty(obj4, 'x', {
    	value: 1,
    	writable: true
    });
    document.write(obj4.x); // 值为 1
    document.write("<br />");
    obj4.x = 2;
    document.write(obj4.x); // 值为 2，writable 为 true，值可修改
    document.write("<br />");
    // 设置 enumerable 属性
    Object.defineProperty(obj4, 'y', {
    	value: 1,
    	enumerable: true
    });
    document.write(obj4.propertyIsEnumerable('x')); // false
    document.write("<br />");
    document.write(obj4.propertyIsEnumerable('y')); // true
    document.write("<br />");
    // 设置 configurable 属性
    Object.defineProperty(obj4, 'z', {
    	value: 1,
    	configurable: true
    });
    document.write(obj4.z); // 值为 1
    document.write("<br />");
    delete obj4.z;
    document.write(obj4.z); // 值为 undefined，configurable 为 true，值可以被删除
    document.write("<br />");

> 属性包括自身属性和继承自原型的属性，如果只对自身属性做处理，需要冻结 Object.prototype；或者将 `__proto__` 属性指向空。

	var obj5 = {
		__proto__: null // 没有继承来的属性
	}
	obj5.x = 5;
	document.write(obj5.x);
	document.write("<br />");

> 如果 configurable 为 false，可以把 writable 的 true 改成 false，但不可以把 writable 的 false 改成 true

	var obj6 = {};
	Object.defineProperty(obj6, 'x', {
		value: 1,
		writable: true,
		enumerable: false,
		configurable: false
	});
	document.write(obj6.x); // 值为 1
	document.write("<br />");
	obj6.x = 3;
	document.write(obj6.x); // 值为 3，值可以修改
	document.write("<br />");
	Object.defineProperty(obj6, 'x', {
		writable: false
	});
	obj6.x = 5;
	document.write(obj6.x); // 值为 3，值不可以修改
	document.write("<br />");

## 存取器属性

> 存取器属性特性有：set = 设置属性值，get = 获取属性值

	// 通过对象字面量的方式使用 get/set
	var person = {
		name: 'person',
		get age() {
			return 18;
		},
		set age(val) {
			document.write('age is : ' + val);
			document.write("<br />");
		}
	}
	document.write(person.name);
	document.write("<br />");
	document.write(person.age);
	document.write("<br />");
	person.age = 15;

    // 通过 Object.defineProperty 方式使用 get/set
	var person2 = {};
	Object.defineProperty(person2, 'age', {
		get: function() {
			return 18;
		},
		set: function(val) {
			document.write('age is : ' + val);
			document.write("<br />");
		}
	});
	document.write(person2.age);
	document.write("<br />");
	person2.age = 16;

## 一次定义多个属性以及获取属性的描述

    // 使用 Object.defineProperties 定义多个属性
	var person3 = {};
	Object.defineProperties(person3, {
		name: {
			value: 'person3',
			writable: true,
			enumerable: true,
			configurable: true
		},
		age: {
			value: 18,
			writable: true
		}
	});
	console.log(person3.name);
	console.log(person3.age);
    // 使用 Object.getOwnPropertyDescriptor 获取对某个属性的描述
	console.log(Object.getOwnPropertyDescriptor(person3, 'name'));
	console.log(Object.getOwnPropertyDescriptor(person3, 'age'));
