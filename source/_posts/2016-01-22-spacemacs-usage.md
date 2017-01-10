---
title: Spacemacs 使用总结
date: 2016-01-22 15:20:38
category: emacs
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
`SPC q R` 重启 emacs


#### 帮助文档
`SPC h d` 查看 describe 相关的文档
`SPC h d f` 查看指定函数的帮助文档
`SPC h d b` 查看指定快捷键绑定了什么命令
`SPC h d v` 查看指定变量的帮助文档


#### 文件管理
`SPC f f` 打开文件（夹），相当于 `$ open xxx` 或 `$ cd /path/to/project`
`SPC /` 用合适的搜索工具搜索内容，相当于 `$ grep/ack/ag/pt xxx` 或 ST / Atom 中的 `Ctrl + Shift + f`
`SPC s c` 清除搜索高亮
`SPC f R` 重命名当前文件

`SPC b k` 关闭当前 buffer (spacemacs 0.1xx 以前)
`SPC b d` 关闭当前 buffer (spacemacs 0.1xx 以后)
`SPC SPC` 搜索当前文件 


#### 窗口管理
`SPC f t 或 SPC p t` 用 NeoTree 打开/关闭侧边栏，相当于 ST / Atom 中的 `Ctrl(cmd) + k + b`
`SPC f t` 打开当前文件所在的目录
`SPC p t` 打开当前文件所在的**根**目录

`SPC 0` 光标跳转到侧边栏（NeoTree）中
`SPC n(数字)` 光标跳转到第 n 个 buffer 中

`SPC w s 或 SPC w -` 水平分割窗口
`SPC w v 或 SPC w /` 垂直分割窗口
`SPC w c` 关闭当前窗口 (spacemacs 0.1xx 以前)
`SPC w d` 关闭当前窗口 (spacemacs 0.1xx 以后)


#### 项目管理
`SPC p p` 切换项目
`SPC p D` 在 dired 中打开项目根目录
`SPC p f` 在项目中搜索文件名，相当于 ST / Atom 中的 `Ctrl + p`
`SPC p R` 在项目中替换字符串，根据提示输入「匹配」和「替换」的字符串，然后输入替换的方式：
- `E` 修改刚才输入的「替换」字符串
- `RET` 表示不做处理
- `y` 表示只替换一处
- `Y` 表示替换全部
- `n` 或 `delete` 表示跳过当前匹配项，匹配下一项
- `^` 表示跳过当前匹配项，匹配上一项
- `,` 表示替换当前项，但不移动光标，可和 `n` 或 `^` 配合使用


#### 对齐
`SPC j =` 自动对齐，相当于 beautify


#### Shell 集成 (必须先配置 Shell layer)
`SPC '(单引号)` 打开/关闭 Shell
`C-k` 前一条 shell 命令，相当于在 shell 中按上箭头
`C-j` 后一条 shell 命令，相当于在 shell 中按下箭头


#### 快速翻页 (在 spacemacs 0.1xx 中没测试过)
`SPC n , 或 . 或 < 或 >` 进入 `scrolling transient state`
然后重复按 `,` 或 `.` 或 `<` 或 `>` 即可，
按其他键会退出 `scrolling transient state`
`,` 向上翻一页
`.` 向下翻一页
`<` 向上翻半页
`>` 向下翻半页


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
3. 创建 `.editorconfig` 文件，写上自己喜欢的配置。
4. 在 `~/.spacemacs` 中的 `docspacemacs/user-config` 中加入 `(editorconfig-mode 1)`。
5. 完。

## Git 集成 (必须先配置Magit 的使用)

Git 是一个优秀的版本控制工具，我们可以在 `.spacemacs` 的 `dotspacemacs-configuration-layers` 列表中添加 `git` 就可以集成 git 了。

下面是一些常用的 git 命令，前缀为 `g`。

spacemacs 0.1xx:

| Git | Magit |
|-----|-------|
| `git init`            | `SPC g i` |
| `git status`          | `SPC g s` |
| `git add`             | `SPC g s` 弹出层选中文件然后按 `s` |
| `git add currentFile` | `SPC g S` |
| `git commit`          | `SPC g c c` |
| `git push`            | `SPC g P` 按提示操作 |
| `git checkout xxx`    | `SPC g C` |
| `git checkout -- xxx` | `SPC g s` 弹出层选中文件然后按 `u` |
| `git log`             | `SPC g l l` |

spacemacs 0.2xx:

大部分命令整合到 `SPC g s` 中，需要按照提示执行命令

在 commit 时，我们输入完 commit message 之后，需要按 `C-c C-c` 来完成 commit 操作，也可以按 `C-c C-k` 来取消 commit 。


## 设置文件默认的主模式

虽然我们可以通过 `M-x` 来设置文件的主模式，但这种方式只是在单独修改某个文件的主模式时好用，如果要把所有同类型的文件都改成其他模式，这种方式的效率就太低了。

在 Spacemacs 中，我们可以用 `auto-mode-alist` 来设置某一类文件默认的主模式。

我们只需要在 `~/.spacemacs` 中的 `user-config` 中加入下面代码即可：

```elisp
(add-to-list 'auto-mode-alist '("\\.js\\'" . react-mode))
```

上面代码会用 `react-mode` 打开所有 `.js` 文件。


## 设置主模式的 hook

主模式的 hook 是一个 elisp 函数，这个函数会在主模式加载时调用，常用于为特定的主模式自定义配置。
如我想在 `js2-mode` 中作出如下配置：

- 移除检查分号的提示 (否则当行末缺少了分号的时候，会给出烦人的提示)
- 支持 Node.js 内建函数 (否则当出现 `require`, `module` 之类的时候会提示未定义变量)
- 设置缩进为 2

这时候我可以通过如下方式来设置：

```elisp
;; hooks
(defun my-js-mode-hook ()
  (setq js2-basic-offset 2)
  (setq js-indent-level 2)
  (setq js2-include-node-externs t)
  (setq js2-strict-missing-semi-warning nil))

(add-hook 'js2-mode-hook 'my-js-mode-hook)
```


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

https://github.com/syl20bnr/spacemacs/blob/master/doc/DOCUMENTATION.org
http://brannonlucas.com/using-editorconfig-and-spacemacs-on-os-x/
https://www.gnu.org/software/emacs/manual/html_node/elisp/Mode-Hooks.html
