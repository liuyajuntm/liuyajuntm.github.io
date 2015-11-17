---
layout: post
title: "COMMONJS&ES6模块加载机制"
date: 2015-11-09 15:48:33
category: "javascript"
---

## COMMONJS模块加载机制如何解决循环依赖

源文链接：

[JavaScript 模块的循环加载](http://www.ruanyifeng.com/blog/2015/11/circular-dependency.html)

1. COMMONJS是NODEJS使用的模块加载机制
2. COMMONJS是同步加载的
3. 每一个模块都有一个module对象，该对象在第一require模块的时候被创建
4. module.exports本质上就是exports
5. exports上的对象是在执行到该行的时候就被绑定到module.exports上了，而不是在模块退出的时候再统一绑定
6. COMMONJS除了是同步加载以外，和我们的模块系统还有两个本质的区别，这导致了它比较不容易因为循环依赖而挂掉
	- 进入模块文件第一步就是创建module对象，并且将该对象绑定到全局对象上，这样下一步如果有对象要调用该模块，该模块一定是存在的(此时可能是个空对象，但是这个对象是存在的)
	- exports.xx马上就把xx绑定到了已经在全局对象上的module上，如果下一步其他模块要使用该函数，是能够取到的(也就是说就算模块只加载了一半，被加载的一半已经可以被使用了，而不是要等整个模块全部加载成功再一起绑定到全局对象上)
7. module对象上除了有exports属性还有很多其他属性
8. require("xx.js")返回值就是module.exports对象


以下代码的测试环境是NODEJS

module.exports和exports本质上是同一个

	console.log(module.exports === exports);

输出为：
	
	true

一进入模块立即就创建了module对象，该对象上有很多属性

	var ele;
	for(ele in module){
		console.log("module." + ele);
	}

输出为：
	
	module.id
	module.exports
	module.parent
	module.filename
	module.loaded
	module.children
	module.paths
	module.load
	module.require
	module._compile

module对象上的各个属性挺复杂的没仔细研究

	var ele;
	for(ele in module){
		console.log("\nmodule." + ele + ": " + module[ele]);
	}

输出为：
	
	module.id: .

	module.exports: [object Object]

	module.parent: null

	module.filename: C:\Users\tangmin\Desktop\c.js

	module.loaded: false

	module.children:

	module.paths: C:\Users\tangmin\Desktop\node_modules,C:\Users\tangmin\node_module
	s,C:\Users\node_modules,C:\node_modules

	module.load: function (filename) {
	  debug('load ' + JSON.stringify(filename) +
	        ' for module ' + JSON.stringify(this.id));
	  assert(!this.loaded);
	  this.filename = filename;
	  this.paths = Module._nodeModulePaths(path.dirname(filename));
	  var extension = path.extname(filename) || '.js';
	  if (!Module._extensions[extension]) extension = '.js';
	  Module._extensions[extension](this, filename);
	  this.loaded = true;
	}

	module.require: function (path) {
	  assert(path, 'missing path');
	  assert(typeof path === 'string', 'path must be a string');
	  return Module._load(path, this);
	}

	module._compile: function (content, filename) {
	  var self = this;
	  // remove shebang
	  content = content.replace(/^\#\!.*/, '');
	  function require(path) {
	    return self.require(path);
	  }
	  require.resolve = function(request) {
	    return Module._resolveFilename(request, self);
	  };
	  Object.defineProperty(require, 'paths', { get: function() {
	    throw new Error('require.paths is removed. Use ' +
	                    'node_modules folders, or the NODE_PATH ' +
	                    'environment variable instead.');
	  }});
	  require.main = process.mainModule;
	  // Enable support to add extra extension types
	  require.extensions = Module._extensions;
	  require.registerExtension = function() {
	    throw new Error('require.registerExtension() removed. Use ' +
	                    'require.extensions instead.');
	  };
	  require.cache = Module._cache;
	  var dirname = path.dirname(filename);
	  // create wrapper function
	  var wrapper = Module.wrap(content);
	  var compiledWrapper = runInThisContext(wrapper, { filename: filename });
	  if (global.v8debug) {
	    if (!resolvedArgv) {
	      // we enter the repl if we're not given a filename argument.
	      if (process.argv[1]) {
	        resolvedArgv = Module._resolveFilename(process.argv[1], null);
	      } else {
	        resolvedArgv = 'repl';
	      }
	    }
	    // Set breakpoint on module start
	    if (filename === resolvedArgv) {
	      // Installing this dummy debug event listener tells V8 to start
	      // the debugger.  Without it, the setBreakPoint() fails with an
	      // 'illegal access' error.
	      global.v8debug.Debug.setListener(function() {});
	      global.v8debug.Debug.setBreakPoint(compiledWrapper, 0, 0);
	    }
	  }
	  var args = [self.exports, require, self, filename, dirname];
	  return compiledWrapper.apply(self.exports, args);
	}

### 循环依赖详解

a.js

	exports.done = false;
	var b = require('./b.js');
	console.log('在 a.js 之中，b.done = %j', b.done);
	exports.done = true;
	console.log('a.js 执行完毕');

b.js

	exports.done = false;
	var a = require('./a.js');
	console.log('在 b.js 之中，a.done = %j', a.done);
	exports.done = true;
	console.log('b.js 执行完毕');
	
node ./a.js输出为：

	在 b.js 之中，a.done = false
	b.js 执行完毕
	在 a.js 之中，b.done = true
	a.js 执行完毕
	
执行流程如下：

	1. 执行a.js创建a的module，module绑定到全局对象上  
	2. 将done绑定到a的module.exports上  
	3. 加载b.js(注意：此时a.js并没有加载完)  
	4. 执行b.js创建b的module，module绑定到全局对象上  
	5. 将done绑定到b的module.exports上  
	6. require("./a.js");此时a模块已经存在了而且具有done属性  
	7. 输出"在 b.js 之中，a.done = false"  
	8. 修改b的module.exports.done的值为true  
	9. 输出"b.js 执行完毕"  
	10. b.js执行完毕，返回a.js,此时var b的值为ｂ的module.exports对象,a.js继续执行  
	11. 输出"在 a.js 之中，b.done = true"  
	12. 修改a的module.exports.done的值为true  
	13. 输出"a.js 执行完毕"  

如果执行node　./b.js输出为
	
	在 a.js 之中，b.done = false
	a.js 执行完毕
	在 b.js 之中，a.done = true
	b.js 执行完毕

其实COMMONJS并不能根本上解决循环依赖的问题，我们只需要修改一行代码，程序就会出错
a.js 把require放在第一行

	var b = require('./b.js');
	exports.done = false;
	console.log('在 a.js 之中，b.done = %j', b.done);
	exports.done = true;
	console.log('a.js 执行完毕');

b.js 中加一行log,该log只是为了使得结果更直观，不加也行
	
	exports.done = false;
	var a = require('./a.js');
	console.log("在b.js　a的值为 : " + JSON.stringify(a))
	console.log('在 b.js 之中，a.done = %j', a.done);
	exports.done = true;
	console.log('b.js 执行完毕');

执行 node ./a.js,输出为：

	在b.js　a的值为 : {}
	在 b.js 之中，a.done = undefined
	b.js 执行完毕
	在 a.js 之中，b.done = true
	a.js 执行完毕

此时仍然能够取到a模块，但是此时a模块还没有done属性，是一个空对象，但是这种也比永远都加载不起来的死循环强

如果执行node ./b.js,则输出为：

	在 a.js 之中，b.done = false
	a.js 执行完毕
	在b.js　a的值为 : {"done":true}
	在 b.js 之中，a.done = true
	b.js 执行完毕

