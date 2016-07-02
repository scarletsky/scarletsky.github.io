---
title: 「译」Node.js Streams 基础
date: 2016-07-01 13:45:12
categories: nodejs
tags: [nodejs, stream]
---

Node.js 天生异步和事件驱动，非常适合处理 I/O 相关的任务。如果你在处理应用中 I/O 相关的操作，你可以利用 Node.js 中的流(stream)。因此，我们先具体看看流，理解一下它们是怎么简化 I/O 操作的吧。

## 流是什么

流是 unix 管道，让你可以很容易地从数据源读取数据，然后流向另一个目的地。
简单来说，流不是什么特别的东西，它只是一个实现了一些方法的 `EventEmitter`。根据它实现的方法，流可以变成可读流(Readable)，可写流(Writable)，或者双向流(Duplex，同时可读可写)。
可读流能让你从一个数据源读取数据，而可写流则可以让你往目的地写入数据。


如果你已经用过 Node.js，你很可能已经遇到过流了。
例如，在一个 Node.js 的 HTTP 服务器里面，`request` 是一个可读流，`response` 是一个可写流。
你也可能用过 `fs` 模块，它能帮你处理可读可写流。
 

现在让你学一些基础，理解不同类型的流。本文会讨论可读流和可写流，双向流超出了本文的讨论范围，我们不作讨论。
 
## 可读流 (Readable Streams)

我们可以用可读流从一个数据源中读取数据，这个数据源可以是任何东西，例如系统中的一个文件，内存中的 buffer，甚至是其他流。因为流是 `EventEmitter`，它们会用各种事件发送数据。我们会利用这些事件来让流工作。

### 从流中读取数据

从流中读取数据最好的方式是监听 `data` 事件，添加一个回调函数。当有数据流过来的时候，可读流会发送 `data` 事件，回调函数就会触发。看看下面的代码片段：

```js
var fs = require('fs');
var readableStream = fs.createReadStream('file.txt');
var data = '';

var readableStream.on('data', function(chunk) {
  data += chunk;
});

readableStream.on('end', function() {
  console.log(data);
});
```

`fs.createReadStream` 会给你一个可读流。
最开始的时候，这个流不是流动态的。当你添加了 `data` 的事件监听器，加上一个回调函数时，它才会变成流动态的。在这之后，它就会读取一小块数据，然后传到你的回调函数里面。
流的实现者决定了 `data` 事件的触发频率，例如 HTTP request 会在读取到几 KB 数据的时候触发 `data` 事件。 当你从一个文件中读取数据的时候，你可能会决定当一行被读完的时候就触发 `data` 事件。


当没有数据可读的时候 (读到文件尾部时)，流就会发送 `end` 事件。在上面的例子中，我们监听了这个事件，当读完文件的时候，就把数据打印出来。


还有另一种读取流的方式，你只要在读到文件尾部前不断调用流实例中的 `read()` 方法就可以了。

```js
var fs = require('fs');
var readableStream = fs.createReadStream('file.txt');
var data = '';
var chunk;

readableStream.on('readable', function() {
  while ((chunk = readableStream.read()) != null) {
    data += chunk;
  }
});

readableStream.on('end', function() {
  console.log(data);
});
```

`read()` 方法会从内部 buffer 中读取数据，当没有数据可读的时候，它会返回 `null`。
因此，在 while 循环中我们检查 `read()` 是不是返回 `null`，当它返回 `null` 的时候，就终止循环。
需要注意的是，当我们可以从流中读取数据的时候，`readable` 事件就会触发。


### 设置编码

默认情况下，你从流中读取到的是 `Buffer` 对象。如果你要读取的是字符串的话，这并不适合你。因此，你可以像下面的例子那样通过调用 `Readable.setEncoding()` 来设置流的编码：

```js
var fs = require('fs');
var readableStream = fs.createReadStream('file.txt');
var data = '';

readableStream.setEncoding('utf8');

readableStream.on('data', function(chunk) {
  data += chunk;
});

readableStream.on('end', function() {
  console.log(data);
});
```

