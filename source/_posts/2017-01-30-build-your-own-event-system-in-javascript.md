---
title: 用 JavaScript 构建事件系统
date: 2017-01-30 08:30:21
category: javascript
tags: [javascript]
---


## 简介
在组件化开发越来越流行的今天，事件系统演者着非常重要的角色，它经常作为组件间通讯的桥梁。
本文将讨论如何用 JavaScript 实现一个简单的事件系统。


## 基本结构
我们先回想一下使用事件系统的方式。
通常，我们要先通过 `.on` / `.listen` 方法注册为一个事件注册一个回调函数。
然后调用 `.emit` / `.fire` 来触发一个事件，该事件对应的回调函数就会一一触发。
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


## 参考资料
https://corcoran.io/2013/06/01/building-a-minimal-javascript-event-system/