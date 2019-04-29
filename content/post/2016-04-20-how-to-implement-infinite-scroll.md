---
title: 如何实现无限滚动
date: 2016-04-20 17:14:21
categories: [javascript]
tags: [javascript, infinite scroll]
---

## 简介

无限滚动对我们来说已经是很常见的功能了，具体表现为当页面滚动到某个位置时就自动加载数据，本文将探讨无限滚动的实现原理以及优化。

## 原理

我们先看看最简单的无限滚动的例子：

```js
function fetchData() {
  fetch(path).then(res => doSomeThing(res.data));
}

window.addEventListener('scroll', fetchData);
```

上面就是无限滚动最简单的例子啦~
其实就是监听 `window` 对象的 `scroll` 事件，然后再触发获取数据的函数~

然而，上面的例子中还有很多问题，其中最大的问题就是 **获取数据的函数(以后叫 fetch 函数)没有触发条件**， 我们还需要不断优化，才能在生产环境下使用。

## 添加触发条件

我们先想想，一般情况下，fetch 函数的触发条件有哪些呢 ？

- 在 fetch 过程中不能重复触发
- 没有更多数据的时候不能再触发
- 屏幕距离容器边缘 xxx 的时候触发

前两点很好处理，只要加个 `isLoading` 和 `isEnd` 的变量就可以了。
添加这两个变量之后，我们的代码就变成下面的样子啦：

```js
var isLoading = false;
var isEnd = false;

function fetchData() {

  if ( !isLoading && !isEnd ) {

    isLoading = true;

    fetch(path).then(res => {
      isLoading = false;
      res.data.length === 0 && isEnd = true;
      doSomething(res.data);
    });

  }

}
window.addEventListener('scroll', fetchData);
```

第三点对不熟悉 DOM 的童鞋来说就有点难度了~

## 计算屏幕与容器边缘的距离

我们以计算屏幕底部与容器底部边缘为例:

如果有 api 可以直接得到元素底部与屏幕底部的距离就最好啦，可以省去麻烦，但实际上并没有这样的 api。
然而，我们可以通过浏览器提供的两个 api，计算出元素底部与屏幕底部之间的距离。

第一个 api 是 `window.innerHeight`，它返回的是屏幕（viewport）高度。
第二个 api 就是 `Element.getBoundingClientRect` ，这个方法用来计算元素边缘与屏幕（viewport）之间的距离。
需要提醒一下，`Element.getBoundingClientRect` 会得到这么一个类 Object 对象：

```js
ClientRect {
  width: 760,   // 元素宽度
  height: 2500, // 元素高度
  top: -1352,   // 元素上边缘与屏幕上边缘的距离
  bottom: 1239, // 元素下边缘与屏幕上边缘的距离
  left: 760,    // 元素左边缘与屏幕左边缘的距离
  right: 860    // 元素右边缘与屏幕左边缘的距离
}
```

可以看看下面这图：

```
     +------> +--------------------------------------------------------+
     |        |                     document.body                      |
     |        |                                                        |
     |        |                                                        |
body.getBoundingClientRect().top                                       |
     |        |                                                        |
     |        |                                                        |
     |        +--------------------------------------------------------+
     |        | browser                                              x |
     +------> +--------------------------------------------------------+ <--+
     |        | window                                                 |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
window.innerHeight                                                     |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                               body.getBoundingClientRect().bottom
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     |        |                                                        |    |
     +------> +--------------------------------------------------------+    |
              |                                                        |    |
              |                                                        |    |
              |                                                        |    |
              |                                                        |    |
              |                                                        |    |
              |                                                        |    |
              +--------------------------------------------------------+ <--+




```

有了这两个 api，我们很容易就可以计算出元素底部边缘与屏幕底部边缘的位置啦~

我们再修改下我们的代码：

```js
var isLoading = false;
var isEnd = false;
var triggerDistance = 200;

function fetchData() {

  var distance = container.getBoundingClientRect().bottom - window.innerHeight;
  if ( !isLoading && !isEnd && distance < triggerDistance ) {

    isLoading = true;

    fetch(path).then(res => {
      isLoading = false;
      res.data.length === 0 && isEnd = true;
      doSomething(res.data);
    });

  }

}
window.addEventListener('scroll', fetchData);
```

修改之后，当容器底部与屏幕底部距离小于 200 的时候，才会触发 fetch 函数，这样我们的无限滚动就更加实用啦！


## 支持 window 以外的元素

然而，并不是只有 window 才可以滚动，拥有高度的级块元素只要设置了 `overflow: scroll` 都是可以滚动的。
我们需要再修改一下代码来让级块元素也支持无限滚动！

```js
function fetchData() { /* do something */ }
window.addEventListener('scroll', fetchData);
document.getElementById('container').addEventListener('scroll', fetchData);
```

很简单吧！只需要为该容器元素添加一个 scroll 的事件监听器就好啦！


## 参考资料
https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect
