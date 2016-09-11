---
title: 浅谈单页应用中前端分页的实现方案
date: 2016-09-11 18:07:06
category: javascript
tags: [javascript]
---

## 简介

分页是开发中最常见的需求之一。
对于分页，我们讨论的最多的是后端的数据库分页，这关乎到我们应用程序的性能，也是分页这个需求的核心。
而前端要做的，是把后端返回的数据呈现在页面上，工作被认为是简单琐碎的。
在单页应用中，我们有很多中分页方案，最常见的是无限滚动、上一页 & 下一页和页码。
本文将谈谈这三种分页方式。


## 通用

无论使用哪种分页方案，我们都需要处理一些通用的需求，如：

- 解析 url，提取当前页面的参数
- 根据返回数据生成自定义 DOM
- 移除某个 Node 节点中的所有子元素
- 往某个 Node 节点中插入元素列表

```js
// 解析 url 中的查询字符串
// 如 http://host/items?page=5 中提取 page=5 中的 5
function parsePage() {
    var searchString = window.location.search.substr(1).split('&').filter(v => v.indexOf('page') !== -1)[0];
    var page = Number(searchString.split('=')[1]);
    return isNaN(page) ? 1 : page;
}

// 生成自定义 DOM
// generateItemView :: Object -> DOM Node
function generateItemView(object) { /* implementation */ }

// 移除 Node 中所有子节点
function removeItems(node) {
    while (node.firstChild) {
        node.removeChild(node.firstChild);
    }
}

// 往 Node 中插入元素列表
function insertItems(node, items) {
    items.forEach(item => node.appendChild(generateItemView(item)));
}
```

下文的示例代码中会直接调用这些函数，不再重复定义。


## 无限滚动

无论对从前端还是后端来说，无限滚动都是我认为最简单的分页方案。
对后端来说，按照 `page` 和 `limit` 直接查出范围，然后返回一个数组给前端即可，不需要像其他方案那样还要查询总数。
对前端来说，直接根据后端返回的数据进行拼接即可，当后端返回一个空数组时，可以认为已经到最后一页，这时候就不需要再发请求到后端了。

```js
// 后端返回的数据结构
// GET /items?page=5
{ items: [...] }
```

```js
// 前端处理
function getItems(page) {
    fetch(`/items?page=${page}`)
        .then(res => res.json())
        .then(res => {
            if (res.items.length > 0) {
                insertItems(
                    document.getElementById('container'),
                    res.items
                );
            } else {
                alert('No more data');
            }
        });
}

```

无限滚动虽然实现起来简单，用户体验也不错，但有一些致命的缺点：

- 容易出现性能问题
- 容易丢失浏览进度

目前有一些方案可以解决这些缺点：性能问题可以通过动态渲染来解决，而丢失浏览进度则可以通过简单的新开窗口来解决。


## 上一页 & 下一页

这种分页方式和无限滚动比起来，会复杂一点点。
最主要是因为后端需要查询总数，然后根据当前页数来计算是否可以查询上一页或下一页。
当然，计算这部分可以在后端做，也可以在前端做。

### 后端计算

如果在后端计算，那么后端要做的事情就有：

- 查询总数
- 计算 `hasPrev` 和 `hasNext`
- 查询元素列表

而前端方面则相对简单：

- 根据后端返回的 `hasPrev` 和 `hasNext` 来判断是否需要显示上一页/下一页按钮
- 移除容器内的所有元素，再插入新的元素（即用新元素替换旧元素）

```js
// 后端返回数据结构
// GET /items?page=5
{
    // hasPrev 和 hasNext 都需要后端去查询总数，然后计算出来
    hasPrev: true,
    hasNext: true,
    items: [...]
}

```

```js
// 前端处理
function getItems(page) {
    fetch(`/items?page=${page}`)
        .then(res => res.json())
        .then(res => {
            res.hasPrev
                ? document.getElementById('prevButton').style.display = 'block'
                : document.getElementById('prevButton').style.display = 'none';

            res.hasNext
                ? document.getElementById('nextButton').style.display = 'block'
                : document.getElementById('nextButton').style.display = 'none';

            var container = document.getElementById('container');
            removeItems(container);
            insertItems(container, res.items);
        });
}
```

这个方案实现起来比较简单，但缺点是每次分页都需要查询总页数，浪费资源。

### 前端计算

