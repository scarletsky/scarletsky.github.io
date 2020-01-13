---
title: JavaScript 进度管理
date: 2020-01-13T14:25:30+08:00
categories: [javascript]
---

## 前言

我们写程序的时候会经常遇到显示进度的需求，如加载进度、上传进度等。
最常见的实现方式是通过记录 `已完成数量(loadedCount)` 和 `总数量(totalCount)`，然后算一下就能得到进度了。
这种方式简单粗暴，容易实现，但不好扩展，必须有个地方维护所有 `loadedCount` 和 `totalCount`。
本文将会基于上述实现方式，实现一种更容易扩展的进度管理方式。


## 问题

笔者在写 WebGL 应用，在应用预加载阶段需要计算加载进度。
加载的内容包括：模型资源、贴图资源、脚本资源等。
其中模型资源中又会包含材质资源，材质资源里面又会包含贴图资源。
画图来表示的话就是如下的结构：

```text
+-------------------------------------------------------------+
|                                                             |
|  resources                                                  |
|                                                             |
|  +----------+   +-----------------+   +-----------------+   |
|  | script1  |   | model1          |   | model2          |   |
|  +----------+   |                 |   |                 |   |
|                 | -------------+  |   | -------------+  |   |
|  +----------+   | |model1.json |  |   | |model2.json |  |   |
|  | script2  |   | +------------+  |   | +------------+  |   |
|  +----------+   |                 |   |                 |   |
|                 | +------------+  |   | +------------+  |   |
|  +----------+   | | material1  |  |   | | material1  |  |   |
|  | texture1 |   | | +--------+ |  |   | | +--------+ |  |   |
|  +----------+   | | |texture1| |  |   | | |texture1| |  |   |
|                 | | +--------+ |  |   | | +--------+ |  |   |
|  +----------+   | | +--------+ |  |   | | +--------+ |  |   |
|  | texture2 |   | | |texture2| |  |   | | |texture2| |  |   |
|  +----------+   | | +--------+ |  |   | | +--------+ |  |   |
|                 | +------------+  |   | +------------+  |   |
|                 |                 |   |                 |   |
|                 | +------------+  |   | +------------+  |   |
|                 | | material2  |  |   | | material2  |  |   |
|                 | +------------+  |   | +------------+  |   |
|                 +-----------------+   +-----------------+   |
|                                                             |
+-------------------------------------------------------------+
```

这里有个前提：当加载某个资源的时候，必须保证这个资源及它引用的资源全部加载完成后，才能算加载完成。
基于这个前提，我们已经实现了一个 `onProgress` 接口，这个接口返回的进度是已经包含了子资源的加载进度的了。

既然有了这个接口，如果沿用全局维护 `loadedCount` 和 `totalCount` 的形式的话，处理起来其实挺麻烦的。
本文接下来要介绍的，就是一种变通的做法。


## 原理

基本思想就是**分而治之**。把一个大任务拆分成多个小任务，然后分别计算所有小任务的进度，最后再把所有小任务的进度归并起来得到总进度。
如下图表示：

```text

+--------------------------------------------------------------------+
|                                                                    |
|                                                                    |
|   total progress                                                   |
|                                                                    |
|   +---------+---------+----------+----------+--------+--------+    |
|   | script1 | script2 | texture1 | texture2 | model1 | model2 |    |
|   |  (0~1)  |  (0~1)  |   (0~1)  |   (0~1)  |  (0~1) |  (0~1) |    |
|   +---------+---------+----------+----------+--------+--------+    |
|                                                                    |
|   model1                                                           |
|   +-------------+-----------------------+-----------+              |
|   | model1.json |      material1        | material2 |              |
|   |   (0~1)     |        (0~1)          |   (0~1)   |              |
|   +------------------------+------------------------+              |
|                 | texture1 |  texture2  |                          |
|                 |   (0~1)  |    (0~1)   |                          |
|                 +----------+------------+                          |
|                                                                    |
|   model2                                                           |
|   +-------------+-----------------------+-----------+              |
|   | model2.json |      material1        | material2 |              |
|   |    (0~1)    |        (0~1)          |   (0~1)   |              |
|   +------------------------+------------------------+              |
|                 | texture1 |  texture2  |                          |
|                 |   (0~1)  |    (0~1)   |                          |
|                 +----------+------------+                          |
|                                                                    |
+--------------------------------------------------------------------+
```

基于这个原理去实现进度，实现方式就是通过一个列表去保存所有资源当前的加载进度，然后每次触发 `onProgress` 的时候，执行一次归并操作，计算总进度。

```js
var progresses = [
  0, // script1,
  0, // script2,
  0, // texture1,
  0, // texture2,
  0, // model1,
  0, // model2
];

function onProgress(p) {
    // TODO: progresses[??] = p;
    return progresses.reduce((a, b) => a + b, 0) / progresses.length;
}
```

但这里面有个难点，当触发 `onProgress` 回调的时候，如何知道应该更新列表中的哪一项呢？
利用 JavaScript 的闭包特性，我们可以很容易实现这一功能。

```js
var progresses = [];
function add() {
    progresses.push(0);
    var index = progresses.length - 1;

    return function onProgress(p) {
        progresses[index] = p;
        reduce();
    };
}

function reduce() {
    return progresses.reduce((a, b) => a + b, 0) / progresses.length;
}
```

利用闭包保留资源的索引，当触发 `onProgress` 的时候，就能根据索引去更新列表中对应项的进度了。最后归并的时候就能计算出正确的进度了。
最后一步就是整合我们所有的代码了。


## 测试

我们可以用下面的代码来模拟一下整个加载过程：

```js
class Asset {
    constructor(totalCount) {
        this.loadedCount = 0;
        this.totalCount = totalCount;
        this.timerId = -1;
    }

    load(onProgress) {
        if (typeof onProgress !== 'function') {
            onProgress = (_p) => {  };
        }

        return new Promise((resolve) => {
            this.timerId = setInterval(() => {
                this.loadedCount++;
                onProgress(this.loadedCount / this.totalCount);
                if (this.loadedCount === this.totalCount) {
                    clearInterval(this.timerId);
                    resolve();
                }
            }, 1000);
        });
    }
}

class Progress {
    constructor(onProgress) {
        this.onProgress = onProgress;
        this._list = [];
    }

    add() {
        this._list.push(0);

        const index = this._list.length - 1;

        return (p) => {
            this._list[index] = p;
            this.reduce();
        };
    }

    reduce() {
        const p = Math.min(1, this._list.reduce((a, b) => a + b, 0) / this._list.length);
        this.onProgress(p);
    }
}

const p = new Progress(console.log);
const asset1 = new Asset(1);
const asset2 = new Asset(2);
const asset3 = new Asset(3);
const asset4 = new Asset(4);
const asset5 = new Asset(5);

Promise.all([
    asset1.load(p.add()),
    asset2.load(p.add()),
    asset3.load(p.add()),
    asset4.load(p.add()),
    asset5.load(p.add()),
]).then(() => console.log('all resources loaded'));

/**
  输出
  Promise { <state>: "pending" }
  
  0.2 
  0.3 
  0.36666666666666664 
  0.41666666666666663 
  0.45666666666666667 
  0.5566666666666668 
  0.6233333333333333 
  0.6733333333333333 
  0.7133333333333333 
  0.78 
  0.8300000000000001 
  0.8699999999999999 
  0.9199999999999999 
  0.96 
  1 
  all resources loaded 
 */
```

(完)。
