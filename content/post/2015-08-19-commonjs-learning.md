---
title: CommonJS 学习笔记
date: 2015-08-19 10:11:10
categories: [javascript]
tags: [commonjs]
---

# 概述

CommonJS 是 JavaScript 模块化的规范，Node.js采用了这个规范。

根据 CommonJS 规范，一个 JavaScript 文件就是一个模块，其他模块可以通过 `require` 来获取 `module.exports` 中暴露的内容，而其他部分都是私有的，其他模块不能访问。

# module 对象

每个模块都有一个module变量，该变量指向当前模块。module不是全局变量，而是每个模块都有的本地变量。

+ `module.id` 模块的识别符，通常是带有绝对路径的模块文件名。

+ `module.filename` 模块的文件名。

+ `module.loaded` 返回一个布尔值，表示模块是否已经完成加载。

+ `module.parent` 返回一个对象，表示调用该模块的模块。

+ `module.children` 返回一个数组，表示该模块要用到的其他模块。数组中的内容是其他要用到的模块的 `module` 对象。

+ `module.paths` 返回一个数组，表示模块查找路径，排在越前面的优先级越高。具体请查看 [这里](http://www.infoq.com/cn/articles/nodejs-module-mechanism/)

+ `module.exports` 表示其他模块可以通过 `require` 获取的内容。默认情况下是一个空对象，即 `{}`。

# module.exports 和 exports

默认情况下，`module.exports` 和 `exports` 两者是等价的，即 `module.exports === exports`

`exports` 只是为了方便我们编码而添加的，相当于在每个文件中自动添加了 `var exports = module.exports`。

我们可以根据个人喜好，把需要暴露的东西挂在 `exports` 或 `module.exports` 下。

**但是**，必须注意以下几点：

+ 不能把 `exports` 重新赋值，因为这样会令 `exports` 不再指向 `module.exports`，这样 `exports` 就没用了。

+ 我们可以通过 `module.exports = function() {...}` 这种方式指定 `module.exports` 的指向，这样在其他模块中 `require` 这个模块，获得的就是一个 `function`，而不是一个对象。这样做也会导致 `exports` 失效。

+ 由于上面的原因，我们会在很多地方看到 `module.exports = exports = xxx` 这样的代码。



# 参考资料
http://javascript.ruanyifeng.com/nodejs/commonjs.html
http://www.infoq.com/cn/articles/nodejs-module-mechanism/
http://stackoverflow.com/questions/7137397/module-exports-vs-exports-in-node-js
