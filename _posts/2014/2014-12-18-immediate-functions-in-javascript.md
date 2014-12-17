---
layout: post
title: 深入理解立即函数
categories:
- 编程与开发
tags:
- JavaScript
---

在JS中，立即函数的身影随处可见，一方面它很好地隔离了各模块，另一方面它为私有成员变量的存在提供了可能。

立即函数一般的格式为：

    (function () {})();
    (function () {}());

可是，如果我们尝试写成下面这种：

    function () {} ();

，编译器会毫不留情地告诉我们这是错误的语法。第一眼看过去，你会觉得好奇怪呀，明明连下面这种[奇葩的写法](http://jsperf.com/self-invoking-function)都是没有问题的：

    void function () {} ();
    !function () {} ();
    +function () {} ();
    -function () {} ();
    ~function () {} ();

尼玛，为什么会这样？这还得从javascript中的函数说起。我们知道，在js中，函数是也是一个对象，它可以通过`new function(fun_body)`来构造，但通常更多的是使用**函数表达式**或**函数声明式**：

    // 函数表达式
    var fun = function () {};
    // 函数声明式
    function fun() {}

虽然js是解释性语言，但并不意味着解析器不会对代码进行任何的预处理。函数表达式与函数声明式的区别就在于，在代码执行前，解析器会通过一个*函数声明提升*的过程将函数声明提前读取并放到代码树顶部，所以下面的代码完全可以正常运行：

    fun();
    function fun() {} 

但对于函数表达式，就不会有这样的一个函数声明提升过程，所以上面的代码如果fun是表达式就无法正确地运行。

回到前面提到的`function () {} ();`这种形式的代码，前面的`function () {}`会被解析器认为是函数声明式，函数声明式的后面是不允许有`()`的。如果我们把函数用`()`包起来，因为`()`内部只能是表达式，所以解析器就会认为它是函数表达式，从而运行起来就没有问题，前面提到的所谓的奇葩的方法，也都是相当于告诉解析器这是一个函数表达式，而不至于出现解析上的偏差。所以下面这种情况是没有问题的：

    // var fun = function () {} 表明是一个函数表达式
    var fun = function () {
    } ();

有关于函数表达式和函数声明式的细节，可以参考[这篇文章](http://kangax.github.io/nfe/)，本文是参考了stackoverflow上的[一个回答](http://stackoverflow.com/questions/1634268/explain-javascripts-encapsulated-anonymous-function-syntax/1634321#1634321)。


