---
layout: post
title: 找工作面试总结
categories:
- Life
tags:
- 面试题
- diary
---

找工作到目前应该算是可以收工了，现在把我之前面试过程中记录的一些笔记列举在这里。
当初记录这些内容的目的是给自己一个备忘录，让自己知道应该复习哪些内容，同时也方便
后来回过头来重新复习。

当然，这里的内容比较有限，主要是数据结构和算法的内容，最后包含少量的操作系统和计
算机网络知识点，不过并不完全。具体到每一小节的知识点，有些内容我在博客里单独地作
了一些笔记（如[随机数相关知识点][RAND-NOTES]）。当然，后面也会逐步地把之前记录的
笔记添加进来。这里很多的链接是[**LeetCode**](http://oj.leetcode.com)上的内容，我在
[GitHub](https://github.com/JiaxiangZheng/LeetCode/)上也记录了我自己做的一些题目
的代码。

[RAND-NOTES]: http://octman.sinaapp.com/?p=19

### 栈和队列

* 二叉树层次遍历及其扩展，如[ZigZag](http://oj.leetcode.com/problems/zigzag-conversion/)等
* 栈和队列的互相模拟
* 快速返回最小值的栈和返回最大值队列
    - 返回最大值的队列也是类似的，只不过需要用到栈和队列相互模拟的结果
* 包含四则运算的表达式计算
* 判断扩号是否合法

下面是两个利用**栈**的算法，个人觉得第一次见到的时候还是比较有难度的。

* 最长匹配的扩号<http://oj.leetcode.com/problems/longest-valid-parentheses/>
* 最大面积直方图问题<http://oj.leetcode.com/problems/largest-rectangle-in-histogram/>

### 链表相关算法

关于链表的内容，可以参考之前写的[一篇文章](http://1.octman.sinaapp.com/?p=91)。

* 链表排序问题
    - 包含归并排序和快速排序
* 反转链表
    - 递归与非递归方法
* 寻找第K个结点（快慢指针）
* 复制复杂链表（包含一个随机指针）<http://oj.leetcode.com/problems/copy-list-with-random-pointer/>
* 寻找公共结点（没有环和有环两种情况）
* 检测链表中的环<http://oj.leetcode.com/problems/linked-list-cycle-ii/>
* 链表与二叉树的原地转换---可以用来快速合并两个BST<http://oj.leetcode.com/problems/flatten-binary-tree-to-linked-list/>

### 树相关的算法

关于树的结构主要包括红黑树，B+树，B树，堆，二叉树，BST。对于红黑树这种比较复杂的
数据结构，基本了解一下其来龙去脉就可以了，要在很短的时间内实现它还是很困难的。但
对于**堆**和**BST**，则需要掌握地非常完全。

* 二叉树的遍历
    - 递归遍历的方法
    - 非递归遍历的方法，可以参考之前写的文章<http://1.octman.sinaapp.com/?p=123>。
* 二叉树的插入删除操作
    - 主要可以参考算法导论上的内容
* BST中两个结点交换位置，如何把它们找出来。[参考这里](http://chuansongme.com/n/97180)
* LCA问题
    - 多种[解法](http://www.itint5.com/2013/08/least-common-ancestors/)
    - 另外如果本身有指向父结点的指针话，那实际上也相当于求链表的第一个公共结点

### 图论

国内公司直接问图论的话，可能会问的比较少，最多问到基本概念性的东西就打住了。不过
网易游戏貌似比较喜欢问关于图论方面的东西，所以还是需要好好地看一下。

* BFS，DFS
* 拓扑排序
* 最短路径
    - Dijkstra算法
    - Floyd算法
* 最小生成树
    - Prim算法
    - Kruskal算法
* 连通性问题
    - 可以用DFS或并查集来不断地合并计算连通分支个数
* 二分图
    - 使用BFS不断地把相连的结点标记成相反的颜色

### 字符串相关的操作

出现频率非常高的算法题，基本上每次面试都会被问到字符串相关的算法。

* 最长公共子串
* 最长重复子序列
* 最长回文子串   
这个问题用DP做在线性复杂度内可以求解，可以线性遍历串，同时每次以当前元素为中心进
行扩展（这里注意有可能以当前元素和其下一个元素共同作为中心）
* 模式子串匹配
    - 朴素的匹配算法
    - KMP算法
    - BoyerMoore算法
    - RobinKarp算法   
    使用了滚动哈希计算方法，使得在给定当前子串hash值情况下，计算下一个
    子串hash值可以在常数时间内确定。
* 字符串排序
* 最短摘要
* 最长不重复子串
* 最短包含子串
* 变位词（可以考虑用素数相乘对比）
* STL中字符串类的实现   
实现有多种方式，有的使用到了引用计数，有的使用则包含大的字符串和小的字符串两种类
型，由于使用了两类，需要设置一个标志位指示是哪种类型。PS：小字符串使用默认分配大
小，而长字符串则使用一个指针加容量长度计数器。
* 最小回文分割<http://oj.leetcode.com/problems/palindrome-partitioning-ii/>
* 单词分割<http://oj.leetcode.com/problems/word-break/>

### 其它数据结构

几个高级的数据结构稍作了解：

* Trie树
* KD树和八叉树
* 红黑树
* 线段树----用于区间统计问题
* B/B+
* 后缀数组
* 后缀树----可以在对数时间内解决大部分字符串问题
* 树状数组----使用二进制位解决区间和问题
* 跳表Skiplist
* 树堆Treap

* * * * * * * * * * * * * * * * * * * * * * * * * * *

关于算法方面的内容，永远是准备不完的，这里总结的一小部分，还有诸如搜索算法等内容
并没有写在这里，一般搜索算法是被称为万能算法的，即当没有什么好的思路时，可以先尝
试一下搜索算法，然后考虑能否利用动态规划或者利用子结构将时间复杂度降低。

### 常量空间的整数算法

有几个使用整数将空间效率降为常数空间的算法：

* 已知一个数组为1\~N的排列，现在在其中随机地抽走一个元素，即得到一个大小为N-1的
数组，那么如何在O(N)时间O(1)空间复杂度下找到这个元素呢？如果抽走K个元素，如何在
尽可能低的空间复杂度下找到这K个元素呢？
* 判断一个长串A是否包含字符串P中所有的字符（忽略顺序）。可以参考[google-interviewing-story][4]

[4]: http://www.aqee.net/google-interviewing-story/

### 随机数相关算法

关于这部分内容，可以参考之前的[一篇文章](http://octman.sinaapp.com/?p=19)。

* 随机洗牌问题
* 随机数生成
* 蓄水池抽样
* 从数组中随机地选择一部分元素
* 随机算法的验证

### 动态规划问题

基本上GeeksForGeeks里动态规划问题已经覆盖了所要考的内容，[链接在此](http://www.geeksforgeeks.org/tag/dynamic-programming/)。

* LIS最长递增子序列   
可以利用二分+DP提高效率
* 字符串编辑距离<http://oj.leetcode.com/problems/edit-distance/>
* 最大连续子数组之和-包括首尾相连这种情况<http://www.itint5.com/oj/#9>
* LCS最长公共子串
* Interleaving String问题<http://oj.leetcode.com/problems/interleaving-string/>
* 最长活动安排[^1]

[^1]: 与下面贪心算法中的活动安排不同，最长活动安排需要求出最长的不重叠的活动子集，但活动安排问题只求最多的活动数。

### 贪心算法

GeeksForGeeks里关于贪心算法的[链接](http://www.geeksforgeeks.org/tag/Greedy-Algorithm/)。

* 部分背包问题----允许有小数
* 活动安排问题----按结束时间排序，然后依次选择满足条件的活动
* Huffman编码
* 最小生成树（Prime和Kruskal）
* Dijkstra最短路径

### 排列组合问题

这一部分内容之前已经写过文章，在[这里](http://octman.sinaapp.com/?p=144)。

* 全排列
* 全子集
* 组合
* 电话号码组合问题
* 求某一数组按字典序在所有全排列中第i个排列
* 给定一个排列，求它在字典序中排第几个

### 数字相关问题

* 求1000!末尾有几个0？（编程之美）
* 素数问题
* 利用位运算
  - 判断一个数是否是2的幂指数
  - 数组中只有一个数字出现一次，其它全出现两次，找出这个数字？
* 亲和数问题
* 求解数组中和为特定值的所有组合
* 程序员编程艺术206页
* 最大连续子数组之和（允许首尾相连的情况）
* 调序问题：将`A1 A2 A3 B1 B2 B3`变换成`A1 B1 A2 B2 A3 B3`的形式。
* 奇偶调序
* 众数，K众数等
* 陈利人待字闺中系列<http://chuansong.me/account/daiziguizhongren>

### 二分查找算法相关

参考[这里](http://octman.sinaapp.com/?p=12)。

* 普通的二分查找或返回插入位置
* 返回上界或下界
* 二维的情况（从左到右有序，从上到下也有序）
* 旋转数组查找旋转点和查找某个元素
* 两个有序数组求第K大的数

### 算法参考网站

* [GlassDoor][1]
* [LeetCode][2]
* [ItInt5][3]

### 海量数据处理问题

主要看网上的[专题](http://blog.csdn.net/v_july_v/article/category/1106578)。

### 设计模式

主要是注意**单例模式**[^2]、**组合模式**、**工厂模式**和**抽象工厂**，另外**模板方法**也值得一提。参考[这里](http://octman.sinaapp.com/?p=4)。

[^2]: 特别注意如何实现多线程安全的单例模式。

### 操作系统与计算机网络

* 多线程问题
* 内存管理
* PV操作-信号量
* TCP/IP相关内容

[1]: http://www.glassdoor.com/
[2]: http://www.leetcode.com
[3]: http://www.itint5.com
