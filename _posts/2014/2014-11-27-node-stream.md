---
layout: post
title: Node的stream流
categories:
- 编程与开发
tags:
- NodeJS
- JavaScript
---

NodeJS中的stream是一个非常重要的概念，当进行IO操作的时候，我们可以将IO读取抽象成流的概念（有读取IO流一说）。本文我们分析Node中stream的概念，并分析如何创建出满足接口的自定义流。

简单地来说，stream就是实现了特殊方法的EventEmitter，一定的阶段会触发相应事件，从而引起用户自定义回调函数的调用。根据方法的不同，stream可以分为可读，可写或读写（Duplex）。最常见的流是标准输入流process.stdin和标准输出流process.stdout。另外一些常用的包括：

可读流：`http.request`、`http.response`、TCP中的`sockets`、`fs`中的读文件流等
可写流：`http.request`、`http.response`、`fs`中的写文件流等

由于流是EventEmitter，所以当流中有数据到来的时候（因为不一定能一次性读取所有的数据，所以有可能是一点一点地读取），就会自动触发`data`事件，而当没有数据的时候，就会触发`end`事件。所以读文件用下面的方式：

    var fs = require('fs')
    var data = ''
    var rstream = fs.createReadStream('file.txt')
    rstream.setEncoding('utf8')     // 默认情况下下面的on事件中chunk是Buffer类型的数据，可以更改其编码方式
    rstream.on('data', function (chunk) {
          data += chunk
      })
      .on('end', function (chunk) {
          console.log(data)
      });

当然，可读流还会触发一个叫`readable`的事件，即缓冲区中有数据可读，一般配合`read`方法使用：

    var fs = require('fs')
    var rstream = fs.createReadStream('file.txt')
    rstream.on('readable', function () {
          var chunk
          while ((chunk = rstream.read()) != null) {
              console.log(chunk.length)
          }
    })

从前面的描述中，不难看出，stream的数据是源源不断地过来，即会持续的触发`data`事件。但我们是可以通过可读流的pause和resume方法来暂停或恢复`data`事件的触发。

UNIX中有管道（pipe）这个概念，Node的stream也使用了这一思想，使得IO流的传输变得更加容易。举个例子来说，读文件并将内容写到另一个文件这一过程，使用pipe的思想即从一个创建的文件输入流传输到输出流中。

    fs.createReadStream('read.txt').pipe(fs.createWriteStream('write.txt'));
    process.stdin.pipe(process.stdout)      // 将输入的信息打印出来

甚至于，我们可以使用多个pipe将多个流连接起来（常用于我们需要对可读流中的内容作一定的处理以后再传输到输出流中），比如，我们希望把标准输入流中的所有字母全部轮换成大写并写入标准输出流，`through`包就提供了一个这样的功能：

    var through = require("through");
    var tr = through(function (chunk) {
        // write
        this.queue(chunk.toString().toUpperCase());
    }, function () {
        // end
        this.queue(null);
    });

既然有pipe管道连接两个流，自然也有unpipe来解除二者间的连接。其使用与pipe类似，只不过如果流之间本身就没有管道连接的话，其实就不会做任何事情；而如果调用的时候不指定写入流，就会断开所有与当前可读流连接的写入流。

前面已经提到了可写流，它有一个write方法用于写数据（接口为`write(chunk, [encoding],[callback])`），end方法通知可写流已经完成写入操作同时触发finish事件，所以前面的pipe代码可以写成：

    raadStream.on('data', function (chunk) {
        writeStream.write(chunk);   // return true if writes successfully
    });
    raadStream.on('end', function () {
        writeStream.end('\nEOF\n');   // return true if writes successfully
    });
    // writeStream.on('finish', function () {
    //    console.log("successfully writes data");
    // });

// TODO: analysis of creating custom stream

下面分析如何创建自定义的流：

    var util = require('util');
    util.inherits(Counter, stream.Readable);
    function Counter(opt) {
        Readable.call(this, opt);   
        this._max = 1000;
        this._index = 1;
    }
    Counter.prototype._read = function () {
        var i = this._index++;
        if (i > this._max)
            this.push(null);
        else {
            var str = '' + i;
            var buf = new Buffer(str, 'ascii');
            this.push(buf);
        }
    }
