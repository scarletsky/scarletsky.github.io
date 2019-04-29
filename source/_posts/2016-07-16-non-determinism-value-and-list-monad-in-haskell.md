---
title: Non-deterministic value and List Monad in Haskell
date: 2016-07-16 16:01:39
category: haskell
tags: [haskell]
---

## 简介

看 《Learn you a haskell for a great good》 这本书的过程中，有时候会看到 **non-determinism** 这个词，虽然具体不知道什么意思，但不影响阅读，所以就没深究。
最近看到 List Monad 部分的时候，这个词的出现频率比以前高得多。这时候我才留意到，**non-determinism** 之前也是在 List 相关的地方出现的。
那么，**non-determinism** 到底指的是什么呢？


## deterministic value 与 non-deterministic value

在理解 **non-deterministic value** 之前，我们先看看什么是 **deterministic value**。

从字面意思推测，**deterministic value** 是确定的值，那确定的值到底是什么呢？书中有这么一段话：

> A value like 5 is deterministic. It has only one result and we know exactly what it is. On the other hand, a value like [3,8,9] contains several results, so we can view it as one value that is actually many values at the same time.

像 5 这种只表示一种结果的值，它就是确定的值。
像 [3,8,9] 那样，它包含了多个值，我们可以把它看成是用一个值来表示多个结果，例如它可以表示 3，也可以表示 8，也可以表示 9，因此它是非确定值。

**deterministic value** 和 **non-deterministic value** 的区别就在这里。

那么 **non-deterministic value** 到底有什么用呢？
我们知道了它可以表示多个结果，那么在动态计算的时候就很方便了，先看看下面这个简单的例子：

```hs
do
  x <- [1,2]
  y <- [4,5,6]
  return (x + y)
```

因为 `x` 可以 1 或者 2， `y` 可以表示 4 或者 5 或者 6，所以 `x+y` 就可以表示出 `1+4`, `1+5`, `1+6`, `2+4`, `2+5`, `2+6` 这么多结果了！


## List Monad

理解了 **non-deterministic value** 之后，我们再来看看 List Monad。
我们知道，不同的 Monad 带有不同的 context，如 Maybe Monad 带有可能失败的 context，那 List Monad 带有怎样的 context 呢？
答案就是 **non-determinism**。

先看看 List Monad 的定义：

```hs
instance Monad [] where
  return x = [x]
  xs >>= f = concat (map f xs)
  fail _ []
```

我们主要看看 `>>=` 的实现，它把 f 应用到 List 中的每个元素中，然后再把结果扁平化而已。
虽然看起来很简单，但用起来很强大：

```hs
-- 1
ghci> [3,4,5] >>= \x -> [x,-x]  
[3,-3,4,-4,5,-5]

-- 2
ghci> [1,2,3] >>= \x -> [x+1] >>= \y -> [y*2]
[4,6,8]

-- 3
ghci> [1,2] >>= \n -> ['a','b'] >>= \ch -> return (n,ch)
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]

-- 4
ghci> [ (n, ch) | n <- [1,2], ch <- ['a', 'b'] ]
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

留意最后两个例子，事实上列表推导 (List Comprehension) 只是 List Monad 的一种语法糖而已。

最后提一下一个很炫的技巧：用 `filterM` 来生成一个列表的幂集 (Power Set)，即该列表中的所有子列表，如：

```hs
-- [1,2,3] 的幂集就是
[1,2,3]  
[1,2]  
[1,3]  
[1]  
[2,3]  
[2]  
[3]  
[]
```

利用 `filterM` 和 List Monad 可以非常轻松地实现幂集：

```hs
import Control.Monad

powerset :: [a] -> [[a]]
powerset xs = filterM (\x -> [ True, False ]) xs

-- in ghci
ghci> powerset [1,2,3]
[[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]
```

很神奇吧！想知道为什么的话，去看看下面的链接吧：
http://stackoverflow.com/questions/28872396/haskells-filterm-with-filterm-x-true-false-1-2-3 


## 参考资料
http://stackoverflow.com/questions/29886852/why-is-the-following-haskell-code-non-deterministic
http://stackoverflow.com/questions/20638893/how-can-non-determinism-be-modeled-with-a-list-monad
http://learnyoua.haskell.sg/content/zh-cn/ch12/a-fistful-of-monads.html
http://learnyoua.haskell.sg/content/zh-cn/ch13/for-a-few-monads-more.html
