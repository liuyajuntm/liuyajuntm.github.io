---
layout: post
title: "解构"
date: 2016-01-25 14:11:33
category: "javascript"
---

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值

	let getValue1 = ()=>{
		let a = 1;
		let b = 3;
		return [a,b];
	}

	let [c,d] = getValue1();//1,3

	[c,d] = [d,c];//3,1

	console.log(c);//3
	console.log(d);//1



	let getValue2 = ()=>{
		let a = 1;
		let b = 3;
		return {a,b};//本质上是return {a : a , b: b};只是简写为了{a,b};
	}

	let {a:m, b:n} = getValue2();//1,3

	console.log(m);//1
	console.log(n);//3

	let x = 1, y = 3;
	let {y:i, x:j} = {x,y};

	console.log(i);//3
	console.log(j);//1

	[i,j] = [x,y];

	console.log(i);//1
	console.log(j);//3



