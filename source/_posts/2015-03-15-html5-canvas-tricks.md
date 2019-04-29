---
title: HTML5 Canvas 技巧
date: 2015-03-15 20:55:00
categories: [javascript]
tags: [html5, canvas]
---

# 注意事项

用 `<canvas>` 标签进行绘图时必须要设置 `width` 和 `height` 属性，这里并不是指 CSS 属性中的 `width` 和 `height`，而是 `<canvas>` 标签本身的属性！

```html
<!-- 正确 -->
<canvas width="500" height="500">

<!-- 错误 -->
<canvas style="width: 500px; height: 500px;">

<!-- 正确 -->
<canvas id="canvas" style="width: 500px; height: 500px;">
<script>
var canvas = document.getElementById('canvas');
canvas.setAttribute('width', '500');
canvas.setAttribute('height', '500');
</script>
```

我以前一直以为 `<canvas>` 的 `width` 和 `height` 属性和 CSS 中的 `width` 和 `height` 是同一个东西，直到我看到 Stackoverflow 上面的[这个帖子](http://stackoverflow.com/questions/2588181/canvas-is-stretched-when-using-css-but-normal-with-width-height-properties)...


# 读取图像文件，并绘制到 canvas 的中央

下面演示一下如何让用户选择一张图片，然后把图片绘制到 canvas 中央。

HTML
```html
<canvas id="canvas" width="500" height="500" style="border: 1px solid black;">
<input id="input" type="file" style="display: none;">
```

Javascript
```js
// 为了方便起见，还是用 jQuery 吧
var $canvas = $('#canvas');
var $input = $('#input');
var canvas = $canvas[0];
var input = $input[0];
var ctx = canvas.getContext('2d');

// 通过点击 canvas 触发 input 的 click 事件，用来选择文件
$canvas.on('click', function(e) {
    input.click();
});

// 选择文件后，会触发 change 事件
$input.on('change', function(e) {

    // 通过 FileReader 来读取文件
    var reader = new FileReader();
    var file = e.target.files[0];

    // 绑定读取文件后的回调函数
    reader.onload = function (e) {
        var img = new Image();
        var dataURL = e.target.result;

        // 通过计算可以得到
        img.onload = function () {
            var hRatio = canvas.width  / img.width    ;
            var vRatio =  canvas.height / img.height  ;
            var ratio  = Math.min ( hRatio, vRatio );

            var centerShiftX = ( canvas.width - img.width*ratio ) / 2;
            var centerShiftY = ( canvas.height - img.height*ratio ) / 2;  

            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(img,
                0, 0, img.width, img.height, // 原始图像
                centerShiftX, centerShiftY, img.width*ratio, img.height*ratio); // 目标图像
      };

      img.src = dataURL;

    };

    // 如果用户在选择文件时点了取消，file 就会为 undefined
    if (file) {
        // 读取文件，并返回 DataURL，可以通过 e.target.result 来得到这个 DataURL
        reader.readAsDataURL(file);
    }
});
```



# 参考资料：

[http://stackoverflow.com/questions/2588181/canvas-is-stretched-when-using-css-but-normal-with-width-height-properties](http://stackoverflow.com/questions/2588181/canvas-is-stretched-when-using-css-but-normal-with-width-height-properties)

[http://stackoverflow.com/questions/2588181/canvas-is-stretched-when-using-css-but-normal-with-width-height-properties](http://stackoverflow.com/questions/2588181/canvas-is-stretched-when-using-css-but-normal-with-width-height-properties)
