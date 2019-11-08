---
title: 实现一个简单限流的 Promise 队列
date: 2019-11-02T16:47:29+08:00
categories: [javascript]
---

## 前言

`Promise.all` 可以用来并行处理一组 Promise，在 Promise 数量不大的时候是一个很好的方案。
但当 Promise 的数量非常庞大的时候，`Promise.all` 很容易出现磁盘/网络的 IO 激增，而导致其他 Promise 无法处理，最后整个 `Promise.all` 一直无法返回。
而解决这钟问题的关键是需要一个限流的 Promise 队列，也就是本文需要讨论的对象。


## 问题

最近遇到一个问题，用 `Promise.all` 从 CDN 中下载几百个文件到本地的时候，系统的监控显示已经没有网络 IO 了，但 `Promise.all` 一直没返回。

代码如下：

```js
var fs = require('fs');
var path = require('path');
var axios = require('axios');
var resources = [ 'url1', 'url2', 'url3' ]; // length is 400
var folder = path.resolve(__dirname, '.');
Promise.all(
    resources.map(url => new Promise((resolve, reject) => {
        axios({
            method: 'GET',
            url: url,
            responseType: 'stream'
        }).then(res => {
            res.data
               .on('end', resolve)
               .on('error', reject)
               .pipe(fs.createWriteStream(
                   path.resolve(folder, path.basename(url))
            ));
        });
    }))
).then(console.log).catch(console.error);
```


很明显，这是因为并发量太大，某些 Promise 无法被处理，最后才导致整个 `Promise.all` 无法返回的。


## 解决方案

既然不能同时下载所有资源，那就一个一个下载吧。这就是我们要实现的 Promise 队列了。

```ts
type PromiseQueue = Array<() => Promise<any>>;
```

需要注意的是，Promise 队列的类型不能是 `Array<Promise<any>>`，这是因为当我们通过 `new Promise` 来创建 Promise 的时候，Promise 就会马上执行。
因此需要通过调用函数来创建 Promise，这样就能延迟创建 Promise 了。

有了队列之后，我们还需要一个 `loadNext` 的方法，当执行完上一个 Promise 的时候就去执行下一个 Promise。这个实现起来很简单：

```js
// ...
var queue = [];
var MAX_PARALLEL_COUNT = 10;
var loadingCount = 0;
var loadedCount = 0;

function loadNext() {
    if (loadingCount === MAX_PARALLEL_COUNT || queue.length === 0) return;

    loadingCount++;
    var promiseFn = queue.shift()
    promiseFn().then(onLoaded).catch(onLoaded);
}

function onLoaded() {
    loadingCount--;
    loadedCount++;
    console.log('Queue Length: ', queue.length, ' Loaded Count: ', loadedCount, ' Loading Count: ', loadingCount);

    if (loadingCount === 0) {
        console.log('Done.');
    } else {
        loadNext();
    }
}

for (var i = 0; i < resources.length; i++) {
    var url = resources[i];
    queue.push(() => new Promise((resolve, reject) => {
        axios({
            method: 'GET',
            url: url,
            responseType: 'stream'
        }).then(res => {
            res.data
               .on('end', resolve)
               .on('error', reject)
               .pipe(fs.createWriteStream(
                   path.resolve(folder, path.basename(url))
            ));
        });
    }));

    loadNext()
}
```

通过这种限流的 Promise 队列，我们就可以解决 `Promise.all` 并发太高而导致的奇怪问题了。


## 封装

最后封装一下上面的例子：

```js
class PromiseQueue {
    constructor(options = {}) {
        this.concurrency = options.concurrency || 1;
        this._current = 0;
        this._list = [];
    }

    add(promiseFn) {
        this._list.push(promiseFn);
        this.loadNext();
    }

    loadNext() {
        if (this._list.length === 0 || this.concurrency === this._current) return;

        this._current++;
        const fn = this._list.shift();
        const promise = fn();
        promise.then(this.onLoaded.bind(this)).catch(this.onLoaded.bind(this));
    }

    onLoaded() {
        this._current--;
        this.loadNext();
    }
}

const q = new PromiseQueue();
[1, 2, 3, 4, 5].forEach(v => {
    q.add(() => new Promise((resolve) => {
       setTimeout(() => {
           console.log(v);
           resolve();
       }, 1000);
    }));
});
```

这样，我们就能实现了一个简单限流的 Promise 队列了。

(完)。