上面的例子中，我们把流的编码设置成 `utf8`，数据就会被解析成 `utf8`，回调函数中的 `chunk` 就会是字符串了。

### 管道 (Piping)

管道是一个很棒的机制，你不需要自己管理流的状态就可以从数据源中读取数据，然后写入到目的地中。我们先看看下面的例子：

```js
var fs = require('fs');
var readableStream = fs.createReadStream('file1.txt');
var writableStream = fs.createWriteStream('file2.txt');

readableStream.pipe(writableStream);
```

上面的例子利用 `pipe()` 方法把 file1 的内容写到 file2 中。因为 `pipe()` 会帮你管理数据流，你不需要担心数据流的速度。这让 `pipe()` 变得非常简洁易用。
需要注意的是，`pipe()` 会返回目的地的流，因此你可以很轻易让多个流链接起来！

### 链接 (Chaining)

假设有一个归档文件，你想要解压它。有很多方式可以完成这个任务。但最简洁的方式是利用管道和链接：

```js
var fs = require('fs');
var zlib = require('zlib');

fs.createReadStream('input.txt.gz')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('output.txt'));
```

首先，我们通过 `input.txt.gz` 创建了一个可读流，然后让它流向 `zlib.createGunzip()` 流，它会解压内容。最后，我们添加一个可写流把解压后的内容写到另一个文件中。


### 其他方法

我们已经讨论了一些可读流中重要的概念了，这里还有一些你需要知道的方法：

1. `Readable.pause()` - 这个方法会暂停流的流动。换句话说就是它不会再触发 `data` 事件。
2. `Readable.resume()` - 这个方法和上面的相反，会让暂停流恢复流动。
3. `Readable.unpipe()` - 这个方法会把目的地移除。如果有参数传入，它会让可读流停止流向某个特定的目的地，否则，它会移除所有目的地。


## 可写流 (Writable Streams)

可写流让你把数据写入目的地。就像可读流那样，这些也是 `EventEmitter`，它们也会触发不同的事件。我们来看看可写流中会触发的事件和方法吧。

### 写入流

要把数据写如到可写流中，你需要在可写流实例中调用 `write()` 方法，看看下面的例子：

```js
var fs = require('fs');
var readableStream = fs.createReadStream('file1.txt');
var writableStream = fs.createWriteStream('file2.txt');

readableStream.setEncoding('utf8');

readableStream.on('data', function(chunk) {
  writableStream.write('chunk');
});
```

上面的代码非常简单，它只是从输入流中读取数据，然后用 `write()` 写入到目的地中。
这个方法返回一个布尔值来表示写入是否成功。如果返回的是 `true` 那表示写入成功，你可以继续写入更多的数据。 如果是 `false`，那意味着发生了什么错误，你现在不能继续写入了。可写流会触发一个 `drain` 事件来告诉你你可以继续写入数据。

### 写完数据后

当你不需要在写入数据的时候，你可以调用 `end()` 方法来告诉流你已经完成写入了。假设 `res` 是一个 HTTP response 对象，你通常会发送响应给浏览器：

```js
res.write('Some Data!!');
res.end();
```

当 `end()` 被调用时，所有数据会被写入，然后流会触发一个 `finish` 事件。注意在调用 `end()` 之后，你就不能再往可写流中写入数据了。例如下面的代码就会报错：

```js
res.write('Some Data!!');
res.end();
res.write('Trying to write again'); //Error !
```

这里有一些和可写流相关的重要事件：

1. `error` - 在写入或链接发生错误时触发
2. `pipe` - 当可读流链接到可写流时，这个事件会触发
3. `unpipe` -  在可读流调用 `unpipe` 时会触发


## 结论

这些是关于流的基础知识。流，管道，链接是核心，它们是 Node.js 中最强大的功能。如果使用得当，流可以帮助你写出简洁而且高性能的代码。


## 参考资料
https://www.sitepoint.com/basics-node-js-streams/
