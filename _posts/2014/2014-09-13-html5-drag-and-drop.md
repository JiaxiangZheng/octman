---
layout: post
title: HTML5中的原生拖放事件
categories:
- 编程与开发
tags:
- HTML5
- Web
- JavaScript
---

原生HTML5可实现元素的拖放功能，其中涉及到拖放元素及目标元素（即被放置的元素）。但默认情况下，只有图像与链接是可拖动的，对于其余的元素，需要设置它的`draggable`属性为`true`才能开启其可拖动性。在拖放的整个过程中，依次发生的事件有：

* 拖放元素
    1. `dragstart`：当开始拖动时触发
    2. `drag`：拖动过程中持续触发（类比`mousemove`）
    3. `dragend`：松开鼠标拖动停止时触发
* 放置目标
    1. `dragenter`：有元素被拖动到放置目标上时被触发，类似于`mouseover`
    2. `dragover`：被拖动元素在目标内移动时触发
    3. `dragleave`或`drop`：移出放置目标时触发`dragleave`；但是当元素被放置到目标中，则触发`drop`

注意：除了某些元素（输入框）外，其余元素都是默认不可放置的，这会导致拖放元素放置到该元素上的时候，还会显示不可放置的标志，当在该元素里释放鼠标时，并不会触发`drop`事件，而是`dragleave`事件。**要将元素设置为可放置目标，需要重写`dragenter`和`dragover`的默认行为，即在其中加`evt.preventDefault();`语句。**对于FireFox，还要取消`drop`的默认行为以阻止浏览器默认的操作（如拖动URL的时候就会打开链接）。

一般情况下，拖动元素需要实现与目标元素的数据交互，这可以通过event的`dataTransfer`对象实现。它主要有`getData(type)`和`setData(type, value)`方法，在`drag`的时候，我们可以将要传输的数据添加到`dataTransfer`对象中，当`drop`被触发后，就可以取得数据进行后续的处理了。需要注意的是两个方法中的第一个参数，HTML5允许扩展指定成任意的MIME类型，但IE只定义了`text`和`URL`两种，HTML5为实现兼容，也支持这两种类型，但会被自动映射为`text/plain`和`text/uri-list`。此外，FireFox 5 以前的版本只能将 `Text` 映射为`text/plain`，因此为保证跨浏览器，最好以 `Text` 代替 `text`。

利用dataTransfer对象，不仅可以传输数据，还可以设置被拖动元素及放置目标可接收什么操作。这可以通过 dataTransfer 的 dropEffect 和 effectAllowed 属性获得。在 `dragstart` 中我们可以设置 effectAllowed 以限制该拖动元素可允许行为， 然后在`dragenter`中设置目标元素能接收什么行为。（事实上，我们可以通过修改这些值观察浏览器生成的鼠标边上的小图标）

<style>
  #dnd-test div {
    margin: 10px auto;
    border: 1px solid #fe0000;
  }
  #dragme {
    border-radius: 5px;
    width: 30px; height: 30px;
    background-color: #7CC5EC;
  }
  #dropme {
    width: 100px; height: 100px;
  }
</style>

<div id="dnd-test">
    <div id="dragme">dragme</div>
    <div id="dropme" width="100" height="100">dropme</div>
</div>

<script>
  (function () {
    var $ = function (name) {
      return document.querySelector(name);
    }
    HTMLElement.prototype.on  = HTMLElement.prototype.on || function (type, callback) {
        this.addEventListener(type, callback);
    }
    var dragme = $('#dragme'), 
        dropme = $('#dropme');

    dragme.setAttribute('draggable', true);
    dragme.on('dragstart', function () {
        console.log('drag start');
    });
    dragme.on('drag', function () {
        console.log('I am being dragged');
    });
    dragme.on('dragend', function () {
        console.log('I am being released');
    });

    dropme.on('dragenter', function () {
        console.info('something entered');
        evt.preventDefault();
    });
    dropme.on('dragover', function () {
        console.info('something is hovering on me');
        evt.preventDefault();
    });
    dropme.on('dragleave', function () {
        console.info('something leaved from me');
    });
    dropme.on('drop', function () {
        console.info('something dropped to me');
    });
  }());
</script>

