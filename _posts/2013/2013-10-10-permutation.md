---
layout: post
title: 排列组合面试题
categories:
- 算法与基础
tags:
- 面试题
- 算法
- 排列组合
---

面试题中，经常有一些求集合的全排列，组合等问题，本文主要就这些问题进行探讨，作为
自己学习的一些记录笔记。本文主要内容包括：

### 全排列

给定一个集合$S$，比如$S = \{1, 2, 3\}$，则它对应的全排列为 `{1,2,3}, {1,3,2},
{2,1,3}, {2,3,1}, {3,1,2}, {3,2,1}`。如何对于一个给定的集合求
其全排列呢？计算机程序设计艺术（四）中对于排列组合问题进行了非常详细的讨论
，这里主要是分析两种常用的方法，分别是递归和非递归方法。

#### 非递归求解全排列

显然，集合$S$全排列中排列的个数为$n!$个（假设$S$中元素个数为$n$），那么可以考虑
将这所有的排列与1到$n!$作线性映射，这显然可以通过定义一个比较函数将两个排列进行
大小比较，用字典序就可以达到这个目的。什么叫字典序？就是从前向后比较两个排列，第
一次碰到元素更小的那个排列在字典序中就小。举个例子，`1,2,3`和`1,3,2`两个排列中，
`1,2,3`显然小于`1,3,2`，因为从前向后比较，第一个排列的2号元素小于第二个排列的2号
元素。显然，如果把所有的字典序排列一遍，就是所有的全排列了。给定一个排列，如果我
们可以生成它字典序中对应的下一个排列，然后迭代$n!-1$次就可以解决非递归求解全排列
了。

STL中有一个函数叫`next_permutation`函数，它的输入为一个排列，输出为其字典序对应
的下一个排列。

**next\_permutation算法**：给定一个排列后，从最后一个元素向前找到第一个满足$S_i < S_{i+1}$
的元素，然后从最后向前找到第一个大于$S_i$ 的元素$S_j$，交换$S_i$和$S_j$，将
`S[i+1:N]`反向，输出新的$S$即为下一个排列。其C++代码可以描述如下：

    // 如果当前排列已经是字典序中最后一个了，那么就会返回false
    bool has_next_permutation(std::vector<int>& arr) {
        int i = arr.size() - 1, j = arr.size() - 1;
        if (i < 1) return false;
        for (; i > 0 && arr[i] < arr[i-1]; --i);    // 从后向前找到满足S[i] < S[i+1]的元素，代码中的i-1相当于正文中的i
        if (i == 0) return false;
        // assert (arr[i-1] < arr[i]);
        for (; j >= 0 && arr[j] < arr[i-1]; --j);    // 从后向前找到满足S[i] < S[j]的元素，代码中的i-1相当于正文中的i
        std::swap(arr[i-1], arr[j]);        // 交换
        std::reverse(arr.begin() + i, arr.end());   // 反向
        return true;
    }

可以用反证法证明这个算法的正确性。

#### 递归求解全排列

递归的思想是假设子问题已经求解，然后利用求解的子问题来解决父问题。假设我们知道如
何求解`S[2:n]`的全排列，那么将每个元素与第一个元素交换并输出其对应的子集合的全排
列，当子集合中只有一个元素，其全排列就是它自身了，非常的容易理解。于是算法可以描
述为如下：

    func Permutation(arr []int, i int) {    // 求解S[i:N)的全排列
        if i == len(arr) - 1 {      // 处理最后一个元素，显然它的全排列就是它自己，输出最后的结果
            fmt.Println(arr)
        } else {
            for j := i; j<len(arr); j++ {   // 将S[i:N)中每个元素与S[i]交换并输出对应的全排列
                arr[i], arr[j] = arr[j], arr[i]     // 交换
                Permutation(arr, i+1)               // 处理S[i+1, N)子集全排列
                arr[i], arr[j] = arr[j], arr[i]     //
            }
        }
    }

### 全子集

