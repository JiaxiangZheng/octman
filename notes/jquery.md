---
title: jQuery源码学习笔记
layout: page
comments: yes
---

# jQuery源码学习笔记

jQuery是前端开发过程中广泛使用的一个库。据统计，Alex排名前100的网站几乎都直接或间接地使用了jQuery库，由此可见其适用性之广，这也从另一个侧面说明其代码的健壮性。因此，学习和研究jQuery是很有价值的，本文也将作为我学习jQuery的笔记，将持续更新。

首次使用jQuery选择器的时候，也许会陷入一个认识误区：调用jQuery选择DOM时返回的结果与`document.querySelectorAll`是一样的。但事实并非如此。jQuery返回的是jQuery对象组成的数组，对象内部包含了一个context属性指向相应的DOM结点。对于支持`querySelector*`的浏览器，jQuery选择器引擎会先优先使用`querySelector*`方法；而对于不支持的浏览器（如IE6、IE7等），才会使用选择器引擎去匹配找出相应的结果。

## 基础模块

关于jQuery的整体框架结构，我们可以参考它提供的一个sub方法（返回新的jQuery对象，修改这个对象不影响原来的jQ对象）：

	sub: function() {
        // 工厂方法
		function jQuerySub( selector, context ) {
			return new jQuerySub.fn.init( selector, context );
		}
        // 注意这里使用了深度拷贝
		jQuery.extend( true, jQuerySub, this );
		jQuerySub.superclass = this;
		jQuerySub.fn = jQuerySub.prototype = this();
		jQuerySub.fn.constructor = jQuerySub;
		jQuerySub.sub = this.sub;
		jQuerySub.fn.init = function init( selector, context ) {
			if ( context && context instanceof jQuery && !(context instanceof jQuerySub) ) {
				context = jQuerySub( context );
			}

			return jQuery.fn.init.call( this, selector, context, rootjQuerySub );
		};
		jQuerySub.fn.init.prototype = jQuerySub.fn;
		var rootjQuerySub = jQuerySub(document);
		return jQuerySub;
	},

这里需要注意的是`jQuery.fn`，即jQuery这个工厂函数的原型。它是一个对象：

    jQuery.fn = jQuery.prototype = {
        constructor: jQuery,    // constructor指回jQuery工厂函数
        init: function () {},   // jQuery最核心的选择器匹配函数
        selector: "",
        jquery: '1.7.1',
        each: function () {},
        ready: function (fn) {},
        ...
    };

从源码中，我们可以看到，jQuery这个工厂函数实际上最终也是调用了`jQuery.prototype.init`这个方法实例出来的一个新对象，所以`jQuery.prototype.init`可以看作一个构造函数，它的原型也是指向`jQuery.prototype`。

jQuery提供了一个each方法，可以对对象每一个属性成员都调用同一个回调函数`callback.apply()`。

为了实现多层对象的继承，jQuery提供extend这个API。其接口为：

    // 支持深度拷贝
    // 如果参数只有一个，则是扩展jQuery本身，否则扩展传入的目标对象
    $.extend([deep], target, [object1], [object2], [...]);

这里如果提供的参数包含深度拷贝，则需要使用递归的方法扩展其中的某些属性（处理对象Object和数组Array两种情形）。

有了extend这个方法，对jQuery本身（或其原型）的扩展就变得非常地方便了：

    jQuery.extend({
        prop: value,
        fun:  function () {}
    });

`jQuery(document).ready | jQuery(fn)`与`dom.onload`区别在于，一旦DOM构建好以后，ready事件就可以被触发了，而onload只会在所有DOM及脚本图像等资源处理好以后才开始。jQuery是怎么实现的呢？它会有一个计时器(setTimeout)每隔一定时间定期查询`document.body`是否已经构建完成，完成的时候就会触发相应的回调队列（见后节），这个回调队列的创建由bindReady方法构建。

此外，为了确定浏览器的版本信息，jQuery使用了正则表达式进行处理以匹配出合适的浏览器内核名称、版本等相关信息。PS：对`navigator.userAgent`字符串进行解析就可以获得这些信息。

## 属性操作

jQuery的属性操作包括HTML属性操作（attr）、DOM属性操作（prop）、CSS操作（class）和值操作（val）。总体上说，`jQuery.prototype`和`jQuery`都包含了同名的各操作方法，但它们之间的关系是（以attr为例）：`jQuery.fn.attr`里会通过`jQuey.access`方法作一定的检查，而后再调用到`jQuery.attr`处理具体的问题。所以本质上，主要的实现是在jQuery相关的方法中。

