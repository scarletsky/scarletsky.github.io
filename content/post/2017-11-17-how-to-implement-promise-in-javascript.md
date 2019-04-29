---
title: Promise 实现原理
date: 2017-11-17 11:52:30
categories: [javascript]
---

## 简介

Promise 是目前流行的处理异步操作结果的方式。
本文假设读者已经了解过 Promise/A+ 规范，并熟悉 Promise 的用法。

## 构造

我们先看看如何构造一个 Promise ：

```js
new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve(1);
  }, 1000);
});
```

观察上面的 Promise，我们可以知道一个 Promise 接受一个函数作为参数，而这个函数也提供了两个函数提供 Promise 内部使用。利用 JavaScript 的特性，我们可以很容易实现这一点：

```js
function MyPromise(fn) {
  fn(function resolve(value) {
    console.log(value);
  }, function reject(reason) {
    console.log(reason);
  });
}

new MyPromise(function(resolve, reject) {
  setTimeout(function() {
    resolve(1);
  }, 1000);
});
```


## 状态转换

Promise/A+ 规范中定义了三个状态：`pending`、`fulfilled`、`rejected`，任何一个 Promise 必须处于其中一个状态。

- 当 Promise 处于 `pending` 状态时:
  - 它可以转换成 `fulfilled` 或 `rejected` 状态。

- 当 Promise 处于 `fulfilled` 或 `rejected` 状态时：
  - 它不允许转换成其他状态。
  - 它必须拥有一个不能被改变的值(或原因)。

这本质上是一个状态机，通过 `resolve` 或 `reject` 来改变它内部的状态。因此，我们只需要做个小改动就可以实现了：

```js
var STATUS_PENDING = 0;
var STATUS_FULFILLED = 1;
var STATUS_REJECTED = 2;

function MyPromise(fn) {
  var status = STATUS_PENDING;

  fn(function resolve(value) {
    status = STATUS_REJECTED;
  }, function reject(reason) {
    status = STATUS_FULFILLED;
  });
}
```

上面的示例中，我们添加了一个内部变量 `status` 来实现 Promise 的状态转换。


## Then

Promise/A+ 规定任何一个 Promise 都必须实现 `then` 方法，用来访问 Promise 操作的结果(或原因)。`then` 方法接受两个参数：`onFulfilled` 和 `onRejected`，即：

```
promise.then(onFulFilled, onRejected);
```

这两个参数有如下规定：

- `onFulfilled` 和 `onRejected` 都是可选的，如果他们不是函数，那必须忽略它。
- 如果 `onFulfilled` 是一个函数：
  - 当 Promise 状态变成 `fulfilled` 时必须被调用，它的第一个参数是 Promise 的结果。
  - 它禁止在 Promise 状态变成 `fulfilled` 前被调用。
  - 它只能被调用一次。
- 如果 `onRejected` 是一个函数：
  - 当 Promise 状态变成 `rejected` 时必须被调用，它的第一个参数是 Promise 的原因。
  - 它禁止在 Promise 状态变成 `rejected` 前被调用。
  - 它只能被调用一次。
- `then` 方法可以在同一个 Promise 中多次调用:
  - 当 Promise 状态变成 `fulfilled` 时，所有 `onFulfilled` 回调函数必须按顺序执行。
  - 当 Promise 状态变成 `rejected` 时，所有 `onRejected` 回调函数必须按顺序执行。
- `then` 方法必须返回一个 Promise。

要实现上面的规范，我们需要两个变量来保存回调函数列表：`onFulfilledCallbacks` 和 `onRejectedCallbacks`，然后在状态转换的时候按顺序调用这些回调函数即可：

```js
var STATUS_PENDING = 0;
var STATUS_FULFILLED = 1;
var STATUS_REJECTED = 2;

function MyPromise(fn) {
  var status = STATUS_PENDING;
  var onFulfilledCallbacks = [];
  var onRejectedCallbacks = [];

  this.then = function(onFulfilled, onRejected) {
    onFulfilledCallbacks.push(onFulfilled);
    onRejectedCallbacks.push(onRejected);
    return this;
  };

  fn(function resolve(value) {
    status = STATUS_REJECTED;
    onFulfilledCallbacks.forEach(function(fn) { fn(value); });
  }, function reject(reason) {
    status = STATUS_FULFILLED;
    onRejectedCallbacks.forEach(function(fn) { fn(reason); });
  });
}
```

上面的例子中实现了最简单的 Promise，但离「可用」还有很长的距离，而且上面只是实现了 Promise/A+ 规范中的一小部分而已，还有很多细节需要处理的。

## 参考资料
https://promisesaplus.com/
https://www.promisejs.org/implementing/
https://github.com/Jocs/jocs.github.io/issues/7
