---
layout: post
title: 读书笔记-《High Performance JavaScript》
categories:
- 阅读与思考
tags:
- JavaScript
- 读书笔记
---

JS是一门脚本语言，其设计之初并没有像Go一样带有与生俱来的性能优势，它最初主要设计用于替代服务器端作简单的合法性验证，因此性能问题并没有在一开始设计的考虑范围之内。但随着Web的发展，JS已经发展得越来越复杂，性能问题也呼之欲出，因此写出高性能的代码很有必要。《High Performance JavaScript》这本书总结了一些JS的坑及可能优化的地方。

## 加载与执行

要在网页中使用JS，需要将它加载到浏览器端的JS引擎中执行。JS的加载方式有多种，如文件加载、直接嵌入在 HTML 的 `<script>` 标签中。但是，当浏览器遇到JS时，会阻塞所有其余内容的执行（如网页的渲染），从而导致极有可能出现长时间的网页空白情形。当外部文件过多的时候，还需要进行HTTP请求，这也会延长整个的加载时间。为此，加载JS文件的时候：

1. 尽量减少HTTP请求，即减少外部JS文件的数目（这可以通过将多个JS文件合并到一个文件达到目的）
2. 在`script`标签中尽量使用`defer`和`async`以非阻塞形式加载JS文件
3. 动态脚本文件加载：即在完成DOM以后通过创建`script`结点添加到DOM中实现动态加载，而且我们还可以为该结点添加`onload`事件（IE还支持`readystatechange`事件），当文件加载完成后进行相应的回调处理
    * 注意如果多个JS文件间是有依赖关系的，那就只能通过不断地嵌套callback函数进行处理了，这也进入了所谓的*callback hell*状态
4. XHR注入加载：可以通过AJAX异步读取服务器上JS文件再创建script结点，但其劣势很明显，即只能读取同域名下的文件，无法引用外部文件
5. gzip压缩有时候往往可以极大地减小文件的大小，减少传输时间

## 变量读取

TODO

## DOM操作性能优化

通常说的浏览器端的JavaScript包括ECMAScript和DOM，DOM的操作相对于核心的ECMAScript来说性能很低，因此在程序中能使用ECMAScript处理的内容就少用DOM，而且为了保证跨浏览器，最小化DOM与ECMAScript的耦合也是非常重要的。

为实现JS丰富地操作动态页面，经常需要动态插入或删除DOM结点。但是，这往往会造成页面布局的重新计算（即所谓的页面回流），引起一定的性能问题。如果我们能够尽量避免页面回流，或者减小回流影响范围，就可以提高整个程序的性能了。

1. 与其通过修改DOM的style来切换样式（每设置一个就会导致一次reflow），不如设置它的`className`。此外，这也更好地实践了样式与行为的分离。
2. 在flow以外对DOM进行操作
3. 创建DOM的时候，与其插入结点再设置相关样式属性，倒不如先设置属性再插入到页面的DOM中。
4. 有很多情况下，我们需要创建不止一个结点，按前面第2点，先把所有结点的属性设置好，然后再插入DOM中。使用`document.createDocumentFragment`当作这些结点的父结点，然后再将这个`DocumentFragment`当作真实DOM中父结点的子结点。

关于第2点，可以参考如下代码：

	/**
	 * Remove an element and provide a function that inserts it into its original position
	 * @param element {Element} The element to be temporarily removed
	 * @return {Function} A function that inserts the element into its original position
	 **/
	function removeToInsertLater(element) {
	  var parentNode = element.parentNode;
	  var nextSibling = element.nextSibling;
	  parentNode.removeChild(element);
	  return function() {
	    if (nextSibling) {
	      parentNode.insertBefore(element, nextSibling);
	    } else {
	      parentNode.appendChild(element);
	    }
	  };
	}
	function updateAllAnchors(element, anchorClass) {
	  var insertFunction = removeToInsertLater(element);
	  var anchors = element.getElementsByTagName('a');
	  for (var i = 0, length = anchors.length; i < length; i ++) {
	    anchors[i].className = anchorClass;
	  }
	  insertFunction();
	}

具体量化的结果大致可以参考下图：

![](/images/2014/reflow-chart.png)

