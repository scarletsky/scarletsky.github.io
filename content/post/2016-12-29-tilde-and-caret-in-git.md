---
title: Git 中的 ~ 和 ^
date: 2016-12-29 21:37:31
categories: [tools]
tags: [git]
---

## 简介

在使用 Git 的时候，我们经常会看见 `~` 和 `^`，如 `HEAD~2`, `HEAD^3` 等。
初学者经常会对这些符号感到疑惑，本文将讨论这两个符号的用途。

## 为何需要 ~ 和 ^

不知道大家有没体会到，我们经常需要根据一个提交去查找它的祖先提交，如查找 `HEAD` 的第三个祖先提交。
要找到对应的提交，我们可以直接通过 `git log`，然后手动选中第三个提交。

```bash
$ git log --graph --oneline
* a19bf31 D
* 85ce81b C
* 73d1f3b B
* 078e0e6 A
...
```

然后我们选中 `078e0e6 (A)` 这个提交，接着进行余下的操作。

这种方式虽然可以实现我们的需求，如果我们想要 `HEAD` 的第 10 个祖先提交呢？
那是要把 log 打印出来，然后一条一条慢慢找吗？这样的话就太低效了。

我们需要有一种方式，根据一个提交快速找到它的祖先提交，因此，我们就需要 `~` 和 `^` 这两个符号拉。


## ~ 的作用

如果我们想要 `HEAD` 的第 10 个祖先提交，我们直接用 `HEAD~10` 就可以了。
`<rev>~<n>` 用来表示一个提交的第 n 个祖先提交，如果不指定 n，那么默认为 1。
另外，`HEAD~~~` 和 `HEAD~3` 是等价的。

```bash
$ git rev-parse HEAD
a19bf31 (D)

$ git rev-parse HEAD~0
a19bf31 (D)

$ git rev-parse HEAD~
85ce81b (C)

$ git rev-parse HEAD~1
85ce81b (C)

$ git rev-parse HEAD~~
73d1f3b (B)

$ git rev-parse HEAD~3
078e0e6 (A)
```



## ^ 的作用

先看看下面这幅图：

```bash
$ git log --graph --oneline

* f44239d D
*   7a3fb3d C
|\
| * 07b920c B
|/
* 71bd2cf A
...
```

我们知道，很多情况下一个提交并不是只有一个父提交。
就如上图表示那样，`7a3fb3d (C)` 就有两个父提交：

- `07b920c (B)`
- `71bd2cf (A)`。

这时候，我们是不能通过 `~` 去找到 `07b920c (B)` 这个提交的。
如果一个提交有多个父提交，那么 `~` 只会找第一个父提交。
那么我们应该怎么找到 `07b920c (B)` 呢？
答案是：`HEAD~^2`

`<rev>^<n>` 用来表示一个提交的第 n 个父提交，如果不指定 n，那么默认为 1。
和 `~` 不同的是，`HEAD^^^` 并不等价于 `HEAD^3`，而是等价与 `HEAD^1^1^1`。

```bash
$ git rev-parse HEAD~
7a3fb3d (C)

$ git rev-parse HEAD~^
71bd2cf (A)

$ git rev-parse HEAD~^0
7a3fb3d (C)

$ git rev-parse HEAD~^2
07b920c (B)

$ git rev-parse HEAD~^3
fatal: ambiguous argument 'HEAD~^3': unknown revision or path not in the working tree.

$ git rev-parse HEAD^2
fatal: ambiguous argument 'HEAD^2': unknown revision or path not in the working tree.
```


## ~ 与 ^ 的关系

我们知道，`~` 获取第一个祖先提交，`^` 可以获取第一个父提交。
其实第一个祖先提交就是第一个父提交，反之亦然。
因此，当 n 为 1 时，`~` 和 `^` 其实是等价的。
譬如：`HEAD~~~` 和 `HEAD^^^` 是等价的。

最后，引用 [kernel.org](https://www.kernel.org/pub/software/scm/git/docs/git-rev-parse.html) 中非常形象的一段话：

>Here is an illustration, by Jon Loeliger. Both commit nodes B and C are parents of commit node A. Parent commits are ordered left-to-right.

```
G   H   I   J
 \ /     \ /
  D   E   F
   \  |  / \
    \ | /   |
     \|/    |
      B     C
       \   /
        \ /
         A
A =      = A^0
B = A^   = A^1     = A~1
C = A^2  = A^2
D = A^^  = A^1^1   = A~2
E = B^2  = A^^2
F = B^3  = A^^3
G = A^^^ = A^1^1^1 = A~3
H = D^2  = B^^2    = A^^^2  = A~2^2
I = F^   = B^3^    = A^^3^
J = F^2  = B^3^2   = A^^3^2
```

## 参考资料

- http://stackoverflow.com/questions/2221658/whats-the-difference-between-head-and-head-in-git
- https://www.kernel.org/pub/software/scm/git/docs/git-rev-parse.html
