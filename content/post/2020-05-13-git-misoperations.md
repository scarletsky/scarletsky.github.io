---
title: Git 常见错误操作 (持续更新)
date: 2020-05-13T09:26:55+08:00
---

# 简介

本文是基于笔者的操作经验来记录 Git 的常见错误操作，会长期更新。（遇到再更新）


# Merge and Rebase

## 背景

```text
服务端 develop 分支：
                  HEAD
                   |
    a -> b -> c -> d


本地 develop 分支:
            HEAD
              |
    a -> b -> c

A 分支：
                      HEAD
                        |
    a -> b -> c -> e -> f

A 分支(更新后)：
                           HEAD
                             |
    a -> b -> c -> e -> f -> g
```


## 操作

```sh
# 当前在 develop 分支
# 1. merge A
git merge A

# 2. 由于本地与服务端有冲突导致 push 操作失败
git push

# 3. rebase
git pull --rebase

# 4. push again
git push

# 5. A 更新后，又要合并 A
git merge A
git push
```


## 结果

```text
本地 develop 分支:
                                            HEAD
                                              |
    a -> b -> c -> d -> e' -> f' -> e -> f -> g
```

e 和 e'，f 和 f' 实际上是同一个提交，但它们有不同的 commit hash。


## 分析

进行 `git pull --rebase` 操作后，e 和 f 在 develop 分支上已经变成了 e' 和 f'，它们实际上是同一个修改，但 commit hash 发生了改变。

```text

                             HEAD
                              |
    a -> b -> c -> d -> e' -> f'
```

这时候我们再合并 A，就自然变成了上面的结果了。


## 解决

- 避免重复合并同一个分支（master 合并 develop 分支除外）。
- 进行 merge 操作之前先同步服务端分支。
- 用 merge 代替 rebase。
