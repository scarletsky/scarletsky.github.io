---
title: What is applicative in haskell ?
date: 2016-03-07 11:52:28
categories: haskell
tags: [haskell, applicative]
---

## 初探

什么是 `Applicative` ?

> Applicative 是加强版的 Functor，是一个类型类。

加强版的 `Functor` 是什么意思 ?

> 还记得 Functor 的限制吗？
> `fmap f x` 中的 `f` 只接受一个参数。
> `fmap f x` 中的 `f` 不能带有上下文 (换句话说只能是 `(+42)` 不能是 `Just (+42)`)。
> 所谓的加强版的 Functor 就没有这些限制。

这样有什么用 ?

> 我们先看看 Applicative 的声明吧:
> ```hs
class (Functor f) => Applicative f where
    pure :: a -> f a
    <*> :: f (a -> b) -> f a -> b
```
> 你能从 Applicative 的声明中看出什么吗？

`Applicative` 是 `Functor` ?

> 是的，所谓加强版的 Functor，它首先就是一个 Functor，外加一些额外的功能嘛。

`pure` 有什么用 ?

> 把一个值变成一个 Applicative Functor 。

`<*>` 呢 ?

> 和 fmap (也就是 <$>) 一起看吧。
> ```hs
<$> ::   (a -> b) -> f a -> f b
<*> :: f (a -> b) -> f a -> f b
```

`<$>` 传进的是普通的函数，而 `<*>` 传进的是一个包装过的函数 ?

> 是的，这就是 Applicative 和 Functor 的区别。

有没有简单易懂的例子 ?

> 先看看下面几个例子吧。
> ```hs
ghci> (+33) <$> Just 9
Just 42
ghci> Just (+33) <*> Just 9
Just 42
ghci> pure (+33) <*> Just 9
Just 42
```
> 能看出什么吗？

包装过的函数和普通函数一样，都可以接受包装过的值作为参数，并返回正确的结果？

> 正确！
> 我们再看看下面两个例子:
> ```hs
ghci> Just (+) <*> Just 3 <*> Just 8
Just 11
ghci> pure (+) <*> Just 3 <*> Just 8
Just 11
```
> 能看出什么吗 ?

包装过的函数可以接受多个参数了 ?

> 是的！
> Applicative 就是通过接受一个带有上下文的函数来突破 Functor 的限制的。
> 我们先凭直觉来用一下 Applicative 吧！


## 小试

`Just (++) <*> Just "Hello" <*> Just " World"` 的结果是 ?

> Just "Hello World"

`(++) <$> Just "Hello " <*> Just "Haskell~"` 的结果是 ?

> Just "Hello Haskell~"

`(++) <$> Just 42 <*> Nothing` 的结果是 ?

> Nothing

`[(*0), (+100), (^2)] <*> [1, 2, 3]` 的结果是 ?

> [0,0,0,101,102,103,1,4,9]

为什么不是 `[0, 102, 9]` ?

> 因为列表是 Applicative 的实例，它实现了自己的 <*> 方法。

所以其实 `Maybe` 也是 `Applicative` 的实例 ?

> 是的。

## 理解

`Maybe` 是怎么实现 `<*>` 的 ?

> 如下所示:
> ```hs
instance Applicative Maybe where
    pure          = Just
    Just f  <*> m = fmap f m
    Nothing <*> _ = Nothing
```
> 现在可以试试理解 <*> 是如何应用在 Maybe 中的。

`Just (+) <*> Just 3` 的结果是 ?

> Just (+3)

所以 `Just (+) <*> Just 3 <*> Just 8` 是 `Just 11` ?

> 是的！

那列表是怎么实现 `<*>` 的 ?

> 如下所示:
> ```hs
instance Applicative [] where
    pure x = [x]
    fs <*> xs = [f x | f <- fs, x <- xs]
```
> 列表是通过「列表解析(List Comprehension)」来实现 `<*>` 的。
> 其中的 fs 可以看成是一个由函数组成的列表，如 [(+1), (*2), (^3)]
> 整个过程可以看作 `concatMap (\f -> map f xs) fs` 的另一个写法。

还有哪些是 `Applicative` 的实例 ?

> `IO`, `(->) r`

`IO` 怎么实现 `<*>` ?

> 看看下面的代码:
> ```hs
instance Applicative IO where
    pure = return
    a <*> b = do
        f <- a
        x <- b
        return (f x)
```

不太懂，有没有具体的例子 ?

> 下面有个简单易懂的例子:
> ```hs
myAction :: IO String
myAction = do
    a <- getLine
    b <- getLine
    return $ a ++ b
```
> 其实我们可以可以写成下面这样:
> `myAction = (++) <$> getLine <*> getLine`

那 `(->) r` 又是什么 ?

> `(->) r` 其实是函数，如果语法允许的话，我们写成 `r ->` 也可以，但是语法只允许 `(->) r` 这种写法。

`(->) r` 是怎么实现 `<*>` 的 ?

> 看看下面的代码:
> ```hs
instance Applicative ((->) r) where
    pure x = (\_ -> x)
    f <*> g = \x -> f x (g x)
```

`(->) r` 的 `<*>` 实现不是太好理解。

> 确实不太好理解，它的意思是给 `<*>` 两个函数，然后生成一个新的函数。

有具体例子吗？

> 这里要先留意一下 pure 的实现:
> `pure x = (\_ -> x)`
> 它会包装传进的函数，返回一个新的函数。这个新函数会无视第一个传进的参数，然后返回原来的函数。 

看起来很绕，可以给一些例子吗 ?

