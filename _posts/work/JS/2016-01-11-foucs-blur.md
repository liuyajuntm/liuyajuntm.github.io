---
layout: post
title: "事件的捕获和冒泡"
date: 2016-01-11 11:18:33
category: "javascript"
---

[生动详细解释javascript的冒泡和捕获，包懂包会](http://www.cnblogs.com/hh54188/archive/2012/02/08/2343357.html)

[说说focus /focusin /focusout /blur 事件](https://segmentfault.com/a/1190000003942014)

1. 浏览器的事件传递有两种：
	- 捕获(从上向下)
	- 冒泡(从下向上)

2. 有的浏览器支持捕获，有的支持冒泡，有的都支持，有的都不支持(现在基本不存在都不支持的)

3. focus和blur事件不会冒泡
	- 对应的focusin和focusout会冒泡
	- 所以如果需要冒泡的话，需要将focus转换为focusin事件，将blur转换为focusout