如果是前端计算的话，那么后端要做的事情就相对简单，只要再提供一个查询总数的接口即可。
而前端方面，需要做更多的事情，同时要考虑当前端数据丢失时（如用户刷新页面）的处理方案。

- 第一次加载页面时需要调用一次查询总数的接口，同时调用获取元素的接口
- 返回数据后计算 `hasPrev` 和 `hasNext`，用来判断是否需要显示上一页/下一页按钮
- 移除容器内的所有元素，再插入新的元素（即用新元素替换旧元素）


```js
// 后端返回数据结构
// GET /itemsCount
{ total: 100 }

// GET /items?page=5
{ items: [...] }
```

```js
// 前端处理
var total = 0;
var limit = 10;

window.onload = getItemsCount(getItems);

// 获取总数
function getItemsCount(callback) {
    fetch('/itemsCount')
        .then(res => res.json())
        .then(res => {
            total = res.total;
            callback.call(null, parsePage());
        });
}

function getItems(page) {
    fetch(`/items?page=${page}`)
        .then(res => res.json())
        .then(res => {
            var hasPrev = page != 1;
            var hasNext = page != Math.ceil(total / limit);
            hasPrev
                ? document.getElementById('prevButton').style.display = 'block'
                : document.getElementById('prevButton').style.display = 'none';

            hasNext
                ? document.getElementById('nextButton').style.display = 'block'
                : document.getElementById('nextButton').style.display = 'none';

            var container = document.getElementById('container');
            removeItems(container);
            insertItems(container, res.items);
        });
}
```

这种方案可以让后端甩锅给前端，前端的活又变多拉！


## 页码

最后我们谈谈页码分页。
这个方案和「上一页 & 下一页」的方案很类似，不同的地方在于这个方案需要根据当前页面和总数来生成页码。
生成页码是这个方案最麻烦的地方。举个简单的例子，假设我们的数据有 50 页，我们不可能把所有页码都显示出来，需要生成一组不连续的页码。

我们可以采用下面的形式来显示页面：

```js
// ------------------------------
// 我个人比较喜欢用 -1 来表示省略的区域
// 在生成 DOM 的时候，可以用省略号来展示
// ------------------------------
// 假设当前是第 1 页
[1, 2, 3, -1, 50]

// 假设当前是第 3 页
[1, 2, 3, 4, 5, -1, 50]

// 假设当前是第 25 页
[1, -1, 23, 24, 25, 26, 27, -1, 50]

// 假设当前是第 48 页
[1, -1, 46, 47, 48, 49, 50]

// 假设当前是第 50 页
[1, -1, 48, 49, 50]
```

生成页码的原则通常都是：

- 第一页和最后一页必须展示
- 其他页面按需展示，通常是当前页面的前后两页（即 x +- 2）
- 当页数少于 10 页的时候，直接显示出所有页码（为什么是 10 页？其实在满足前两个原则的情况下，只要 7 页省略号就会正常显示了。但页数较少的情况下显示省略号感觉怪怪的。）

```js
var lastPage = Math.ceil(total / limit);

// 根据当前页生成动态页码
function genPages() {

    if (lastPage <= 10) {
        return Array(10).fill().map((v, i) => i + 1);
    }

    // dynamicPages 为除第一页和最后一页之外的页码，-1 表示省略号
    var dynamicPages;

    if (page === 1) {
        dynamicPages = [2, 3, -1];

    } else if (page === 2) {
        dynamicPages = [2, 3, 4, -1];

    } else if (page === 3) {
        dynamicPages = [2, 3, 4, 5, -1];

    } else if (page === lastPage - 2) {
        dynamicPages = [-1, page - 2, page - 1, page, page + 1];

    } else if (page === lastPage - 1) {
        dynamicPages = [-1, page - 2, page - 1, page];

    } else if (page === lastPage) {
        dynamicPages = [-1, page - 2, page - 1];

    } else {
        dynamicPages = [-1, page - 2, page - 1, page, page + 1, page + 2, -1];
    }

    dynamicPages.unshift(1);
    dynamicPages.push(lastPage);

    return dynamicPages;
}
```

生成动态页码这部分的逻辑，无论放在前端还是后端都影响不大，可以按照自己需要去选择。
至于其他部分的细节，和「上一页 & 下一页」类似，这里就不再重复了。


## 参考资料
https://github.com/xitu/gold-miner/blob/master/TODO/ux-infinite-scrolling-vs-pagination.md
