---
title: Lodash 中 assign，extend 和 merge 的区别
date: 2016-04-02 10:51:19
categories: [javascript]
tags: [javascript]
---

## 简介

我们经常在别人的代码中看见 `assign`，`extend`，`merge` 函数，这三个函数用起来很相似，都是合并源对象的属性到目标对象中。

既然都是合并对象，为什么还分三个不同的函数呢？它们之间到底有什么区别呢？

## assign(object, [sources])

我们先看看官方网站上面的定义：

> Assigns own enumerable string keyed properties of source objects to the destination object. Source objects are applied from left to right. Subsequent sources overwrite property assignments of previous sources.

把源对象(sources)的属性分配到目标对象(object)，源对象会从左往右地调用，后面对象的属性会覆盖前面的。

看看下面的例子:

```js
assign({}, { a: 1 }, { b: 2 });
// { a: 1, b: 2 }

// 后面的 { a: 2 } 把前面的 { a: 1 } 覆盖了
assign({}, { a: 1 }, { b: 2 }, { a: 2 });
// { a: 2, b: 2 }

// 观察下面两个例子，如果属性值为 object，后面的值会覆盖前面的值
assign(
  {},
  { a: 1 },
  { b: { c: 2, d: 3 } }
)
// { a: 1, b: { c: 2, d: 3 } }

assign(
  {},
  { a: 1 },
  { b: { c: 2, d: 3 } },
  { b: { e: 4 } }
)
// { a: 1, b: { e: 4 } }

// `assign` 函数会忽略原型链上的属性。
function Foo() { this.c = 3; }
Foo.prototype.d = 4;
assign({ a: 1 }, new Foo());
// { a: 1, c: 3 }

// `assign` 会修改原来的对象
var test = { a: 1 };
assign(test, { b: 2 }); // { a: 1, b: 2 }
console.log(test);      // { a: 1, b: 2 }
```

## extend(object, [sources])

在 3.x 版本中，`extend` 是 `assign` 的别名，它们的作用是一模一样的。
在 4.x 版本中，`extend` 是 `assignIn` 的别名，和 `assign` 有点区别。

官方定义如下：

> This method is like _.assign except that it iterates over own and inherited source properties.

在上面的例子中，我们知道 `assign` 函数不会把原型链上的属性合并到目标对象，而 `extend` 或 `assignIn` 函数则会！

```js
// Important !! this is Lodash 4.x !!

// 把源对象原型链上的属性也合并到目标对象上！
function Foo() { this.c = 3; }
Foo.prototype.d = 4;
extend({ a: 1 }, new Foo());
// { a: 1, c: 3, d: 4 }
```

## merge(object, [sources])

我们看看 `merge` 函数的定义：

> This method is like _.assign except that it recursively merges own and inherited enumerable string keyed properties of source objects into the destination object. Source properties that resolve to undefined are skipped if a destination value exists. Array and plain object properties are merged recursively.Other objects and value types are overridden by assignment. Source objects are applied from left to right. Subsequent sources overwrite property assignments of previous sources.

`merge` 也和 `assign` 类似，不同的地方在于 `merge` 遇到相同属性的时候，如果属性值为纯对象(plain object)或者集合(collection)时，不是用后面的属性值去覆盖前面的属性值，而是会把前后两个属性值合并。
如果源对象的属性值为 `undefined`，则会忽略该属性。

```js
assign(
  {},
  { a: 1 },
  { b: { c: 2, d: 3} },
  { b: { e: 4 } }
)
// { a: 1, b: { e: 4 } }
merge(
  {},
  { a: 1 },
  { b: { c: 2, d: 3} },
  { b: { e: 4 } }
)
// { a: 1, b: { c: 2, d: 3, e: 4 } }

// 合并集合
var users = {
  'data': [{ 'user': 'barney' }, { 'user': 'fred' }]
};
var ages = {
  'data': [{ 'age': 36 }, { 'age': 40 }]
};
merge({}, users, ages)
// { data: [ { user: 'barney', age: 36 }, { user: 'fred', age: 40 } ] }

// merge 函数会修改原来的对象！
merge(users, ages)
console.log(users) // { data: [ { user: 'barney', age: 36 }, { user: 'fred', age: 40 } ]
```

## 总结

### 相同之处

- 都可以用来合并对象
- 都会修改原来的对象 (如果原来的对象是作为函数的第一个参数的话)

### 不同之处

- `assign` 函数不会处理原型链上的属性，也不会合并相同的属性，而是用后面的属性值覆盖前面的属性值

- `extend`
  - 3.x 版本中和 `assign` 一样
  - 4.x 版本中会合并原型链上的属性

- `merge` 遇到相同属性名的时候，如果属性值是纯对象或集合的时候，会合并属性值

## 参考资料
https://lodash.com/docs
http://stackoverflow.com/questions/19965844/lodash-difference-between-extend-assign-and-merge
