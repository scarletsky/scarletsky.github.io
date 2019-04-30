---
title: Hubot 的简单用法
date: 2016-04-03 00:12:49
categories: [hubot]
tags: [hubot]
---

## 简介

Hubot 是 Github 的开源聊天机器人，可以用来做一些自动化任务，如部署网站，翻译语言等等。

你可能会说，这些只要写个脚本就可以做到了吧？

确实，但你写完脚本之后还是需要手动运行那些脚本。

你有没想过其实你可以在常用的聊天软件上说 `@xxx, 部署新版本的网站`，然后机器人就自动登录服务器，然后执行部署脚本，部署成功后告诉你 `新版本的网站已经部署成功`。

是的，如果你的聊天软件上集成了 Hubot，你就可以轻松地用它来管理一些繁琐的事情啦！


## 安装

官方推荐我们用 yeoman + hubot 生成器来生成我们的聊天机器人，方法如下：

```shell
$ npm install -g yo generator-hubot
$ mkdir myhubot && cd myhubot
$ yo hubot
```

回答一些基本的问题后，我们的聊天机器人就生成好啦~


## 基本用法

我们的聊天机器人的执行文件是 `bin/hubot`，我们先看看里面写什么：

```shell
$ cat ./bin/hubot

#!/bin/sh

set -e

npm install
export PATH="node_modules/.bin:node_modules/hubot/node_modules/.bin:$PATH"

exec node_modules/.bin/hubot --name "myhubot" "$@"
```

这份执行文件只是先执行 `npm install`，然后设置环境变量，再执行 `node_modules/.bin/hubot` 而已，没什么神秘的。

我们试试运行一下这份可执行文件：

```shell
$ ./bin/hubot
myhubot>
```

我们看到了一个类似 shell 的东东！试试随便输入一些东西：

```shell
myhubot> hello
myhubot> world
myhubot> how are you?
myhubot> can you hear me?
```

我们发现无论我们输入什么，我们的机器人都没有反应，是不是坏掉了？
其实并不是这样的，它没反应是因为我们没有对「输入」的处理，如果我们输入一些特定的「输入」，它就会有反应啦！

```shell
myhubot> myhubot ping
myhubot> PONG

myhubot> myhubot pug me
myhubot> http://28.media.tumblr.com/tumblr_locinzasB91qzj3syo1_500.jpg

myhubot> myhubot help
myhubot> myhubot adapter - Reply with the adapter
myhubot animate me <query> - The same thing as `image me`, except adds a few parameters to try to return an animated GIF instead.
myhubot echo <text> - Reply back with <text>
myhubot help - Displays all of the help commands that Hubot knows about.
myhubot help <query> - Displays all help commands that match <query>.
myhubot image me <query> - The Original. Queries Google Images for <query> and returns a random top result.
myhubot map me <query> - Returns a map view of the area returned by `query`.
myhubot mustache me <url|query> - Adds a mustache to the specified URL or query result.
myhubot ping - Reply with pong
myhubot pug bomb N - get N pugs
myhubot pug me - Receive a pug
myhubot the rules - Make sure hubot still knows the rules.
myhubot time - Reply with current time
myhubot translate me <phrase> - Searches for a translation for the <phrase> and then prints that bad boy out.
myhubot translate me from <source> into <target> <phrase> - Translates <phrase> from <source> into <target>. Both <source> and <target> are optional
ship it - Display a motivation squirrel
```

看到了吧！如果我们输入了特定的「输入」，机器人就会有反应啦！

当我们输入 `myhubot help` 的时候，返回的东东其实就是预定义的「输入」，这些预定义的「输入」只在 shell adapter 下有效哦！


## Adapter

什么是 shell adapter ？ 我们运行 `./bin/hubot` 时默认的 adapter 就是 shell adapter。

什么是 adapter ？ 所谓的 adapter 其实是一些让机器人接收输入的接口。 

刚才提到，shell adapter 是默认情况下的 adapter，主要是用来测试 adapter 是否生效。说白了，其实就是没什么用！

觉得很坑爹是吧？说好的让我们的聊天软件整合我们的机器人呢？

实际上社区已经为我们提供了各种各样的 adapter，我们只要下载就可以用啦！具体请看看 https://hubot.github.com/docs/adapters/

那么我们如何指定用某个 adapter 呢？很简单啦，只要启动机器人的时候带上 `-a` 参数就好了。
譬如如果我们想让机器人整合到 telegram，我们只要执行下面的命令就可以了：

```shell
$ npm install --save hubot-telegram
$ ./bin/hubot -a telegram
```

当然我们还需要设置一下，这些设置会根据不同的 adapter 而有所不同，具体请看对应的文档！

如果你所用的聊天软件并不在社区的支持列表中，又想把整合 Hubot 的话，可以自己写 adapter，文档在这里：https://hubot.github.com/docs/adapters/development/


## Scripts

我们一直说 Hubot 是聊天机器人，机器人最基本的是根据不同的「输入」给出不同的「输出」。
在 Hubot 应该怎么处理不同「输入」，给出不同的「输出」呢？
答案就是用 Scripts 啦！

有没有发现我们机器人的目录下有个 `scripts/` 文件夹？我们可以在这个文件夹下添加各种脚本文件，根据不同的「输入」给出不同的「输出」。
在我们启动 Hubot 的时候，它会加载 `scripts/` 文件夹下的脚本，赋予 Hubot 强大的交互能力！

需要注意的是，`scripts/` 下的脚本必须是 `.coffee` 或者 `.js` 格式的，而且必须暴露一个接受 robot 参数的函数！
我们还是先打开 `scripts/example.coffee` 看看吧！

```js
// coffee
module.exports = (robot) ->

// js
module.exports = function(robot) {}
```

在这个函数里面，我们可以利用 `robot.hear`、`robot.response`、`robot.send`、`robot.reply` 等 api 为不同的「输入」给出不同的「输出」！
我们还可以用 `robot.http(url).get()` 等方法来发出 http 请求！这样我们的机器人就可以有更强大的交互能力了！

想知道更多 api 的用法的话，可以参考文档：https://hubot.github.com/docs/scripting/


## 写在最后

Hubot 真的是一个简单易用的聊天机器人，我们可以把它整合到我们的聊天软件中，让那些简单但繁琐的任务自动化起来，提高我们的工作效率！
最后强烈推荐各位同学去读一下 Hubot 的源码，简单易懂，之后会对 Hubot 有更深刻的认识！


## 参考资料

- https://hubot.github.com/
