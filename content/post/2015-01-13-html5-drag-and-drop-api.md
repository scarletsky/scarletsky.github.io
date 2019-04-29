---
title:  HTML5 Drag and Drop API
date:   2015-01-13 18:28:04
categories: [HTML5]
tags: [html5, dnd]
---

# 基本使用

## 创建可拖放对象
在 HTML5 中创建可拖动内容非常简单，只需要在元素的属性中加上 `draggable="true"` 就能创建可拖放对象了。

```html
<div class="rect red" draggable="true"></div>
<div class="rect green" draggable="true"></div>
<div class="rect blue" draggable="true"></div>
```


## 如何拖动
有了可拖放对象之后，我们还需要处理拖放事件才能实现拖放。拖放包含下面的事件：

- `dragstart` 发生在元素开始被拖动时
- `drag` 发生在元素被拖动(按着鼠标不放时)
- `dragenter` 发生在有元素被拖进一个可放区域
- `dragover` 发生在有元素被拖在可放区域的上面
- `dragleave` 发生在有元素被拖离一个可放区域
- `drop` 发生在元素被放到可放区域内
- `dragend` 发生在拖放事件结束(松开鼠标或者按 esc 键)

```js
var draggableElements = document.querySelectorAll('[draggable]');
// draggableElements 是 NodeList 对象，并不是 Array，因此没有 forEach 方法。
[].forEach.call(draggableElements, function (element) {
    element.addEventListener('drag', drag, false);
    element.addEventListener('dragstart', dragStart, false);
    element.addEventListener('dragenter', dragEnter, false);
    element.addEventListener('dragleave', dragLeave, false);
    element.addEventListener('dragover', dragOver, false);
    element.addEventListener('drop', drop, false);
    element.addEventListener('dragend', dragEnd, false);
});
```

## DataTransfer 对象
dataTransfer 属性正是这一神奇 DnD 功能的动力来源，控制着拖动操作中发送的数据。dataTransfer 可在 dragstart 事件中进行设置，并在 drop 事件中读取/处理。调用 e.dataTransfer.setData(format, data) 会将对象内容设置成 MIME 类型，并将数据有效负载作为参数传递。

拖放不同对象需要设置不同的格式，下面是 MDN 推荐 drag types。

- 文字：`text/plain`
- 超链接：`text/uri-list`, `text/plain`
- HTML & XML：`text/html`, `text/plain`

其他类型可以查看 MDN 上面的文档：[https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Recommended_Drag_Types](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Recommended_Drag_Types)

**PS：综合自己试验和网上的说法，其实设置 `setData` 的 format 设置成自定义值也是可以使用的。**

## DataTransfer 属性

`effectAllowed` 和 `dropEffect` 是 DnD 里面常看见的属性，也是对新手来说最摸不着头脑的属性。

这两个属性最主要的作用是，用于配置拖放过程中鼠标的指针的类型，以便提示用户操作。其次作用是用来控制 `drop` 事件是否触发。

这两个属性的有效值可以查阅 [https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)。

## **注意**

- `effectAllowed` 只能在 `dragstart` 事件中设置

- `dropEffect` 只能在 `dragenter` 和 `dragover` 事件中设置

- `dropEffect` 和 `effectAllowed` 不相等时，不会触发 `drop` 事件


## 使用心得

- 必须禁止 `dragover` 的默认事件才能正常触发 `drop` 事件

- `e.dataTransfer.getData` 只能在 `drop` 事件中正常访问，在其他事件中访问的话会返回空值。具体请看 [http://stackoverflow.com/questions/11927309/html5-dnd-datatransfer-setdata-or-getdata-not-working-in-every-browser-except-fi](http://stackoverflow.com/questions/11927309/html5-dnd-datatransfer-setdata-or-getdata-not-working-in-every-browser-except-fi)

- 拖放操作的结果一般情况下会产生新的 `DOM` 结构，因此不能使用 `$(selector).on(event, fn)` 来进行事件绑定，因为这样对新产生的 DOM 不起作用。要进行动态绑定的话，可以使用下面的写法：`$(document).on(event, selector, fn)`。

## 使用示例
```html
<!doctype html>
<html>

<head>
    <style>
        [draggable] {cursor: move;}
        .dragging {opacity: 0.2;}
        .over {border: 1px dashed black;}
        .rect {width: 500px; height: 100px; margin: auto;}
        .red, .green, .blue, .black {width: 100%; height: 100%;}
        .red {background-color: #FF8C8C;}
        .green {background-color: #8CFFB5;}
        .blue {background-color: #8CA3FF;}
        .black {background-color: #3A3C44;}
    </style>
</head>

<body>
    <div class="rect" draggable="true">
        <div class="red"></div>
    </div>

    <div class="rect" draggable="true">
        <div class="green"></div>
    </div>

    <div class="rect" draggable="true">
        <div class="blue"></div>
    </div>

    <div class="rect" draggable="true">
        <div class="black"></div>
    </div>

    <script>
        var dragSrc = null;
        var draggableElements = document.querySelectorAll('[draggable]');
        // draggableElements 是 NodeList 对象，并不是 Array，所以没有 forEach 方法。

        function resetDragSrc () {dragSrc = null;}

        function dragStart (e) {
            console.log('drag start');
            dragSrc = this;
            e.dataTransfer.effectAllowed = 'move';
            e.dataTransfer.setData('text/html', this.innerHTML);

            this.classList.add('dragging');
        }

        function dragOver (e) {
            console.log('drag over');
            if (e.preventDefault) {
                e.preventDefault(); // Necessary. Allows us to drop.
            }

            e.dataTransfer.dropEffect = 'move';  // See the section on the DataTransfer object.

            return false;
        }

        function dragEnter (e) {
            console.log('drag enter');
            this.classList.add('over');
        }

        function dragLeave (e) {
            console.log('drag leave');
            this.classList.remove('over');
        }

        function drop (e) {
            console.log('drop!');
            if (e.stopPropagation) {
                e.stopPropagation(); // stops the browser from redirecting.
            }
            dragSrc.innerHTML = this.innerHTML;
            this.innerHTML = e.dataTransfer.getData('text/html');
        }

        function dragEnd(e) {
            console.log('drag end!');
            [].forEach.call(draggableElements, function (element) {
                element.classList.remove('over');
                element.classList.remove('dragging');
            });
            resetDragSrc();
        }

        function drag (e) {
            console.log('dragging!');
        }

        [].forEach.call(draggableElements, function (element) {
            element.addEventListener('dragstart', dragStart, false);
            element.addEventListener('dragenter', dragEnter, false);
            element.addEventListener('dragleave', dragLeave, false);
            element.addEventListener('dragover', dragOver, false);
            element.addEventListener('drag', drag, false);
            element.addEventListener('drop', drop, false);
            element.addEventListener('dragend', dragEnd, false);
        });

    </script>
</body>

</html>

```

# 参考资料
[http://www.html5rocks.com/zh/tutorials/dnd/basics/](http://www.html5rocks.com/zh/tutorials/dnd/basics/)
[https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)
[https://developer.mozilla.org/en-US/docs/Web/Events](https://developer.mozilla.org/en-US/docs/Web/Events)
[http://stackoverflow.com/questions/203198/event-binding-on-dynamically-created-elements](http://stackoverflow.com/questions/203198/event-binding-on-dynamically-created-elements)
[http://www.cnblogs.com/fsjohnhuang/p/3961066.html](http://www.cnblogs.com/fsjohnhuang/p/3961066.html)
