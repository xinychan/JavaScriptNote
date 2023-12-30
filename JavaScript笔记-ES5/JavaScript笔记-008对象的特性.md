# 对象的特性

## 对象的原型

> prototype 对象的原型：指向另外一个对象，本对象的属性继承自它的原型对象

> Object.isPrototypeOf() 方法可以检测一个对象是否是另一个对象的原型，或是否处于另一个对象的原型链中

    // 使用对象字面量创建对象，使用 Object.prototype 作为原型
    var test = {};
    console.log(Object.prototype.isPrototypeOf(test)); // true

    // 使用new关键字创建对象，使用构造函数的 prototype 属性作为原型
    function Person() {};
    var person1 = new Person();
    console.log(Person.prototype.isPrototypeOf(person1)); // true

    // 使用 Object.create() 创建对象，使用第一个参数（可以为null）作为原型
    var obj = {};
    var obj1 = Object.create(obj);
    // obj 是 obj1 的原型，返回 true
    console.log(obj.isPrototypeOf(obj1));
    // Object.prototype 也是 obj1 的原型，返回 true
    console.log(Object.prototype.isPrototypeOf(obj1));

    // Object.create() 第一个参数使用 null
    var obj2 = Object.create(null);
    // 此时 Object.prototype 不再是 obj2 的原型，返回 false
    console.log(Object.prototype.isPrototypeOf(obj2));

## 对象的类

> class 对象的类：标识对象类型的字符串

    // 通过对象的 toString 方法间接获取对象的类型
    var objClass = {};
    console.log(objClass.toString()); // [object Object]
    // 内置对象的 toString 方法被重写，需要使用 call 方法回调查看类型
    var objNum = 1; // 1
    console.log(Object.prototype.toString.call(objNum)); // [object Number]

## 对象的可扩展性

> extensible 对象的可扩展性：表明该对象是否可以添加新属性

> 所有内置对象和自定义对象都是显式可扩展的，宿主对象的可扩展性由 JavaScript 引擎定义

> 可扩展性的目的是将对象锁定，防止外界干扰，通常和对象的属性和可配置性与可写性配合使用

**Object.preventExtensions()**

    // 通过 Object.preventExtensions() 将对象设置成不可扩展，且不可再转成可扩展
    var obj = {};
    obj.x = 1;
    console.log(Object.isExtensible(obj)); // true
    var obj1 = Object.preventExtensions(obj);
    // 通过 Object.isExtensible() 检测对象是否可扩展
    console.log(Object.isExtensible(obj1)); // false
    obj1.y = 1;
    console.log(obj1.y); // 不可扩展，值为 undefined
    obj1.x = 2;
    console.log(obj1.x); // 从 obj 继承来的属性，已经有值，现在值为 2

**Object.seal()**

    // Object.seal() 与 Object.preventExtensions() 类似，除了将对象设置成不可扩展
    // 还将对象的所有自身对象都设置成不可配置
    var obj1 = {};
    obj1.x = 1;
    console.log(obj1.x);
    // 未封闭前，configurable 为 true
    console.log(Object.getOwnPropertyDescriptor(obj1, 'x')); // configurable: true
    obj1 = Object.seal(obj1);
    console.log(obj1.x);
    // 封闭后，configurable 为 false
    console.log(Object.getOwnPropertyDescriptor(obj1, 'x')); // configurable: false
    // Object.isSealed() 检测对象是否是封闭的
    console.log(Object.isSealed(obj1)); // true
    obj1.y = 2;
    console.log(obj1.y); // 设置成封闭的，不再可扩展，y 属性赋值失败，值为 undefined

**Object.freeze()**

    // Object.freeze() 更严格锁定对象-将对象冻结。除了将对象设置成不可扩展
    // 还将对象的所有自身对象都设置成不可配置之外，还将自身所有数据属性设置成只读
    // 如果存取器属性定义了 setter 方法，存取器属性将不受影响，可通过 setter 赋值
    var obj1 = {};
    obj1.x = 1;
    console.log(obj1.x);
    // 未冻结前，configurable 为 true，writable 为 true
    console.log(Object.getOwnPropertyDescriptor(obj1, 'x')); // configurable: true/writable: true
    obj1 = Object.freeze(obj1);
    console.log(obj1.x);
    // 冻结后，configurable 为 false，writable 为 false
    console.log(Object.getOwnPropertyDescriptor(obj1, 'x')); // configurable: false/writable: false
    // Object.isFrozen() 检测对象是否被冻结
    console.log(Object.isFrozen(obj1)); // true
    obj1.y = 2;
    console.log(obj1.y); // 设置成冻结的，不再可扩展，y 属性赋值失败，值为 undefined
    obj1.x = 2;
    console.log(obj1.x); // 设置成冻结的，值为可读，不可再赋值，值为 1

## 对象的特性注意事项

    // 默认对象是可扩展的，同时也是非冻结的
    console.log(Object.isFrozen({})); // false
    // 一个不可扩展的空对象，同时也是一个冻结对象
    var obj = Object.preventExtensions({});
    console.log(Object.isFrozen(obj)); // true

---

    // 一个非空对象默认是非冻结的，除非所有属性被删除
    var obj = {
    	x: 1
    };
    console.log(Object.isFrozen(obj)); // false
    Object.preventExtensions(obj);
    console.log(Object.isFrozen(obj)); // false
    delete obj.x;
    console.log(Object.isFrozen(obj)); // true

---

	// 一个不可扩展的对象，但有一个可写或可配置的属性，仍然是非冻结的
	// 必须属性是不可写，且不可配置，才是冻结的
	var obj = {
		x: 1
	};
	console.log(Object.isFrozen(obj)); // false
	Object.preventExtensions(obj);
	Object.defineProperty(obj, 'x', {
		writable: false
	});
	console.log(Object.isFrozen(obj)); // false
	Object.defineProperty(obj, 'x', {
		configurable: false
	});
	console.log(Object.isFrozen(obj)); // true

---

	// 一个不可扩展的对象，拥有一个存取器属性，仍然是非冻结的
	var obj = {
		get test() {
			return 1;
		}
	};
	console.log(Object.isFrozen(obj)); // false
	Object.preventExtensions(obj);
	console.log(Object.isFrozen(obj)); // false
	// 将存取器属性设置成不可配置的，则是冻结的
	Object.defineProperty(obj, 'test', {
		configurable: false
	});
	console.log(Object.isFrozen(obj)); // true

---

	// 一个冻结的对象，同时也是封闭的，同时也是不可扩展的
	var obj = {
		x: 1
	};
	Object.freeze(obj);
	console.log(Object.isFrozen(obj)); // true
	console.log(Object.isSealed(obj)); // true
	console.log(Object.isExtensible(obj)); // false