注：关于页面的回流与重绘性能优化，参考了谷歌开发者中的[一篇文章](https://developers.google.com/speed/articles/javascript-dom)。

## 算法层面的优化

算法的好坏对程序的性能会有重大的影响，这适用于任何编程语言，而流程控制的使用也会影响程序的性能。

在JS中，不同循环的使用性能也有一定的影响，如`for in`的性能就明显不如普通的`for`循环（因为`for in`在循环过程中会找prototype里的属性）。另外对长度的缓存也可以很大幅度地提高性能：

    // 第一种情况性能不如第二、三种情况，
    for (var i = 0; i < arr.length; i++)
    for (var i=0, len=arr.length; i < len; i++)
    for (var i = arr.length - 1; i--; )

如果其中的arr是类似于`getElementsByTagName`这种函数返回的与DOM相关的数组，性能差距更是巨大。JS的数组支持`forEach`方法，它的性能也不如普通的`for`循环，其原因就在于其中的每次循环会对元素作各种检查，同时关联的一个额外函数调用也会导致性能的下降。

对于条件判断比较少，用`if-else`；而一旦判断的条件多了，用`switch-case`是一个更明智的选择。事实上，`switch`语句大部分情况下都比`if-else`要快，当条件多的时候，这个差异就会很明显。但是，真的是条件太多，其实查表更快，而且还有一个好处是这样代码会写得很清晰简洁，我们平时应该提倡这个方法的使用。

递归：想想二叉树的很多算法，通过递归是多么地简洁。但递归并不是没有缺陷，首先是函数调用的消耗，其次是受调用栈的大小限制。对JS而言，不同浏览器的实现，调用栈大小不同。一般来说，如果出现stack overflow的情况，就会抛出异常。一般情况下，如果递归比较容易转换成迭代的方法，最好使用迭代的方法。

## 字符串与正则表达式

拼接字符串是一个常见的操作，不同的实现方法性能也不一样。最简单的是`+`和`+=`运算符，现代浏览器基本上都对它们进行了优化。但是我们还是需要深入看一下它们：

    str += "one" + "two"    // 会分配一个临时的字符串存放"one" + "two"的结果
    str += "one"; str += "two"; // 可以避免临时分配
    str = str + "one" + "two"; // 可以避免临时分配
    str = "one" + str + "two"; // 出现临时分配，优化失效

当然，我们知道`Array.join`方法也是可以用来拼接字符串的，但值得一提的是，它在现代浏览器中的性能比前二个都要低，*但在IE7及更早IE版本里，这却是唯一高效的拼接方法*（IE7原生拼接的方法需要反复地复制及分配内存，从而导致效率的低下）。

而`Array.concat`与简单的`+`和`+=`运算符效率差别不是太大。

下面的代码用于测试字符串连接与数组连接生成字符串的对比：

    console.time("String Plus");
    var str = "I'm a thirty five character string.",
        newStr = "",
        appends = 5000000;
    while (appends--) {
        newStr += str;
    }
    console.timeEnd("String Plus");

    console.time("Array Concatting");
    var str = "I'm a thirty five character string.", 
        newStr = "", 
        arr = [],
        appends = 5000000; 
    while (appends--) {
        arr.push(str);
    }
    newStr = arr.concat("");
    console.timeEnd("Array Concatting");

正则表达式：TODO

## 响应式接口

TODO

## AJAX

AJAX可以在不刷新页面的情况下对服务器发请求，从而极大提高了 Web 页面可交互性。各客户端通过 XMLHttpRequest 对象（IE使用ActiveObjectX）提供 AJAX 服务。出于安全性上的考虑，XHR无法提供跨域的数据获取，即只能请求服务器上的数据。另外，XHR提供`GET`和`POST`方法，但对于我们通常的请求，更多的是`GET`方法。事实上，`GET`请求结果可以被缓存到客户端，这极大提高了页面的性能，但它的局限在于所有的参数都是从URL上可见的，而且整个的URL长度有一定的限制，这时就只能用`POST`请求了。

打开一个 WEB 页面的时候，会去请求与这个页面相关的所有资源，一份资源就是一个请求，从而可能引起页面加载时间过长的问题。为此，我们需要尽量减少HTTP请求，这也是为什么JS和CSS相关的文件普遍被打包到一个文件中（压缩主要是降低文件大小，减小传输时间）。对于 XHR，如果我们需要请求多份文件，也是会引起多个请求。一个方法是可以在服务器端将多个文件合并成一个，在客户端负责重新分块。举个例子，如果请求多张图片，可以在服务器端将图片使用base64编码，使用与客户端特殊约定的分隔符分隔不同图片，组成一个长的字符串，然后在客户端重新划分得到原来的多张图片，而这些全在一个请求中完成，从而提高性能。

多次对同一份资源进行请求，我们可以通过缓存提高性能，从而就只需要首次发出请求了。两种普遍的方法可以阻止不必要的缓存：

1. 服务器端在HTTP头中设置响应的生存时间（如`Expires: Mon, 28 Jul 2014 20:00:00 GMT`）
2. 客户端使用本地存储

## 编程实践及应用

在平时的编程过程中，我们经常会被教导要这么写而非那么写，后面这几章就是在讲一些最佳实践相关的内容。

举个很简单的例子，很多人都告诫我们不要用`eval`，问及原因，他们会说，性能上的考虑。好吧，那为什么使用`eval`就会有性能上的劣势呢？原因在于这会引起两次evaluation penalty，即会对代码进行两次预处理。PS：事实上，近年来eval的性能问题已经没有那么明显了，更重要的问题在于调试困难、**易受攻击**。与之相关的还有`Function`、`setTimeout`和`setInterval`，可以参考[^1]。

与浏览器相关的JS代码中很多的API需要根据客户端的不同作一定的处理，比如`addEventListener`和`attachEvent`，通常的做法是包装一个自己的`addHandler`函数，在其中使用条件判断选择使用不同的API。但每次调用`addHandler`都会进入条件判断，更好的一个方法是只在第一次调用这个函数的时候进入条件判断，因为浏览器作为一个执行环境，一旦确定了客户端那么API也就固定下来了。

JS中有许多内置的方法，如`Math.log(num)`，在平时的实践中，能用这些方法就不要用自己写的方法。因为内置的方法是通过低层语言如 C++ 编译成机器码而集成的执行环境中的，那肯定比我们自己写的要快了。


PS：我发现[jsPerf](http://jsperf.com/)真是好东西！

[^1]: http://www.nczonline.net/blog/2013/06/25/eval-isnt-evil-just-misunderstood/
