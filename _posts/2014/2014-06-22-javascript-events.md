---
layout: post
title: JavaScript事件处理
categories:
- 编程与开发
tags:
- JavaScript
---

JavaScript用于操纵HTML页面的行为，而它与HTML之间的交互主要就是通过**事件**进行实现的。无论是Web上，还是传统的界面UI（如QT等），一般都是使用观察者（Observer）模式监听事件并触发行为的发生。

早期，众多浏览器都有其自己的事件实现方式，导致API也是各异，而DOM2级作为规范在一定程度上统一了这些浏览器的事件API。下面的内容主要包括：

* 事件流
* 事件处理程序
* 事件对象

### 事件流

事件流用于描述从页面中接收事件的顺序，举个例子，当我们点击页面中一个按纽时，相当于是触发了一个事件，但我们点击按纽的同时，事实上还点击了包围按纽的元素，还点击了相应的window对象，还点击了相应的html页面……。这些元素可以看作是一个一层套一层的结构，当我们点击最里层的元素时，实际上也点击了所有包围它的所有元素。事件流指的就是事件触发的一个顺序，到底是从最里面到最外面元素依次触发还从外到里触发的一个顺序。

事实上IE和网景提出两种相反的事件流概念：IE提出**事件冒泡流**（从最里层元素到最外层），而网景提出**事件捕获流**（从最外层元素到最里层）。

而DOM2级规范则规定事件流包括如下三个阶段：

1. 事件捕获阶段：从document开始到最具体的一个元素上层，这期间目标不会接收事件
2. 处于目标阶段：事件在具体元素上发生并在事件处理中被看作冒泡阶段的一部分
3. 事件冒泡阶段：事件传播回文档

### 事件处理程序

事件都有其名称，如单击鼠标的事件名称为`click`，而响应某个事件的函数即事件处理程序，以`on-`开头，如`onclick`。为事件指定处理程序的方式有如下几种：

**HTML事件处理程序**：可以在HTML页面中可以直接在标签内指定事件属性，比如下面的一段HTML语句就指定单击按纽的时候会触发一个警告窗口显示`click me`，当然也可以直接`onclick="fun()"`来指定一个函数作为要执行的动作。

    <input type="button" value="click" onclick="alert('clicked me')" />

**DOM0级事件处理程序**：由于元素有`onclick`这个属性，我们可以直接对其进行赋值设置其触发动作，可以将该属性值设置为`null`以删除其指定的事件处理程序。

    var btn = document.getElementById("btn");
    btn.onclick = function() {
    }

**DOM2级事件处理程序**：使用`addEventListener`可以将事件处理添加到监听列表中，它的最后一个参数接受布尔值，为真则表示捕获阶段调用事件处理程序；为假则在冒泡阶段调用。注意，使用`addEventListener`添加的事件处理程序只能由`removeEventListener`删除。大部分情况下，最后一个参数应该设置为`false`以便程序添加到事件的冒泡阶段。另外，需要注意的是，在删除事件处理程序的时候，接受的第二个参数需要与添加时的参数完全一样，即不能使用匿名函数，而应该是一个函数的引用。

    var btn = document.getElementById("btn");
    btn.addEventListener("click", function() {}, false);

**IE事件程序程序**：IE8及前面的版本中处理事件程序方法与DOM方法不太一样，首先，它对应的方法名分别为`attachEvent`和`detachEvent`；其次，事件的名称是以`on`开头的；另外，它只支持事件冒泡，因此它只有两个参数。

    var btn = document.getElementById("btn");
    btn.attachEvent("onclick", function() {});

总结来说，要保证事件处理程序能够兼容各浏览器，我们需要处理上面的所有情况，自己编写代码的话，可以使用类似于如下的代码：

    var EventUtil = {
        addHandler: function(evt, type, handler) {
            if (evt.addEventListener) { // DOM2
                evt.addEventListener(type, handler, false);
            } else if (evt.attachEvent) {   // IE
                evt.attachEvent("on"+type, handler);
            } else {    // DOM0
                evt["on"+type] = handler;
            }
        },
        removeHandler: function(evt, type, handler) {
            // ...
        }
    };

### 事件对象

在前面提到的事件处理程序（即handler）中，没有任何的参数；而实际上，触发DOM上某一个事件的时候，会自动产生一个事件对象event，包含着与该事件相关的所有信息。例如，鼠标点击导致的事件对象中，会包含鼠标位置及点击的元素等信息。

注意：事件对象并不是一个特殊的JS类型，只是一个增加特定属性的对象Object。

对于HTML事件处理程序，它会自动包含一个event对象，所以前面的程序中可以直接使用event对象。

    <input type="button" value="click" onclick="alert(event.type)" />

兼容DOM标准（DOM0和DOM2）的浏览器会将事件对象传入事件处理程序中，即handler程序会有一个event参数。由于不同事件对象的触发事件类型不一样，所以可用方法及属性也不完全相同，有几个比较重要的通用属性和方法：

1. `bubbles`表明事件是否冒泡
2. `cancelable`表明是否可以取消事件默认行为
3. `eventPhase`表示调用事件处理程序的阶段（1表示捕获，2表示处于目标，3表示冒泡阶段）
4. `preventDefault()`用于取消事件默认行为（如单击链接会跳转，鼠标右键会调用浏览器系统菜单）
5. `stopPropagation()`取消事件的进一步捕获或冒泡
6. `target`表示事件的目标

IE的事件对象又与前面的有所不同，首先它的事件是window对象的一个属性，因此需要使用`window.event`得到，它的事件冒泡就是直接修改`cancelBubble`属性的值，而事件的目标则是`srcElement`属性，默认行为的取消也只是通过设置`returnValue`为`false`。

总结来说，要想得到一个兼容多浏览器的事件对象解决方案，需要重新写一个函数考虑上面的多种方法。

    function preventDefault(evt) {
        if (evt.preventDefault) {
            evt.preventDefault();
        } else {
            evt.returnValue = false;
        }
    }
    function getEvent(evt) {
        return evt ? evt : window.event;
    }
    function getTarget(evt) {
        return evt.target || evt.srcElement;
    }
    function stopPropagation(evt) {
        if (evt.stopPropagation()) {
            evt.stopPropagation();
        } else {
            evt.cancelBubble = true;
        }
    }
