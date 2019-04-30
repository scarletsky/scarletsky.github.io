---
title:  D3.js 学习笔记1
date:   2014-12-29 16:28:04
categories: [javascript]
tags: [d3.js]
---

# 基础操作

+ `d3.select`, `d3.selectAll` 选择元素
+ `selection.append` 在选择集合中添加元素
+ `selection.data` 为选择元素进行数据绑定，常与 `enter` 和 `exit` 一起使用
+ `selection.data(data).enter` 返回缺失元素集合
+ `selection.data(data).exit` 返回多余元素集合
+ `selection.text` 设置或获取选定元素的标签体文本内容
+ `selection.attr` 设置或获取指定属性
+ `selection.transition` 启用动画效果，只要设置前和设置后的属性发生变化就会发生动画，一般和 `duration`，`delay` 一起用。

更多操作可以参考本文末尾的官方 API 文档。


# 尺度(Scale)

尺度(Scale)在 D3 中是一个很重要的概念。

根据官方的定义，尺度是将输入域映射为输出范围的函数。尺度可以让用户定义参数的函数。一旦生成一个尺度，你可以调用此尺度函数，传入一个数据值，然后它会返回一个缩放后的值。你可以任意定义和使用尺度。

D3 中有两种尺度，分别是 **定量变换(Quantitative)** 和 **序数变换(Ordinal)**。


# 定量变换(Quantitative)

定量变换有一个连续的域，如 [0, 1000] 这种线性区间，它可以很方便的进行数据转换。

例如当我们有一个 [0, 1000] 的区间，但我们在作图的时候不想显示这么大的区间，只想让它显示 [0, 100] 的区间，这时候我们就可以利用定量变换进行数据转换了。

```js
// 创建定量变换
var yScale = d3.scale.linear()
                     .domain([0, 1000])
                     .range([0, 100])
// 使用定量变换
yScale(1000) // 100
yScale(50) // 5

yScale.domain() // [0, 1000]
yScale.range() // [0, 100]
```


# 序数变换(Ordinal)

序数变换是包含一组离散的值，如 ['January', 'February', 'March']，像这样的值是无法创建连续区间的，只能用来创建离散区间。

```js
var dataset = ['January', 'February', 'March'];
var xScale = d3.scale.ordinal()
                     .domain(d3.range(dataset.length))
                     .rangeBands([0, 100 * dataset.length]);

xScale.range() // [0, 100, 200];
xScale.rangeBand() // 100
xScale(0) // 0
xScale(1) // 100
xScale(2) // 200
xScale(3) // undefined
```

看了上面代码之后可以发现，序数变换和定量变换在创建和使用上都是不同的。

创建定量变换时传进 `.domain` 中的是一个区间，而创建序数变换时传进去的是一组数，序数变换利用这组数去创建一个等宽度的区间。

使用定量变换时，无论我们传什么数值进去，它都能返回一个和 `domain` 对应的值。但使用序数变换时，我们只能通过下标去获取值，当下标大于数组长度时，它会返回 `undefined` ，和数组类似。


# 坐标轴

利用尺度(Scale)，我们可以很轻松的创建坐标轴。

```js
var xAxis = d3.svg.axis()  // 创建坐标轴
                  .scale(xScale) // 设置坐标轴的尺度变换，该尺度变换设定了数值和像素位置的转换规则
                  .orient('bottom') // 设置坐标轴刻度的方向
                  .ticks(3) // 控制坐标轴刻度的产生方式，需要注意的是这个只是建议值，D3 会自动判断最合适的值
                  .tickFormat(function (d) {
                    return dataset[d].keyword; // 设置刻度文字的格式，注意这里的 d 的值是索引值。
                  });
```


# 绘图

D3 是利用 svg 进行绘图的，在操作上和用 `jQuery` 操作 `dom` 类似。

在进行绘图的时候需要注意一点：绘图方向是从上往下，从左往右的。也就是左上角的坐标是 (0, 0)，右下角的坐标是 (width, height)。

我们先看看如何用 D3 生成一个柱形图。

