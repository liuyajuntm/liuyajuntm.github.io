---
layout: post
title: "CSS局部作用域"
date: 2016-01-26 15:11:33
category: "javascript"
---

1. css没有局部作用域,html5提供了scoped属性支持局部作用域,但是支持程度很低,目前只有firefox支持

2. 有很多人实现了css的局部作用域,但是做得都比较不好。本质上不外乎以下几种:
	a. 给css加前缀后缀 .myClass -> .001_myClass(同事需要改动html,class="001_myClass")
		+ 无法解决div{backgroud-color}的全局作用域
	b. 给css加前置条件 .myClass -> .preClass_001 .myClass(同时需要改动html,也加上class="preClass_001")
		+ 相对较优的方案
	c. 使用属性选择器  .myClass -> .myClass[myAttr="xxxx_001"](同时需要改动html,也加上myAttr="xxxx_001")
		+ 属性选择器性能很差
3. 然后这三种方案都不是适合组件系统，原因：
	a. 浏览器会在你加入<style></style>标签的时候重排整个页面
	b. 而组件系统会不停的插入组件删除组件，会不停的删除<style></style>插入<style></style>
	c. 唯一可选的解决方案是：全部预编译为内联样式，内联样式不会引发重排的问题

