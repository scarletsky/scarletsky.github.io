---
title: What is functor in Haskell ?
date: 2016-02-09 22:50:13
category: haskell
tags: ['haskell', 'functor']
---

## Functor 简介

什么是 Functor ?

> 现在你可以认为 Functor 是一种数据类型。

Functor 有什么用 ?

> 我们可以对 Functor 使用 `fmap`。

`fmap` 是什么东西 ?

> `fmap` 是一个函数。

`fmap` 是函数的话，那它的类型签名是什么 ?

> `fmap :: (a -> b) -> f a -> f b`。

我应该怎么看这个类型签名 ?

> 它接受一个函数和一个 Functor 类型作为参数，然后返回另一个 Functor 。

`fmap` 有什么用 ?

> 类似于 `map`
> `map (+1) [1,2,3,4,5]  -- 返回 [2,3,4,5,6]`
> `fmap (+1) [1,2,3,4,5] -- 返回 [2,3,4,5,6]`

所以列表是 Functor ?

> 是的，List 是 Functor。

为什么列表是 Functor ?

> 因为列表实现了 `fmap`。
>
```hs
instance Functor [] where
  fmap = map
```

实现了 `fmap` 的数据类型都是 Functor ?

> 不一定。

为什么 ?

> 除了要实现 `fmap` 之外，还需要满足一些条件才能成为 Functor。

满足什么条件 ?

> 1. 必须保证 `fmap id = id`，也就是说 `fmap id xs` 和 `id xs` 必须返回相同的值。
> 2. 必须是可组合的，两个 `fmap` 组合使用的结果应该和两个函数组合起来再用 `fmap` 的结果相同。
>    也就是说 `fmap f . fmap g` 必须等于 `fmap (f . g)`。

为什么 `fmap id = id` ?

> 因为
> `id :: a -> a`
> `fmap id :: T(a) -> T(a)`
> 令 `T(a) = a`
> 即 `fmap id :: a -> a`
> 所以 `fmap id = id`

所以条件一是什么意思 ?

> 意思是 `fmap` 只能对值调用 `f`，不能做额外的事情。

有具体例子吗 ?

> 看看这个经典的自定义数据类型，C表示计数器：
>
```hs
data CMaybe a = CNothing | CJust Int a deriving (Show)

instance Functor CMaybe where
  fmap f CNothing          = CNothing
  fmap f (CJust counter x) = CJust (counter + 1) (f x)

-- ghci

ghci> fmap (++ "ha") (CJust 0 "ho")
CJust 1 "hoha"
ghci> fmap (++ "he") (fmap (++ "ha") (CJust 0 "ho"))
CJust 2 "hohahe"
ghci> fmap (++ "blah") CNothing
CNothing
```
> 这里的 `fmap` 除了对值调用 `f` 之外，还对 `counter` 加一。


这有什么问题吗 ?

> 再看看 `fmap id` 和 `id`
```hs
ghci> fmap id (CJust 0 "haha")
CJust 1 "haha"
ghci> id (Cjust 0 "haha")
CJust 0 "haha"
```
> 看出问题了吗 ?

`fmap id` 和 `id` 返回的结果不相等 ?

> 是的，所以即便 `CMaybe a` 实现了 `fmap`，但它也不是 Functor。

为什么 `fmap (f . g) = fmap f . fmap g` ?

> 假设
> `f :: a -> b`, `g :: b -> c`
> 那么
> `f . g :: a -> c`
> 即
> `fmap (f . g) = T(a) -> T(c)`
> 又因为
> `fmap f = T(a) -> T(b)`, `fmap g = T(b) -> T(c)`
> 所以
> `fmap f . fmap g = T(a) -> T(c)`
> 即 `fmap (f . g) = fmap f . fmap g`


条件二有点像乘法分配律。

> 是的。
> 乘法分配律是 `(a + b) x c = a x c + b x c`。
> 而条件二是 `fmap (f . g) = fmap f . fmap g`。

条件二有具体例子吗 ?

> 可以类比函数，因为函数本身也是 Functor，所以函数会满足**可组合**这个条件。
> 而实际应用中，我们也经常使用到函数组合这个特性。

实现了 `fmap` ，同时满足两个条件的数据类型就是 Functor 吗？

> 不，还有一个规则，就是该数据类型要有一个类型参数。

能举个例子吗 ?

> 我们已经知道 List 是一个 Functor，先看看 List 的定义：
> ```hs
data [] a = [] | a : [a]
```
> 列表有一个类型参数 a，表示一个列表中可以包含相同类型的元素。

Functor 只能有一个类型参数吗？

