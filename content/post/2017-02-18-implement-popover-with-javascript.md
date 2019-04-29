---
title: 如何用 JavaScript 实现 Popover
date: 2017-02-18 16:21:21
categories: [javascript]
tags: [javascript]
---

## 简介
Popover 是我们日常开发中用得比较多的组件，通常用于给定一个触发元素，当某特定事件 (`hover`, `click`等) 在该元素上触发时，弹出相关的菜单供用户选择。

## 目标
我们的 Popover 需要实现如下特性：

- 点击触发元素时，Popover 出现/消失
- 点击 Popover 内部时，Popover 不消失
- 点击 Popover 外部时(不包含触发元素)，Popover 消失

先看如下的 HTML 结构和 CSS 代码：

```html
<div>
  <button id="trigger">click me</button>
  <div id="popover" class="popover">
    i am popover
    <button id="hide">hide popover</button>
  </div>
</div>
```

```css
.popover {
  padding: 10px;
  border: 1px solid black;
  display: none;
}

.popover.show {
  display: block;
}
```

这是一个基本的 Popover 的 HTML 结构和用来实现出现/消失的 CSS，再添加一些 JavaScript 代码，我们就能实现一个 Popover 了。
实现 Popover 中最麻烦的是上面提到的最后一点，即： 如何判断点击事件是发生在 Popover 内部还是外部。
对于这个问题，常见的有两种解法：

- 监听 document 中的 click 事件
- 利用 tabindex + focus + blur


## 监听 document 中的 click 事件
一种常见的方式是在 `document` 中添加事件监听器，用 `e.target` 来判断点击是否发生在 Popover 内部。


```js
var isVisible = false;
var trigger = document.getElementById('trigger');
var hide = document.getElementById('hide');
var popover = document.getElementById('popover');

function showPopover() {
  isVisible = true;
  popover.classList.add('show');
}

function hidePopover() {
  isVisible = false;
  popover.classList.remove('show');
}

function togglePopover() {
  if (isVisible) {
    hidePopover();
  } else {
    showPopover();
  }
}

trigger.addEventListener('click', function (e) {
  e.stopPropagation();
  togglePopover();
});

hide.addEventListener('click', function (e) {
  hidePopover();
});

document.addEventListener('click', function (e) {
  if (isVisible &&
      e.target !== popover &&
      e.target.parentElement !== popover
  ) {
    hidePopover();
  }
});
```

上面的实现虽然非常简单，但有一点需要注意的：
由于我们是在 `document` 中来监听 `click` 是否触发在 Popover 外部的，而触发元素也是在 Popover 之外，由事件冒泡的流程可以知道，浏览器会先调用触发元素中的函数，再调用 `document` 中的函数，这样一来 Popover 就不会显示了。所以我们需要调用 `e.stopPropagation` 来告诉浏览器不在派发事件。


## 利用 tabindex + focus + blur
这种方式是利用 `tabindex` 让 Popover 能够响应 `focus` 和 `blur` 事件。

```html
<div>
  <button id="trigger">click</button>
  <div id="popover2" class="popover" tabindex="0">
    i am popover
    <button id="close">close</button>
  </div>
</div>
```

```css
#popover2 {
  outline: none;
}
```

```js
var trigger = document.getElementById('trigger');
var popover = document.getElementById('popover2');
var hide = document.getElementById('hide');
var shouldTriggerBlur = true;

function showPopover() {
  popover.classList.add('show');
  popover.focus();
}

function hidePopover() {
  shouldTriggerBlur = true;
  popover.classList.remove('show');
}

trigger.addEventListener('click', showPopover);

hide.addEventListener('click', hidePopover);

popover.addEventListener('blur', function (e) {
  if (shouldTriggerBlur) {
    hidePopover();
  }
});

popover.addEventListener('mousedown', function (e) {
  if (e.target === hide) {
    shouldTriggerBlur = false;
  }
});
```

当点击触发元素时，调用 Popover 的 `focus` 方法，这样如果鼠标点击 Popover 外部时，就能自动触发 `blur` 事件了。
有一点是需要注意的：如果 Popover 中有其他可以响应 `click` 事件的元素 (如 `<button />`)，当我们点击这些元素时，会先触发 Popover 的 `blur` 事件，这时候 Popover 已经隐藏了，那么 Popover 中的 `<button />` 自然就点击不到了。
因此，上面的例子中，我用了一个 `shouldTriggerBlur` 变量来判断是否需要触发 `blur` 事件。

另外，这种实现有一个缺陷：在 Popover 出现后再点击触发元素，Popover 并不会消失。原因和刚才提到的一样，因为 `blur` 事件比 `click` 先触发。
