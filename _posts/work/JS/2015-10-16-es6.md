---
layout: post
title: ECMAScript 6 笔记
date: 2015-10-16 17:45:33
category: "javascript"
--- 

原文地址：  

[ES6 入门教程](http://es6.ruanyifeng.com/#docs/number)

### let:

__ES 5没有局部作用域的概念，只有函数作用域和全局作用域，会有很多问题，let实现了局部作用域__

例如，经常会遇到的如下问题，期望输出6实际输出了10，使用let就能解决

ES 5:

	var a = [];
	for (var i = 0; i < 10; i++) {
	  a[i] = function () {
	    console.log(i);
	  };
	}
	a[6](); // 10

ES 6: 

	var a = [];
	for (let i = 0; i < 10; i++) {
	  a[i] = function () {
	    console.log(i);
	  };
	}
	a[6](); // 6

__var 有变量提升的副作用，而let没有变量提升__
	
	console.log(key);//undefined
	var key = 1;

这段代码等同于

	var key;
	console.log(key);//undefined
	key = 1;	

而如果是let

	console.log(key);//ReferenceError
	let key = 1;	


__let有暂时性死区的弊端__

	var tmp = 123;
	if (true) {
	  tmp = 'abc'; // ReferenceError
	  let tmp;
	}

如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些命令，就会报错

__let不允许重复声明__

### 块级作用域

__变量的块级作用域可以使用let和var区分，而function在ES 5中也会被提升，但是在ES 6中是默认在块里面的__

同样一段代码，ES 5 输出：inside， ES 6输出：outside

	function f() { console.log('I am outside!'); }
	(function () {
	  if(false) {
	    // 重复声明一次函数f
	    function f() { console.log('I am inside!'); }
	  }

	  f();
	}());

### const

__const本质上可以理解为const let，他继承了let的所有特性__
__const更像一个常量指针，即一旦被赋值了A，就不能再赋值为B了，但是A里面的内容是可以修改的__

	const a = [];
	a.push("Hello"); // 可执行
	a.length = 0;    // 可执行
	a = ["Dave"];    // 报错

__如果真的想将对象冻结，应该使用Object.freeze方法__

### 全局对象的属性

__ES6规定，var命令和function命令声明的全局变量，属于全局对象的属性；let命令、const命令、class命令声明的全局变量，不属于全局对象的属性。__

	var a = 1;
	// 如果在Node的REPL环境，可以写成global.a
	// 或者采用通用方法，写成this.a
	window.a // 1

	let b = 1;
	window.b // undefined