> 不是，我们可以通过其他方法让多于一个类型参数的数据类型都能成为 Functor 的实例。

什么手段 ?

> 你需要先知道怎么定义一个 Functor。

## 自定义 Functor

我应该怎么自定义 Functor ?

> 先定义一个数据类型，再让该类型成为 Functor 的实例。
> ```hs
data MyFunctor a = Data a deriving (Show)

instance Functor MyFunctor where
  fmap f (Data x) = Data (f x)
```
> 这样，我们定义的 MyFunctor 就是一个 Functor 了。

刚才提到的让多于一个类型参数的数据类型成为 Functor 实例的方法是？

> 利用 Haskell 中不全调用的特性。

可以给个例子吗？

> ```hs
data MyFunctor2 a b = Data2 a b deriving (Show)

instance Functor (MyFunctor2 a) where
  fmap f (Data2 x y) = Data2 x (f y)
```
> 在 Haskell 中，我们可以利用 Haskell 不全调用的特性，把 MyFunctor2 a 当成一个整体，这样就相当于只有 b 一个类型参数了。


## 真 · Functor

我从上面看到，Functor 是一个类型类？

> 是的。事实上，**Functor 是一个类型类，表示满足一些条件的数据类型。**

满足上面提到的条件？

> 是的！

有哪些常见的 Functor ？

> `List`, `Maybe`等等。
> 你可以在 ghci 中输入 `:i Functor` 来查看更多预定义的 Functor。

这些 Functor 有什么特点？

> 它们都带有上下文：即可以表示有值，也可以表示空值。
> [] 表示空值，[a] 表示有值；
> Nothing 表示空值，Just a表示有值；

这样有什么好处吗？

> 好处是显然易见的。考虑下下面的伪代码：
> ```py
post = Posts.find_by_id(1)
if post
  return post.title
else
  return None
```
> 为什么这段伪代码需要判断 post 是否为空？因为 post 没有上下文环境，不能表示空值。
> 如果 post 有上下文环境 (也就是 post 可以表示空值)，那么我们的代码就可以直接写成：
> ```py
post = Posts.find_by_id(1)
return post.title
```
> 因此，如果一个值可以带有上下文环境的话，我们的代码就可以写的非常简洁。

把刚才的伪代码写成 Haskell 代码 ?

> `fmap (getPostTitle) (findPosts 1)`

`if else` 不见了？

> 是的，这里假设 post 是一个 Functor，它可以表示带有空值的情况。所以 `if else` 就不需要了。

那 `fmap` 呢？ 它事实上是什么东西？

> `fmap` 确确实实是一个函数，它知道怎么把传进的函数应用到 Functor 中，并返回一个新的 Functor。

`fmap` 对 Functor 调用函数的过程发生了什么？

> 看下面两张图 (图出自 [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html))：
> ![fmap_just](https://cloud.githubusercontent.com/assets/2386165/12949440/c8753e16-d042-11e5-84ff-b47753d65c52.png)
> ![fmap_nothing](https://cloud.githubusercontent.com/assets/2386165/12949441/c87a69b8-d042-11e5-80d6-2ee458e13e37.png)
> 实际上，`fmap` 先取出 Functor 中的值，然后把值传进函数中，再把函数的返回值放回到 Functor 中，最后返回新的 Functor。


Functor 有什么限制？

> `fmap f x` 中的 `f` 只接受一个参数。
> `fmap f x` 中的 `f` 不能带有上下文 (换句话说只能是 (+42) 不能是 `Just (+42)`)。


关于 Functor 的知识，还有什么我是需要知道的 ？

> `fmap` 可以中缀调用，即 ```f `fmap` xs```
> `<$>` 是 `fmap` 的别名，一般用于中缀调用，即 `f <$> xs`。


## 总结

Functor 是类型类，只要满足以下条件的数据类型都可以成为 Functor 的实例：

- 实现 `fmap`。
- 保证 `fmap id = id`。
- 保证 `fmap (f . g) = fmap f . fmap g`。
- 该数据类型必须有一个以上的类型参数。

最后，强烈建议看看 [这篇文章](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)，相当形象生动。

## 特别感谢

[@bramblex](https://github.com/bramblex) 给出 `fmap id = id` 和 `fmap (f . g) = fmap f . fmap g` 的证明过程。

## 参考资料

[Functors, Applicative Functors and Monoids](http://learnyouahaskell.com/functors-applicative-functors-and-monoids)
[Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
[Functor 简介](http://cnhaskell.com/chp/10.html#functor)
http://stackoverflow.com/questions/2030863/in-functional-programming-what-is-a-functor
