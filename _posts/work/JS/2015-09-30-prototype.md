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

