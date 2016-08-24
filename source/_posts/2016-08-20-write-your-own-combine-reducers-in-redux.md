---
title: 自定义 Redux 中的 combineReducers
date: 2016-08-20 20:05:28
category: javascript
tags: [javascript, redux]
---

## 简介
Redux 中的 `combineReducers` 能让我们很方便地把多个 reducers 组合起来，成为一个新的 reducer。 
然而，随着我们的应用变得越来越复杂，`combineReducers` 有可能不能满足我们的需求。
正如 Redux 官方文档所说:

> This helper is just a convenience! You can write your own combineReducers that works differently, or even assemble the state object from the child reducers manually and write a root reducing function explicitly, like you would write any other function.

`combineReducers` 只是方便我们使用而已，我们可以自定义一个完全不同的 `combineReducers` 来满足我们特殊的需求。

## 原理
我们先回忆一下 reducer 的写法：

```js
const reducer = (oldState, action) => newState;
```

reducer 是一个普通的函数，接受两个参数：`oldState` 和 `action`，然后返回一个 `newState`。
为了把多个 reducers 组合起来，我们通常会用 Redux 自带的 `combineReducers` 来实现:

```js
const rootReducer = combineReducers({
  key1: key1Reducer,
  key2: key2Reducer
});
```

先留意一下我们传了什么给 `combineReducers`:
```js
{
  key1: function(state.key1, action) { /*...*/ },
  key2: function(state.key2, action) { /*...*/ },
}
```

好了，让我们先来想一想，经过 `combineReducers` 的处理之后，我们得到了什么呢？
不用想了，很显然我们得到了一个新的 reducer。
那这个新的 reducer 又长什么样呢？
```js
const rootReducer = (oldState, action) => newState;
```

你应该不会惊讶，因为所有 reducer 都长这个样子，即使它是已经被组合过的 reducer，它也是长这个样子。
现在你应该猜到 `combineReducers` 做了什么了吧？其实它最基本形态是这样子的:

```js
function combineReducers(reducers) {
  return function (state, action) { /*...*/ };
}
```

它接受 `reducers` 作为参数，然后返回一个标准的 reducer 函数。

> 注意:
> 其实到了这一步，我们就可以自定义 `combineReducers` 了，我们完全可以写一个类似的函数，然后在里面写各种 `switch...case` 语句来达到自定义的目的。
> 但我觉得我们还是先看看 Redux 自带的 `combineReducers` 做了什么比较好，因为我们自定义的 `combineReducers` 很有可能需要原来的功能。

还记得我刚才叫你留意的地方吗？没错，就是下面这个:
```js
// reducers
{
  key1: function(state.key1, action) { /*...*/ },
  key2: function(state.key2, action) { /*...*/ }
}
```

我们来回想一下 `store.dispatch(action)` 的过程：当一个 `action` 触发的时候，所有 reducers 都应该响应这个 `action`，做出相应的改变，最后返回一个新的 `store`。
对着上面这个结构，我们其实很容易就能写出这样的效果，还能加上一些其他的处理：

```js
function reCombineReducers(reducers) {
  return function (state, action) {
    switch (action.type) {
      case SP_ACTION:
        return Object.assign({}, state, { /* do something */ });
      default:
        return Object.keys(reducers)
                    .map(k => ({ [k]: reducers[k](state[k], action) }))
                    .reduce((prev, next) => Object.assign({}, prev, next));
    }
  }
}
```

上面的例子模拟了原来 `combineReducers` 的功能，还对 `SP_ACTION` 进行了特殊的处理，很简单吧！

然而，上面的例子虽然模拟了 `combineReducers` 的功能，但失去了 `combineReducers` 的检查对象变化的功能，因为现在的 default block 中会返回一个全新的对象。
有没有方法可以既保留 `combineReducers` 的全部功能，又能扩展它呢？
其实很简单，我们只要利用 `combineReducers` 返回的函数就可以了！
(感谢liximomo 指出上面例子中的缺陷)

```js
function reCombineReducers(reducers) {
  let fn = combineReducers(reducers);
  return function (state, action) {
    switch (action.type) {
      case SP_ACTION:
        return Object.assign({}, state, { /* do something */ });
      default:
        return fn(state, action);
    }
  }
}
```


## 实例
按照 Redux 的原则，不同的 reducer 应该相互独立的，它们之间不应该有任何依赖。
这个原则看着是很美好的，但在实际使用中还是会有一些例外的情况。
一个很简单的例子，也是我遇到过的例子，就是实现一个简单的表格 (其实我的情况复杂的多，需要实现类似 Excel 那样的操作，同时支持其他额外的功能)。
我们先来设计一下 `state`:

```js
// state
{
  rows: { ... },
  cells: { ... },
  data: { ... }
}
```

`rows`, `cells`, `data` 都会响应一些特定的 `action` (如 `CHANGE_ROW_PROPS`, `CHANGE_CELL_PROPS`, `CHANGE_DATA`)，做出相应的改变，这些都是我们所期望的。
然而，当出现一些特殊的 action (如 `GET_TABLE_SUCCESS`，表示成功从服务端获取数据) 的时候，灾难就出现了：
所有的 reducer 都需要监听 `GET_TABLE_SUCCESS` 这个 action，这意味着如果我们有 n 个 reducer 的话，我们就需要修改 n 个文件！
当我再加上 `UPDATE_TABLE_SUCCESS`，`REMOVE_TABLE_SUCCESS` 之类的 `action` 时，我要再修改 n 个文件！
这不合理啊，为什么我加一个简单的功能，需要修改这么多文件，最重要的是，这些修改都是非常类似！

这时候，我们就需要自定义 `combineReducers` 来解决我们的需求拉：

```js
function reCombineReducers(reducers) {
  let fn = combineReducers(reducers);
  return function (state, action) {
    switch (action.type) {
      case GET_TABLE_SUCCESS:
      case UPDATE_TABLE_SUCCESS:
        return Object.assign({}, state, action.payload.table);
      case REMOVE_TABLE_SUCCESS:
        return initState;
      default:
        return fn(state, action);
    }
  }
}

const table = reCombineReducers({
  sections,
  suites,
  rows,
  cells,
  toys,
  data,
  logics
})
```

怎么样，是不是比修改多个文件舒服很多？

(完)


## 参考资料
http://redux.js.org/docs/api/combineReducers.html
https://github.com/reactjs/redux/blob/master/src/combineReducers.js
