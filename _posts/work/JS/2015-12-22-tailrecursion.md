---
layout: post
title: "尾递归"
date: 2015-12-22 11:07:33
category: "javascript"
---

1. 尾递归是在编译器级别进行优化的，内存不会爆掉
2. 但是并不是所有递归都能改为尾递归，像最简单的汉诺塔就不能改为尾递归
3. 递归函数在一次递归中多次调用自身，肯定不能改为尾递归
4. 递归函数在一次递归中单次调用自身，一般能改为尾递归

[尾递归](http://es6.ruanyifeng.com/#docs/function#%E5%B0%BE%E8%B0%83%E7%94%A8%E4%BC%98%E5%8C%96)