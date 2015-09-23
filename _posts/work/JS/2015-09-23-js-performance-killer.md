---
layout: post
title:  [转]JavaScript 性能优化杀手
date:   2015-09-23 10:27:11
category: "javascript"
---

[原文链接](http://dev.zm1v1.com/2015/08/19/javascript-optimization-killers/)

### 引言

这篇文档包含了如何避免使代码性能远低于预期的建议. 尤其是一些会导致 V8 (牵涉到 Node.js, Opera, Chromium 等) 无法优化相关函数的问题.

### 一些 V8 背景

在 V8 中并没有解释器, 但却有两个不同的编译器: 通用编译器和优化编译器. 这意味着你的 JavaScript 代码总是会被编译为机器码后直接运行. 这样一定很快咯? 并不是. 仅仅是编译为本地代码并不能明显提高性能. 它只是消除了解释器的开销, 但如果未被优化, 代码依旧很慢.

举个例子, 使用通用编译器, a + b 会变成这个样子:

	mov eax, a
	mov ebx, b
	call RuntimeAdd

换言之它仅仅是调用了运行时的函数. 如果 a 和 b 一定是整数, 那可以像这样:

	mov eax, a
	mov ebx, b
	add eax, ebx

相比而言这会远快于调用需要处理复杂 JavaScript 运行时语义的函数.

通常来说, 通用编译器得到的是第一种结果, 而优化编译器则会得到第二种结果. 使用优化编译器编译的代码可以很容易比通用编译器编译的代码快上 100 倍. 但这里有个坑, 并非所有的 JavaScript 代码都能被优化. 在 JavaScript 中有很多种写法, 包括具备语义的, 都不能被优化编译器编译 (回落到通用编译器*).

记下一些会导致整个函数无法使用优化编译器的用法很重要. 一次代码优化的是一整个函数, 优化过程中并不会关心其他代码做了什么 (除非代码在已经被优化的函数中).

这个指南会涵盖多数会导致整个函数掉进 “反优化火狱” 的例子. 由于编译器一直在不断更新, 未来当它能够识别下面的一些情况时, 这里提到的处理方法可能也就不必要了.

### 索引


* 工具和方法
* 不支持的语法
* 使用 arguments
* switch…case
* for…in
* 无限循环  

### 1. 工具和方法

你可以通过添加一些 V8 标记来使用 Node.js 验证不同的用法如何影响优化结果. 通常可以写一个包含了特定用法的函数, 使用所有可能的参数类型去调用它, 再使用 V8 的内部函数去优化和审查.

test.js

	// 包含需要审查的用法的函数 (这里是 with 语句)
	function containsWith() {
	    return 3;
	    with({}) { }
	}

	function printStatus(fn) {
	    switch(%GetOptimizationStatus(fn)) {
	        case 1: console.log("Function is optimized"); break;
	        case 2: console.log("Function is not optimized"); break;
	        case 3: console.log("Function is always optimized"); break;
	        case 4: console.log("Function is never optimized"); break;
	        case 6: console.log("Function is maybe deoptimized"); break;
	    }
	}

	// 告诉编译器类型信息
	containsWith();
	// 为了使状态从 uninitialized 变为 pre-monomorphic, 再变为 monomorphic, 两次调用是必要的
	containsWith();

	%OptimizeFunctionOnNextCall(containsWith);
	// 下一次调用
	containsWith();

	// 检查
	printStatus(containsWith);

执行:

	$ node --trace_opt --trace_deopt --allow-natives-syntax test.js
	Function is not optimized

作为是否被优化的对比, 注释掉 with 语句再来一次:

	$ node --trace_opt --trace_deopt --allow-natives-syntax test.js
	[optimizing 000003FFCBF74231 <JS Function containsWith (SharedFunctionInfo 00000000FE1389E1)> - took 0.345, 0.042, 0.010 ms]
	Function is optimized

使用这个方法来验证处理方法有效且必要是很重要的.

### 2. 不支持的语法

优化编译器不支持一些特定的语句, 使用这些语法会使包含它的函数无法得到优化.

有一点请注意, 即使这些语句无法到达或者不会被执行, 它们也会使相关函数无法被优化.

比如这样做是没用的:

	if (DEVELOPMENT) {
	    debugger;
	}

上面的代码会导致包含它的整个函数不被优化, 即使从来不会执行到 debugger 语句.

目前不会被优化的有:

* generator 函数
* 包含 for…of 语句的函数
* 包含 try…catch 的函数
* 包含 try…finally 的函数
* 包含复合 let 赋值语句的函数 (原文为 compound let assignment)
* 包含复合 const 赋值语句的函数 (原文为 compound const assignment)
* 包含含有 __proto__ 或者 get/set 声明的对象字面量的函数

可能永远不会被优化的有:

* 包含 debugger 语句的函数
* 包含字面调用 eval() 的函数
* 包含 with 语句的函数

最后一点明确一下, 如果有下面任何的情况, 整个函数都无法被优化:

	function containsObjectLiteralWithProto() {
	    return { __proto__: 3 };
	}

	function containsObjectLiteralWithGetter() {
	    return {
	        get prop() {
	            return 3;
	        }
	    };
	}

	function containsObjectLiteralWithSetter() {
	    return {
	        set prop(val) {
	            this.val = val;
	        }
	    };
	}

提一下直接使用 eval 和 with 的情况, 因为它们会造成相关嵌套的函数作用域变为动态的. 这样一来则有可能也影响其他很多函数, 因为这种情况下无法从词法上判断相关变量的有效范围.

__处理方法__

之前提到过的一些语句在生产环境中是无法避免的, 比如 try...finally 和 try...catch. 为了是代价最小, 它们必须被隔离到一个最小化的函数, 以保证主要的代码不受影响.

	var errorObject = { value: null };
	function tryCatch(fn, ctx, args) {
	    try {
	        return fn.apply(ctx, args);
	    } catch(e) {
	        errorObject.value = e;
	        return errorObject;
	    }
	}

	var result = tryCatch(mightThrow, void 0, [1,2,3]);
	// 不带歧义地判断是否调用抛出了异常 (或其他值)
	if(result === errorObject) {
	    var error = errorObject.value;
	} else {
	    // 结果是返回值
	}  


### 3. 使用 arguments

有不少使用 arguments 的方式会导致相关函数无法被优化. 所以在使用 arguments 的时候需要非常留意.

#### 3.1. 给一个已经定义的参数重新赋值, 并且在相关语句主体中引用 (仅限非严格模式). 典型的例子:

	function defaultArgsReassign(a, b) {
	     if (arguments.length < 2) b = 5;
	}

处理方法则是赋值该参数给一个新的变量:

	function reAssignParam(a, b_) {
	    var b = b_;
	    // 与 b_ 不同, b 可以安全地被重新赋值
	    if (arguments.length < 2) b = 5;
	}

如果仅仅是在这种情况下在函数中用到了 arguments, 也可以写为是否为 undefined 的判断:

	function reAssignParam(a, b) {
	    if (b === void 0) b = 5;
	}

然而如果之后这个函数中用到 arguments, 维护代码的同学可能会容易忘掉要把重新赋值的语句留下**.

第二个处理方法: 对整个文件或者函数开启严格模式 ('use strict').

#### 3.2. 泄露 arguments:

	function leaksArguments1() {
	    return arguments;
	}

	function leaksArguments2() {
	    var args = [].slice.call(arguments);
	}

	function leaksArguments3() {
	    var a = arguments;
	    return function() {
	        return a;
	    };
	}

arguments 对象不能被传递或者泄露到任何地方.

处理方法则是使用内联的代码创建数组:

	function doesntLeakArguments() {
	                    // .length 只是一个整数, 它不会泄露
	                    // arguments 对象本身
	    var args = new Array(arguments.length);
	    for(var i = 0; i < args.length; ++i) {
	                // i 始终是 arguments 对象的有效索引
	        args[i] = arguments[i];
	    }
	    return args;
	}

写一堆代码很让人恼火, 所以分析是否值得这么做是值得的. 接下来更多的优化总是会带来更多的代码, 而更多的代码又意味着语义上更显而易见的退化.

然而如果你有一个 build 的过程, 这其实可以被一个不必要求 source map 的宏来实现, 同时保证源代码是有效的 JavaScript 代码.

	function doesntLeakArguments() {
	    INLINE_SLICE(args, arguments);
	    return args;
	}

上面的技巧就用到了 Bluebird 中, 在 build 后会被扩充为下面这样:

	function doesntLeakArguments() {
	    var $_len = arguments.length;var args = new Array($_len); 
	    for(var $_i = 0; $_i < $_len; ++$_i) {args[$_i] = arguments[$_i];}
	    return args;
	}

#### 3.3 对 arguments 赋值

在非严格模式下, 这其实是可能的:

	function assignToArguments() {
	    arguments = 3;
	    return arguments;
	}

处理方法: 没必要写这么蠢的代码. 说来在严格模式下, 它也会直接抛出异常.
怎样安全地使用 arguments?

仅使用:

* arguments.length
* arguments[i] 这里 i 必须一直是 arguments 的整数索引, 并且不能超出边界
* 除了 .length 和 [i], 永远不要直接使用 arguments (严格地说 x.apply(y, arguments) 是可以的,但其他的都不行, 比如 .slice. Function#apply 比较特殊)

另外关于用到 arguments 会造成 arguments 对象的分配这一点的 FUD (恐惧), 在使用限于上面提到的安全的方式时是不必要的.

### 4. switch…case

一个 switch…case 语句目前可以有最多 128 个 case 从句, 如果超过了这个数量, 包含这个 switch 语句的函数就无法被优化.

	function over128Cases(c) {
	    switch(c) {
	        case 1: break;
	        case 2: break;
	        case 3: break;
	        ...
	        case 128: break;
	        case 129: break;
	    }
	}

所以请保证 switch 语句的 case 从句不超过 128 个, 可以使用函数数组或者 if…else 代替.

### 5. for…in

for…in 语句在一些情况下可能导致包含它的函数无法被优化.

以下解释了 “for…in 不快” 或者类似的原因.
键不是局部变量:

	function nonLocalKey1() {
	    var obj = {}
	    for(var key in obj);
	    return function() {
	        return key;
	    };
	}

	var key;
	function nonLocalKey2() {
	    var obj = {}
	    for(key in obj);
	}

因此键既不能是上级作用于的变量, 也不能被子作用域引用. 它必须是一个本地变量.

#### 5.2. 被枚举的对象不是一个 “简单的可枚举对象”

##### 5.2.1. 处于 “哈希表模式” 的对象 (即 “普通化的对象”, “字典模式” – 以哈希表为数据辅助结构的对象)不是简单的可枚举对象.

	function hashTableIteration() {
	    var hashTable = {"-": 3};
	    for(var key in hashTable);
	}

如果你 (在构造函数外) 动态地添加太多属性到一个对象, 删除属性, 使用不是合法标识符 (identifier) 的属性名称, 这个对象就会变为哈希表模式. 换言之, 如果你把一个对象当做哈希表来使用, 它就会转变为一个哈希表. 不要再 for…in 中使用这样的对象. 判断一个对象是否为哈希表模式, 可以在开启 Node.js 的 --allow-natives-syntax 选项时调用 console.log(%HasFastProperties(obj)).

##### 5.2.2. 对象的原型链中有可枚举的属性

	Object.prototype.fn = function() {};

添加上面的代码会使所有的对象 (除了 Object.create(null) 创建的对象) 的原型链中都存在一个可枚举的属性. 由此任何包含 for…in 语句的函数都无法得到优化 (除非仅枚举 Object.create(null) 创建的对象).

你可以通过 Object.defineProperty 来创建不可枚举的属性 (不推荐运行时调用, 但是高效地定义一些静态的东西, 比如原型属性, 还是可以的).

##### 5.2.3. 对象包含可枚举的数组索引

一个属性是否是数组索引是在 ECMAScript 规范 中定义的.

>A property name P (in the form of a String value) is an array index if and only if ToString(ToUint32(P)) is equal to P and ToUint32(P) is not equal to 232−1. A property whose property name is an array index is also called an element

通常来说这些对象是数组, 但普通的对象也可以有数组索引: normalObj[0] = value;

	function iteratesOverArray() {
	    var arr = [1, 2, 3];
	    for (var index in arr) {

	    }
	}

所以使用 for…in 遍历数组不仅比 for 循环慢, 还会导致包含它的整个函数无法被优化.

如果传递一个非简单的可枚举对象到 for…in, 会导致整个函数无法被优化.

处理方法: 总是使用 Object.keys 再使用 for 循环遍历数组. 如果的确需要原型链上的所有属性, 创建一个单独的辅助函数.

	function inheritedKeys(obj) {
	    var ret = [];
	    for(var key in obj) {
	        ret.push(key);
	    }
	    return ret;
	}

### 6. 退出条件较深或者不明确的无限循环

写代码的时候, 有时会知道自己需要一个循环, 但不清楚循环内的代码会写成什么样子. 所以你放了一个 while (true) { 或者 for (;;) {, 之后再在一定条件下中断循环接续之后的代码, 最后忘了这么一件事. 重构的时间到了, 你发现这个函数很慢, 或者发现一个反优化的情况 – 可能它就是罪魁.

将循环的退出条件重构到循环自己的条件部分可能并不容易. 如果代码的退出条件是结尾 if 语句的一部分, 并且代码至少会执行一次, 那可以重构为 do { } while (); 循环. 如果退出条件在循环开头, 把它放进循环本身的条件部分. 如果退出条件在中间, 你可以尝试 “滚动” 代码: 每每从开头移动一部分代码到末尾, 也复制一份到循环开始之前. 一旦退出条件可以放置在循环的条件部分, 或者至少是一个比较浅的逻辑判断, 这个循环应该就不会被反优化了.