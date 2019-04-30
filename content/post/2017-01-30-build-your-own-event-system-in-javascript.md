---
title: 用 JavaScript 构建事件系统
date: 2017-01-30 08:30:21
categories: [javascript]
tags: [javascript]
---


## 简介

在组件化开发越来越流行的今天，事件系统演者着非常重要的角色，它经常作为组件间通讯的桥梁。
本文将讨论如何用 JavaScript 实现一个简单的事件系统。


## 基本结构

我们先回想一下使用事件系统的方式。
通常，我们要先通过 `on` / `listen` 方法注册为一个事件注册一个回调函数。
然后调用 `emit` / `fire` 来触发一个事件，该事件对应的回调函数就会一一触发。
这样就是我们平常使用的事件系统的工作方式。

事件系统本质上是一个键值对的合集，一个事件名对应多个函数。
在 JavaScript 中，我们很容易用一个 `Object` 来模拟这种行为。如：

```js
{
  ADD_ENTITY: [fn1, fn2, fn3],
  REMOVE_ENTITY: [fn4],
  UPDATE_ENTITY: [fn5, fn6]
}
```

上面的结构表示有三个事件：`ADD_ENTITY`, `REMOVE_ENTITY`, `UPDATE_ENTITY`，它们对应了一些回调函数。
当 `ADD_ENTITY` 触发的时候，`fn1`, `fn2`, `fn3` 都会依次调用，如此类推。


## on & off & emit

接下来我们将利用上述的结构来实现事件系统。其中最基本的三个操作是：

- `on` 监听事件
- `off` 移除事件
- `emit` 触发事件

按照上述的结构，我们很容易可以写出如下的代码：

```js
class EventEmitter {
  constructor() {
    this._events = {};
  }

  on(event, callback) {
    let callbacks = this._events[event] || [];
    callbacks.push(callback);
    this._events[event] = callbacks;
  }

  off(event) {
    delete this._events[event]
  }

  emit(event) {
    let callbacks = this._events[event];

    if (!callbacks || callbacks.length === 0) {
      throw new Error('You should register listener for event ' + event);
    }

    let args = [].slice.call(arguments, 1);
    callbacks.forEach(fn => fn.apply(this, args));
  }
}
```

上面的代码非常简单，`on` 和 `off` 只是在操作 `Object`，为其添加、更新或删除一些键值对。
而 `emit` 方法只是根据给定的事件名来调用它相关的回调函数。

这样，我们就实现了一个简单的事件系统：

```js
let ee = new EventEmitter();
ee.on('TEST1', (x) => console.log('In test1, x is: ', x));
ee.on('TEST2', () => console.log('In test2'));
ee.on('TEST2', () => console.log('In test2 again'));

ee.emit('TEST1', 1);
// In test1, x is:  1

ee.emit('TEST2');
// In test2
// In test2 again

ee.off('TEST1');
ee.off('TEST2');
console.log(ee._events);
// {}
```


## once

有些时候，我们需要指定某些回调函数只触发一次。
如：当页面加载完成后，执行初始化操作，这个操作就只会执行一次，之后再执行就不会生效。
上面的事件系统无法实现这种需求，因为每次触发事件，所有绑定的回调事件都会执行，不存在只执行一次的情况。

其实，我们只需要在执行回调函数后把该函数从列表中移除，就可以实现这种需求。
在这之前，我们先想想应该如何存储这些需要移除的函数。
存储方式大概有两种，一种是和不需要移除的函数保存在一起，即：

```js
{
  ADD_ENTITY: [
    { callback: fn1, once: false },
    { callback: fn2, once: true }
  ],
  REMOVE_ENTITY: [
    { callback: fn3, once: true }
  ]
}
```

这种方式在用 `emit` 触发事件的时候，需要根据 `once` 属性来判断是否需要移除该回调函数。
另外，这种方式还有一个好处，它能保证回调函数的调用顺序。

另一种方式则是分开存储，通过引入一个新的变量来区分一次生效和多次生效的回调函数，如：

```js
this._events = {
  ADD_ENTITY: [fn1, fn2],
  REMOVE_ENTITY: [],
};
this._onceHandlers = {
  ADD_ENTITY: [fn2],
  REMOVE_ENTITY: [fn3]
}
```

这种方式把两种函数隔离开，在移除回调函数的时候变得非常方便，直接移除整个 key 即可。

无论我们选择哪种存储方式，对 `once` 的实现影响都不大，因为只需要对 `on` 方法做一点修改就可以了。
同样，对于删除函数的操作，第一种方式只需要找出 `once` 为 `true` 的列表索引，然后根据索引来移除即可，而第二种方式直接删除 key 就可以了。
实现起来都比较简单，因此就不给出示例代码了。


## 参考资料

- https://corcoran.io/2013/06/01/building-a-minimal-javascript-event-system/
