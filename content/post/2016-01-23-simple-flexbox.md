---
title: 简单易懂的 Flexbox
date: 2016-01-23 09:58:37
categories: [css]
tags: [css, flexbox]
---

## 简介

Flexbox 是 CSS 3 的布局方式，可以轻松实现传统布局中难以实现的布局。

## 基本用法

1. 设置父容器的 `display` 为 `flex`，然后调节容器相关的属性。
2. 调节子元素相关的属性。

```html
<div class="container">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
</div>
```

```css
.container { display: flex; ... }
.item { ... }
```

## 具体用法

要使用 Flexbox 布局，你需要设置父容器和子元素的属性。

### 父容器设置
1. 启用 flex 布局 `display: flex | inline-flex`

2. 设置主轴方向 `flex-direction: row | row-reverse | column | column-reverse`
   - `row` 默认值，从左到右
   - `row-reverse` 从右到左
   - `column` 从上到下
   - `column-reverse` 从下到上

3. 设置子元素的换行方式 `flex-wrap: nowrap | wrap | wrap-reverse`
   - `nowrap` 默认值，让所有子元素排在一行中
   - `wrap` 自动换行，方向为从右到左。(这里的方向是指换行方向，不是指排列方向)
   - `wrap-reverse` 自动换行，方向为从左到右

4. `flex-direction` 和 `flex-wrap` 的简写：`flex-flow: <'flex-direction'> || <'flex-wrap'>` 

5. 设置子元素的在主轴中对齐方式 `justify-content: flex-start | flex-end | center | space-between | space-around`
   - `flex-start` 默认值，在起始位置对齐。和 `flex-direction: row` 一起用的话相当于左对齐，和 `flex-direction: column` 一起用的话相当于上对齐
   - `flex-end` 在终点位置对齐
   - `center` 居中对齐
   - `space-bewteen` 第一个子元素会在起始位置，最后一个子元素会在终点位置，它们之间的元素会在剩余位置中平均分布
   - `space-around` 所有元素都会平均分布在容器中。注意，视觉上元素不是平均分布的。那是因为所有元素所占的空间都被平均分了，元素两边都有空间，第一个元素和最后一个元素靠近容器的边缘只有一份空间，其他空白的地方都是有两份空间组成的，所以看起来两边的空间少，而中间的空间多。好好体会一下 `space-around` 字面上的意思就能理解了。

6. 设置子元素在侧轴中的对齐方式 `align-items: flex-start | flex-end | center | baseline | stretch`
   - `stretch` 默认值，拉伸元素来填充父容器
   - `flex-start` 在侧轴的起始位置对齐
   - `flex-end` 在侧轴的终点位置对齐
   - `center` 居中于侧轴
   - `baseline` 在基线对齐

7. 设置侧轴中行(不是元素)的对齐方式 `align-content: flex-start | flex-end | center | space-between | space-around | stretch`
   - `stretch` 默认值，拉伸行来填充剩余的空间
   - `flex-start` 所有行在容器的起始位置对齐
   - `flex-end` 所有行在容器的终点位置对齐
   - `center` 所有行居中于容器
   - `space-between` 类似 `justify-content: space-between`
   - `space-around` 类似 `justify-content: space-around`

### 子元素设置

1. 设置元素的排序方式 `order: <integer>`，数字越小，排越前面。默认情况下是以文档流的先后顺序排序，负值合法。

2. 调节元素的扩展能力 `flex-grow: <number>`，默认为1，增大该值表示该元素所占空间是其他元素的 n 倍，负值不合法。

3. 调节元素的收缩能力 `flex-shrink: <number>`，默认为 1，减少该值表示该元素所站空间是其他元素的 1/n，负值不合法。

4. 调节元素的基本大小：`flex-basis: <length> | auto`，默认为 auto。

5. 上面属性的缩写：`flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]`

6. 指定元素的排列方式，作用和容器中的 `align-items` 类似，优先级比 `align-items` 高。

**注意：`float`, `clear`, `vertical-align` 在子元素中不起作用。**



## 参考资料

- [https://css-tricks.com/snippets/css/a-guide-to-flexbox/](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
- [http://www.w3cplus.com/css3/a-guide-to-flexbox.html](http://www.w3cplus.com/css3/a-guide-to-flexbox.html)
- [http://zh.learnlayout.com/flexbox.html](http://zh.learnlayout.com/flexbox.html)

