---
layout: post
title: Godoc-一个Go代码文档化工具
categories:
- 翻译与转载
tags:
- Golang
- Godoc
---

本文翻译自[Godoc: documenting Go code](http://blog.golang.org/godoc-documenting-go-code)一文，一方面当作自己翻译的第一篇练习，另一方面也熟悉一下Go的文档化工具。下面是翻译内容：

* * *

Go项目十分重视代码的文档。事实上，在软件设计中，文档对于软件的可维护和易使用具有重大的影响。因此，文档必须是书写良好并准确的，与此同时它还需要易于书写和维护。理想情况下，文档应该与代码一起，当更新代码的时候，文档也能够同步得到更新。对于程序员来说，越容易生成好的文档，对大家就越有好处。

为此，我们开发了文档化工具[godoc](http://golang.org/cmd/godoc/)。本文介绍使用godoc生成文档的方法，同时解释如何使用我们的约定规则与工具为你自己的项目书写良好的文档。

Godoc通过解析包含注释的Go代码来生成HTML或文本类型的文档。这样做的结果就是文档与它所涉及到的代码紧密地放到了一起。比如，通过godoc的Ｗeb界面，只需简单的一个点击，即可从一个函数的[文档](http://golang.org/pkg/strings/#HasPrefix)跳转到其[实现](http://golang.org/src/pkg/strings/strings.go?#L312)。

在概念上，Godoc与Python的[Docstring](http://www.python.org/dev/peps/pep-0257/)和Java的[Javadoc](http://www.oracle.com/technetwork/java/javase/documentation/index-jsp-135444.html)类似，但Godoc的设计更简单。Godoc读入的注释并非语言层面进行构建（如Docstring），也无需按特殊的机器可读的语法规则编写（如Javadoc）。Godoc注释本身就是良好的注释，即便godoc不存在，也照样简单易读。

Go的注释规则很简单，为类型，变量，常量，函数或包编写注释时，直接在这些声明前编写普通形式的注释，中间不留空行即可。Godoc将这些注释与后面的声明连接到一起，达到文档化的目的。比如，下面就是`fmt`包中[`Fprint`](http://golang.org/pkg/fmt/#Fprint)函数的文档：

    // Fprint formats using the default formats for its operands and writes to w.
    // Spaces are added between operands when neither is a string.
    // It returns the number of bytes written and any write error encountered.
    func Fprint(w io.Writer, a ...interface{}) (n int, err error) {

注意，这个注释以其描述的元素开头，形成完整的一个句子。这个重要的规则使得我们可以生成多种形式的文档，从文本类型到HTML类型，甚至于UNIX的帮助页面类型，或为了使其更容易阅读而进行截断，例如只显示第一行或者第一句的时候。

包的声明注释应该提供这个包的一般化文档，其注释可以简洁到像[`sort`](http://golang.org/pkg/sort/)包这样：

    // Package sort provides primitives for sorting slices and user-defined
    // collections.
    package sort

也可以像[gob包](http://golang.org/pkg/encoding/gob/)那样作一个全面而详细的介绍。当包需要大量的介绍性文档时，使用的是包文档化的另一条规则：包的注释放在一个单独的文件[doc.go](http://golang.org/src/pkg/encoding/gob/doc.go)，它只包含这些注释和包的许可条款。

无论编写的包注释大小，都需要牢记于心的是，它们的首句将出现在godoc的[包列表](http://golang.org/pkg/)中。

godoc的输出将忽略那些与顶层声明不相邻的注释，只有一个例外，那就是以`BUG(who)`开头的注释。这些注释将被识别为已知的bug，并包含在文档的`BUGS`区。而其中的`who`应该是那些可以提供关于这个BUG更多信息的用户名。比如，下面就是一个[bytes](http://golang.org/pkg/bytes/#bugs)包中已知的问题：

    // BUG(r): The rule Title uses for word boundaries does not handle Unicode punctuation properly.

如果需要将注释转化成HTML形式的文档，Godoc用户还需要掌握一些额外的格式化规则：

-   段落以空行格开

-   预格式化的文档应该缩进（参考gob包的[doc.go](http://golang.org/src/pkg/encoding/gob/doc.go)）

-   URL将被转化为HTML链接，无需其它的特殊标记

注意到，上述的这些规则并没有任何不同寻常之处。

事实上，godoc简单规则的最大好处就是其易用性。事实上，包含所有Go语言标准库包在内的许多Go代码都已经遵守着上述的这些规则。

对于你自己的代码，遵循上述的注释方式，就可以产生出良好的文档。任何安装在`$GOROOT/src/pkg`和`GOPATH`工作区下的Go包文档都自动地被godoc索引分析并通过godoc的命令行及HTTP接口查看，同时你还可以通过指定`-path`标志设定额外的索引路径，或者单纯地运行`godoc .`获得当前代码的文档。更多的细节可以参考[godoc文档](http://golang.org/cmd/godoc/)。

* * *

UPDATE: 发现已经有人翻译过这个了，见[这个](http://mikespook.com/2011/04/%E3%80%90%E7%BF%BB%E8%AF%91%E3%80%91godoc%EF%BC%9A%E6%96%87%E6%A1%A3%E5%8C%96-go-%E4%BB%A3%E7%A0%81/)
