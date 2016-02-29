---
title: FlexibleInstances 和 TypeSynonymInstances 编译指令的区别
date: 2016-02-28 22:31:21
tags: [haskell]
---


## FlexibleInstances

先看看下面这个简单的例子：

```hs
-- Learning.hs
data Vector a = Vector a a deriving (Show)

class MyClass a where
    myFun :: a -> a

instance MyClass (Vector a) where
    myFun = id
```

这样的定义看起来是没有问题的，因为不需要任何编译指令就能通过编译了。
我们可以运行看看：

```hs
ghci> :l Learning.hs
[1 of 1] Compiling Main             ( Test.hs, interpreted )
Ok, modules loaded: Main.
ghci> myFun (Vector 1 2)
Vector 1 2
ghci> myFun (Vector 1 2) :: Vector Int
Vector 1 2
ghci> myFun (Vector 1 2) :: Vector Double
Vector 1.0 2.0
```

但如果我们需要为 `Vector a` 不同的类型参数实现不同的 `myFun` 的话呢？

```hs
instance MyClass (Vector Int) where
    myFun = undefined

instance MyClass (Vector Double) where
    myFun = undefined
```

我们再看看编译时会发生什么？

```hs
ghci> :r
[1 of 1] Compiling Main             ( Learning.hs, interpreted )

Learning.hs:10:10:
    Illegal instance declaration for ‘MyClass (Vector Int)’
          (All instance types must be of the form (T a1 ... an)
           where a1 ... an are *distinct type variables*,
           and each type variable appears at most once in the instance head.
           Use FlexibleInstances if you want to disable this.)
    In the instance declaration for ‘MyClass (Vector Int)’
Failed, modules loaded: none.
```

编译失败！为什么会编译失败呢？

因为通常情况下，我们不能给多态类型（polymorphic type）的特化版本（specialized version）写类型类实例。 
在这个例子中，`Vector Int` 和 `Vector Double` 就是 `Vector a` 的特化版本。
如果我们需要为这些特化版本写类型类实例的话，我们就需要开启 `FlexibleInstances` 编译指令来取消这个限制。


## TypeSynonymInstances

理解了上面的 `FlexibleInstances` 后，`TypeSynonymInstances` 就容易理解了。

如果我需要为 `Vector Int` 添加一个别名，然后让这个别名成为 MyClass 类型类的实例，我们就会需要用到 `TypeSynonymInstances` 编译指令了。

```hs
type VectorInt = Vector Int

instance MyClass VectorInt where
    myFun = undefined
```

默认情况下，ghc 编译上面的代码时会报错，原因是 Haskell 98 并不支持这种语法。
要让 ghc 成功编译上面的代码，我们就需要开启 `TypeSynonymInstances` 这个编译指令了。


## 参考资料
http://rwh.readthedocs.org/en/latest/chp/6.html
http://book.realworldhaskell.org/read/using-typeclasses.html
