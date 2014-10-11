---
layout: post
title: Emmet：一个快速编写HTML的工具
categories:
- 编程与开发
tags:
- CSS
- HTML
---

写HTML的时候，最痛苦的莫过于标签的不断重复。虽然有的编辑器支持自动提示，但碰到一层套一层的结构，还是写得很蛋疼。因为平时使用VIM这种不够现代化的编辑器，所以写起来更是郁闷。幸亏找到了Emmet（以前也叫Zen-Coding），瞬间生产力得到了大大的解放。一个简短的演示如下：

![](http://dl.iteye.com/upload/attachment/0083/2327/301ff5c9-3604-3dd3-a206-6d3516861ec4.jpg)

VIM下Emmet的安装很简单，把[对应的插件]()安装到VIM的配置文件夹下，然后下次打开一个HTML页面的时候，就可以开始使用了。而Sublime的安装似乎更简单，直接`ctrl+shift+p`调用Package Control后，输入install，回车再输入`emmet`就可以找到emmet插件包了。两个编辑器的插件编写规则是一样的，当按规则编写完后，VIM使用`ctrl+y`再按`,`就可以转化成HTML语言了，而Sublime则直接TAB键就OK了。

这篇文章存在的目的正是给自己作一个备忘录，以便在不太熟悉的时候可以查一下。不过，话说编写规则很简单，用多了也就容易记住。

<del>在CSS中，`#`和`.`分别表示ID选择器与类选择器，使用Emmet规则`div#A.B`可以生成`<div id="A" class="B"></div>`。</del>

在CSS中`>`表示**子元素选择器**，而`+`表示**兄弟元素选择器**，使用Emmet规则`div>span+p`可生成：

    <div>
        <span></span>
        <p></p>
    </div>

为生成多个同标签元素，可以组合使用`*`和`$`：（`div>p*3+div#grid$*4`）

    <div>
        <p></p>
        <p></p>
        <p></p>
        <div id="grid1"></div>
        <div id="grid2"></div>
        <div id="grid3"></div>
        <div id="grid4"></div>
    </div>

当使用`>`处于子层后希望回到父层，那么可以使用`^`进行提升，`div>p^span`就可以生成``：

    <div>
        <p></p>
    </div>
    <span></span>

{}：文字结点
()：群组

此外，除了这些统一的规则以外，emmet还规定了非常多的定制规则简化HTML与CSS的编写，可以参考相应的文档。

* [https://zen-coding.googlecode.com/files/ZenCodingCheatSheet.pdf](https://zen-coding.googlecode.com/files/ZenCodingCheatSheet.pdf)。
* [http://www.iteye.com/news/27580](http://www.iteye.com/news/27580)
* [http://docs.emmet.io/](http://docs.emmet.io/)
