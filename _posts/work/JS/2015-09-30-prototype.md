---
layout: post
title: prototype/__proto__/constructor
date: 2015-09-30 14:56:00
category: "javascript"
---

[原型和原型链详解](http://segmentfault.com/a/1190000000662547)

##  文字

1.  区分原型对象和实例对象

	    function A(){
	    }
	    var a = new A();

	    A是原型对象，a是实例对象

2.  prototype是函数特有的属性，普通对象并没有该属性  
3.  \_\_proto\_\_理论上是普通对象一个不可访问的私有属性，指向了原型对象的prototype
4.  完整的继承应该是这样的：

	    function Father(){

	    }

	    function Child(){

	    }
	    Child.prototype = new Father();
	    Child.prototype.constructor = Child;

	    var chi = new Child();
	    chi.__proto__ === Child.prototype

5. 另一个例子:

		var A = function(){

		}
		A.price = 1;
		A.prototype.price = 2;
		var a = new A();

		var B = function(){

		}
		B.prototype = A;
		B.prototype.constructor = B;
		B.price = 3;
		var b = new B();

		var C = function(){

		}
		C.prototype = new A();
		C.prototype.constructor = C;
		C.price = 4;
		var c = new C();
		console.log("test 1");
		console.log(A.price);//1
		console.log(a.price);//2
		console.log(B.price);//3
		console.log(b.price);//1
		console.log(C.price);//4
		console.log(c.price);//2

		console.log("test 2");
		A.prototype.price = 5;
		console.log(A.price);//1
		console.log(a.price);//5
		console.log(B.price);//3
		console.log(b.price);//1
		console.log(C.price);//4
		console.log(c.price);//5

		console.log("test 3");
		C.prototype.price = 6;
		console.log(A.price);//1
		console.log(a.price);//5
		console.log(B.price);//3
		console.log(b.price);//1
		console.log(C.price);//4
		console.log(c.price);//6