**TODO: jQuery.attr和jQuery.prop的实现及jQuery.access方法**。


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

## 异步队列

异步队列模块（Deferred）在jQuery中主要用于实现异步任务与回调函数的解耦，为其它模块提供了基础，它基于Callbacks实现。先了解一个简单的异步队列使用情形：

传统上在jQuery中发一个ajax请求代码如下：

    $.ajax({
        url: "/echo/json/",
        data: {json: JSON.stringify({firstName: "Jose", lastName: "Romaniello"})} ,
        type: "POST",
        success: function(person){ 
            alert(person.firstName + " saved."); 
        },
        error: function(){ 
            alert("error!");
        } 
    });

其中使用了两个回调，一个在请求成功返回后将被调用，而另一个则是处理错误发生的情况。但是，使用异步队列后，上面的请求就可以很简单地写成：

    // $.ajax返回的是一个具有Promise接口的对象
    $.ajax({
        url: "/echo/json",
        data: {json: JSON.stringify({firstName: "Jose", lastName: "Romaniello"})},
        type: "POST",
    }).done(function (person) { alert(person.firstName + " saved."); } )
      .done(function () { console.log("成功返回"); })
     .fail(function () { alert("error!"); });

下面先大致地描述一下`jQuery.Callbacks`的一些接口：

TODO：Callbacks的实现。

事实上，Deferred模块提供了三种状态的回调列表：resolved、rejected和progress状态（每种状态对应一个回调队列Callbacks实例）。每种状态的回调队列都有各自的添加回调函数的方法，分别是done、fail和progress，实际上它们都是Callbacks的add方法的一个包装。当然，jQuery还提供了一个always方法用于添加回调函数，实际上就是done和fail都会添加这个回调，这样看来还是调用了done和fail。然后jQuery提供了三个执行（触发）函数：resolve、reject和notify（当然，还提供了对应的With函数用于设定自定义的上下文）。对于这三种情况，jQuery的实现还是很值得学习的：

    // tuples定义一个数组，包含三种状态的触发函数名、回调函数添加方法、异步队列及状态名称
    // 这样接下来对三种情况的处理就可以放在一个forEach里了
    var tuples = [
        // action, add listener, listener list, final state
        [ "resolve", "done", jQuery.Callbacks("once memory"), "resolved" ],
        [ "reject", "fail", jQuery.Callbacks("once memory"), "rejected" ],
        [ "notify", "progress", jQuery.Callbacks("memory") ]
    ],

对于Deferred的实现，jQuery通过复制了一个只读的副本promise的方法来扩展Deferred的方法，从而使得自身具有Promise属性。**TODO**：Promise可以看作是将异步回调嵌套线性化的一种标准，有机会将深入研究。promise提供了一个叫promise的方法，通过传入一个对象，会将promise所拥有的方法扩展到该对象上，Deferred对象的实现就是如此（`promise.promise(deferred)`）。

由于其它方法比较简单，这里只谈一下`promise.then`方法，实际上它在低版本中的方法名为pipe，用于返回新的异步队列的只读副本，通过过滤函数过滤当前异步队列的状态及值：

    // TODO: 分析
    then: function( /* fnDone, fnFail, fnProgress */ ) {
        var fns = arguments;
        return jQuery.Deferred(function( newDefer ) {
            jQuery.each( tuples, function( i, tuple ) {
                var fn = jQuery.isFunction( fns[ i ] ) && fns[ i ];
                // deferred[ done | fail | progress ] for forwarding actions to newDefer
                deferred[ tuple[1] ](function() {
                    var returned = fn && fn.apply( this, arguments );
                    if ( returned && jQuery.isFunction( returned.promise ) ) {
                        returned.promise()
                            .done( newDefer.resolve )
                            .fail( newDefer.reject )
                            .progress( newDefer.notify );
                    } else {
                        newDefer[ tuple[ 0 ] + "With" ](
                            this === promise ? newDefer.promise() : this,
                            fn ? [ returned ] : arguments
                        );
                    }
                });
            });
            fns = null;
        }).promise();
    }

以上就是异步队列模块的全部内容。

## 数据缓存

## 队列模块

## 事件系统

## 异步请求

## 动画生成

## 参考

* [http://www.zouyesheng.com/jquery.html](http://www.zouyesheng.com/jquery.html)
* [http://davestewart.io/resources/javascript/deconstructed/jquery/](http://davestewart.io/resources/javascript/deconstructed/jquery/)
