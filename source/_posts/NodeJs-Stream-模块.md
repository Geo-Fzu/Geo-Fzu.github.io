---
title: NodeJs Stream 模块
date: 2016-07-24 10:16:29
tags: [NodeJs, Stream, 概念]
categories: tech
---

# 一 、什么是流？

> A stream is an abstract interface for working with streaming data in Node.js. The stream module provides a base API that makes it easy to build objects that implement the stream interface.
> There are many stream objects provided by Node.js. For instance, a request to an HTTP server and process.stdout are both stream instances.
> Streams can be readable, writable, or both. All streams are instances of EventEmitter.

在 Node.js 中，流是一个用来处理流式数据的抽象接口。`stream` 模块提供基础的 API 来帮助更简单得创建对象。HTTP 服务器的一个 request 请求、process.stdout 等都是流的实例。
流可以分为可读、可写或者两者兼得。所有的流都是 EventEmitter 的实例。（**注： 所以，流可以通过事件的注册和监听来对数据进行处理。**）

<!--more-->

# 二、流的分类

Node.js 中有四种基本的流的类型：

- Readable - 可以从中读取数据的流（例如：fs.createReadStream()）
- Writable - 可以向其中写入数据的流（例如：fs.createWriteStream()）
- Duplex - 既可读又可写的流（例如：net.Socket）
- Transform - 当被读写是可以修改和转换其中数据的 Duplex 流（例如：zlib.createDeflate()）

## 1. Readable Streams

> Readable streams are an abstraction for a source from which data is consumed.
> All Readable streams implent the interface defined by the `stream.Readable` class.

可读流通过 `push` 方法产生数据，存入 readable 的缓存中。当调用 push(null) 时，便结束流的数据产生。正常情况下，需要为流实例提供一个 `_read` 方法，在这个方法中使用 `push` 产生的数据。
在需要数据时，流的内部会自动调用 `_read` 方法来向缓存中添加数据。

```javascript
var stream = require("stream");
var readable = Stream.Readable();
var source = ["a", "b", "c"];

readable._read = function() {
  var data = source.shift() || null;
  console.log("push", data);
  this.push(data);
};
```

- data 事件
  `data` 事件在流把一个数据块的的所有权转让给消耗者时触发。在 `flowing` 模式下，可以通过 `readable.pipe()`, `readable.resume()` 或者 通过给 `data` 事件添加监听回调函数触发。
  如果之前未调用 pause 方法进入 paused 模式，则监听 `data` 事件也会调用 `resume` 方法，从而使流进入 `flowing` 模式。**在 `flowing` 模式下，readable 的数据会持续不断的生产出来。**

  > The listener callback will be passed the chunk of data as a string if a default encoding has been specified for the stream using the readable.setEncoding() method; otherwise the data will be passed as a Buffer.

  如果 readable 有使用 `setEncoding()` 方法，回调函数会返回一个字符串，否则返回一个 `Buffer`。

- end 事件
  `end` 事件在流中没有可以消耗的数据时被触发。

  ```javascript
  // readable 定义见上面代码
  readable.on("end", function() {
    console.log("end");
  });

  readable.on("data", function(data) {
    console.log("data", data);
  });

  // 输出：
  /*
      push a
      push b
      data:  <Buffer 61>
      push c
      data:  <Buffer 62>
      push null
      data:  <Buffer 63>
      end
  */
  ```

- readable.pipe(destination[,options]) 方法

  > The `readable.pipe()` method attaches a Writable stream to the readable, causing it to switch automatically into `flowing` mode and push all of its data to the attached Writable. The flow of data will be automatically managed so that the destination Writable stream is not overwhelmed by a faster Readable stream.

  readable 使用 pipe() 后，将会自动转化为 `flowing` 模式。**read.pipe() 方法返回一个对 destination 流的引用**，这样我们就可以进行链式调用了。下面是一个 `gulp` 中链式调用的例子：

  ```javascript
  gulp
    .src("client/templates/*.jade")
    .pipe(jade())
    .pipe(minify())
    .pipe(gulp.dest("build/minified_templates"));

  // 中间环节的流，必须是既可写又可读的，即Duplex或Transform。
  ```

  默认情况下，destination 可读流将会在在可写流触发 `end` 事件是调用 stream.end() 方法，这样 destination 就不再可写了。pipe() 方法自动处理了 'data', 'end', 'write' 等事件和方法，使得关联变得更简单。

## 2. Writable Streams

- `_write(data, _, next)` 中调用 next(err) 来声明“写入”操作已完成， 可以开始写入下一个数据
- next 的调用时机可以是异步的
- 调用 write(data) 方法来往 writable 中写入数据。将触发 \_write 的调用，将数据写入底层
- 必须调用 end()方法来告诉 writable，所有数据均已写入
