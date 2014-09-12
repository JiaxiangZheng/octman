---
layout: post
title: 重新理解text-decoration属性
categories:
- 编程与开发
tags:
- CSS
---

几周以前，改过一个与text-decoration相关的issue，这里记录一下，以免以后犯同样的错误。

在CSS中，有些元素的样式是可以继承的，即在`<div><span>text</span></div>`的div结构上设置字体加粗后，span也会继承这个样式。但是，也有一类无法继承的样式，其中就包括text-decoration。

单纯地说它没有继承的特性可能不那么直观，这里举个很简单的例子。`<p style="text-decoration:underline"><strong>should be underline</strong></p>`结构中， strong 标签内的元素实际上是没有继承到下划线的特性的。也许你试验后会发现 `should be underline` 这几个文字下面是出现了下划线的，但这个下划线其实并非`strong`元素本身留下来的，而是它的父结点 p 所留下来的。为了验证这一点，可以将strong标签的style设置为`style="text-decoration:none"`，你会发现下划线依然存在。在Chrome的调试窗口中，你会发现`strong`标签的CSS确实包含了`style="text-decoration:none"`这一项。事实上，这是因为父结点的样式一不小心正好重叠在子结点上了。如果你把子结点的位置移开（如使用position），你就会发现子结点事实上是没有下划线的。

下面我们要讨论两个比较具体的问题：

1. 父结点上有下划线，当子结点是漂浮 (float) 在父结点上，如何将下划线显示出来？
2. 父结点上有下划线，如何将重叠在子结点的下划线去掉？

针对第一个问题，如果子结点与父结点的layout关系不是浮动的，那么我们是可以利用父结点的下划线重叠解决这个问题的。但是，关键在于，如果子结点是浮动于父结点上的时候，就会出现前面无法重叠的情况了。可以参考这个[演示](http://jsfiddle.net/Tsung/8bFZx/)。关于这一点，我们看看W3标准是怎么说的：

> Note that text decorations are not propagated to floating and absolutely positioned descendants, nor to the contents of atomic inline-level descendants such as inline blocks and inline tables.

因此，我们不能通过在父结点上设置text-decoration来达到子结点显示的目的。目前，我用的解决方法是在父结点加一个`class`，然后在CSS中写一个长的class链直接指定到子结点上。另一种方法当然，也可以直接在子结点上加相应的`class`，然后就可以只定义子结点的style了。

那么，如果是类似于本文最开始的例子那样，想要把strong标签设置成`text-decoration: none`却没有成功，怎么解决呢？其实第一个问题可以提供一个很好的答案，即设置strong的显示为`display: inline-block`。

但是，最好的方法，其实还是尽量避免利用父结点的text-decoration显示来叠加到子结点的显示上。

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

前面我们提到如果在父结点上设置`text-decoration`属性，并且在子结点上能看到这个效果的话，那是因为父结点的效果重叠在了子结点上。但是，这个重叠就真的会恰好与子结点本来应该有的效果叠在一起吗？看下面这个例子：

HTML：

    <div style="text-decoration: line-through;">
        <span>text line-through is not align well. </span>
        <ul>
            <li>2008</li>
            <li>2009</li>
            <li>2010</li>
        </ul>
    </div>

CSS：

    div span {
        text-decoration: line-through;
        font-size: 50px;
    }
    div ul li {
        text-decoration: line-through;
        font-family: Arial;
        font-size: 50px;
    }

在Chrome中执行上述的代码，不管我们如何增大子结点的font-size，我们会发现看到的始终都只是一条线。但是，在FireFox中，随着字体的增大，两条线分离得越来越明显。这个问题怎么破呢？没办法，这是浏览器自身的实现差异，也许是FF的一个BUG，无解了。**看来，最好的方法，就是在设置text-decoration属性的时候，应该把它直接设置在要显示的元素结点上**。

另外，如果细心一点，会发现例子中的`line-through`会显得离中间位置偏下，这一点也是浏览器的绘制引擎本身所导致的。要想解决这个问题，只能使用一点小小的trick（事实上不太被推荐），如使用带删除线的背景图片来代替CSS的删除线，或者可以使用类似下面的方法，用边界线来替代CSS删除线（极度依赖位置的调整，如果字体变了，又得重新调整）：

HTML：

    <div>
        <p class="strikethrough">This is an 
            <span class="mark">
                <span class="offsetMark">example </span>
            </span> of how I'd do it.
        </p>
    </div>

CSS：

    .mark {
        border-bottom: 1px solid #000;
        top: -9px; /* Tweak this and the other top in equal, but opposite values */
        position: relative;
    }
    .offsetMark {
        position: relative;
        top: 9px; /* Tweak this and the other top in equal, but opposite values */
    }

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 

归根结底，我觉得**在使用`text-decoration`这一类属性的时候，需要特别注意将它直接作用到需要作用的元素上，而非其父结点上**。

