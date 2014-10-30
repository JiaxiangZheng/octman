---
title: jQuery源码学习笔记
layout: page
comments: yes
---

# jQuery源码学习笔记

jQuery是前端开发过程中广泛使用的一个库。据统计，Alex排名前100的网站几乎都直接或间接地使用了jQuery库，由此可见其适用性之广，这也从另一个侧面说明其代码的健壮性。因此，学习和研究jQuery是很有价值的，本文也将作为我学习jQuery的笔记，jQuery代码将基于1.7.1版本。将持续更新，

首次使用jQuery选择器的时候，也许会陷入一个认识误区：调用jQuery选择DOM时返回的结果与`document.querySelectorAll`是一样的。但事实并非如此。jQuery返回的是jQuery对象组成的数组，对象内部包含了一个context属性指向相应的DOM结点。对于支持`querySelector*`的浏览器，jQuery选择器引擎会先优先使用`querySelector*`方法；而对于不支持的浏览器（如IE6、IE7等），才会使用选择器引擎去匹配找出相应的结果。

## 基础模块

TODO：浏览器客户端检测、防御式noConflict、对象辅助函数等。

## Sizzle选择器

jQuery最核心的一个功能就是可以通过CSS选择器的方式获取DOM结点，其底层使用的是[Sizzle选择器引擎](https://github.com/jquery/sizzle)。Sizzle选择器的目标很简单：在不支持`querySelectorAll`的浏览器上模拟该方法的查找过程。注：由于多个并列选择器表达式（由逗号分隔的多个表达式）的情况可以归约到单个选择器表达式的情况，因此下面只讨论单个选择器表达式情形。

在Sizzle中，选择器表达式由块表达式（`div`、`.class`、`#id`等，对应`TAG`、`CLASS`和`ID`块表达式类型）和块间关系符（`>`、`+`、`~`和` `）组合形成。当得到表达式后，很直观的想法就是查找匹配的元素（以`div.class>p`），这个过程有自左向右过滤和自右向左过滤两种方式，而Sizzle使用的从右向左扫描的方法。主要原因还是在于效率：从左向右扫描的话，我们总是需要面对未知数量的后代元素，而从右向左的话，父元素或祖先元素总是有限的。

总的来说，Sizzle选择元素的过程大致如下：

1. 解析块表达式与块间关系符
2. 存在伪类，从左向右查找：（注：因为`div p:first`需要过滤的`div p`返回结果中的第一个，因此这时应该从左向右，否则就是先查找`p:first`然后过滤祖先结点为`div`的元素，显然这是不对的。）
    * 查找第一个块表达式匹配的元素集合
    * 遍历剩余块表达式与块间关系符，缩小上下文元素集合
3. 否则从右向左查找：
    * 查找最后一个块表达式匹配的元素集合，得到候选集合
    * 遍历剩余块表达式与块间关系符，对候选集合进行过滤
4. 排序并返回最终匹配的元素（对于并列选择器表达式的情况，依次处理各选择器表达式后还需要对结果进行去重排序）

当然，具体的实现细节可以参考代码中的`Sizzle`入口函数，通过`Sizzle.find`和`Sizzle.filter`方法查找并过滤元素。当然，`Sizzle.find`和`Sizzle.filter`可以看作是比较高层的方法，而底层的查找实际上还是需要根据块表达式类型的不同使用不同的查找方法，底层的过滤细节也需要根据块间关系及伪类标记的不同而有所不同。这里不详细赘述，只记录一些值得注意的地方。

首先是正则表达式的大量运用。Sizzle有一个属性叫`selectors`，映射了各类选择器块表达式类型同相应正则表达式（ID，CLASS，NAME，ATTR，TAG，CHILD，POS，PSEUDO）的关系，这样做的好处是方便扩展。块表达式的各种类型对应的正则表达式如下：

    Sizzle.selectors = {
        match = {
            ID: /#((?:[\w\u00c0-\uFFFF\-]|\\.)+)/,
            CLASS: /\.((?:[\w\u00c0-\uFFFF\-]|\\.)+)/,
            NAME: /\[name=['"]*((?:[\w\u00c0-\uFFFF\-]|\\.)+)['"]*\]/,
            ATTR: /\[\s*((?:[\w\u00c0-\uFFFF\-]|\\.)+)\s*(?:(\S?=)\s*(?:(['"])(.*?)\3|(#?(?:[\w\u00c0-\uFFFF\-]|\\.)*)|)|)\s*\]/,
            TAG: /^((?:[\w\u00c0-\uFFFF\*\-]|\\.)+)/,
            CHILD: /:(only|nth|last|first)-child(?:\(\s*(even|odd|(?:[+\-]?\d+|(?:[+\-]?\d*)?n\s*(?:[+\-]\s*\d+)?))\s*\))?/,
            POS: /:(nth|eq|gt|lt|first|last|even|odd)(?:\((\d*)\))?(?=[^\-]|$)/,
            PSEUDO: /:((?:[\w\u00c0-\uFFFF\-]|\\.)+)(?:\((['"]?)((?:\([^\)]+\)|[^\(\)]*)+)\2\))?/
        },
    }

此外，为了实现不同类型的表达式调用不同函数，还加入字典成员映射ID，CLASS，NAME，TAG到各自的选择器所需的查找函数，如下：

    Sizzle.selectors = {
        find: {
            ID: function () { context.getElementById(); },
            CLASS: function () { context.getElementsByClassName(); },  // 实际上初始化的时候没有CLASS这一项，需要判断浏览器是否支持getElementsByClassName来决定要不要添加
            NAME: function () { context.getElementsByName(); },
            TAG: function () { context.getElementsByTagName(); },
        }
    };

事实上，这种代码形式在Sizzle的filter过滤方法及处理块间关系的时候也是大量地被使用到，可以参考`Sizzle.selectors.relative`属性和`Sizzle.selectors.filter`属性。

## 事件系统

## 参考

* [http://www.zouyesheng.com/jquery.html](http://www.zouyesheng.com/jquery.html)
* [http://davestewart.io/resources/javascript/deconstructed/jquery/](http://davestewart.io/resources/javascript/deconstructed/jquery/)
