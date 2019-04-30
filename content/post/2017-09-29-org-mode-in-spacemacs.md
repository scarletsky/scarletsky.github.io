---
title: Org mode in Spacemacs
date: 2017-09-29 23:30:13
categories: [emacs]
tags: [emacs, spacemacs, org-mode]
---

## 简介

Org mode 是 Emacs 中的一个 Major mode，本质上只是一种类似记事本的纯文本格式。它与 Markdown 非常相似，都是通过自定义标记来赋予文本样式和语义。我们可以用 Org mode 来记录各种各样的事情，如项目管理，TODO / GTD 管理等等。
而 Spacemacs 只是一份强大的 Emacs 配置文件，当 Org mode 与 Spacemacs 结合时，用起来会非常舒服，会给人一种相见恨晚的感觉。
本文主要总结在 Spacemacs 中使用 Org mode 的常用功能。


## 安装

在 `dotspacemacs-configuration-layers` 中添加 `org`，然后重启 Spacemacs。


## 打开

打开任意一个 `.org` 结尾的文件，Emacs 会以 Org mode 的处理该文件。
如果 Spacemacs 正在处理一个项目中的文件，这时候可以按 `SPC p o`，它会尝试打开该项目根目录中的 `TODO.org`，如果找不到该文件，就会创建该文件。


## 格式

Markdown 文件通常以 `.md` 结尾，而 Org mode 文件通常以 `.org` 结尾。
在语法上，Org mode 和 Markdown 非常相似， 下表总结了常用的格式：

|              | Markdown               | Org mode             |
|--------------|------------------------|----------------------|
| h1           | \# text                | \* text              |
| h2           | \#\# text              | \*\* text            |
| h3           | \#\#\# text            | \*\*\* text          |
| h4           | \#\#\#\# text          | \*\*\*\* text        |
| h5           | \#\#\#\#\# text        | \*\*\*\*\* text      |
| h6           | \#\#\#\#\#\# text      | \*\*\*\*\*\* text    |
| italics      | \*text\*               | /text/              |
| bold         | \*\*text\*\*           | \*text\*             |
| through line | \~\~text\~\~           | \+text\+             |
| underline    | <u>text</u>            | \_text\_             |
| link         | \[description\](url)   | [[url][description]] |
| image        | \!\[description\](url) | [[url][description]] |
| list         | \- item                | \- item              |
| code inline  | \`code\`               | =code= or \~code\~   |
| code block   | \`\`\`js               | #+BEGIN\_SRC js      |
|              | code                   | code                 |
|              | \`\`\`                 | #+END\_SRC           |
| quote        | \> quote text          | #+BEGIN\_QUOTE       |
|              |                        | text                 |
|              |                        | #+END\_QUOTE         |



## 功能

大家可能会问，既然 Org mode 和 Markdown 差不多，那 Org mode 的优势在哪里呢？
答案就是 Org mode 在 Emacs 里面会变得非常强大。

### 大纲

在上面的表中可以看到，我们可以用 `*` 来组织大纲视图。在 Markdown 中，大纲只是用来渲染标题，并没有特殊的功能。
而 Org mode 中的大纲是可以折叠的。假如你有以下大纲：

```
* this is h1
description of h1

** this is h2
write what you want here

*** h3
TODO

