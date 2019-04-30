---
title: 从 Hexo 迁移到 Hugo
date: 2019-04-28T18:28:47+08:00
categories: [technology]
---


博客之前是用 Hexo 来搭建的，现在生成的速度已经有点慢了，于是想趁着假期把 Hexo 迁
移到其他程序。通过搜索发现两款比较合适的，一个是 [Zola](https://github.com/getzola/zola) ，另一个是 [Hugo](https://github.com/gohugoio/hugo)。
一开始我是想用 Zola 的，毕竟我是一个 Rust 粉丝，然而 Zola 的主题真的太少了，没几
个喜欢的，所以就转头到 Hugo 这边了。Hugo 是用 Golang 编写的静态网站生成器，号称
是世界上最快的框架，实际上用起来确实比 Hexo 快多了。


## 安装与使用

我们只需要按照着官方文档来操作就可以了：

```bash
brew install hugo  # 安装 hugo
hugo new site blog  # 使用 hugo 创建 blog 项目
cd blog  # 进入 blog 目录
hugo server # 启动开发服务器
```

非常简单，傻瓜操作，毫无难度。


## 迁移文章

迁移文章时 **必须** 做到不影响以前文章的 URL，不然会影响以前发出去的链接，如果别
人访问的话就会变成 404 了，而且会对 SEO 有影响。

我博客以前的文章是这种命名方式的：

```text
2017-01-02-xxxxx.md
2017-03-04-yyyyy.md
2017-05-06-zzzzz.md
```

然后配置 Hexo 会生成以下的 URL：

```text
/2017/01/02/xxxxx
/2017/03/04/yyyyy
/2017/05/06/zzzzz
```

到了 Hugo 这边，需要添加以下配置才能达到这种效果：

```toml
[frontmatter]
date = [":filename", ":default"]

[permalinks]
posts = "/:year/:month/:day/:slug"
```

### 关于 Front-matter

如果 Front-matter 写得不规范的话会导致一些奇奇怪怪的 Bug。

我迁移博客过程中曾经就遇到过下面这个问题：

```
ERROR 2019/04/29 21:29:15 Failed to render pages: render of "page" failed:
"/Users/scarlex/Projects/scarletsky.github.io/themes/even/layouts/_default/baseof.html:14:82":
execute of template failed: template: post/single.html:14:5:
executing "post/single.html" at <partial "head.html" .>: error calling partial:
"/Users/scarlex/Projects/scarletsky.github.io/themes/even/layouts/partials/head.html:14:82":
execute of template failed: template: _internal/schema.html:14:82:
executing "_internal/schema.html" at <.Params.tags>: range can't iterate over mongodb
```

出现这个 Bug 是因为我以前有些文章的 Front-matter 写的不规范，写成了下面这样：

```
---
title: MongoDB 运维基础
date: 2014-10-16 17:59
categories: [database]
tags: mongodb
---
```

这样就会导致 `tags` 不能迭代，需要改成 `tags: [mongodb]` 才能解决这个 Bug。

另外，如果 Front-matter 缺少了结尾的分隔符(`---`)，那么就会导致报错：

```
failed to unmarshal YAML: yaml: line 7: could not find expected ':'
```

如果缺少了开头的分隔符(`---`)，Hugo 在编译时不会报错，但是会解析出错，会额外生成
`public/1/01/01/index.html` 页面，这样在归档页面就会出现空链接。

因此，记得写好 Front-matter 哦。


## 添加评论

博客之前的评论系统用的是 [Disqus](https://disqus.com/)，用起来感觉很一般，而且不
支持 markdown，代码高亮又麻烦，就趁这个机会换掉它吧。目前有很多基于 Github
Issues 的评论系统，例如 [gitment](https://github.com/imsun/gitment)，
[utterances](https://github.com/utterance/utterances) 等。

目前采用了 utterances，主要喜欢它的样式，嗯。


## 部署

最后就到了部署环节了。用 Hexo 的时候配置一下，然后运行 `hexo d -g` 就能部署到
Github 上了。迁移到 Hugo 后，它官方文档推荐我们写 bash script 去部署，反正都是现
成的了，直接抄过来然后改下就能用了：

```diff
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo # if using a theme, replace with `hugo -t <YOURTHEME>`

# Go To Public folder
cd public

+ git init
+ git remote add origin git@github.com:scarletsky/scarletsky.github.io.git
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master -f

# Come Back up to the Project Root
cd ..

+ rm -rf public
```

总的来说 Hugo 还是很舒服的～

（完）


## 参考资料

https://gohugo.io/documentation/

https://discourse.gohugo.io/t/extracting-date-and-slug-from-filename/