> 是的，直接看例子会好理解一点:
> ```hs
ghci> :t pure (+3)
pure (+3) :: f (a -> a)

ghci> :t pure (+3) 2
pure (+3) 2 :: a -> a

ghci> pure (+3) 2 4
7
```
> 看到了吗 ?
> `pure (+3)` 返回的是 `(\_ -> (+3))`。
> `pure (+3) 2` 返回的是 `(+3)`。
> `pure (+3) 2 4` 其实是 `(+3) 4` 的结果。

OK，我已经了解 `pure` 是如何处理函数的了。
 
> 好，接下来我们看 `<*>` 的实现。
> 看看下面的例子，我们按照定义一步一步来~
> ```hs
-- 例子1
    pure (+3) <*> (*100)
-- 展开 pure (+3)
= (\_ -> (+3)) <*> (*100)
-- 展开 <*>
= \x -> (\_ -> (+3)) x (*100 x)
-- 当 x = 5 时
= \5 -> (\_ -> (+3)) 5 (*100 5)
-- 传进 5 之后的返回值如下
= (\_ -> (+3)) 5 500
-- 无论第一个参数是什么，它都是返回 (+3)
= (+3) 500
= 503

-- 例子2
    (+) <$> (+3) <*> (*100)
-- 展开 <$>
= (+) . (+3) <*> (*100)
-- 展开 <*>
= \x -> (+) . (+3) x (*100 x)
-- 当 x = 5 时
= \5 -> (+) . (+3) 5 (*100 5)
-- 返回值
= (+) . (+3) 5 500
= (8+) 500
= 508

-- 例子3
    (\x y z -> [x,y,z]) <$> (+3) <*> (*2) <*> (/2)
-- 先假设 (\x y z -> [x,y,z]) 为 f
= f <$> (+3) <*> (*2) <*> (/2)
-- 展开 <$>
= f . (+3) <*> (*2) <*> (/2)
-- 展开第一个 <*>
= \x -> (f . (+3)) x (*2 x) <*> (/2)
-- 展开第二个 <*>
= \y -> (\x -> (f . (+3)) x (*2 x)) y (/2 y)
-- 观察 (\x -> (f . (+3)) x (*2 x)) y
-- 把 y 传入函数中可以得到
= \y -> (f . (+3)) y (*2 y) (/2 y)
= \y -> f (+3 y) (*2 y) (/2 y)
-- 当传入 5 作为参数时
= f (+3 5) (*2 5) (/2 5)
= f 8 10 2.5
-- 把 f 换回 (\x y z -> [x,y,z])
= [8.0,10.0,2.5]
```
> 很有趣吧 !

每次都要展开 `<*>` 很麻烦啊 ?

> 从刚才的例子中，你应该可以发现，我们从直觉上就可以知道计算过程。
> 试试化简一下 `f <$> g <*> h` ?

`f <$> g <*> h` = `(\x -> f (g x) (h x))`。

> 正确，再把 5 传入看看 ?

`f <$> g <*> h $ 5` = `f (g 5) (h 5)`。

> 假设 f 是一个需要两个参数的函数的话，就能返回计算结果了 !

所以其实我们可以把 `f <$> g <*> h` 看成是把 `g` 和 `h` 的结果传给 `f` ?

> 是的，我们从直觉上就可以知道计算过程，很有趣吧 !
> 顺便说下, 像 `f <$> g <*> h` 这样的用法叫做 Applicative Style 哦 !

`Applicative` 有没有和 `Functor` 一样需要遵守的规则 ?

> 有的！其中最重要的规则是:
> 必须保证 `pure f <*> x = fmap f x`。

我应该怎么理解这个规则 ?

> 我们说 `Applicative` 是加强版的 `Functor`，它可以应用一个带有上下文的函数到 `Functor` 上。
> 我们看看之前提到的 `Applicative` 实例 :
> 对于 `Maybe` :
> ```hs
  pure f <*> x
= Just f <*> x
= fmap f x
```
> 对于 `List` :
> ```hs
  pure f <*> x
= [f] <*> x
= [f x' | x' <- x]
= map f x
= fmap f x
```
> 对于 `IO` :
> ```hs
  pure f <*> x
= return f <*> x
= return (f x)
= fmap f x
```
> 对于 `(->) r` :
> ```hs
  pure f <*> x
= (\_ -> f) <*> x
= (\y -> (\_ -> f) y (x y))
= (\y -> f (x y))
= (\y -> (f . x) y)
= f . x
= fmap f x
```
> 看到了吧 ! 我们上面提到的 `Applicative` 的实例全部都满足 `pure f <*> x = fmap f x` 哦 !

还有其他需要遵守的规则吗 ?

> 有的，分别如下:
> - `pure id <*> v = v`
> - `pure (.) <*> u <*> v <*> w = u <*> (v <*> w)`
> - `pure f <*> pure x = pure (f x)`
> - `u <*> pure y = pure ($ y) <*> u`
> 这些规则都是可以被证明的，你可以试一试。

还有什么我需要知道的吗 ?

> 忘了说，`Applicative` 需要导入才可以用，它位于 `Control.Applicative` 下。
> 想用的时候记得先写 `import Control.Applicative` 哦 !

## 总结

- `Applicative` 是加强版的 `Functor`。
- `Applicative` 的实例可以使用 `pure` 和 `<*>`。
- 我们可以用 `<$>` 和 `<*>` 将一个普通函数应用到任意数量的 `Applicative Functor` 上。


## 参考资料

[Functors, Applicative Functors and Monoids](http://learnyouahaskell.com/functors-applicative-functors-and-monoids)
