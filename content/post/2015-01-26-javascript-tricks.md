---
title:  JavaScript 常用技巧
date:   2015-01-26 18:05:00
categories: [javascript]
tags: [javascript]
---

# 字符串操作

## 首字母大写
```js
str.replace(/\b\w+/g, function (word) {
    return word.substring(0, 1).toLowerCase() + word.substring(1);
});
```


## 截取字符串最后几位
```js
str.substring(str.length-X)
```


## 进制转换
```js
d.toString(16)    (十进制 -> 十六进制)
parseInt(‘ff’, 16) (十六进制 -> 十进制)
```

## 移除左边多余的 0
```js
// 用正则, '000.00' -> 0.00
// 基于 http://stackoverflow.com/questions/594325/truncate-leading-zeros-of-a-string-in-javascript
str.replace(/^[0]{2}/, '');

// 用 parseInt , '000123' -> '123'
// http://stackoverflow.com/questions/6676488/remove-leading-zeros-from-a-number-in-javascript
parseInt(str, 10);
```

## 生成随机字符串
```js
Math.random().toString(36).substring(7)
// 出自 http://stackoverflow.com/questions/1349404/generate-a-string-of-5-random-characters-in-javascript
```

## is-start-with?
```js
string.lastIndexOf(str, 0) === 0
string.substring(0, str.length) === str
// 出自 http://stackoverflow.com/questions/646628/how-to-check-if-a-string-startswith-another-string
```

## is-end-with?
```js
// 出自 http://stackoverflow.com/questions/280634/endswith-in-javascript
```

## 解析 Base64
```js
var base64str = 'this is base64 string';
Buffer(base64str, 'base64').toString();
```
-----

# 数组操作

## 获取最后数组中最后一个元素
```js
arr.slice(-1).pop() // 简洁
arr[arr.length - 1] // 快速
// 出自 http://stackoverflow.com/questions/3216013/get-the-last-item-in-an-array
```

## 过滤重复元素
```js
arr.filter(function(v, i) {
  return arr.indexOf(v) === i;
});
```

-----

# 日期操作

## 获取当前时间戳
```js
+new Date
// 出自 http://stackoverflow.com/questions/221294/how-do-you-get-a-timestamp-in-javascript
```
