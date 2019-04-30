---
title: JavaScript 中 Object.defineProperty 的使用
date: 2015-08-18 17:57:31
categories: [javascript]
tags: [javascript]
---

# Object.defineProperty

> The Object.defineProperty() method defines a new property directly on an object, or modifies an existing property on an object, and returns the object.

直接在一个对象上定义一个新的属性，或修改一个已经存在的属性。这个方法会返回该对象。

## 语法

`Object.defineProperty(obj, prop, descriptor)`

## 参数

- `Object obj` 目标对象

- `String prop` 需要定义的属性

- `Object descriptor` 该属性拥有的特性，可设置的值有：

    + `value` 属性的值，默认为 `undefined`。

    + `writable` 该属性是否可写，如果设置成 `false`，则任何对该属性改写的操作都无效（但不会报错），默认为 `false`。

    + `get` 一旦目标对象访问该属性，就会调用这个方法，并返回结果。默认为 `undefined`。

    + `set` 一旦目标对象设置该属性，就会调用这个方法。默认为 `undeinfed`。

    + `configurable` 如果为false，则任何尝试删除目标属性或修改属性以下特性（writable, configurable, enumerable）的行为将被无效化，默认为 `false`。

    + `enumerable` 是否能在for...in循环中遍历出来或在Object.keys中列举出来。默认为 `false`。

**注意:** 在 `descriptor` 中不能**同时**设置访问器 (`get` 和 `set`) 和 `wriable` 或 `value`，否则会报以下错误：
```
Invalid property.  A property cannot both have accessors and be writable or have a value
```

## 实际应用

我们知道，在 `Express.js` 升级到 4.0 之后，它把很多功能从核心库中移除了。当我们访问那些被移除的属性时，它会报错，告诉我们该属性已经被移除了。这个功能就是通过 `Object.defineProperty` 来实现的。看看源码吧：

```
[
  'json',
  'urlencoded',
  'bodyParser',
  'compress',
  'cookieSession',
  'session',
  'logger',
  'cookieParser',
  'favicon',
  'responseTime',
  'errorHandler',
  'timeout',
  'methodOverride',
  'vhost',
  'csrf',
  'directory',
  'limit',
  'multipart',
  'staticCache',
].forEach(function (name) {
  Object.defineProperty(exports, name, {
    get: function () {
      throw new Error('Most middleware (like ' + name + ') is no longer bundled with Express and must be installed separately. Please see https://github.com/senchalabs/connect#middleware.');
    },
    configurable: true
  });
});
```


# Object.defineProperties

> The Object.defineProperties() method defines new or modifies existing properties directly on an object, returning the object.

和 `Object.defineProperty` 类似，只不过这个方法可以设置多个属性。

## 语法

`Object.defineProperties(obj, props)`

## 参数

- `Object obj` 目标对象

- `Object props` 要为目标对象添加的属性，其中 `key` 和 `value` 分别代表 `Object.defineProperty` 中的第二和第三个参数。


# 参考资料

- http://www.cnblogs.com/rubylouvre/archive/2010/09/19/1831128.html
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object
- https://github.com/strongloop/express/blob/master/lib/express.js
