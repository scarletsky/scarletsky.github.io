---
title: Why list is non-determinism value in Haskell
date: 2016-07-16 16:01:39
category: haskell
tags: haskell
---

## 简介

看 《Learn you a haskell for a great good》 这本书的过程中，有时候会看到 **non-determinism value** 这个词，虽然具体不知道什么意思，但不影响阅读，所以就没深究。
最近看到 List Monad 部分的时候，这个词的出现频率比以前高得多。这时候我才留意到，**non-determinism value** 之前也是在 List 相关的地方出现的。
那么，**non-determinism value** 到底指的是什么呢？


## determinism value 与 non-determinism value

在理解 **non-determinism value** 之前，我们先看看什么是 **determinism value**。

从字面意思推测，**determinism value** 是确定的值，那确定的值到底是什么呢？书中有这么一段话：

> A value like 5 is deterministic. It has only one result and we know exactly what it is. On the other hand, a value like [3,8,9] contains several results, so we can view it as one value that is actually many values at the same time.

像 5 这种只表示一种结果的值，它就是确定的值。
像 [3,8,9] 那样，它包含了多个值，我们可以把它看成是用一个值来表示多个结果，例如它可以表示 3，也可以表示 8，也可以表示 9，因此它是非确定值。

**determinism value** 和 **non-determinism value** 的区别就在这里。


## List Monad

那么 **non-determinism value** 到底有什么用呢？
我们知道了它可以表示多个结果，那么在动态计算的时候就很方便了，先看看下面这个简单的例子：

```hs
do
  x <- [1,2]
  y <- [4,5,6]
  return (x + y)
```

因为 `x` 可以 1 或者 2， `y` 可以表示 4 或者 5 或者 6，所以 `x+y` 就可以表示出 `1+4`, `1+5`, `1+6`, `2+4`, `2+5`, `2+6` 这么多结果了！


## 参考资料
http://stackoverflow.com/questions/29886852/why-is-the-following-haskell-code-non-deterministic
http://stackoverflow.com/questions/20638893/how-can-non-determinism-be-modeled-with-a-list-monad
http://learnyoua.haskell.sg/content/zh-cn/ch13/for-a-few-monads-more.html
