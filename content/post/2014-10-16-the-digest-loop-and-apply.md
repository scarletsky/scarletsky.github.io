---
title: The Digest Loop and $apply
date: 2014-10-16 11:12
categories: [javascript]
tags: [angular.js]
---

让我们来看一下 `Angular` 底下是怎么工作的。我们是怎样通过几行代码让神奇的数据绑定起作用的呢？我们先来理解一下 `$digest` 循环和 `$apply` 方法是怎样工作的。

在通常的浏览器工作流程中，当一个事件发生（例如 click），浏览器就会执行该事件注册过的回调函数。

当页面被加载，当一个 `$http` 请求被响应，当鼠标移动或者一个按钮被点击等等，都会触发事件。

当事件被触发后，JavaScript 就会创建一个 event object 并通过这个 event object 去执行任何监听该事件的函数。这个回调函数会运行在一个会返回给浏览器，可能更新 DOM 的 JavaScript 函数里面。

不可能有两种事件同时发生，浏览器会一直执行一个事件处理器（event handler）直到它完成为止，之后才会调用下一个事件处理器。

在非 Angular 的 JavaScript 中，我们可以把一个函数绑定到一个 div 的 click 事件中。任何情况下一个 click 事件触发后，该函数就会马上运行：

```js
var div = document.getElementById("clickDiv");
div.addEventListener("click",
  function(evt) {
    console.log("evt", evt);
});
```

打开 Chrome 的开发者工具，复制上面的代码到任何一个 web 页面，然后点击该页面。

任何时候当浏览器检测到一个 click 事件，浏览器就会运行通过 `addEventListener` 注册的函数。

当我们把 Angular 混合到该流程中后，它会扩展这种普通的浏览器流程，创建一个 Angular context。这个 Angular context 是指专门运行在 Angular event loop 中的代码，也就是 `$digest` 循环。要理解 Angular context，我们需要明确知道它里面发生了什么。在 `$digest` 循环里面，有两个主要的组件：
- `$watch list`
- `$evalAsync list`


# $watch list

每当我们追踪 view 中的一个事件的时候，我们都在注册一个回调函数，当一个事件发生在页面中的时候我们希望这个函数被调用。想想我们的第一个例子：

```html
<!DOCTYPE html>
<html ng-app>
<head>
  <title>Simple app</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.13/angular.js"></script>
</head>
<body>
  <input ng-model="name" type="text" placeholder="Your name">
  <h1>Hello {{ name }}</h1>
</body>
</html>
```

任何时候用户去更新 input field 的值，{{ name }} 都会在 UI 中发生改变。会发生这种改变是因为我们把 input field 的值**绑定**到 `$scope.name` 属性中。为了更新 view ，Angular 需要追踪这个变化。它通过把一个 watch function 添加到 `$watch List` 中来达到这个目的。

在 `$scope` 对象中的属性如果在视图中被使用的话，它们才会被绑定。上面的例子中，我们已经添加了一个函数到 `$watch list`。

记住：对于**所有**被绑定到 `$scope` 对象的 UI 元素来说，一个 `$watch` 被添加到 `$watch list` 中。
Remember: For **all** UI elements that are bound to a $scope object, a $watch is added to the `$watch list`.

这些 `$watch` 列表都会在 `$digest` 循环中通过一个叫 `dirty checking` 的过程被解决。


# Dirty Checking
Dirty checking 是一个简单的过程，可以归结为一个非常基本的概念：检查是否一个值改变了，整个应用程序尚未同步。

Dirty checking 策略在许多不同的应用程序中是很普遍的，远远超过 Angular 所做的。例如游戏引擎，数据库引擎，对象关系映射器（ORMs）都是这样。

我们的 Angular app 一直追踪着目前在 watch（在 watch 对象里面） 的值。Angular 会一个一个地检查 `$watch list` 里面的值，如果值还没改变的话，它会继续检查下一个。如果值改变了，app 会记录新的值，然后继续检查下面的。

![Image Title](https://www.ng-book.com/images/digest_loop/digest.png)

一旦 Angular 检查完整个 `$watch list`，如果任何值改变了，app 就会回到 `$watch` 循环中，直到它检测到没有东西改变。


# 参考资料
[The Digest Loop and $apply](https://www.ng-book.com/p/The-Digest-Loop-and-apply/)
