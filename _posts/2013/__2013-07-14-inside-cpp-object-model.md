---
layout: post
title: C++对象内存模型
categories:
- Programming Language
tags:
- c++
- programming
- 内存模型
---

### 基本的C++内存模型

对于C++的类，有两类成员变量：静态与非静态，三类成员函数：静态、非静态和虚函数。
其中静态成员（变量和函数）都是存在于对象模型之外的，而非静态成员变量则存在于对象
之中，即将sizeof作用于对象得到的结果中是包含非静态成员变量的，非静态成员函数也是
存在于对象之外的。对于虚函数，则情况稍微复杂了一点点。首先是需要创建一个额外的虚
函数表，存放所有的虚函数指针和类型信息，然后将指向该虚函数表的指针放在对象模型中
。此外，为了保证字节对齐，编译器有可能在对象中加入填充字节，所以导致最终一个对象
占用的内存空间由如下要素决定：非静态成员变量、字节对齐、虚函数指针[^1]。

[^1]: 对于多个继承，虚函数指针可能不止一个。

### 构造函数

### intrivial constructor

1. the member object has a default constructor
2. base class has a default constructor
3. the class contains virtual function(s)
4. contains a virtual base class

keep in mind that if you do not declare and implement a default constructor, the
compiler might construt one for you. but only the above 4 cases are constructed
as non-trivial, others are trivial, and for compiler, these trivial default
constructor is useless, so it will not construct them.


### copy constructor

the application of copy constructor are various.

- explicitly assignment like `X xx = x`   
- function return value  
- function value parameters

like constructor, copy constructor might construct default copy constructor for
you. **bitwise copy** is used in a plain class without copy constructor. some
non-trivial copy constructor cases are list as follow:  

* member class contains a copy constructor  
* the class is derived from a base class that contains a copy constructor
* virtual function (virtual table should be re-initialized rather than bitwise
copy in some case)  
* inheritage hierachy has one or more virtual base class (*to be further read*)

### 2.3program transformation semantics


### Name Return Value optimization
if the original code is:

    X foo() 
    { 
        X xx; 
        if(...) 
            return xx; 
        else 
            return xx; 
    }

then the compiler might convert to :

    void  foo(X &result)
    {
        result.X::X();
        if(...)
        {
            //process result
            return;
        }
        else
        {
            //process result
            return;
        }
    }


for cfont, only you explicitly define a copy constructor, does the compiler
perform a NRV optimization; for vs2010, the compiler is defaultly open the
optimization.


* * *
## Data语意学

影响一个类大小的因素主要包括：

1. 支持虚拟机制
2. 优化产生出来的隐含对象
3. 字节对齐

关于字节对齐，不同编译器情况不同。可以使用`#pragma pack(n)`来改变对齐字节的大小
。

在VS2010中，成员对象在内存布局的顺序为：（*相关平台下待测试*）

1. 虚函数表指针
2. 虚基类表指针
3. 基类元素
4. 自身声明元素
5. 虚基类元素

### Data Member的存取成本 

考虑如下代码：`Point3d p; Point3d* p_ptr = &p;`，分别通过`p`和`p_ptr`访问成员`x`
，存取成本如何呢？

1. 当成员`x`是一个静态成员的情况下，数据实际上是存放在一个全局的数据段中，因此编
译器在转化的过程中会直接转化为对其地址数据的访问，不管`Point3d`类是一个虚拟继承
，多重继承情况，因为不管类层次如何，编译器转化的结果最终都是`Point3d::x`，因此两
种访问方式实际上最终是一样的。**说明**：不同类静态成员如果同名，放在全局数据区域
中会造成命名冲突，因此编译器暗自会进行name-mangling转化。

2. 非静态成员情况：
   - 一般情况下，编译器会转化`this->x`，因此对于两种存取的方式实际上是等价的。
   - 如果`Point3d`类是一个包含虚拟继承体系的基类，那么指针存取的方式会比直接对象
   存取要慢，因为通过指针我们无法指明它具体指向的对象是何种数据类型（不确定类型
   offset无法计算），直接使用对象存取显然可以避免这个情况。

### 继承与Data Member

如果一个子类继承基类，而基类对象中由于对齐导致有一些填充比特，子类如果独立定义新
的成员变量且其大小小于填充大小的时候，会不会占用填充位置呢？不会的，考虑将基类对
象复制到子类对象，预期应该是基类变量一致，而子类新定义的变量则元素不变。

### C++对象成员的内存布局

不包含继承体系的类比较简单，其对象大小主要靠非静态成员，字节对齐和一个虚表指针来
决定。

而对于有继承体系的类，其情况比较复杂，分四点讨论。

