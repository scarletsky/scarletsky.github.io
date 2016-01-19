title:  D3.js 学习笔记2
date:   2015-01-04 17:28:04
categories: javascript
tags: d3.js
---

# Layout

和它的名字相比，D3 中的 Layout 并不会放任何东西在屏幕上。实际上，Layout 方法和可视化输出并没有直接关系。D3 Layout 会把你交给它的数据转换成更加方便处理的数据。

D3 中包含的全部的 Layout 列表如下：

- Bundle
- Chord
- Cluster
- Force
- Histogram
- Pack
- Partition
- Pie
- Stack
- Tree
- Treemap

每一种 Layout 都会提供一些特别的方法帮助你处理数据。

本文主要介绍一下 `Pie Layout` 和 `Stack Layout`。

## Pie Layout

Pie Layout 可以帮助你生成类似饼状图的数据结构。我们先看看它的使用方法。

```js
var dataset = [ 5, 10, 20, 45, 6, 25 ];
var pie = d3.layout.pie();
```

如下图所示，你会看到 Pie Layout 会帮助你生成新的数据。
![](http://7tebgv.com1.z0.glb.clouddn.com/d3js-learning-2-01.png)

我们来看看如何用这些数据来生成饼状图。

```js
var dataset = [ 5, 10, 20, 45, 6, 25 ];
var pie = d3.layout.pie();

var w = 300;
var h = 300;

var outerRadius = w / 2;
var innerRadius = 0;
var arc = d3.svg.arc()  // 创建弧形。
                .innerRadius(innerRadius)  // 内径
                .outerRadius(outerRadius); // 外径

//Create SVG element
var svg = d3.select("body")
            .append("svg")
            .attr("width", w)
            .attr("height", h);

var arcs = svg.selectAll("g.arc")
        .data(pie(dataset))
        .enter()
        .append("g")
        .attr("class", "arc")
        .attr("transform", "translate(" + outerRadius + ", " + outerRadius + ")");  // 圆心位置

var color = d3.scale.category10();  // 创建颜色

//Draw arc paths
arcs.append("path")
    .attr("fill", function(d, i) {
        return color(i);  // 会返回不同的颜色
    })
    .attr("d", arc);  // d 是 path 的属性，用来定义路径的生成规则。

arcs.append("text")
    .attr("transform", function(d) {
        return "translate(" + arc.centroid(d) + ")";  // arc.centroid 是弧形中一个重要方法，会返回该弧形的中心位置。
    })
    .attr("text-anchor", "middle")
    .text(function(d) {
        return d.value;
    });
```

你可以在下面这个连接里看到效果：
[http://examples.oreilly.com/0636920026938/chapter_11/01_pie.html](http://examples.oreilly.com/0636920026938/chapter_11/01_pie.html)


## Stack Layout

Stack Layout 可以帮你处理如下图所示的类似“叠加柱形图”那样的数据结构。

![](http://7tebgv.com1.z0.glb.clouddn.com/d3js-learning-2-02.png)

我们先来看看一组数据：

```js
var _dataset = [
        { apples: 5, oranges: 10, grapes: 22 },
        { apples: 4, oranges: 12, grapes: 28 },
        { apples: 2, oranges: 19, grapes: 32 },
        { apples: 7, oranges: 23, grapes: 35 },
        { apples: 23, oranges: 17, grapes: 43 }
];  // 转换前的数据

var dataset = [
        [
                { x: 0, y: 5 },
                { x: 1, y: 4 },
                { x: 2, y: 2 },
                { x: 3, y: 7 },
                { x: 4, y: 23 }
        ],
        [
                { x: 0, y: 10 },
                { x: 1, y: 12 },
                { x: 2, y: 19 },
                { x: 3, y: 23 },
                { x: 4, y: 17 }
        ],
        [
                { x: 0, y: 22 },
                { x: 1, y: 28 },
                { x: 2, y: 32 },
                { x: 3, y: 35 },
                { x: 4, y: 43 }
        ]
];  // 转换后的数据
```

要使用 Stack Layout，首先要把我们的数据处理一遍，把相同类型的数据归为一组，例如上面代码所示那样，第一组数据是 apples，第二组数据是 oranges， 第三组数据室 grapes。注意，分组后的数据必须同时包含 x 和 y，其中 x 只是代表他们的 id，y 才是真正的数据。

处理好数据之后我们可以试试调用 stack 方法。

```js
var stack = d3.layout.stack();
stack(dataset);
```

之后我们可以看到下面的结果：
![](http://7tebgv.com1.z0.glb.clouddn.com/d3js-learning-2-03.png)

看到规律了吗？用 stack 方法处理过的数据，都会多了一个 `y0` 的键。而这个键的值刚好就是前面数据的 y 值之和。利用这个 `y0`，我们就可以很方便的创建“叠加柱形图”了。

```js
//Width and height
var w = 500;
var h = 300;

//Original data
var dataset = [
    [
        { x: 0, y: 5 },
        { x: 1, y: 4 },
        { x: 2, y: 2 },
        { x: 3, y: 7 },
        { x: 4, y: 23 }
    ],
    [
        { x: 0, y: 10 },
        { x: 1, y: 12 },
        { x: 2, y: 19 },
        { x: 3, y: 23 },
        { x: 4, y: 17 }
    ],
    [
        { x: 0, y: 22 },
        { x: 1, y: 28 },
        { x: 2, y: 32 },
        { x: 3, y: 35 },
        { x: 4, y: 43 }
    ]
];

//Set up stack method
var stack = d3.layout.stack();

//Data, stacked
stack(dataset);

//Set up scales
var xScale = d3.scale.ordinal()
    .domain(d3.range(dataset[0].length))
    .rangeRoundBands([0, w], 0.05);

var yScale = d3.scale.linear()
    .domain([0,
        d3.max(dataset, function(d) {
            return d3.max(d, function(d) {
                return d.y0 + d.y;
            });
        })
    ])
    .range([0, h]);

//Easy colors accessible via a 10-step ordinal scale
var colors = d3.scale.category10();

//Create SVG element
var svg = d3.select("body")
            .append("svg")
            .attr("width", w)
            .attr("height", h);

// Add a group for each row of data
var groups = svg.selectAll("g")
    .data(dataset)
    .enter()
    .append("g")
    .style("fill", function(d, i) {
        return colors(i);
    });

// Add a rect for each data value
var rects = groups.selectAll("rect")
    .data(function(d) { return d; })
    .enter()
    .append("rect")
    .attr("x", function(d, i) {
        return xScale(i);
    })
    .attr("y", function(d) {
        return yScale(d.y0);
    })
    .attr("height", function(d) {
        return yScale(d.y);
    })
    .attr("width", xScale.rangeBand());
```

结果可以看看下面连接：
[http://examples.oreilly.com/0636920026938/chapter_11/03_stacked_bar.html](http://examples.oreilly.com/0636920026938/chapter_11/03_stacked_bar.html)

什么？你不想要从上往下的柱形图？你想要从下往上的柱形图？

其实要生成从下往上的柱形图很简单，只需要把 yScale 的 range 反过来，然后画图的时候调整一下 y 和 height 属性就好了。

```js
var yScale = d3.scale.linear()
    ...
    .range([h, 0]); // 原来是 [0, h]

...
var rects = groups.selectAll("rect")
    ...
    .attr("y", function(d) {
        return yScale(d.y0) - (h - yScale(d.y)); // 原来是 return yScale(d.y0)
    })
    .attr("height", function(d) {
        return h - yScale(d.y);  // 原来是 return yScale(d.y)
    })
    ...

```

# 参考资料
[http://chimera.labs.oreilly.com/books/1230000000345/ch11.html](http://chimera.labs.oreilly.com/books/1230000000345/ch11.html)