** this is another h2
```

Org mode 可以对大纲进行折叠，先把光标移动到大纲标题所在的行，然后按 `TAB`，当前标题下的内容会被折叠：

```
* this is h1...
```

再按一次 `TAB` 会展开大纲:

```
* this is h1
** this is h2...
** this is another h2...
```

再按一次 `TAB` 可以展开所有内容。

大纲常用快捷键：

| 快捷键            | 功能                    |
|-------------------|-------------------------|
| `TAB`             | 展开/折叠当前标题的内容 |
| `SHIFT-TAB`       | 展开/折叠所有标题的内容 |
| `ALT-<up>/<down>` | 上移/下移当前标题       |

### 表格

在 Org mode
中可以非常方便的制作表格。在 Emacs 中，我们可以按 `C-c |`，而在 Spacemacs 中，我们可以按 `SPC m t n` 或 `, t n` 来触发表格的制作。
按了快捷键之后，在 minibuffer 里面输入你想制作的表格大小就可以自动生成表格。

```
| id | name    | age |
|----|---------|-----|
|  1 | scarlex |  10 |
```

Org mode 的表格有一个非常好用的功能：自动对齐。
当我们在一个单元格中输入完之后，按 `TAB` 可以跳到下一个单元格中，然后就会触发单元格的自动对齐，这个功能真的非常好用，我们不用花时间区对齐单元格。


表格常用快捷键：

| 快捷键        | 功能                                   |
|---------------|----------------------------------------|
| `, t n`       | 创建表格                               |
| `TAB`         | 跳转到下一个单元格                     |
| `S-TAB`       | 跳转到上一个单元格                     |
| `, t a`       | 触发自动对齐                           |
| `, t i r`     | 在当前行上方插入一行                   |
| `, t i c`     | 在当前列左边插入一列                   |
| `, t i h`     | 在当前行下面插入水平线                 |
| `, t d r`     | 删除当前行(vim 模式下可以通过 dd 删除) |
| `, t d c`     | 删除当前列                             |
| `ALT-<up>`    | 上移当前行                             |
| `ALT-<down>`  | 下移当前行                             |
| `ALT-<left>`  | 左移当前列                             |
| `ALT-<right>` | 右移当前列                             |


### 待办事项

文章开头提到过，我们可以用 Org mode 来做待办事项管理。
要在 Org mode 中添加 TODO 项，只需要用 `- [ ] TODO ` 的形式即可。如：

```
- [ ] Fix bug
- [ ] Support new feature
- [ ] Release new version
```

当我们把光标移动到其中一项上面，然后按 `C-c C-c` 或 `, ,`，就可以标记该项为完成状态:

```
- [X] Fix bug
- [ ] Support new feature
- [ ] Release new version
```

值得一提的是，Org mode 中可以自动计算待办事项的完成情况，只需要在标题的后面添加 `[/]` 或 `[%]`，在完成 TODO 项的时候就会自动计算进度。
两者的区别在于进度的表现方式：

- `[/]` 会显示成 `[1/4]` 的形式。
- `[%]` 会显示成 `[25%]` 的形式。

```org
TODO List [1/3] // 按 C-c C-c 或 , , 的时候会自动计算

- [X] Fix bug
- [ ] Support new feature
- [ ] Release new version
```

除了上面形式的待办事项外，Org mode 还有另外一种形式的表示方式：通过设置 `TODO` 来标记。
默认情况下，Emacs 支持两种标记：`TODO` 和 `DONE`。
在任意一级的标题中按 `C-c C-t` 或 `, T T`，即可为该标题添加标记。
来看看下面这个例子：

```org
** Fix bug [/]

- [ ] login failed
- [ ] database crash
```

假如有以上内容，当我们在这级标题内按 `, T T`，会自动为其添加 `TODO` 标记：

```org
** TODO Fix bug [/]

- [ ] login failed
- [ ] database crash
```

当我们完成了所有子任务，再按 `, T T`，会把 `TODO` 替换成 `DONE`：

```org
** DONE Fix bug [2/2]
  CLOSED: [2017-09-30 六 20:48]

- [X] login failed
- [X] database crash
```

默认情况下的 `TODO` 和 `DONE` 这两种状态在一些情况下可能不够用，不用担心，我们可以自行添加更多的状态。
添加方法有以下两种：

- (只对当前 Org 文件生效) 在当前 Org 文件的开头添加以下内容：

```
#+SEQ_TODO: NEXT(n) TODO(t) WAITING(w) | DONE(d) CANCELLED(c)
```

然后把光标移动到该行上，按 `C-c C-c` 即可。

- (对所有 Org 文件生效) 在 Emacs 的配置文件中添加以下内容：

```
(setq org-todo-keywords
      '((sequence "NEXT(n)" "TODO(t)" "WAITING(w)"  "|" "DONE(d)" "CANCELLED(c)")))
```

然后重启 Emacs 即可。

设置完后按 `, T T`，会激活一个新窗口，我们可以看到新添加的状态，并且可以通过自定义的快捷键(括号里的字母)来激活对应的状态。

下面列出待办事项常用的快捷键：

| 快捷键          | 功能                          |
|-----------------|-------------------------------|
| `, ,`           | 把子任务设置成完成/未完成状态 |
| `, T T`         | 激活 TODO 项的设置            |
| `SHIFT-<left>`  | 设置 TODO 状态                |
| `SHIFT-<right>` | 设置 TODO 状态                |


### 导出

Org mode 还可以导出各种格式，如 HTML、LaTeX、PDF 等。
只需要按 `SPC m e e` 或 `, e e`，然后根据提示选择即可。


## 参考资料

- http://spacemacs.org/layers/+emacs/org/README.html
- https://www.youtube.com/playlist?list=PLVtKhBrRV_ZkPnBtt_TD1Cs9PJlU0IIdE
- http://notex.life/t/emacs-key-binding/18
- https://www.zhihu.com/question/19851600
- http://blog.csdn.net/u014801157/article/details/24372485

