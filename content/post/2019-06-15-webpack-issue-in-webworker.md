---
title: 记一次 webpack 打包 web worker 的 bug
date: 2019-06-15T15:06:30+08:00
categories: [technology]
tags: [webpack]
---

## 前言

最近需要在项目中引入 web worker ，但是用 webpack 打包完后请求 worker 的时候总是会报 404 错误。


## 现象

这是一个很常见的配置：

```js
const WorkerPlugin = require('worker-plugin');

module.exports = {
    // ...
    output: {
        path: path.resolve(__dirname, '../public'),
        publicPath: '/',
        filename: 'static/scripts/[name].js',
    },
    // ...
    plugins: [
       // ... 
       new WorkerPlugin({
            globalObject: 'self'
       })
    ]
}
```

这样打包之后运行项目，我们会发现请求 web worker 的时候会请求 `/static/scripts/static/scripts/5.worker.js`，这个路径明显是错误的，整个的路径应该是 `/static/scripts/5.worker.js` 。


## 调查

**先调查下为什么它会请求这个路径呢？**

因为我在 web worker 里面引入了其他依赖，所以 web worker 里面就会通过 `importScripts` 来加载依赖。

**那么这个路径是怎么得到的呢？**

打开 webpack 打包出来的文件，我们可以找到相关的地方：

```js
__webpack_require__.e = function requireEnsure(chunkId) {
    var promises = [];
    promises.push(Promise.resolve().then(function() {
        // "1" is the signal for "already loaded"
        if(!installedChunks[chunkId]) {
            importScripts("static/scripts/" + ({}[chunkId]||chunkId) + ".worker.js");
        }
    }));
    return Promise.all(promises);
};
```

很明显，`importScripts` 里面少了 `publicPath` ！


## 解决方案

找到问题的根源了，接下来就很容易解决了，只要在打包的时候给它加上 `publicPath` 就可以了。

解决方案我提交 [pull request](https://github.com/webpack/webpack/pull/9270) 了，大家只要升级到 `v4.34.0+` 就能解决这个问题了。

~~没想到我居然成为 webpack 的贡献者了~~

(完)
