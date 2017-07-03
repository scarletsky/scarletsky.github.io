---
title: 我所了解的函数式编程
date: 2017-06-30 22:01:07
category: technology
tags: [functional-programming]
---

## 前言
我从去年年初开始接触函数式编程，看了很多和函数式编程相关的书和博客，如[JS函数式编程](https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details)、
[Haskell趣学指南](https://learnyoua.haskell.sg/content/zh-cn/ch01/introduction.html)、[Real World Haskell](https://rwh.readthedocs.io/en/latest/)，接触到很多函数式语言，如 Haskell, Elixir, Lisp系列等等。随着函数式代码写得越来越多，我越来越喜欢函数式的编程方式，渐渐成为了一名函数式编程的爱好者。本文是我学习函数式编程以来的一些总结。

## 简介
函数式编程是一种编程范式，主要是利用函数把运算过程封装起来，通过组合各种函数来计算结果。举个简单的例子，假设我们要把字符串 `functional programming is great` 变成每个单词首字母大写，我们可以这样实现：

```js
var string = 'functional programming is great';
var result = string
  .split(' ')
  .map(v => v.slice(0, 1).toUpperCase() + v.slice(1))
  .join(' ');
```

或许对很多人来说这并没有什么特别之处，但这里已经是利用了函数式编程的核心思想： **通过函数对数据进行转换**。

上面的例子先用 `split` 把字符串转换数组，然后再通过 `map` 把各元素的首字母转换成大写，最后通过 `join` 把数组转换成字符串。 整个过程就是 `join(map(split(str)))`，我们只是利用面向对象编程的接口来把这个转换过程写成上面那样。

细心的同学可以发现，其实这个过程和 Unix 的管道操作非常相似，管道操作通过 `|` 来把上一个程序的输出重定向到下一个程序的输入，如：`cat log.txt | grep mystring`。这种方式非常符合我们对计算机的理解：给定特定输入，返回计算结果。函数式编程正好是这种方式的一种「实现」。

## 函数式编程与面向对象编程
我们知道，函数式编程是一种编程范式，它经常与面向对象编程作比较。
经常编写面向对象程序的人们会发现，面向对象是让**操作围绕数据**，我们会定义一个类的属性，然后定义一些方法，这些方法会围绕这些属性来进行操作。如：

```js
class ObjectManager {
  constructor() {
    this._objects = [];
  }

  add(object) {
    this._objects.push(obj);
    return this;
  }

  get(index) {
    return this._objects[index];
  }
}

var objects = new ObjectManager();
objects.add({ key: 1 }).add({ key: 2 }).add({ key: 3 });
objects.get(2); // { key: 3 }
```

可以看见，上面代码中的 `add`、`get` 方法其实是围绕 `this._objects` 这个数据来进行操作。如果是函数式编程的话，更多的是让**数据围绕操作**。我们会定义一系列函数，然后让数据「流」过这些函数，最后输出结果。

```js
// basic implementation
function add(array, object) {
  return array.concat(object);
}

function get(array, index) {
  return array[index];
}

var objects = add(add(add([], { key: 1 }), { key: 2 }), { key: 3});
get(objects, 2); // { key: 3 }
```

这就是用函数式编程改写过的代码，尽管非常难以阅读，一点也不符合函数式编程简洁的特点，但它确实代表了函数式编程的基本思想：**通过函数对数据进行转换**。我们可以用以下方式来改写：

```js
// improvement
Array.prototype.get = function(index) { return this[index]; };
[].concat({ key: 1 }) // [{ key: 1 }]
  .concat({ key: 2 }) // [{ key: 1 }, { key: 2 }]
  .concat({ key: 3 }) // [{ key: 1 }, { key: 2 }, { key: 3 }]
  .get(2); // { key: 3 }
```

这个版本才像函数式的写法，它有两个要点：

1. 用 `get` 代替直接通过 `[index]` 来访问数据，它和面向对象版本的 `get` 非常相似，是为了避免直接访问数组。
2. 利用 `concat` 代替 `push` 拼接数组，这和面向对象版本的 `add` 有本质的区别。尽管大家看起来都是链式调用，但面向对象版本是通过返回 `this` 来实现链式调用，而函数式版本是通过返回一个新数组的方式来实现链式调用的。这点可以很好的体现出 **操作围绕数据** 和 **数据围绕操作** 的区别。


## 基本特点

函数式编程的基本特点有两个：

- 通过函数来对数据进行转换
- 通过串联多个函数来求结果

我们来看看下面这个例子：假设有三个函数，`f(x)` 可以把 x 转换成 a，`g(a)` 可以把 a 转换成 b，`h(b)` 可以把 b 转换成 c，即：

```
f(x) -> a
g(a) -> b
h(b) -> c
```

那么怎么把 x 转换成 c ？相信这难不倒大家，我们只需要通过如下式子就可以实现：

```
h( g( f(x) ) )
```

上面的式子可以解读成把 `f(x)` 的结果传给 g 函数(即 `g(f(x))`), 再把结果传给 h 函数，即是上式子的意思。

由于串联函数在函数式编程中实在太常见了，各个函数式语言都有自己的表达方式。下面我们来看看各个函数式语言是怎么表达这个式子的：

Lisp 家族
```lisp
(h (g (f x)))
```

Haskell
```hs
let result = h . g . f x
```

Elixir:
```ex
result = x |> f |> g |> h
```

函数式编程除了提倡串联函数之外，还有很多有趣的特性，接下来我们来看看函数式编程中一些常见的特性。

## 常见特性

很多时候你去查阅函数式编程的相关资料，你都会看到下面这些术语：

### 无副作用

指调用函数时不会修改外部状态，即一个函数调用 n 次后依然返回同样的结果。

```js
var a = 1;
// 含有副作用，它修改了外部变量 a
// 多次调用结果不一样
function test1() {
  a++
  return a;
}

// 无副作用，没有修改外部状态
// 多次调用结果一样
function test2(a) {
  return a + 1;
}
```


### 透明引用

指一个函数只会用到传递给它的变量以及自己内部创建的变量，不会使用到其他变量。

```js
var a = 1;
var b = 2;
// 函数内部使用的变量并不属于它的作用域
function test1() {
  return a + b;
}
// 函数内部使用的变量是显式传递进去的
function test2(a, b) {
  return a + b;
}
```

### 不可变变量

指的是一个变量一旦创建后，就不能再进行修改，任何修改都会生成一个新的变量。使用不可变变量最大的好处是线程安全。多个线程可以同时访问同一个不可变变量，让并行变得更容易实现。
由于 JavaScript 原生不支持不可变变量，需要通过第三方库来实现。 (如 [Immutable.js](https://facebook.github.io/immutable-js/)，[Mori](http://swannodette.github.io/mori/) 等等)

```js
var obj = Immutable({ a: 1 });
var obj2 = obj.set('a', 2);
console.log(obj);  // Immutable({ a: 1 })
console.log(obj2); // Immutable({ a: 2 })
```


### 函数是一等公民

指的是函数和其他数据类型一样，可以赋值给变量，可以作为参数传递，可以作为返回值，可以作为数组中的函数，可以是对象中的一个值等等。下面这些术语都是围绕这一特性的应用：

  * **高阶函数 (Higher order function)**
  如果一个函数接受函数作为参数，或者返回值为函数，那么该函数就是高阶函数。

  ```js
// 接受函数的函数
function test1(callback) {
  callback()
}

// 返回函数的函数
function test2() {
  return function() {
    console.log(1);
  }
}
  ```

  * **闭包 (Closure)**
  如果一个函数引用了自由变量，那么该函数就是一个闭包。何谓自由变量？自由变量是指不属于该函数作用域的变量(所有全局变量都是自由变量，严格来说引用了全局变量的函数都是闭包，但这种闭包并没有什么用，通常情况下我们说的闭包是指函数内部的函数)。

  ```js
// test1 是普通函数
function test1() {
  var a = 1;

  // test2 是内部函数
  // 它引用了 test1 作用域中的变量 a
  // 因此它是一个闭包
  return function test2() {
    return a + 1;
  }
}
  ```

  * **函数组合 (Composition)**
  前面提到过，函数式编程的一个特点是通过串联函数来求值。然而，随着串联函数数量的增多，代码的可读性就会不断下降。函数组合就是用来解决这个问题的方法。假设有一个 `compose` 函数，它可以接受多个函数作为参数，然后返回一个新的函数。当我们为这个新函数传递参数时，该参数就会「流」过其中的函数，最后返回结果。

```
// 组合 f, g 函数, 从左到右求值
var composed = compose(f,g);
composed(x) = g(f(x));

// 组合 f, g, h 函数, 从右到左求职
var composed = composeRight(h, g, f);
composed(x) = h(g(f(x)));
```

  * **柯里化 (Currying)**
  柯里化是对函数的封装，当调用函数时传递参数数量不足时，会返回一个新的函数，直到参数数量足够时才进行求值。

```js
// 假设函数 f 接受三个参数
f = curry(f);
f(x,y,z) = f(x,y)(z) = f(x)(y,z) = f(x)(y)(z);
```

* **模式匹配 (Pattern matching)**
模式匹配是指可以为一个函数定义多个版本，通过传入不同参数来调用对应的函数。形式上有点像「方法重载」，但方法重载是通过传入*参数类型*不同来区分的，模式匹配没有这个限制。利用模式匹配，我们可以去掉函数中的「分支」(最常见的是 `if`)，写出非常简洁的代码。

```js
// 普通版本，需要在函数内部对参数进行判断
function fib(x) {
  if (x === 0) return 0;
  if (x === 1) return 1;
  return fib(x-1) + fib(x-2);
}

// 模式匹配版本。
// 由于 JavaScript 不支持模式匹配，
// 下面代码只是作演示用途
var fib = (0) => 0;
var fib = (1) => 1;
var fib = (x) => fib(x-1) + fib(x-2);
// 调用
fib(10);
```

可以看到，如果能在语言层面支持模式匹配，那么在函数定义中就可以省略很多「分支」。
上面的例子可能不够明显，请想象一下在编写 Restful API 时，我们需要解析请求中的参数，然后返回对应内容。传统方式下，我们需要写很多 `if` 语句来进行判断，如果该 API 支持非常多参数，那么该函数就会有非常多的 `if` 语句，这种函数在维护的时候会是一个噩梦。
如果支持模式匹配，那么我们只需要定义多个函数即可。


## 常见函数
接下来来看看几个常见的函数。

* `map`
映射函数，通过定义函数来实现数据的映射，简单来说就是令一个数组变换成另一个数组，如要将 `[1,2,3]` 变换成 `[4,5,6]`，我们只需要定义一个 `x+3` 的变换即可，即：

```js
[1,2,3].map(x => x+3); // 返回 [4, 5, 6]
```

`map` 函数的实现非常简单，只需要遍历数组，然后对每个元素调用一次变换函数，最后返回结果即可。

```js
function map(array, fn) {
  var result = [];
  for (var i = 0; i < array.length; i++) {
    var transformed = fn(array[i], i);
    result.push(transformed);
  }
  return result;
}
```


* `reduce` (有些语言里面叫 `fold`)
归纳函数，定义一个函数来归纳数组，最后返回一个值，简单来说就是令一个数组变成一个值。
 `[1,2,3]` -> 6


```js
function reduce(array, acc, fn) {
  for (var i = 0; i < array.length; i++) {
  acc = fn(acc, array[i]);
  }
  return acc;
}
```

* filter - 过滤
 `[1,2,3]` -> `[1,3]`

实现


```js
function filter(array, fn) {
  var result = [];
  for (var i = 0; i < array.length; i++) {
  if (fn(array[i], i)) {
  result.push(array[i]);
  }
  }
  return result;
}
```

混合使用
`http://realibox.com?type=1&keyword=hello` ->
`type=1&keyword=hello ` ->
`[ 'type=1', 'keyword=hello' ]` ->
`[ [ 'type', '1' ], [ 'keyword', 'hello' ] ]` ->
`{ type: 1, keyword: 'hello' }`

```js
url
.split('?')[1]
.split('&')
.map(v => v.split('='))
.reduce((prev, next) =>
Object.assign({}, prev, { [next[0]]: next[1] }),
{});
```

## 列表
表示形式
`[ head | tail ]`

```
[1,2,3]
[1 | [2,3]]
[1 | [2 | [3]]]
[1 | [2 | [3 | []]]]

head([1,2,3]) -> 1
tail([1,2,3]) -> [2,3]
```

## 递归
`[1,2,3,4,5]`

```
sum [] = 0
sum (x:xs) = x + sum(xs)

1 + sum([2,3,4,5])
1 + 2 + sum([3,4,5])
1 + 2 + 3 + sum([4,5])
1 + 2 + 3 + 4 + sum([5])
1 + 2 + 3 + 4 + 5 + sum([])
1 + 2 + 3 + 4 + 5 + 0
15

sum ([], acc) = acc
sum (x:xs, acc) = sum(xs, x + acc)

sum([2,3,4,5], 1 + 0)
sum([3,4,5], 2 + 1 + 0)
sum([4,5], 3 + 2 + 1 + 0)
sum([5], 4 + 3 + 2 + 1 + 0)
sum([], 5 + 4 + 3 + 2 + 1 + 0)
13
```

## 优点
* 代码简洁
* 不须考虑死锁，非常适合并行执行
* 测试和调试非常方便

## 缺点
* 性能比命令式编程差
* 不适合处理可变状态
* 不适合做 IO 操作
* 不适合写 GUI

## 参考资料
[什么是函数式编程思维？](https://www.zhihu.com/question/28292740)
[函数式编程初探](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)
[声明式编程和命令式编程的比较](http://www.vaikan.com/imperative-vs-declarative/)
