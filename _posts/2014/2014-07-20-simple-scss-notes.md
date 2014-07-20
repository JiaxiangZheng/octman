---
layout: post
title: SCSS简介
categories:
- 编程与开发
tags:
- CSS
---

本文参考自[官方文档](http://sass-lang.com/documentation/file.SASS_REFERENCE.html)及[阮一峰博客](http://www.ruanyifeng.com/blog/2012/06/sass.html)。

在Web开发中，写CSS是一件很痛苦的事情，不像其它的编程语言，CSS没有变量和条件判断等特性，要设置对应的CSS，只能自己一点一点地添加。这样写出来的CSS，模块化不足，维护起来也不方便，SCSS的出现弥补了这个不足，使得CSS的书写更加容易和规范。SCSS是一个用Ruby编写的CSS的开发工具，为使用它首先需要安装Ruby并执行`gem install sass`进行安装。你也看到这里名字是SASS而非SCSS，事实上，其开发团队提供了SCSS和SASS两种不同风格的工具编写CSS，其区别只是风格上的不同，底层的逻辑是一样的。这里我们只讨论SCSS，而SASSS也是完全类似，事实上，SASS和SCSS也可以相互转换的。（官网提供了一个在线工具：[http://sassmeister.com/](http://sassmeister.com/)）

SCSS文件后缀名为`.scss`，内部可直接使用CSS语法，为生成对应的CSS文件，可以通过使用`sass style.scss style.css`命令。

SASS提供了四个编译风格选项（`sass --style compressed style.sass test.css`）：

* `nested` - 嵌套缩进的CSS代码（默认）
* `expanded` - 无缩进及扩展的CSS代码
* `compact` - 简洁格式的css代码
* `compressed` - 压缩后的css代码（生产环境中常用）

在SASS中，注释分为两种，`//`和`/**/`，其中`//`只保留在SASS源文件中，在编译的时候将被忽略；而`/**/`则是可能会保留到编译后的文件中作为标准CSS注释。另外，当编译选项设置为压缩模式的时候，块注释也是可能会被忽略的。在`/*`后面添加一个感叹号，则表明这是重要注释，这种情况下注释将会得到保留。

在SCSS中，变量的定义可以使用`$var: value;`，使用 `$var` 引用变量，为在字符串中引用变量，可以使用 `#{$var}` 。例如：

    $side: left;    
    div {
        border-#{$side}-radius: 5px;
    }

同时，SASS支持在代码中使用计算表达式，如`margin: (20px/2);`。

为简化CSS中的层级选择器，SASS允许选择器嵌套，如下：（`CSS->SCSS`）

    div h1 {                             div {
        color: red;         ==>              hi {
    }                                           color: red; 
                                            }
                                        }

此外，属性也是允许嵌套的，与选择器嵌套的区别就在于属性后面的分号，如下：（`SCSS->CSS`）

    nav {
        display: block;                         nav {
        text: {                       ==>           display: block;
            decoration: none;                       text-decoration: none;
        };                                      }
    }

为直接在嵌套结构中引用父选择块，可以使用`&`符号表示父结构。一个很常用的例子便是：当我们需要分别为`a:hover`等添加CSS样式时，前面的嵌套结构会在`a`和`:hover`之间加一个空格，使用`&`可以避免这个情况。具体例子如下：

     a { 
       font-weight: bold;
       text-decoration: none;
       &:hover { text-decoration: underline; }
       body.firefox & { font-weight: normal; }
     }

通过转换成CSS如下：

    a {
      font-weight: bold;
      text-decoration: none; }
      a:hover {
        text-decoration: underline; }
      body.firefox a {
        font-weight: normal; }

为了使得整个CSS能够模块化，可以使用`import`导入不同的SCSS文件或CSS，它允许我们将多个SCSS文件合并到一个文件中进行编译，例如：`@import pack`。需要特别注意的是，如果我们只希望通过多个文件生成一个主CSS文件，可以将其余的SCSS文件名以下划线开头，然后在导入的时候，导入名可以只取下划线后面的那部分。注意，`import`实际上就是将导入的文件合并成一个大文件，即`import`被完全用对应文件的内容所替换。

SASS提供的`mixin`用于复用代码块，有点类似于C语言中的宏但与宏又不一样，因为`mixin`是用于利用规则而非值，因此最好带参数。在CSS3中，经常需要重复写多个语句以兼容多种浏览器，这时，`mixin`就非常有用。SCSS使用`@mixin name ($value: default_value) {}`定义，`@include name(para)`进行调用。

    @mixin border-radius($radius) {               .box {
      -webkit-border-radius: $radius;               -webkit-border-radius: 10px;
      -moz-border-radius:    $radius;    ==>         -moz-border-radius: 10px;
      -ms-border-radius:     $radius;                -ms-border-radius: 10px;
      border-radius:         $radius;                 border-radius: 10px;
    }                                             }
    .box {
      @include border-radius(10px);
    }

`extend`允许我们共享多个属性值，在CSS中，我们经常需要设置多个选择器同样的属性值，这个时候需要使用`h1, h2, h3 {...}`实现，而使用`extend`，则可以以更清晰的结构表现出这种共享的结果（`SCSS->CSS`）。

    .message {                          .message, .success, .error {
      border: 1px solid #ccc;             border: 1px solid #ccc;
      padding: 10px;                      padding: 10px;
      color: #333;                        color: #333;
    }                                   }
    .success {                ==>       
      @extend .message;                 .success {
      border-color: green;                border-color: green;
    }                                   }
    .error {                            
      @extend .message;                 .error {
      border-color: red;                  border-color: red;
    }                                   }

有时候，我们希望事先能够定义好一系列属性值，然后多个选择器能够共享这个值，按前面的方法，我们需要先设置其中一个选择器，然后在其它选择器中包含`@extend sel`，有可能会导致不易理解，这时，可以使用`%`扩展符，它与正常的选择器类似，只不过将ID选择器中的`#`替换成了`%`。此外，它只能在`extend`中使用，这样我们就可以将它当作一个变量赋予一个更有意义的变量名了。

SASS模拟编程语言，因此也提供了控制流语句。

`@if`与`@else`配套使用，作为条件判断语句。

    $type: monster;
    p {
      @if $type == ocean {
        color: blue;
      } @else if $type == matador {   ==> p {  color: green; }
        color: red;
      } @else if $type == monster {
        color: green;
      } @else {
        color: black;
      }
    }

`@for`和`@while`提供循环语句。

    // SCSS
    @for $i from 1 through 3 {
      .item-#{$i} { width: 2em * $i; }
    }
    // CSS
    .item-1 {  width: 2em; }
    .item-2 {  width: 4em; }
    .item-3 {  width: 6em; }

`@each`命令与`for`类似，只不过它用于遍历一个map或list。

    // SCSS
    @each $animal in puma, sea-slug, egret, salamander {
      .#{$animal}-icon {
        background-image: url('/images/#{$animal}.png');
      }
    }
    // CSS
    .puma-icon {
      background-image: url('/images/puma.png'); }
    .sea-slug-icon {
      background-image: url('/images/sea-slug.png'); }
    .egret-icon {
      background-image: url('/images/egret.png'); }
    .salamander-icon {
      background-image: url('/images/salamander.png'); }

至此，所涉及到的SCSS就已经足够日常开发的使用了。当然，还有内置函数等特性，可以在使用的时候直接参考[官方的文件][1]。


  [1]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html

