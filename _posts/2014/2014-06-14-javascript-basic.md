---
layout: post
title: JavaScript基础笔记
categories:
- 编程与开发
tags:
- JavaScript
---

最近在学习JavaScript，这里记录一下其中最基本的语法内容作为笔记，主要参考[JavaScript高级程序设计](http://book.douban.com/subject/10546125/)和[JavaScript: The Definitive Guide](http://book.douban.com/subject/10549733/)。

* * * * * * * * * * * * * * * * * * * * 

JavaScript是现代Web世界的语言，与HTML的DOM进行交互控制页面的行为。它于90年代中期被提出，借Java之名气使得很多人误以为它与Java有所关系，但事实上它们是完全没有关系的二种编程语言。需要特别注意的是，当初刚提出来的时候，各大浏览器厂商都有其自己的JS实现，而且名字也不是叫JavaScript，接口也不一致，这导致后来兼容性的问题日益明显。等到1997年，ECMA组织指定了标准化的ECMASt作为JavaScript脚本语言标准。所以ECMAScript与JavaScript二者的关系也很明显，一个是标准，另一个是实现。完整的JavaScript包括三个核心内容：

* ECMAScript
* DOM（文档对象模型）
* BOM（浏览器对象模型）

由此可见，JavaScript相较于ECMAScript来说范围更大。下面说到的JavaScript基本语法实际上是基本的ECMAScript语法，DOM和BOM这里就不提及了。

### 对象与变量

**命名规则**：虽然JS可以嵌套在HTML中，但与HTML不同，JavaScript区分大小写。并且标识符命名规则与C语言非常相似，除了一点（**它的第一个字符也可以是以`$`开头**，PS：事实上也可以用Unicode字符，但不推荐）。另外，JS使用分号结尾，分号也是可以省略的（这时解释器自动在每行末尾添加分号，这一点与Go有点相似），但这样容易引入诸多不完整的输入错误。

JavaScript只有5种简单类型和1种复杂类型，简单类型分别是`Undefined`、`Null`、`Boolean`、`Number`和`String`，而复杂类型为`Object`。其中JavaScript只包含一种数字类型`Number `，即双精度64位的浮点数，遵循IEEE754标准。而`Object`允许我们自己生成需要的类型（由key-value对组成）。为判断一个数据的类型，可以使用`typeof`动态萃取出其类型。这里需要特别注意的是`Null`和`Undefined`类型，它们都只有一个值，分别是`null`和`undefined`，尽管它们在概念上大体相似，但`undefined`是在我们使用`var`声明一个对象而不指定其值的时候自动生成的，最好不要手工指定一个对象为`undefined`类型；而`null`则为可以是手工指定。

`Boolean`类型虽然字面上只有`true`和`false`两个值，但所有类型的值实际上都可以与这两个值进行等价，可以使用函数`Boolean()`实现。这一点怎么理解呢？例如：对于非空字符串，全为`true`，而空字符串为`false`；对于数字类型，`0`和`NaN`为`false`，`null`和`undefined`也都是`false`，这些为`false`的值称为falset，其余非falset全为`true`。

数字类型中，需要特别注意`NaN`，它表示非数，这是一个特殊的值。在C语言中，如果在表达中出现了除零，则表达式本身错误，最好就需要预先避免这种计算情形；但在JavaScript中，它是可以存在的，用`NaN`表示。尽管语言提供了`Number()`方法将任何其它类型的数据转化成数字类型（不能转换则用`NaN`表示），但最好还是使用`parseInt()`或`parseFloat()`来实现转换。

`Object`是使用得最多的一个类型，它允许我们创建自己需要的对象，通常使用`var obj = new Object();`或`var obj = {name: "", sex: "male"};`达到。在引用某个属性的时候，允许使用点号的方式，也可以使用类似于哈希数组`obj["name"]`的方式，用`[]`的好处在于如果属性名中间含有空格的时候可以正确的引用出其值。由于对象在JS中是一个比较复杂的概念，`Object`类型的详细信息将单独写一篇，见后续笔记。

在JS中，函数也是一种对象。见后文。

由于JS使用松散的类型定义变量，即变量可以存储任何类型的对象值。虽然不推荐动态更改变量类型，下面的代码是完全正确的：

    var message = "hello";  // 字符串类型
    message = 124;          // 数字类型

JS中的变量可以划分为简单类型和Object类型，就其赋值时的表现来说，可以将变量划分成基本类型值和引用类型值。五种基本数据类型（`Undefined`、`Null`、`Boolean`、`Number`和`String`）是基本类型值，而引用类型值则是保存在内存中的对象。首先，引用类型允许自定义其属性及方法，但基本类型是不允许的；其次，在复制变量值的时候（`val = val_copy`），引用类型实际上复制的是变量内存空间的地址，即所谓的指针（类比C++的函数参数中传值和传引用）。但是，注意，即使基本类型，在C++中函数参数也可以使用传引用的，然而在JS中，基本类型只能使用传值方式传递参数，而引用类型则是传递指针的。

### 控制流

首先，需要说明的是`break`和`continue`关键词。大部分情况下，与C语言类似，但是在JavaScript中，`continue`和`break`还可以支持标签引用（如下例），直接跳出标签所定义的区域。

    label: 
    for (var i=0; i<100; i++) {
        for (var j=0; j<100; j++) {
            if (cond(i, j)) {
                break label;    // break out the whole for loop
            }
        }
    }

同大部分语言类似，JavaScript支持`for`和`while`类型的循环（包括`do while`）。简单的`for`遍历与C语言一样，但JavaScript还支持`for in`这种循环遍历对象的方式（即遍历对象中所有属性）。

    // simple loop
    for (int i=0; i<100; i++) {
        // do somthing
    }
    
    // for in loop
    var person = {name: "Jiaxiang Zheng", age: 24, sex: "male"}
    for (x in person) { // x is the key
      // do something
    }

需要特别注意的是`for in`方式的遍历，它允许遍历一个对象的不同属性（其遍历顺序是不可预测的）。`for (attri in obj) { obj[attri]; }`

与C语言类似，条件判断在JavaScript中也是`if/else/else if`。这里需要特别注意的是JS中的等号与全等表达式：

- 不同类型的元素之间是可以划分等号的，它们遵循特定的规则进行转换
- 全等只有在数据未经转换就相等的情况下
- `null == undefined`会返回`true`，但`null === undefined`可能为`false`

JS允许使用`&&`或`||`用于多个条件的组合判断，这一点与其它语言一致。但在JS中，用得比较多的一种是直接使用`expression1 && expression2`进行赋值，表示的是如果`expression1`为假则将`expression1`进行赋值，反之则`expression2`。而`||`则是将为真的表达式进行赋值。

当然，也可以使用`switch`，其使用方式也是非常简单，如下：

    switch (n) {
    case 1:
      // do something()
      break;
    case 2:
      // do something()
      break;
    default:
    }

`break`在循环和`switch`语句中经常会被使用，在带标签的语句块中，`break`也是允许使用的，但一定是要在`break`后面加标签名，否则会报错。

`with`表达式用于将代码作用域设置到特写对象中，示例如下（两者等价）：

    with (object) {                      var qs = object.attri1
        var qs = attri1     <====>       var ns = object.attri2
        var ns = attri2
    }

注意，`with`表达式在严格模式下不可使用。

### 函数

JavaScript中的函数参数与其它编程语言有所不同，它的内部维持一个参数数组`arguments`（事实上并非`Array`类型），所以在调用的时候，解析器并不会介意到底使用多少个参数。可以使用`arguments.length`得到参数个数。

在JavaScript中，一个非常重要的概念便是支持闭包，即支持函数式编程。

与其它语言类似，一个变量是有其作用域的，这里也称为执行环境。可以认为在Web浏览器中，全局执行环境是window对象，因此所有定义的类型和方法都是隶属于window对象的。但是与其它语言又有所不同的是，代码块不能算作是一个环境，因此在`if`代码块里定义的变量在函数体内还是有效的。每个函数体有自己的执行环境。

    var color = "blue";
    function changeColor() {
        var anotherColor = "red";
        function swapColors() {
            var tempColor = anotherColor;
            anotherColor = color;
            color = tempColor;
            // 可以访问color、anotherColor和tempColor
        }
        // 可以访问color和anotherColor，但不能访问tempColor
        swapColors();
    }
    // 只能访问color
    changeColor();

当在一个函数体内使用`var variable`定义了一个变量时，这个变量是局部变量，但是如果在函数体内直接使用`variable = XX`，此时，variable将自动转变为全局变量。由于JS中最重要也最难的一个概念便是函数中的闭包，后面将会有专门的一篇笔记分析JS中的函数，这里不再赘述。

### 异常

因为JS不是编译型语言，而是动态地执行，所以有可能在执行的过程中出现某一语句错误，如果不使用异常处理，可能造成后继任务的无法进行。当错误发生时，JS引擎通常会停止并生成错误的消息，即抛出一个错误。`try catch`语句块允许我们在执行时进行错误测试和执行。

    function message() {
      try {
          adddlert("Welcome guest!"); //这里出现拼写错误，将抛出异常
        } catch(err) {
        txt="There was an error on this page.\n\n";
          txt+="Error description: " + err.message + "\n\n";
          txt+="Click OK to continue.\n\n";
          alert(txt);
        }
    }

与此同时，也允许我们使用`throw`自定义错误。

    function myFunction() {
      try {
        var x=document.getElementById("demo").value;
          if(x=="")    throw "empty";
          if(isNaN(x)) throw "not a number";
          if(x>10)     throw "too high";
          if(x<5)      throw "too low";
        } catch(err) { 
          var y=document.getElementById("mess");
          y.innerHTML="Error: " + err + ".";
        }
    }


### 参考

* [https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
* [http://learnxinyminutes.com/docs/javascript/](http://learnxinyminutes.com/docs/javascript/)
* [https://d396qusza40orc.cloudfront.net/startup%2Flecture_slides%2Flecture10-intermediate-js.pdf](https://d396qusza40orc.cloudfront.net/startup%2Flecture_slides%2Flecture10-intermediate-js.pdf)
* [JavaScript Garden](http://bonsaiden.github.io/JavaScript-Garden/zh/)

* * * * * * * * * * * * * * * * * * * * 

[1]: http://www.w3school.com.cn/jsref/dom_obj_event.asp
  
[^1]: JS速查卡：http://www.addedbytes.com/cheat-sheets/download/javascript-cheat-sheet-v1.pdf 
