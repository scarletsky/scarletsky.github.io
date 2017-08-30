---
title: Emacs key bindings for vim users
date: 2017-08-30 21:12:59
category: technology
tags: [emacs]
---

## 简介
Emacs 是一个文本编辑器，号称是伪装成编辑器的操作系统。提起 Emacs，必定会联想到它的竞争对手 Vim，它们都是古老而强大的编辑器。虽然我已经用 Vim 好几年了，也听说过 Emacs 的各种强大的能力，但一直没有动力去学习它，直到最近看到强大的 org mode，才鼓起勇气去学习 Emacs。本文主要记录我学习 Emacs 的过程中与 Vim 的键位比较。

## 核心键位
Emacs 使用组合键的形式来触发不同的功能，最常见的是以下几个键位：

- C 即 Ctrl 键，如 `<C-x>` 表示 按着 Ctrl 再按 x
- S 即 Shift 键
- M 即 Meta 键，通常 Esc 键，如 `<M-x>` 表示先按下 Esc，然后松手，再按 x

## 常用键位绑定

### Edit

|             | Vim  | Emacs         |
|:------------|:-----|:--------------|
| 向上移动        | k    | C-p           |
| 向下移动        | j    | C-n           |
| 向左移动        | h    | C-b           |
| 向右移动        | l    | C-f           |
| 移动到行首       | 0    | C-a           |
| 移动到行尾       | $    | C-e           |
| 移动到行首非空字符   | ^    | M-m           |
| 移动到下一个单词    | w    | M-f           |
| 移动到上一个单词    | b    | M-b           |
| 移动到第10行     | 10gg | M-g g 10 RET  |
| 移动到下一页      | C-d  | C-v           |
| 移动到上一页      | C-u  | M-v           |
| 移动到文件首行     | gg   | M-<           |
| 移动到文件尾行     | G    | M->           |
| 把当前行移动到屏幕中央 | zz   | C-l           |
| 进入选择模式      | v    | C-SPC         |
| 删除光标中的字符    | x    | C-d           |
| 删除光标到行尾的字符  | d$   | C-k           |
| 删除光标所在行     | dd   | C-w 或 C-a C-k |
| 删除下10行      | 10dd | C-u 10 C-w    |
| 撤销上一次操作     | u    | C-/           |
| 重复上一次命令     | .    | C-z z         |
| 向前搜索        | /    | C-s           |
| 向后搜索        | ?    | C-r           |
| 搜索下一项       | n    | C-s           |
| 搜索上一项       | N    | C-r           |
| 开始录制宏       | qq   | F3            |
| 停止录制宏       | q    | F4            |
| 执行宏         | @q   | F4            |
| 执行宏10次      | 10@q | C-u 10 F4     |


### Window

|         | Vim         | Emacs |
|:--------|:------------|:------|
| 把窗口水平切分 | :new RET    | C-x _ |
| 把窗口垂直切分 | :vnew RET   | C-x 竖线(打出来会破坏布局，所以就不打了) |
| 切换窗口    | C-w j/k/h/l | C-x o |
| 关闭当前窗口  | :q RET      | C-x 0 |


### Buffer

|           | Vim      | Emacs          |
|:----------|:---------|:---------------|
| 创建 Buffer | :new RET | C-x b name RET |
| 列出 Buffer | :ls RET  | C-x b          |
| 删除 Buffer | :bd RET  | C-x k RET      |





## 参考资料
https://www.gnu.org/software/emacs/manual/
http://notex.life/t/emacs-key-binding/18

