---
title: ExpressJS 源码学习 —— Application 篇
date: 2015-08-20 10:11:10
categories: javascript
---

# 概述

ExpressJS 是 NodeJS 世界中使用最广泛的 Web 框架。它非常轻量，最大的特点是可以组合各种各样的中间件来处理请求，每个中间件都会对请求进行处理，最后再返回给客户端。目前最新版的 ExpressJS 是 4.13.3。

这系列的文章是出于学习目的而写的，主要是想了解目前主流的 JavaScript 框架是如何设计的，使用了哪些我还不知道的技术。希望写完这系列的文章之后，对我自己的水平会有一定的提高。这系列的文章会先分析各个模块的 API 的工作流程，最后再作总结。

本文先从 application.js 文件开始分析。

# 私有 API

- `app.init` 对 app 进行初始化

    + 初始化 `cache`, `engines`, `settings` 为 `{}`

    + 调用 `this.defaultConfiguration` 来设置 app 的默认配置。

- `app.defaultConfiguration` 设置 app 的默认配置

    + 启用 `x-powered-by` (`middleware/init`这个中间件会用到)

    + 调用 `this.set` 来设置 `etag`, `env`, `query parser`, `subdomain offset`, `trust proxy`, `view`, `views`, `jsonp callback name` 等属性

    + 初始化 `this.locals`

    + 为 `production` 环境启用 `view cache`

    + 禁止用户直接访问 `app.router`

    + 监听 `mount` 事件，该事件会在调用 `app.use` 时触发

    + 很奇怪为什么一个普通的 `Object` 为什么可以监听和发送事件吧？想想就知道这个 `Object` 要么就继承了 `EventEmitter`，要么就和 `EventEmitter` 合并了。答案是后者，详细见 `express.js` 这个文件。

- `app.lazyrouter` 初始化 Router, 只会执行一次

    + 初始化 Router

    + 为这个 Router 添加 `query` 和 `init` 这两个中间件。

    + 把这个 API 放在 `defaultConfiguration` 之外是因为这个方法会读取 `settings`，如果把它放到 `defaultConfiguration` 中就会忽略用户的设置直接运行。试想，当你调用 `var app = express()` 的时候，默认的路由就已经新建好了，这时候你再怎么用 `app.set(xxx)` 也没效果的。

- `app.handle(req, res, callback)` 用来把 `req`, `res`, `callback` 交给 Router 来处理

    + 检查是否带有 `callback`，如果没有就提供一个默认的 `callback`

    + 检查是否已经初始化过 Router，如果没有就不会传递参数给 Router

- `app.path()` 返回绝对路径，看源码吧


# 公开 API
- `app.use(fn)` 挂载中间件，代理到 `Router.use`

    + 初始化 `offer`, `path`

    + 根据 `fn` 来调整 `offset` (要兼容 `app.use([path1, path2, path3...], fn1, [fn2, fn3...])` 这种语法)

    + 通过 `slice.call(arguments, offset)` 获取传入的所有函数，保存到 `fns` 中

    + 初始化 Router

    + 遍历 `fns`, 挂载所有 `fn`

    + 通过 `this.emit('mount', this)` 的形式发送 `mount` 事件 (`app.defaultConfiguration` 会监听这个事件)

- `app.render(name, options, callback)` 渲染指定名字的页面，`callback` 中返回渲染后的模板字符串

    + 初始化 `cache`，`engine`，`renderOptions`

    + 把 `this.locals`，`options._locals`，`options` 合并到 `renderOptions` 中

    + 根据 `this.settings['view cache']` 来判断是否需要缓存 `view`，在生产环境下，该值默认为 `true` (在 `defaultConfiguration` 设置的默认值)

    + 如果需要渲染的 `view` 已经缓存过，就直接找到该 `view` 对象，否则就生成 `view` 对象。

    + 尝试去渲染 `view` 对象。PS：渲染过程在 `View` 模块中实现的

- `app.route(path)` 代理到 `Router.route(path)`

- `app.param(name, fn)` 代理到 `Router.param(name, fn)`

    + 如果 `name` 参数为 `Array`，则会递归调用该函数

- `app.engine(ext, fn)` 注册模板引擎

    + 必须传入 `fn`，且 `fn` 必须是函数

    + 解析 `ext` 参数，会确保 `ext` 前面带有 `.`，即如果传入 `ejs`，它会帮你转换成 `.ejs`。在实现上只是简单判断第一个字符是不是 `.`

    + 把解析过的 `ext` 和 `fn` 以 `key` 和 `value` 的形式添加到 `this.engines` 中

- `app.set(setting, val)` 设置相关属性值

    + 如果传入一个参数，就去获取相关的属性值。`app.get(setting)` 就是这样实现的

    + 传入两个值的话，就会用 `this.settings[setting] = val` 的形式来设置值

    + `etag`，`query parser`，`trust proxy` 并不是用上面的形式来设置的，具体请看源码

- `app.get(setting)` 获取相关属性值

    + 事实上，就如上面 `app.set` 中所说，并没有通过 `app.get(setting)` 这种形式来设计这个 API

    + 由于 `get` 也是 HTTP 请求方法中的一个，所以它被放在 `app.VERB` 中实现了，但对它做过特殊处理。看下面的 `app.VERB` 吧。

- `app.VERB(...)` 委托 `.VERB` 去调用 `Router.VERB`

    + `methods` 是一个含有最常用的 HTTP 请求方法的列表，如 `['get', 'post', 'put', 'delete' ...]`

    + 遍历 `methods`，把每一个方法名都代理到 Router

    + 对 `get` 进行特殊处理：如果方法名为 `get`，而且参数只有一个，就去调用 `this.set(xxx)`

- `app.all(path)`

    + 初始化 Router

    + 通过 `route[method].apply(route, args)` 来实现本功能

- `app.enable(setting)` 把 `setting` 设置为 `true`

    + 通过 `this.set(setting, true)` 来实现

- `app.disable(setting)` 把 `setting` 设置为 `false`

    + 通过 `this.set(setting, false)` 来实现

- `app.enabled(setting)` 验证 `setting` 是否为 `true`

    + 通过 `Boolean(this.set(setting))` 来实现

- `app.disabled(setting)` 验证 `setting` 是否为 `false`

    + 通过 `!this.set(setting)` 来实现

- `app.listen` 创建服务器



# 学习心得

- ExpressJS 本质上就是让我们根据自己需要去配置 `app` 对象。

- `fn.call(target, param)` 和 `fn.apply(target, args)` 配合 `Array.prototype.slice.apply(argument, offset)` 可以很好的调节参数。这一点在 ExpressJS 中大量使用。

- 实现挂载中间件的方法和我预想不一样，我原本以为 `app.use` 简单添加到一个数组里面，然后按顺序执行这些中间件，结果原来是通过事件机制来实现的。

- 涉及到 Router 的地方还没看懂，等到时候再来补一补。


# 参考资料
https://github.com/strongloop/express
http://expressjs.com/api.html
https://cnodejs.org/topic/545720506537f4d52c414d87
