---
title: 编写 Hubot Scripts
date: 2016-05-02 16:05:37
categories: [hubot]
tags: [hubot]
---

## 简介

我们在上一篇中介绍了 Hubot 的简单用法，里面提到我们可以为机器人编写脚本来让它根据不同的「输入」来给出不同的「输出」。
本文将会介绍如何编写我们的 Hubot Scritps。


## 基础

我们的脚本应该放在哪里才能让 hubot 找到并且正常加载呢？在上一篇文章中我们提到过，hubot 在启动时会加载 `scripts/` 目录中的脚本文件。
但它到底是怎么加载的呢？我们可以打开 `bin/hubot` 文件看一下：

```coffee
# ......
loadScripts = ->
  # 加载 scripts 中的脚本
  scriptsPath = Path.resolve ".", "scripts"
  robot.load scriptsPath

  # 加载 src/scripts 中的脚本
  scriptsPath = Path.resolve ".", "src", "scripts"
  robot.load scriptsPath

  # 加载 hubot-scripts.json 中列出的脚本
  hubotScripts = Path.resolve ".", "hubot-scripts.json"
  if Fs.existsSync(hubotScripts)
    data = Fs.readFileSync(hubotScripts)
    if data.length > 0
      try
        scripts = JSON.parse data
        scriptsPath = Path.resolve "node_modules", "hubot-scripts", "src", "scripts"
        robot.loadHubotScripts scriptsPath, scripts
      catch err
        console.error "Error parsing JSON data from hubot-scripts.json: #{err}"
        process.exit(1)

  # 加载 external-scripts.json 中列出的脚本
  externalScripts = Path.resolve ".", "external-scripts.json"
  if Fs.existsSync(externalScripts)
    Fs.readFile externalScripts, (err, data) ->
      if data.length > 0
        try
          scripts = JSON.parse data
        catch err
          console.error "Error parsing JSON data from external-scripts.json: #{err}"
          process.exit(1)
        robot.loadExternalScripts scripts

  # 加载由 process.env.HUBOT_SCRIPTS 和 -r 参数指定的脚本
  for path in Options.scripts
    if path[0] == '/'
      scriptsPath = path
    else
      scriptsPath = Path.resolve ".", path
    robot.load scriptsPath
# ......
```

其实只是先指定脚本路径，然后调用 `robot.load` 和 `robot.loadHubotScripts` 而已拉！
而这两个方法简单来说是长这样子的：

```coffee
# ......
script = require(path)

if typeof script is 'function'
  script @
else
  @logger.warning "Expected #{full} to assign a function to module.exports, got #{typeof script}"
# ......
```

它们只是获取脚本路径，然后去 `require` 脚本，最后把 `this`(即 robot 对象) 作为参数传给 script 拉！
所以明白为什么我们之前说脚本要写成下面这样子了吧！

```
// coffee
module.exports = (robot) ->

// js
module.exports = function(robot) {}
```

知道了最基本的脚本写法之后，我们就可以愉快的编写属于我们的 hubot script 拉！


## 接受消息

作为一个聊天机器人，hubot 最基本的功能是要监听特定的「输入」。
Hubot 给我们提供了三个不同层次的方法来监听输入：`robot.hear`、`robot.respond` 和 `robot.listen`。

### robot.hear

监听任何匹配的「输入」。
即在聊天过程中，只要匹配到特定的消息，就会触发回调函数。

```coffee
module.exports = (robot) ->

  # 匹配任何带有 hello 的消息，如
  # hello
  # hellooooo
  # haha helloooooo
  robot.hear /hello/i, (res) ->
    # do what you want

```

### robot.respond

监听对 hubot 说的「输入」。
即在聊天过程中，前面带有 hubot/hubot:/@hubot 的消息才会被匹配，然后触发回调函数。

```coffee
module.exports = (robot) ->

  # 匹配对 hubot 说的 hi，如
  # hubot hi
  # @hubot hihihi~
  # hubot: hihihi!
  robot.respond /hi/i, (res) ->
    # do what you want

```

### robot.listen

自由度最高的监听器，传入一个函数（Match Function）对消息进行匹配。
该函数返回 `true` 时回调函数会被执行。

```coffee
module.exports = (robot) ->

  # 根据「消息」对象做处理
  robot.listen(
    (message) -> message.user.name is "Scarlex",
    (res) -> # do what you want
  )

```

## 发送消息

聊天机器人除了接收「输入」之外，还需要对消息做出「响应」。
有没有留意到上面接收消息中的回调函数都有一个 `res` 呢？
你猜对拉！和 Node.js 中的 `res` 用来响应 `req` 一样，这里的 `res` 也是是用来响应「输入」的。
其中比较常用的两个方法是 `res.send` 和 `res.reply`。

### res.send

这个方法和 `robot.hear` 相反，会直接把消息发送到聊天室。

```coffee
module.exports = (robot) ->
  # 匹配所有 hi 相关的输入，然后发送 hello 到聊天室
  robot.hear /hi/i, (res) ->
    res.send 'hello'
```

### res.reply

这个方法和 `robot.respond` 相反，谁对 hubot 聊天就会回复谁。