什么叫全子集呢？就是一个集合的所有子集合构成的集合，根据集合论知识，我们知道一个
包含$N$个元素的集合对应的所有子集个数为$2^N$。举个例子，$S={1,2}$，那么它的全子
集$\mathcal{S} = \{\emptyset, \{1\}, \{2\}, \{1,2\}\}$那么如何把这$2^N$个子集全
部输出呢？考虑到其子集的构造过程实际上是每个元素选与不选所构成的情况的汇总，因此
可以对每个元素赋予一个0-1数值，将所有元素的选与不选与0-1对应起来，从而就可以得到
一系列的二进制数据，事实上对应的就是$0~2^N-1$的二进制表示。其中0表示所有元素都不
选，即空集，而$2^N-1$则表示所有元素对应位全为1，即所有元素都选。于是，也就可以很
容易地求出结果了。

那么，前面是非递归的方法，有没有递归求解的方法呢？当然有！算法可以描述如下：

对于每一个元素，可以有选与不选两个状态，因而从排列第一个元素开始，不选它然后求解
`S[2:n]`的全子集；选它然后求解`S[2:n]`的全子集，两个结果的合并就是整个集合的全子
集了。

    void all_permu(std::vector<int>& output, std::vector<int>& arr, int i) {
        if (i == arr.size()) {     // 输出结果
            for (int i=0; i<output.size(); ++i) {
                printf("%d ", output[i]);
            }
            printf("\n");
            return;
        } 
        all_permu(output, arr, i+1);        //  不选 
        output.push_back(arr[i]);           // 选
        all_permu(output, arr, i+1);
        output.pop_back();
        return;
    }

### 组合

跟全子集非常相关的一个问题是组合问题，因为全子集涉及到每个元素的选择与不选择，而
对于组合问题，如果需要从$n$个元素的集合中选择$k$个元素，它的选择方法个数有
${n}\choose{k}$。那么怎么样用程序输出选择的这些结果呢？类似于全子集，每个元素也
都是有选与不选两种情况，但是需要维护一个计数器以保证最终选择的元素个数始终是$k$
个。代码如下：

    void combi(std::vector<int>& output, std::vector<int>& arr, int pos, int count) {
        if (count < 0 || pos >= arr.size()) return;
        else if (count == 0) {
            for (int i=0; i<output.size(); ++i)
                printf("%d ", output[i]);
            printf("\n");
            return;
        }
        output.push_back(arr[pos]);
        combi(output, arr, pos+1, count-1);     // contain arr[i]
        output.pop_back();
        combi(output, arr, pos+1, count);       // not contain arr[i]
        return;
    }

### 电话号码组合问题

