---
layout: post
title: ES6 Proxy & Reflect
date: 2015-10-29 17:45:33
category: "javascript"
---

## Proxy & Reflect

1. proxy本质上就是提供了对默认方法的重载
2. reflect有两层用法：
	- 将一些绑定在object上但是本质上和object无关的方法绑定到reflect上，
		+ 现在既有Object.defineProperty(obj, name, desc)也有Reflect.defineProperty(obj, name, desc)，以后会去掉Object.defineProperty(obj, name, desc)
	- proxy可以拦截的函数在reflect中都可以找到拦截之前的定义

## 二进制数组

1. ArrayBuffer对象/TypedArray视图/DataView视图，一直都作为独立的规范存在，现在把它们纳入了ES 6规范而已

## 新的数据结构

1. Set/Map/WeakSet/WeakMap
	- WeakSet/WeakMap只能存储对象，而且有垃圾回收机制
	

