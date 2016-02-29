---
title: (译) Haskell 中随机数的使用
date: 2016-02-06 17:35:18
category: haskell
tags: [haskell, random]
---

随机数（我指的是伪随机数）是通过显式或隐式的状态来生成的。这意味着在 Haskell 中，随机数的使用（通过 `System.Random` 库）是伴随着状态的传递的。 

大部分需要获得帮助的人都有命令式编程的背景，因此，我会先用命令式的方式，然后再用函数式的方式来教大家在 Haskell 中使用随机数。

## 任务

我会生成满足以下条件的随机列表：

- 列表长度是 1 到 7 
- 列表中的每一项都是 0.0 到 1.0 之间的浮点数


## 命令式

在 IO monad 中有一个全局的生成器，你可以初始化它，然后获取随机数。下面有一些常用的函数：


### `setStdGen :: StdGen -> IO ()`

初始化或者设置全局生成器，我们可以用 `mkStdGen` 来生成随机种子。因此，有一个很傻瓜式的用法：

```hs
setStdGen (mkStdGen 42)
```

当然，你可以用任意的 `Int` 来替换 `42`。

其实，你可以选择是否调用 `setStdGen`，如果你不调用的话，全局的生成器还是可用的。因为在 runtime 会在启动的时候用一个任意的种子去初始化它，所以每次启动的时候，都会有一个不同的种子。


### `randomRIO :: (Random a) => (a,a) -> IO a`

在给定范围随机返回一个类型为 `a` 的值，同时全局生成器也会更新。你可以通过一个元组来指定范围。下面这个例子会返回 `a` 到 `z` 之间的随机值（包含 `a` 和 `z`）：

```hs
c <- randomRIO ('a', 'z')
```

`a` 可以是任意类型吗？并非如此。在 Haskell 98 标准中， `Random` 库只支持 `Bool`, `Char`,  `Int`, `Integer`, `Float`, `Double`（你可以自己去扩展这个支持的范围，但这是另外一个话题了）。


### `randomIO :: (Random a) => IO a`

返回一个类型为 `a` 的随机数（`a` 可以是任意类型吗？看上文），全局的生成器也会更新。下面这个例子会返回一个 `Double` 类型的随机数：

```hs
x <- randomIO :: IO Double
```
随机数返回的范围由类型决定。


需要注意的是，这些都是 IO 函数，因此你只可以在 IO 函数中使用它们。换句话说，如果你写了一个要使用它们的函数，它的返回类型也会变成是 IO 函数。

举个例子，上面提到的代码片段都要写在 `do block` 中。这只是一个提醒，因为我们想要用命令式的方式来生成随机数。

下面这个例子展示如何在 IO monad 中完成之前的任务：

```hs
import System.Random

main = do
    setStdGen (mkStdGen 42)  -- 这步是可选的，如果有这一步，你每一次运行的结果都是一样的，因为随机种子固定是 42
    s <- randomStuff
    print s

randomStuff :: IO [Float]
randomStuff = do
    n <- randomRIO (1, 7)
    sequence (replicate n (randomRIO (0, 1)))
```


## 纯函数式

你可能有以下原因想知道如何用函数式的方式生成随机数：

- 你有好奇心
- 你不想用 IO monad
- 因为一些并发或者其他原因，你想几个生成器同时存在，共享全局生成器不能解决你的问题

实际上，有两种方法来用函数式的方式去生成随机数：

- 从 stream（无限列表） 中提取随机数
- 把生成器当成函数参数的一部分，然后返回随机数

这里有一些常用的函数用来创建生成器和包含随机数的无限列表。


### `mkStdGen :: Int -> StdGen`

用随机种子创建生成器。


### `randomRs :: (Random a, RandomGen g) => (a, a) -> g -> [a]`

用生成器生成给定范围的无限列表。例子：用 `42` 作为随机种子，返回 `a` 到 `z` 之间包含 `a` 和 `z` 的无限列表：

```hs
randomRs ('a', 'z') (mkStdGen 42)
```

类型 `a` 是随机数的类型。类型 `g` 看起来是通用的，但实际上它总是 `StdGen`。


### `randoms :: (Random a, RandomGen g) => g -> [a]`

用给定的生成器生成随机数的无限列表。例如：用 `42` 作为随机种子生成 `Double` 类型的列表：

```hs
randoms (mkStdGen 42) :: [Double]
```

随机数的范围由类型决定，你需要查文档来确定具体范围，或者直接用 `randomRs`。

注意，这些都是函数式的 —— 意味着这里面没有副作用，特别是生成器并不会更新。如果你用一个生成器去生成第一个列表，然后用相同的生成器去生成第二个列表...

```hs
g = mkStdGen 42
a = randoms g :: [Double]
b = randoms g :: [Double]
```

猜猜结果，由于透明引用，这两个列表的结果是一样的！（如果你想用一个随机种子来生成两个不同的列表，我等下告诉你一个方法）。

下面一种方法来完成创建 `1` 到 `7` 的随机列表：

```hs
import System.Random

main = do
    let g   = mkStdGen 42
    let [s] = take 1 (randomStuff g)
    print s

randomStuff :: RandomGen g => g -> [[Float]]
randomStuff g = work (randomRs (0.0, 1.0) g)

work :: [Float] -> [[Float]]
work (r:rs)      =
    let n        = truncate (r * 7.0) + 1
        (xs, ys) = splitAt n rs
    in xs : work ys
```

除了必要的打印操作外，这是纯函数式的。它用生成器生成了无限列表，然后再用这个无限列表来生成另一个无限列表作为答案，最后取第一个作为返回值。

我这样做是因为尽管我们今天的人物是生成一个随机数，但你通常会需要很多个，我希望这个例子可以对你有点帮助。

上面的代码的工作原理是：用一个生成器，创建一个包含 `Float` 的无限列表。截取第一个值，并扩大这个值到 `1` 到 `7`，然后用剩下的列表来生成答案。换句话说，把输入的列表分成 `(r:rs)`，`r` 决定生成列表的长度（`1` 到 `7`），`rs` 之后会被计算答案。

### `split :: (RandomGen g) => g -> (g, g)`

用一个随机种子创建两个不同的生成器，其他情况下重用相同的种子是不明智的。

```hs
g = mkStdGen 42
(ga, gb) = split g
-- do not use g elsewhere
```

如果你想创建多余两个的生成器，你可以对新的生成器中的其中一个使用 `split`：

```hs
g = mkStdGen 42
(ga, g') = split g
(gb, gc) = split g'
-- do not use g, g' elsewhere
```

我们可以用 `split` 来获得两个生成器，这样我们就可以产生两个随机列表了。

```hs
randomStuff :: RandomGen g => g -> [[Float]]
randomStuff g = work (randomRs (1, 7) ga) (randomRs (0.0, 1.0) gb)
    where (ga,gb) = split g

work :: [Int] -> [Float] -> [[Float]]
work (n:ns) rs =
    let (xs,ys) = splitAt n rs
    in xs : work ns ys
```

它把生成器分成两个，然后产生两个列表。

我在主程序中硬编码了随机种子。正常情况下你可以在其他地方获取随机种子 —— 从输入中获取，从文件中获取，从时间上获取，或者从某些设备中获取。

这些在主程序中都是 do-able 的，因为它们都可以在 IO monad 中访问。

你也可以通过 `getStdGen` 获取全局生成器：

```hs
main = do
    g <- getStdGen
    let [s] = take randomStuff g
    print s
```


## 参考资料
[原文](http://www.vex.net/~trebla/haskell/random.xhtml)