LeetCode里有[一道题](http://leetcode.com/onlinejudge#question_17)描述的是给定一
个手机键盘数字键与26个字母的对应图，如何在给定一个数字串后输出其可能的所有字母组
合呢？还是跟全排列类似，代码如下，不再赘述。

    const char Phone[10][10] = {"", " ", "abc", "def", "ghi",
                                "jkl", "mno", "pqrs", "tuv", "wxyz"};
    int Len[10] = {0, 1, 3, 3, 3, 3, 3, 4, 3, 4};

    class Solution {
    public:
        vector<string> letterCombinations(string digits) {
            // Start typing your C/C++ solution below
            // DO NOT write int main() function
            std::vector<string> res;
            string temp;
            permutation(digits, 0, temp, res);
            return res;
        }

        void permutation(const string& digits, int pos, std::string temp_output, std::vector<string>& output) {
            if (pos == digits.length()) {
                output.push_back(temp_output);
                return;
            }
            int index = digits[pos] - '0'; 
            if (index == 0 || index == 1) return;
            for (int i=0; i<Len[index]; ++i) {
                temp_output.push_back(Phone[index][i]);
                permutation(digits, pos+1, temp_output, output);
                temp_output.pop_back();
            }
            return;
        }
    };

* * * 
下面是两个比较有趣的问题，可以看作是两个相反的问题，但是思路上却可以相互借鉴。
为了叙述的方便性，我们假设`K\in[0, N!)`，其中$N$表示排列中元素的个数，并假设集
排列中元素全在`[0, N)`之间，如果不是在这之间，我们可以通过大小比较将这些元素与
`[0, N)`作一个映射。

### 给定一个排列求其在所有字典序中的位置

先举一个例子，假设`S={2, 1, 3, 0}`，第一个元素为2，那么我们就可以知道它一定是至
少落在3!以后（这里说的第i个之后是以0开始计数），因为以0开头的排列就有3!个，以1开
头的排列（也有3!个）在0开头的最后一个排列之后，而以2开头的排列（3!）在1开头的最
后一个排列之后。因此，以2开头就告诉我们它至少在2\*3！之后。再看后面子列`{1, 3,
0}`，如果我们仅考虑由0，1，3构成的排列，我们就可以知道以1开头的排列在0开头排列之
前，而在3开头排列之后，而每个元素开头的排列有2!个，所以`{1, 3, 0}`应该是至少在在
这三个元素构成全排列中第1\*2!个以后（*注意为什么是1\*2!而不是其它，因为以3开头的排
列在1打头的排列之后，比1小的元素只有0这一个，因此它仅仅是在1\*2!个元素之后*）。然
后继续再看子列`{3, 0}`，它在至少在由这两个元素构成的排列中第1个之后，而元素`{0}`
则是在第0个之后，也显然是第0个，把这些所有的结果综合起来，`S={2, 1, 3, 0}`在第
`2*3!+1*2!+1*1!+0`位，即第15个排列。正确性很容易得到验证。

于是算法可以描述如下：   

>  初始化位置计数为0，从第一个元素S1开始，S1落在S1\*(N-1)!以后，因此计数器增加
S1\*(N-1)!。注意，这里为了方便计算后面的子排列中后面比当前元素少的个数，将当前元
素以后所有比当前元素大的都减1，这样对于子列，继续使用S2\*(N-2)!增加到计数器中，一
直这样重复直到最后一个元素。

下面是代码实现：

    // make sure input's value is a permutation of [0~input.size()-1]
    int solveK(std::vector<int>& input) {
        int pos = 0;
        int factorial = 1;
        int n = input.size();
        for (int i=2; i<n; ++i) factorial *= i;

        for (int i=0; i<input.size(); ++i) {
            pos += input[i]*factorial;      // factorial is (n - i - 1)!
            if (n-i-1 > 0)
                factorial /= (n - i - 1);
            for (int j=i+1; j<input.size(); ++j) {
                if (input[j] > input[i]) --input[j];
            }
        }
        return pos;
    }



### 求按字典序排列后全排列中第K个排列

这个题目出现在LeetCode第60题中。由前面的思路，我们知道如果求第 $(N-1)!$个排列，
那么以第二小的元素开头，后面从小到大排列的一个排列就是所求的。从这里面我们是否可
以借鉴一些呢？

对于元素个数为N的一个排列，如果K大于$(N-1)!$，那么第一个元素就是$K/(N-1)!$了，然后
$N-1$个元素构成的子排列，$(K\%(N-1)!)/(N-2)!$就是第二个元素，这时把$K$当作
$K\%(N-1)!$，而$N-1$当作$N$，那么就很容易化为一个递归的问题了。

算法代码如下（递归和非递归两个版本）：

    //递归版本
    vector<int> get_nth(int n, int k) {
        if(n == 1)
            return vector<int>(1, 0);
        int first_digit = k / factorial[n-1];     // 
        vector<int> ans(1, first_digit);
        vector<int> rest = get_nth(n-1, k % factorial[n-1]);
        for(int i = 0; i < rest.size(); i++) {
            if(rest[i] >= first_digit)
                rest[i]++;
            ans.push_back(rest[i]);
        }
        return ans;
    }

    //非递归版本
    class Solution {
    public:
        string getPermutation(int n, int k) {
            // Start typing your C/C++ solution below
            // DO NOT write int main() function
            string res;
            int factorial = 1;
            for (int i=1; i<=n; ++i) {
                res.push_back('0' + i);
                factorial *= i;
            }

            k--;
            k %= factorial;

            for (int i=0; i<n; ++i) {
                factorial /= (n-i);
                int index = k / factorial;
                k %= factorial;
                char temp = res[index];     // 注意这里的技巧，将已经处理的元素全移动到尾部，然后我们就一直是在处理一个子问题了
                for (int j=index+1; j<n-i; ++j) {
                    res[j-1] = res[j];
                }
                res[n-i-1] = temp;
            }
            reverse(res.begin(), res.end());
            return res;
        }
    };


