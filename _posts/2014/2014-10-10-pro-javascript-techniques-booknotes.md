---
layout: post
title: 读书笔记-《Pro JavaScript Techniques》
categories:
- 阅读与思考
tags:
- JavaScript
- 读书笔记
---

jQuery的作者John Resig写过两本关于JavaScript的书：《Pro JavaScript Techniques》和《Secrets of the JavaScript Ninja》，都是介绍关于JS的一些比较高层次的用法，对于JS进阶大有裨益。本文是我读《Pro》这本书的笔记。

## 第二章

JS包含对象这个概念，可与其它面向对象不一样的是，它没有类这个概念，不过它却有构造函数这个概念。JS中，函数是一等公民，任何函数都是有成为构造函数的可能性的。一般高级语言使用`new Class()`创造对象，其中`Class`为构造函数，而JS中，使用`new Cstr()`构造一个对象，`Cstr`即这里的构造函数。一旦构造对象后，对象就有一个`constructor`属性指向这个构造函数。

其它高级语言使用类的树状关系来实现继承，但在JS中，使用的是原型链继承。

JS的重载与其它语言也不一样，像C++是通过name mangling达到的，但JS每个名字在同一个作用域下只对应一个函数，如何实现重载呢？通过判断`arguments`这一每个函数都有的属性。当然，另一种方式是判断参数是否为`undefined`。

作用域（scope）的概念在其它语言中由大扩号界定，但在JS中则是只由函数域界定。（当然最新版本的JS标准允许使用let关键字以扩号作为作用域）。

JS作为一种函数式编程语言，闭包的概念必不可少。事实上，闭包在JS中广泛使用。由于JS本身的语言缺陷，导致作用域只能由函数界定，为了避免函数中出现大量的全局变量，可以使用一个匿名立即执行函数来模拟作用域或命名空间。

在面向对象的语言中，在一个成员函数中this指代其所属上下文对象，而JS中也是如此，一个函数中出现了this，那么这个this就表示一个相应的上下文。如果该函数并没绑定到任何对象，那即默认的情况下，它绑定到的是一个全局的对象，在浏览器端是windows，而在NodeJS中是global。可以使用`call`或`apply`改变上下文。

乍看JS似乎觉得它里面并没私有成员、公有成员、静态成员的概念。但是，通过闭包，我们是可以模拟出这些的。

## 第三章 创造可复用的代码

前面提到JS是使用不同于类继承的方式实现面向对象的编程方法，它使用原型继承。下面将通过原型继承实现其它高级编程语言中所包含的类继承与多重继承。

在Douglas Crockford的一篇[关于JS继承探讨](http://javascript.crockford.com/inheritance.html)的文章中，有很多可以值得学习的示例。

    // 可以直接通过调用函数的method方法给该函数的原型添加一个方法了
    Function.prototype.method = function (name, func) {
        this.prototype[name] = func;
        return this;
    }

    // 对于任何构造函数Cstr，通过Cstr.inherits(Parent)即可继承Parent构造函数的特性
    Function.method("inherits", function (Parent) {
        var depth = 0,
            proto = this.prototype = new Parent();

        // 单链式多重继承
        this.method("uber", function (name) {
            var func, ret, v = Parent.prototype;
            if (depth) {
                for (var i = d; i > 0; i--) {
                    v = v.constructor.prototype;
                }
                func = v[name];
            } else {
                func = proto[name];
                if (func == this[name]) {
                    func = v[name];
                }
            }
            depth += 1;

            ret = func.apply(this, Array.prototype.slice.apply(arguments, [1]));

            depth -= 1;
            return rest;
        });

        return this;
    });

    // 继承Parent的部分属性，注意，这里prototype应该是一个普通的Object，而非Parent
    Function.method("swiss", function (Parent/*, methodName1, methodName2, ...*/) {
        for (var i=1; i<arguments.length; i++) {
            var name = arguments[i];
            this.prototype[name] = Parent.prototype[name];
        }
        return this;
    });

另外，可以参考之前的一篇关于`mixin`的笔记。

在C++中，可以使用`namespace`创建出命名空间，从而更好地避免变量名的污染。但是，JS本身是不支持命名空间的，要想达到类似于C++中`namespace`的特性，可以借助于JS的对象概念。

    // 为便于认识，可以与std::vector::size类比
    // std.vector.size 或 std.vector.push_back
    var std = {};
    std.vector = {
        size: function () {},
        push_back: function () {}
    };

但是，值得注意的是，JS中对象是使用哈希表存储，一旦名字链写得很长，性能上会有一定的问题，因此推荐的做法是缓存变量，即`var vec = std.vector`，然后就可以提高名字查找链的效率了。

## 第五章 DOM

JS可谓成也DOM，败也DOM。如果没有DOM，也许JS不会像现在这么流行，但是另一方面，如果没有DOM，也许JS所受的诟病没这么多。


    <html>
    <head>
        <title>Introduction to the DOM</title>
    </head>
    <body>
        <h1>Introduction to the DOM</h1>
        <p class="test">There are a number of reasons why the DOM is awesome, here are some:</p>
        <ul>
            <li id="everywhere">It can be found everywhere.</li>
            <li class="test">It's easy to use.</li>
            <li class="test">It can help you to find what you want, really quickly.</li>
        </ul>
    </body>
    </html>

在上面的HTML页面中，其DOM结构看似比较简单，body后的`firstChild`应该就是`h1`了吧？但是，事实并非如此，body的`firstChild`是一个空的text结点！我们可以把这个问题当作DOM的一个BUG，但现在当务之急是需要清除这些讨厌的空格！可使用如下代码：

    function cleanWhitespace( element ) {
        // If no element is provided, do the whole HTML document
        element = element || document;
        // Use the first child as a starting point
        var cur = element.firstChild;
        // Go until there are no more child nodes
        while ( cur != null ) {
            // If the node is a text node, and it contains nothing but whitespace
            if ( cur.nodeType == 3 && ! /\S/.test(cur.nodeValue) ) {
                // Remove the text node
                element.removeChild( cur );
                // Otherwise, if it's an element
            } else if ( cur.nodeType == 1 ) {
                // Recurse down through the document
                cleanWhitespace( cur );
            }
            cur = cur.nextSibling; // Move through the child nodes
        }
    }

但通常显然我们并不一定要先对文档DOM进行预处理，这时更有效的方法是在查询的时候忽视掉文本结点（document.TEXT_NODE）。

另外，关于DOM的操作包括属性操作、DOM结点的添加删除、文本的提取、innerHTML页面注入等。但是，需要注意的是，涉及到DOM的操作往往耗时（需要考虑到浏览器引擎绘制及页面的重新布局等因素），因此需要尽量减少DOM的直接操作。

