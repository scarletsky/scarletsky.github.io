---
title: The Little JavaScript Closures
date: 2015-12-02 20:55:54
categories: [javascript]
tags: [javascript, closure, 闭包]
---

## 写在前面

本文尝试模仿 [The Little Schema](http://uternet.github.io/TLS/) 的风格，介绍 JavaScript 的闭包。本文同时也是我学习 JavaScript 闭包的一次总结。欢迎一起讨论。

## 简介

什么是闭包？

> 闭包是一个函数

闭包都是函数吗？

> 是

函数都是闭包吗？

> 不

我怎么判断一个函数是不是闭包？

> 你现在还不能回答，因为你还不知道以下概念：
> 全局变量（Global Variable）
> 局部变量（Local Variable）
> 自由变量（Free Variable）
> 词法作用域（Lexical Scope）

## 变量与作用域

`var a = 1;` a 是什么变量？

> 全局变量

`a = 1;` a 是什么变量？

> 全局变量

```js
function foo() {
  a = 1;
  var b = 1;
}
```
这里的 a，b 分别是什么变量？

> a 是全局变量，b 是局部变量

为什么 a 在函数中定义还是全局变量？

> 因为 a 不是用 var 声明的

不用 `var` 声明的变量都是全局变量？

> 是的

用 `var` 声明的变量都是局部变量？

> 不是

为什么？

> 在全局作用域中声明的变量都是全局变量，即使这个变量是用 var 声明的

全局作用域是什么？

> 函数作用域以外的地方都是就是全局作用域

函数作用域又是什么？

> 函数内部

可以举个例子吗？

> ```js
var foo = 1;
function bar() {
  var baz = 2;
}
```
> foo 变量和 bar 函数都处于全局作用域中，baz 变量处于函数作用域中

```js
function foo() {
  var bar = 1;
}
```
这段代码中有多少个作用域？

> 2 个，foo 函数所处的全局作用域和 bar 变量所处的函数作用域

```js
function foo() {
  var bar = 1;
  function baz() {
    var test = 1;
  }
}
```
这段代码中有多少个作用域？

> 3 个，foo 函数所处的全局作用域，bar 所处的函数作用域，和 test 所处的函数作用域

上面的 bar 变量和 baz 函数处于同一个作用域吗？

> 是的，因为它们都在 foo 函数中

上面 test 变量和 bar，baz处于同一个作用域中吗？

> 不是，因为 test 变量在 baz 函数中

JavaScript 用函数来划分作用域吗？

> 是的

```js
function foo() {
  var bar = 1;
}
console.log(bar);
```
会输出什么？

> Uncaught ReferenceError: bar is not defined

为什么会报错呢？

> 因为外部作用域**不能**访问内部作用域

```js
var foo = 1;
function bar() {
  console.log(foo);
}
bar();
```
会输出什么？

> 1

为什么不会报错？

> 因为内部作用域**可以**访问外部作用域

```js
var x = 1;
function foo() {
  var x = 2;
  console.log(x);
}
foo();
```
会输出什么？

> 2

为什么不是输出 1 ？

> 因为局部变量的优先级比外部变量高

```js
var x = 1;
function foo() {
  console.log(x);
  var x = 2;
  console.log(x);
}
foo();
```
会输出什么？

> undefined
> 2

为什么会这么奇怪？

> 因为变量声明有变量提升（Variable Hoisting）的过程

变量提升是什么？

> 声明语句会在执行前被处理，在任何地方声明一个变量，相当于在顶部位置声明

可以举个例子吗？

> ```js
bla = 0;
var bla;

// 相当于

var bla;
bla = 0;
```
这和之前的例子有什么关系？

> 函数内部声明的变量，都会先在函数的顶部声明。所以之前的例子就相当于
> ```js
function foo() {
  var x;
  console.log(x);
  x = 1;
  console.log(x)
}
```
什么是词法作用域？

> 变量的作用域是由它在源代码中所处位置决定的（词法），并且嵌套的函数可以访问到其外层作用域中声明的变量。

这和上面说到的内部作用域可以访问外部作用域有什么区别吗？

> 没有

什么是自由变量？

> 在函数内部使用到，但既不是该函数的参数，也不是该函数的局部变量的变量。

可以举个例子吗？

> ```js
var foo = 1;
function bar() {
  var baz = 2;
  console.log(foo + baz);
}
```
> 这里 bar 函数有三个变量：baz, console, foo
> 其中 baz 是局部变量， console 和 foo 都属于自由变量

为什么 console 和 foo 都是自由变量？

> 因为 console 和 foo 都在全局作用域中，在 bar 函数中是通过引用的方式来使用 console 和 foo 的

还需要了解其他概念吗？

> 不需要，现在已经可以深入了解闭包了


## 闭包

什么是闭包？

> 闭包是一个内部函数 [注1]

内部函数都是闭包吗？

> 不是，引用了自由变量的内部函数才是闭包

```js
var x = 1;
function foo() {
  console.log(x + 1);
}
```
foo 函数是一个闭包吗？

> 不是，因为 foo 函数不是一个内部函数

```js
function foo() {
  function bar() {
    var x = 1;
    return x + 1;
  }
}
```
bar 函数是一个闭包吗？

> 不是，因为它只是一个内部函数，并没有引用自由变量

```js
function foo() {
  var x = 1;
  function bar() {
    return x + 1;
  }
}
```
bar 函数是一个闭包吗？

> 是的，因为它是一个内部函数，同时引用了自由变量

闭包有什么特点？

> 1. 闭包可以访问外部变量
> 2. 闭包可以在外部函数返回之后依然保留外部变量的引用
> 2. 闭包会保留外部变量的引用，不是该变量的值

第一点在前面的例子中已经懂了。

> 很好

第二点还没懂，可以举个例子吗？

> ```js
function add(x) {
  return function(y) {
    return x + y;
  }
}

var add5 = add(5);
console.log(add5(10)) // 15
```
> 即便 add 函数已经返回，add5 中依然可以访问 x

第三点还没懂，可以举个例子吗？

> ```js
function user() {
  var id = 1;

  return {
    getId: function() { return id; },
    setId: function(newId) { id = newId }
  }
}

var foo = user();
foo.getId(); // 1
foo.setId(2);
foo.getId(); // 2
```
> 这里闭包中的 id 是一个引用，不是实际值

有点像私有方法？

> 是的，我们可以用闭包来实现私有方法

闭包还可以用来做什么？

> 闭包是函数式编程的骨架，掌握闭包之后你可以写出函数式 JavaScript 代码。

函数式编程是什么？

> 这不是本文的讨论范围，自己去学习吧。

## One More Thing

[注1] 根据 [Understanding JavaScript Closures](https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/) 这篇文章，事实上所有函数在创建的时候都会形成闭包。但这种闭包并没什么趣味，也没什么特别的用途，所以我们更关注的是由内部函数形成的闭包。


## 参考资料

- [http://uternet.github.io/TLS/](http://uternet.github.io/TLS/)
- [http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/var#var_hoisting)
- [http://javascriptissexy.com/understand-javascript-closures-with-ease/](http://javascriptissexy.com/understand-javascript-closures-with-ease/)
- [http://javascriptissexy.com/javascript-variable-scope-and-hoisting-explained/](http://javascriptissexy.com/javascript-variable-scope-and-hoisting-explained/)
- [http://stackoverflow.com/questions/12930272/javascript-closures-vs-anonymous-functions](http://stackoverflow.com/questions/12930272/javascript-closures-vs-anonymous-functions)
- [https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/](https://javascriptweblog.wordpress.com/2010/10/25/understanding-javascript-closures/)
- [http://www.moye.me/2014/12/29/closure_higher-order-function/](http://www.moye.me/2014/12/29/closure_higher-order-function/)