```coffee
module.exports = (robot) ->
  # 匹配所有对 hubot 说的 hi，然后回复对 hubot 说话的用户，如
  # 输入 @hubot hi
  # 输出 @scarlex hello
  robot.respond /hi/i, (res) ->
    res.reply 'hello'
```

### res.match

只有上面两个方法是远远不够的，因为上面两个方法并不能对「输入」做任何处理。
不知道童鞋们有没有发现，我们其实是用正则表达式来匹配输入的，而正则表达式刚好可以用来做匹配某些关键字！
当匹配到关键字之后，我们从哪里可以提取到这些关键字呢？
答案就是 `res.match` 拉！

```coffee
module.exports = (robot) ->
  # 用 res.match 来获取正则表达式匹配的结果，如
  # 输入 open the first door
  # 输出 opening the first door
  robot.hear /open the (.*) door/i, (res) ->
    res.send "opening the #{res.match[1]} door"
```

## 发出 http 请求

只是匹配消息再回复太简单拉！其实我们可以通过 hubot 发出 http 请求来做出更多的事情！
Hubot 自带一个 [node-scoped-http-client](https://github.com/technoweenie/node-scoped-http-client) 来发 http 请求。
用法如下：

```coffee
robot
  .http('https://github.com')
  .get() (err, response, body) ->
    # do what you want
```

初看会觉得很奇怪，其实这只是一个高阶函数而已，对应的 javascript 是这样的：

```js
robot
  .http('https://github.com')
  .get()(function(err, response, body) {
    // do what you want
  })
```

事实上，由于我们是在 Node.js 环境下运行 hubot 的， 我们可以用任何 http client 库来实现这个需求，如著名的 [request](https://github.com/request/request) 库。
我们要做的只是运行 `npm install request --save` 再 `require` 进来就可以了。

```coffee
request = require 'request'
module.exports = (robot) ->
  robot.hear /get github page/i, (res) ->
    request.get 'https://github.com', (err, response, body) ->
      res.send response.statusCode
```

我在这里只演示了 GET 请求，其他类型的请求相信也难不倒大家拉！遇到什么问题去翻翻类库的文档就好拉！

需要提醒一点，在发出 http 请求的时候，不要搞错了 hubot 的 `res` 对象和 request 的 `res` 对象哦！


## 响应 http 请求

Hubot 内置了一个 [express](https://github.com/expressjs/express) 来响应 http 请求。
它会随 hubot 一并启动，默认端口是 8080，我们可以设置环境变量 `EXPRESS_PORT` 或 `PORT` 来改变默认的端口。
那么我们怎么才能使用它呢？很简单，只要调用 `robot.router` 就可以拉！

```coffee
module.exports = (robot) ->
  # 打开浏览器，然后输入 http://localhost:8080/hubot/haha
  # 会看见浏览器显示 ok
  robot.router.get '/hubot/haha', (req, res) ->
    res.send 'ok'
```

尽情发挥你的想象力去写一些有趣的东西吧！

哦，对了，如果想要禁用这个 express，只要在启动的时候加个 `-d` 或者 `--disable-httpd` 就好了。
或者设置环境变量 `HUBOT_HTTPD` 为 `false` 也可以！
即下面的方式都可以：

```
$ ./bin/hubot -d
$ ./bin/hubot --disable-httpd
$ HUBOT_HTTPD=false ./bin/hubot
```

当看到控制台输出下面这种警告的时候，就表示 express 被禁止启动拉！

```
WARNING A script has tried registering a HTTP route while the HTTP server is disabled with --disabled-httpd.
```

## 事件处理

还有一点需要提的是，hubot 自带了一个 EventEmitter，这意味着我们可以通过 `robot.emit` 和 `robot.on` 来编写基于事件通讯的代码~

```coffee
module.exports = (robot) ->

  # 监听输入 event test，然后用 robot.emit 触发 wow 事件
  robot.hear /event test/i, (res) ->
    args = { id: '12345' }
    robot.emit 'wow', args
    res.send 'emit wow event with args: ' + JSON.stringify args

  # 用 robot.on 来监听 wow 事件，回调函数中可以获取事件发送过来的参数
  # 控制台会输出 { id: '12345' }
  robot.on 'wow', (args) ->
    robot.logger.info args
```

这种基于事件通讯的代码非常适合和 webhook 一起使用哦！
想象一下，当我们 push 代码到 master 分支的时候，触发一个 webhook，然后 hubot 就帮我们自动部署新版网站，很棒吧！


## 错误处理

任何代码都不是完美的，它们都有可能报错，当出现错误的时候，我们就需要对错误进行处理拉！
在 hubot 里，我们可以用 `robot.error` 来捕获错误！

```coffee
module.exports = (robot) ->

  # 输入 error test，会触发一个错误
  robot.hear /error test/i, (res) ->
    JSON.parse([])

  # 触发错误之后会捕获到错误，然后打印 Unexpected Error!
  
  robot.error (err, res) ->
    robot.logger.error "Unexpected Error!"
    if res?
      res.reply "Unexpected Error!!!"
```


## 其他有趣而无用的方法

最后提一下，hubot 自带一些有趣而无用的方法，这些方法很少用，有些需要 adapter 支持才能正常使用。

### robot.topic
### robot.enter
### robot.leave
### res.random

## 参考资料
- https://hubot.github.com/docs/scripting/