```js
var dataset = [
    {keyword: '未检查人数', value: 2345},
    {keyword: '已检查人数', value: 1232},
    {keyword: '放弃治疗人数', value: 4562},
    {keyword: '未知人数', value: 562},
    {keyword: '测试人数', value: 2462},
    {keyword: '全部人数', value: 8678}
];

var chartPadding = 100;
var barLabelWidth = 200; // space reserved for bar labels
var barLabelOffset = 5;
var valueLabelWidth = 40; // space reserved for value labels (right)
var maxBarHeight = 500;
var barHeight = 50;

var xScale = d3.scale.ordinal()
                     .domain(d3.range(dataset.length))
                     .rangeBands([0, barLabelWidth * dataset.length])

var yScale = d3.scale.linear()
                     .domain([0, (function () {
                        var maxValue = d3.max(dataset, function (d) { return d.value; });
                        var valueLength = (maxValue + '').length;
                        return Math.ceil(maxValue / Math.pow(10, valueLength - 1)) * Math.pow(10, valueLength - 1);
                      })()])
                     .range([maxBarHeight, 0])

var xAxis = d3.svg.axis()
                  .scale(xScale)
                  .orient('bottom')
                  .ticks(3)
                  .tickFormat(function (d) {
                    return dataset[d].keyword;
                  });

var yAxis = d3.svg.axis()
                  .scale(yScale)
                  .orient('left')
                  .ticks(6);

var svg = d3.select('#chart2').append('svg');
svg.attr('width', chartPadding * 2 + barLabelWidth * dataset.length)
   .attr('height', chartPadding * 2 + maxBarHeight + barLabelOffset);

svg.selectAll('rect').data(dataset).enter().append('rect')
   .attr('x', function (d, i) { return chartPadding + xScale(i) + xScale.rangeBand() / 4; })
   .attr('height', 0)
   .attr('y', function (d) { return chartPadding + maxBarHeight; })
   .transition()
   .duration(1000)
   .attr('y', function (d) { return chartPadding + yScale(d['value']); })
   .attr('width', xScale.rangeBand() / 2)
   .attr('height', function (d, i) { return maxBarHeight - yScale(d['value']); })

svg.selectAll('text').data(dataset).enter().append('text')
   .attr('x', function (d, i) { return chartPadding + xScale(i) + xScale.rangeBand() / 2; })
   .attr('y', function (d) { return chartPadding + maxBarHeight - barLabelOffset; })
   .attr('fill', 'red')
   .transition()
   .duration(1000)
   .attr('y', function (d) { return chartPadding + yScale(d['value']) - barLabelOffset; })
   .attr('width', xScale.rangeBand() / 2)
   .attr('text-anchor', 'middle')
   .tween("text", function (d) {
      var i = d3.interpolateRound(0, d['value']);
      return function (t) {
        this.textContent = i(t);
      };
    })

svg.append('g')
   .attr('class', 'axis')
   .attr("transform", "translate(" + chartPadding +", " + (maxBarHeight + chartPadding) + ")")
   .call(xAxis);

svg.append('g')
   .attr('class', 'axis')
   .attr("transform", "translate(" + chartPadding +", " + chartPadding + ")")
   .call(yAxis);
```

上面的代码有几个地方是需要注意的。

- **为什么定义 yScale 的时候，domain 是一个正向的区间，但 range 是一个反向的区间呢？**

还记得我们之前说过左上角的坐标是(0, 0)，右下角的坐标是(width, height) 吗？细心想想就可以发现，这和我们平时绘制柱形图时用的纵坐标是相反的。因此，我们需要定义一个符合我们使用习惯的尺度。

- **为什么我们的柱形图是从下往上画出来的吗？之前不是说绘图是从上往下，从左往右的吗？怎么会变成从下往上的呢？**

实际上，绘图确实是从上往下画的，之所以看到柱形图从下往上，是由于我们的绘图的起始位置一直往上移动，而且柱形的高度也在一直变化，这样就做成了一种错觉。

很多时候，我们只需要换一种思路，就能实现我们想要的效果了。

- **为什么坐标轴要最后才添加？一开始就添加坐标轴会有问题吗？**

假设我们先画坐标轴，再画柱形图，最后画柱形图上面的文字。
如果我们用上面的方式去生成柱形图上的文字的话，`svg.selectAll('text')` 会把坐标轴里面的文字都选上，数据绑定就会失效(因为元素数量大于数据数量)。这样结果就是不能生成柱形上面的文字了。

要解决的话其实也很简单，只要能区分柱形图上的文字和坐标轴上的文字就行了。怎么区分？加个 `id`, `class` 什么的，或者该结构也行。


# 事件绑定

D3 的事件绑定和 jQuery 类似，如果熟悉 jQuery 的事件绑定的话，可以很快掌握 D3 的事件绑定。

```js
// 基本用法
d3.select('p')
  .on('click', function () {
    // do some thing
  })；

// 常见用法
svg.selectAll('rect')
   .data(dataset)
   .enter()
   .append('rect')
   ... // 设置属性
   .on('click', function (d) {
     console.log(d); // 该矩形对应的数据
   })
   .on('mouseover', function (d) {
     d3.select(this)
       .attr('fill', 'orange');
   });
```


# 参考资料

- [官方 API 文档](https://github.com/mbostock/d3/wiki/Api-%E5%8F%82%E8%80%83)
- [D3 中文教程](http://pkuwwt.gitcafe.com/d3-tutorial-cn/scales.html)