1. 只有继承没有多态的话，可能会导致子类空间发生膨胀。如果类`A`，`B`，`C`均只包含
一个`char`类型，而`D`继承自`A, B, C`，显然各基类因为对齐的原因均占4个字节，但D本
来只需要3字节+本身大小，但实际上却需要12字节+本身大小。而编译器这样做也很合理，
比如将一个子类对象直接赋给基类对象是合理的，但如果子类中不包含对象字节，显然是无
法满足要求的。

2. 包含单一继承的多态情况下，会导致类中还需要插入一个与多态有关的虚函数表指针，
指向虚函数表，而表中包含的是类型信息和虚函数的真实地址。

3. 包含多重继承，（1）如果子类没有覆盖父类的虚函数时，子类将会把继承的所有类的虚
函数表指针都放到对象中，即有多少个包含虚函数的子类，就有多少个虚函数表指针。即便
不同父类虚函数名称一致也可以很好地调用到正确的函数。（2）如果子类会覆盖父类的虚
函数，那么它将会把所有父类虚函数表中同名函数的地址都覆盖掉！

4. 包含虚拟继承的话，以Visual C++编译器为例，它会在对象中安插一个虚基表指针，而
虚基表中则包含一系列virtual base class的指针。这会导致存取成本的增加。

### 对象成员的效率

对于一个元素存取有几种方式，1）直接存取；2）使用对象引值；3）通过类的inline函数
取得对象的引用；4）get/set函数。当编译器的优化开关被打开后，这些封装的效率成本都
是相同的，但是如果把继承体系加入，则需要分情况讨论：

* 如果是非虚拟继承，则基类元素在子类中也是存在的，同时在编译期就可以确定出元素所
处的偏移量。  
* 对于虚拟继承，由于间接存取的操作，导致编译器优化能力被抑制，从而会使得存取效率
有所降低。

### 指向Data Member的指针

* 虚函数表指针的存放视编译器而定，有的是放在对象最前端，也有的是放在对象尾部。
* 指向Data Member的指针：`float Point3d::*pZ = &Point3d::z`得到的将是一个偏移值
，表示的是`z`成员在类中的偏移量，类型为`float Point3d::*`，注意与一个对象提取出
来的`z`类型和地址的区别。

***题外话***：如果用C语言实现虚函数，那么虚函数表放在内存中的哪一块呢？个人想法
一开始是放在堆上，但转而想到，虚函数表是在每个类创建的时候就可以确定的，即编译期
即可确定它，放在内存的常量区不是更好吗？

* * *
## Function语意学

***注***：实际应用中，虚函数不应该被内联。因为“内联”意味着“在编译时刻用被调用
函数的函数体替换被调用函数”。但“虚函数”则意味着“运行时刻决定被调用的是哪一个
函数。”如果编译器在某个函数的调用点不知道具体调用哪个函数，就能够理解为什么不会
把该函数声明为内联函数了。另一个有意思的问题在于，因为每个类只需要一个vtbl拷贝，
那么vtbl放在哪里是一个问题。大多数程序由多个目标文件构成，vtbl放在哪个目标文件呢
？当然可以每个使用该类的目标文件都保存一份拷贝，但这样显然是浪费；另一种常用的方
法是将一个类的虚函数表放在包含该类的第一个**非内联**、**非纯虚函数**定义的目标文
件里。~~所以当我们把虚函数全部声明为内联后，可能会导致一个程序包含很多同一虚函数
表的拷贝。~~

## member函数调用方式

1. 非静态成员函数：编译器内部将该函数转化为参数为this指针的一般函数，同时作name
mangling，保证其名称在整个编译空间中独一无二。另外，成员变量也将被mangling，以便
在继承的时候定义同名变量不会被覆盖。
2. 虚拟成员函数：使用`::`来直接调用有时候会更有效率。
3. 静态成员函数：不能直接存取非静态成员，不能被声明为const, virtual等，不需要经
过class object调用，可用于call back。

***题外话***：如果一个对象中包含由一组基类指针组成的列表，如何对这个对象进行深拷
贝？对于基类定义一个`virtual Base* clone()`函数，这样在深度拷贝的时候调用该函数
即可。

### 虚拟函数相关

1. 单一继承的情况：
2. 多重继承的情况：
3. 虚继承的情况：

## 第7章 Cusp of Object Model

### 编译器如何处理Template

具现模板类的时机，包括两类：

1. 类本身是何时具现，只有使用到了某一个类型对应的模板类，compiler才会生成该类。
2.  类的成员函数不使用是不会生成的。想象一下如果模板类有100个函数，但使用到的只
有2个。

此外，考虑virtual function与template结合起来。(see the book)

### 异常处理

程序堆栈在跳出异常的时候做了什么，开销如何等等 (see book)

### RTTI (RunTime Type Identification）

cast相关的内容需要看一下。


