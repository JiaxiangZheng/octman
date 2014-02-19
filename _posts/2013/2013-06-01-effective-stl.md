---
layout: post
title: Effective STL 摘录
categories:
- 程序设计
tags:
- c++
- programming
---

[《Effective STL》](http://book.douban.com/subject/1792179/)这本书看起来还是比较快的。但是并不是说里面的内容简单，相反，这本书是前人不断工程实践基础上总结出来的。只是粗略地看一遍，以后再重新读它或许又是种别样的理解了。

包括7个主题内容，分别为：  

1. **容器**
2. **vector和string** 
3. **关联容器**
4. **迭代器**
5. **算法**
6. **仿函数**
7. **使用STL**

下面主要是通过简单地把每一个Item里的主要内容枚举一下，可能会有一点点自己的理解在里面，不过目前还是停留在了一个比较肤浅的层面。

### 容器

这一部分主要是讲了STL中不同的容器应该在什么场合被使用，它们之间有什么相同名字的函数及其功能有何差异。事实上，日常的STL应用过程中，几乎没有不用到容器的场合。至少在我平时的代码中，`vector`是一个最常见的容器，然后map次之。对于这些容器各种操作有所了解，显然可以更好地帮助我们使用容器。

STL中的容器，主要包含了二大类：**顺序容器**（sequence container）和**关联容器** （associative container）。区别在于顺序容器直观地就是一个有序的顺序结构，有一定的前后关系；而关联容器内部是用一种**红黑树**的结构所组织的，拓扑上来说就不是一个有序的结构。

#### Item 1 : Choose your container with care
主要介绍了如何根据实际的需求来选择合适的容器。

#### Ietm 2 : Beware the illusion of container-independent code
如果需要写一些模板算法，而且模板类型可能是某些容器，那么注意选择容器都有的函数使用，如果要求函数支持诸如`vector`和`list`等容器，则应该放弃`[]`算符。
此外，`typedef`的应用可以简化很多操作。

#### Item 3 : Make copying cheap and correct for objects in containers
有时候容器中元素的反复复制会导致效率的下降，这时候可以考虑将容器中的元素替换为对应类的指针，另外裸指针的话可能会出现一些问题，这时候考虑使用智能指针替代。使用指针的另一个好处是支持多态。

#### Item 4 : Call empty instead of checking size() against zero
这个主要是因为像`list`这种容器调用`size()`往往需要`O(n)`的时间复杂度，而`empty()`对于所有容器来说都是常数时间即可以完成。

#### Item 5 : Prefer range member functions to their single-element counterparts
给定两个`vector`，v1和v2，怎样使v1和v2的后半部分内容一样，即把v2后半部分内容复制到初始化的v1中。有四个方法可以解决这个问题。 

1. 使用`assign`：`v1.assign(v2.begin()+v2.size()/2, v2.end())` 
2. 使用循环地`push_back` 
3. 使用`copy`区间函数`copy(v2.begin()+v2.size()/2, v2.end(), back_inserter(v1))` 
4. 使用`insert`成员函数： `v1.insert(v1.end(), v2.begin()+v2.size()/2, v2.end())` 

避免使用显式的循环调用，可以用区间函数的就尽量用区间函数，如容器都支持`copy`和`assign`函数，可以直接进行区域赋值。  
不仅仅是缩短代码量的问题，还有一些细节上的效率优化问题。

#### Item 6 : Be alert for C++'s most vexing parse
这是一个很容易出现的错误，比如定义了一个类Foo之后，我们定义一个对象`Fooobj()`，使用了它的默认构造函数，但是这样编译器会误以为是声明了一个无参数返回Foo对象的函数obj。  
这里有一个从文件流中读出数据的非常简单的方法，略！  
`list<int> data(istream_iterator(dataFile), istream_iterator<int>())>//并非想象中那样`  
事实上上面这行代码是错误的，这只是声明一个函数，解决的方法可以两个参数都再加一个括号，强制告诉编译器这不是函数声明。

#### Item 7 : When using containers of newed pointers, remember to delete the pointers before the container is destroyed
用“出来混，迟早要还的”形容再贴切不过了。智能指针可以派上用场。

#### Item 8 : Never create containers of `auto_ptrs`
`auto_ptr`是一个烂坑，避免使用。

#### Item 9 : Choose carefully among erasing options
容器的`erase`操作需要特别小心地使用，为了达到性能上的最优，不同的容器需要分别对待。  
对于连续内存容器（`vector`, `queue`, `string`）最好采用如下方式`c.erase(std::remove(c.begin(), c.end(), value))`  
对于`list`，直接使用`remove`成员函数即可`c.remove(value)`   
对于关联容器，解决问题的适当方法是用`erase`成员函数`c.erase(value)`  

当然，一个更加普遍的问题是如果把移除特定元素修改为移除满足某个条件的所有元素，则采用的函数不同。假设判定函数为`bool condition_fun(value)`   
对于连续内存容器，把`remove`替换成`remove_if`即可；   
对于关联容器，直接从原容器删除元素的方法是用循环，这样更加高效（注意这个时候循环的i++不应该在for里写，而应该在循环体里面，因为`erase`的缘故）。

#### Item 10 : Be aware of allocator conventions and restrictions.
`allocator`的优点

### vector和string

#### Item 13 : Prefer `vector` and `string` to dynamically allocated arrays   
嗯，用`vector`尽量取代数组，`string`取代`const char*`   

#### Item 14 : Use reserve to avoid unnecessary reallocations   
很明显的一个问题，防止内存分配反复进行，预先分配足够的内存空间   

#### Item 15 : Be aware of variations in `string` implementations   
主要是注意一下`string`的内部实现是如何保证`string`能够高效率地被使用及引用计数。   

#### Item 16 : Know how to pass `vector` and `string` data to legacy APIs   
注意如何保证与C中的接口兼容   

#### Item 17 : Use "the swap trick" to trim excess capacity   
现在[C++11](http://en.cppreference.com/w/)里已经支持了`shrink_to_fit`了。   

#### Item 18 : Avoid using `vector<bool>`
`vector<bool>`并不是想象中的那样每个空间中放一个字节长的bool型，而是为了节省空间只占用一个bit，即如果`vector<bool>`中存放了16个元素，实际上它占用的空间（指的是存放元素的数组）为2bytes或16bits   

### 关联容器

#### Item 19 : Understand the difference between equality and equivalence   
即区别相等与等价这两个不同的概念。   
一个比较好的例子可以说明：比如可以在set的比较函数中忽略字母大小写，这样STL和sTl等价，但实际上它们显然是不等的。而到时候搜索的时候，搜索STL也能返回成功的标志；   
但是如果用`find`函数，显然会区别的。   
所以对于关联容器来说，要提供一个定义有序的比较函数，而对于`find`函数，则提供一个相等或不等的比较函数。   

#### Item 20 : Specify comparison types for associative containers of pointers   
看里面的例子，这是一个坑，如果定义的关联容器模板参数是一个指针，那么比较函数默认将会使用地址大小，这个时候千万要自己重新定义。   

#### Item 21 : Always have comparison functions return false for equal values   
举个例子，如果定义了一个`set<int, less_equal<int> > s;//less_equal<int> 即<=`   

那么插入同一个数两次将不会导致第二次插入失败。为什么？因为`set`在检查新插入的元素是否已经存在的时候，它的检查方式如下：（设元素为0，为了区分，第一次插入的为`0_A`，第二次为`0_B`）`!(0_A<=0_B)
&& !(0_B<=0_A)`，显然这是会返回false的，即表明`0_A`与`0_B`不等价。此外，对于`multi_set`和`multi_map`这类容器，其模板参数中的比较类对于相等的值也应该返回false，因为`euqal_range`这类函数返回的一对迭代器将仅仅指定一系列等价的值的迭代器范围。 

#### Item 22 : Avoid in-place key modification in set and multiset   
主要是在使用`set`的时候避免直接更改键值，应该先移去这个元素再重新插入新的值。否则会破坏其内部的有序结构。   

#### Item 23 : Consider replacing associative containers with sorted `vector`s
注意，关联容器诸如map和set内部是通过红黑树组织数据以使得查找操作达到O(log(n))的 ，它们的主要好处是适合于一个动态的查找过程，即随时可以删除，插入新的元素而保持整 棵树的平衡性。但是另一方面，如果我们的数据本身只是初始化的时候把数据集建立起来， 之后就只涉及到查找操作，这个时候用有序的`vector`反而会更好了。一方面是内存占用更少，另一方面是根据局部性原理，从而提高查找效率。  
此外，还可以用`vector<pair<KeyType, ValueType>>`来模拟map。这个时候注意比较函数的改写，而且为了提高比较效率，可以提供多组比较函数。

#### Item 24 : Choose carefully between `map::operator[]` and map-insert when efficiency is important
insert更加地有效，而`[]operator`先insert一组`key+default_value`，然后再调用insert返回的迭代器来更新对应的值。必要时可以自己重新写一个模板参数函数-addOrUpdate函数接口来高效地实现这一过程。

#### Item 25 : Familiarize yourself with the nonstandard hashed containers
`hash_set`, `hash_map`等并不在STL标准之内，但基本上SGI和MVC的实现中都包含了，但可能实现的方法不太一样，因为标准并没有规定。用的时候需要注意一下。   
未细看。

### 迭代器  
STL中包含了四种迭代器，`iterator`, `const_iterator`, `reverse_iterator`, `const_reverse_iterator` 

#### Item 26 : Prefer iterator to const iterator, reverse_iterator, and const_reverse_iterator
原因见下图：   
![](http://twimgs.com/ddj/cuj/images/cuj0106smeyers/diagram1.gif)

**TO BE ADD**

#### Item 27 : Use distance and advance to convert a container's const_iterators to iterators
如果实在是没有办法在已经给定了`const_iterator`的情况下，我们怎么获得它的非const版本的iterator？首先，应该明确iterator和`const_iterator`是两个完全不同的类，因此隐式转化显然不可能了。  
可以考虑`advance(iter, distance<Container::const_iterator>(iter,
c_iter))`，注意一定要有一个显式的形参模板，以明确地告诉编译器传递给distance的参数是`const_iterator`（iterator可以隐式地转化为`const_iterator`）。  

#### Item28 : Understand how to use a reverse_iterator's base iterator
从**Item26**得知`reverse_iterator`到iterator的转化是通过`reverse_iterator`的成员函数base()，但是呢？`r_it.base()`所指的值与`r_it`所指的是不一样的，`r_it.base()`总是指在`r_it`所指元素的下一个。  

#### Item 29 : Consider istreambuf_iterators for character-by-character input
如题（`istreambuf_iterator`忽视空格等字符），同样对于输出用`ostreambuf_iterator`。

算法

#### Item 30 : Make sure destination ranges are big enough
一些关于inserter, `back_inserter`, `front_inserter`的使用，具体还可以参考[《C++标准库》](http://book.douban.com/subject/1110941/)。

#### Item31 : Know your sorting options
对STL中的排序算法，知道多少呢？哪些是stable，哪些不是，堆排序怎么用？`nth_element`平常好像没怎么用过，有啥用途，它们分别用在哪种场景下？（对list的排序要特别注意一下。）  
对于顺序容器的全部元素进行排序，`sort`和`stable_sort`都可以使用。而如果只需要对前n个元素进行排序，只需要使用`partial_sort`；而如果要将容器中找出前n个元素并使之有序，`nth_element`可被使用；`stable_partition`执行的是把顺序容器中的元素划分为两个部分，使得一个部分中所有元素满足某条件，另一部分不满足某条件；而如果是`list`，则需要特别注意。

#### Item 32 : Follow remove-like algorithms by erase if you really want to remove something
算法库中的`remove`函数并不会减小容器的大小，需要`erase`才可以，`remove`函数只是把后面的不需要移除的元素不断地覆盖前面需要移除的元素。  
对于类似于`list`的成员函数`remove`，则真的会减小容器大小，而在`associate` `container`中相同功能的函数称为`erase`成员函数。  

#### Item 33 : Be wary of remove-like algorithms on containers of pointers
注意`remove`用在包含指针的容器时的情况。  
仔细分析就会发现很容易存在内存泄露的问题。  

#### Item 34 : Note which algorithms expect sorted ranges
有些函数要求输入的容器是有序的，注意一下。     
二分查找、`lower_bound`、`upper_bound`等，还有`merge`也要注意一下。此外，`unique`和`unique_copy`对输入不要求有序，但有序往往效率会更好。  

#### Item 36 : Understand the proper implementation of copy_if
***TO BE READ!!!***

#### Item 37 : Use accumulate or for_each to summarize ranges
注意一个这样的情况：  
`double sum = accumulate(ld.begin(), Id.end(), 0);`   
本来这个函数是没有问题的，返回ld中所有元素之和，但是注意第三个函数参数为0，是int型，将会告诉编译器返回的结果是一个int类型，因此可能会导致计算上的错误，故最好写成0.0 

##仿函数

#### Item 38 : Design functor classes for pass-by-value
仿函数接口采用最好是传值而非传引用，间接说明了仿函数应该尽量短小

#### Item 39 : Make predicates pure functions
那些用于条件判断的仿函数最好不要修改内部的数值，即函数应该为const成员函数。

#### Item 40 : Make functor classes adaptable
使用STL中四大标准函数适配器（not1, not2, bind1st, bind2nd）时，注意它们需要的一些typedef类型。具体包括：`argument_type`, `first_argument_type`, `second_argument_type`,和`result_type`。  
当然，可以通过直接把仿函数继承自STL提供的一些基础仿函数类，如`unary_function`，`binary_function`等。

#### Item 41 : Understand the reasons for ptr_fun, mem_fun, and mem_fun_ref
呃。。。这个从来没用过。。。***要特别看一下***

#### Item 42 : Make sure `less<T>` means `operator<` 
这个可以与前面**Item19**对应，因为一些关联容器要求的就是提供`operator<`作为模板的比较元素。

### 使用STL进行编程

#### Item 43: Prefer algorithm calls to hand-written loops
看下面的例子，十分地方便：  
`int sum = std::accumulate (Vs .begin (), Vs. end(), 0 , [](int x , int y){ return x+ y;});`   
结合`mem_fun`，`mem_fun_ref`等。。。  
这一节提供了一些常见算法的简化使用方法，有空看看

#### Item 44. Prefer member functions to algorithms with the same names
拿`associate`容器来说，它有`find, count, lower_bound`等成员函数，而algorithm库中也有这些函数。  
这个自然，因为成员函数总是更有针对性地设计的，效率上可能有很大的提高。  
举个例子，set中的find函数和algorithm中的find通用函数，前者可以在对数时间内返回结果，而后者则需要一个线性的时间。

#### Item 45. Distinguish among count, find, binary search, lower\_bound, upper\_bound, and equal_range

If you have an unsorted range, your choices are count or
find。注意两者作用不一样。而且find一旦找到就返回结果，而count则需要遍历整个序列
。 `equal_range`函数返回的值包含了`lower_bound`和`upper_bound`的结果。 **仔细看
一下书中最后的表格**

#### Item 46 : Consider function objects instead of functions as algorithm parameters
把函数指针作为参数会抑制内联。

#### Item 47 : Avoid producing write-only code 
代码不仅本身要美，而且要让看的人也感觉美。  
这节的例子可以好好地吸收一下   
