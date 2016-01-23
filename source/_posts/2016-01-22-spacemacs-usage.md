---
title: Spacemacs 使用总结
date: 2016-01-22 15:20:38
tags: [emacs, spacemacs]
---

## 简介

Spacemacs 是一份 emacs 的配置文件，想要使用它，你先要有 emacs。

## 安装 & 使用

```bash
$ git clone https://github.com/syl20bnr/spacemacs ~/.emacs.d
$ emacs
```

## 配置文件

Spacemacs 的配置文件位于 `~/.spacemacs` 中，我们只需要修改这个文件就可以制定自己的配置了。

一般情况下，我们只需要在 `dotspacemacs-configuration-layers` 中添加自己需要的 layer 就可以了。

## 常用快捷键

#### 配置文件管理
`SPC f e d` 快速打开配置文件 `.spacemacs`
`SPC f e R` 同步配置文件

#### 文件管理

`SPC f f` 打开文件（夹），相当于 `$ open xxx` 或 `$ cd /path/to/project`
`SPC p f` 搜索文件名，相当于 ST / Atom 中的 `Ctrl + p`
`SPC s a p` 搜索内容，相当于 `$ ag xxx` 或 ST / Atom 中的 `Ctrl + Shift + f`

`SPC b k` 关闭当前 buffer
`SPC SPC` 搜索当前文件 

#### 窗口管理
`SPC f t` 打开/关闭侧边栏，相当于 ST / Atom 中的 `Ctrl(cmd) + k + b`

`SPC 0` 光标跳转到侧边栏（NeoTree）中
`SPC n(数字)` 光标跳转到第 n 个 buffer 中

`SPC w s | SPC w -` 水平分割窗口
`SPC w v | SPC w /` 垂直分割窗口
`SPC w c` 关闭当前窗口

#### 对齐
`SPC j =` 自动对齐，相当于 beautify

#### Shell 集成 (必须先配置 Shell layer)
`SPC '(单引号)` 打开/关闭 Shell
`C-k` 前一条 shell 命令，相当于在 shell 中按上箭头
`C-j` 后一条 shell 命令，相当于在 shell 中按下箭头

## 让 Spacemacs 支持 EditorConfig

EditorConfig 是一个配置文件，一般位于项目的根目录，它可以让不同的编辑器和IDE 都按照相同的格式来格式化代码，对于项目的维护者来说是一个很好的工具。

Spacemacs 也支持 EditorConfig，只需要在配置文件中添加配置即可。下面以 OS X 为例，通过以下步骤即可让 Spacemacs 支持 EditorConfig：

1. `$ brew install editorconfig`
2. 在 `~/.spacemacs` 中的 `dotspacemacs-additional-packages` 中添加 `editorconfig`：
```
dotspacemacs-additional-packages
 '(
   editorconfig
   )
```
3. 创建 .editorconfig 文件，写上自己喜欢的配置。
4. 完。


## 设置文件默认的主模式

虽然我们可以通过 `M-x` 来设置文件的主模式，但这种方式只是在单独修改某个文件的主模式时好用，如果要把所有同类型的文件都改成其他模式，这种方式的效率就太低了。

在 Spacemacs 中，我们可以用 `auto-mode-alist` 来设置某一类文件默认的主模式。

我们只需要在 `~/.spacemacs` 中的 `user-config` 中加入下面代码即可：

```elisp
(add-to-list 'auto-mode-alist '("\\.js\\'" . react-mode))
```

上面代码会用 `react-mode` 打开所有 `.js` 文件。


## Emacs 服务器

Spacemacs 会在启动时启动服务器，这个服务器会在 Spacemacs 关闭的时候被杀掉。

### 使用 Emacs 服务器

当 Emacs 服务器启动的时候，我们可以在命令行中使用 `emacsclient` 命令：

- `$ emacsclient -c` 用 Emacs GUI 来打开文件
- `$ emacsclient -t` 用命令行中 Emacs 来打开文件

### 杀掉 Emacs 服务器

除了关闭 Spacemacs 之外，我们还可以用下面的命令来杀掉 Emacs 服务器：

- `$ emacsclient -e '(kill-emacs)'`

### 持久化 Emacs 服务器

我们可以持久化 Emacs 服务器，在 Emacs 关闭的时候，服务器不被杀掉。只要设置 `~/.spacemacs` 中 `dotspacemacs-persistent-server` 为 `t` 即可。

但这种情况下，我们只可以通过以下方式来杀掉服务器了：

- `SPC q q` 退出 Emacs 并杀掉服务器，会对已修改的 Buffer 给出保存的提示。
- `SPC q Q` 同上，但会丢失所有未保存的修改。


## 参考资料

[https://github.com/syl20bnr/spacemacs/blob/master/doc/DOCUMENTATION.org](https://github.com/syl20bnr/spacemacs/blob/master/doc/DOCUMENTATION.org)
[http://brannonlucas.com/using-editorconfig-and-spacemacs-on-os-x/](http://brannonlucas.com/using-editorconfig-and-spacemacs-on-os-x/)