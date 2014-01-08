---
layout: post
title: 二分查找及其变形
categories:
- Algorithms
tags:
- 面试题
- 算法
- 二分查找
---

二分查找及其变种非常地多，看似简单，实际的实现却充斥着陷阱，很能筛选掉一批眼高手
低的人。下面就其中一些进行分析，分析的过程我们可以使用循环不变式进行验证，保证程
序是正确的。

#### 旋转数组返回最小元素

当数组本身是有序的时候，最小元素显然是最左元素；而如果数组旋转点在中间某个位置，
可以选择将数组一分为二，判断哪一部分包含了旋转的数组（那另一部分肯定是有序的），
然后将问题减半为从包含旋转的数组中寻找旋转点。

代码可以如下：（无重复元素情况）

    while (A[left] > A[right]) {
        int mid = (left + right) / 2;
        if (A[mid] < A[left]) right = mid;
        else left = mid + 1;
    }
    return left;

#### 旋转数组查找元素

对于没有重复元素的情况下，判断起来会比较简单。还是和上面有些类似，将数组一分为二
。分为有序和旋转有序的两部分，然后判断元素落在哪一个部分，如果在有序的部分中，直
接用二分返回结果（有可能还是找不到），否则就继续递归地调用旋转有序查找函数。

    // Leetcode 033
    class Solution {
    public:
        int search(int A[], int n, int target) {
            // Start typing your C/C++ solution below
            // DO NOT write int main() function
            int left = 0, right = n - 1;
            while (left <= right) {
                int mid = ((left + right) >> 1);
                if (A[mid] == target) return mid;

                if (A[left] <= A[mid]) {
                    if (A[mid] > target && A[left] <= target) right = mid - 1;
                    else left = mid + 1;
                } else {
                    if (A[mid] < target && A[right] >= target) left = mid + 1;
                    else right = mid - 1;
                }
            }
            return -1;
        }
    };


对于有重复元素的情况，几个比较有效的测试数据为`{2,1,1,1,1}`, `1`, `2,1,2,2,2,2,2`, `5,6,1,2,3,4`，`1,2,1,1,1,1`。

    // Leetcode 081
    class Solution {
    public:
        bool search(int A[], int n, int target) {
            // Start typing your C/C++ solution below
            // DO NOT write int main() function
            int left = 0, right = n - 1;
            while (left <= right) {
                int mid = ((left + right) >> 1);
                if (A[mid] == target) return true;
                if (A[left] < A[mid]) {
                    if (target < A[mid] && A[left] <= target) right = mid - 1; 
                    else left = mid + 1;
                } else if (A[left] > A[mid]) {
                    if (target > A[mid] && A[right] >= target) left = mid + 1; 
                    else right = mid - 1;
                } else left++;
            }
            return false;
        }
    };

### 二分查找的变种

正常的二分是最简单的，它通过每次比较直接与一半的元素进行比较而达到目的。不再叙述。

#### 正常二分，只不过返回时不是-1，而是被插入的位置

为了验证，我们使用循环不变式“`[0, left)`所有元素都小于val，而所有`(right,
N-1]`大于val。”显然，当元素不存在的时候，`arr[right+1]`已经是最小的一个大于val
的元素了，因此返回的是`right+1`。

    while (left <= right) {
        if (arr[mid] == val) return mid;
        else if (arr[mid] < val) left = mid + 1;
        else right = mid - 1;
    }
    return right + 1;

#### 存在就返回第一个或最后一个相等的位置

返回最后一个相等的位置：此时循环不变式可以表示为“`[0, left)`所有元素小于等于val
，而`(right, N-1]`所有元素都大于val”，这样当程序终止的时候我们`arr[right]`就是
第一个小于等于val的元素了。

而返回第一个相等的位置：此时循环不变式可以表示为“`[0, left)`所有元素小于val，而
`(right, N-1]`所有元素都大于等于val”，这样当程序终止的时候我们`arr[left]`就是第
一个大于等于val的元素了。

程序如下：

	int lowerBound(std::vector<int>& arr, int val) {
		int left = 0, right = arr.size() - 1;
		while (left <= right) {
			int mid = (left + right) / 2;
			if (arr[mid] < val) left = mid + 1;
			else right = mid - 1;
		}
		if (left < arr.size() && arr[left] == val) return left;
		return -1;
	}
	int upperBound(std::vector<int>& arr, int val) {
		int left = 0, right = arr.size() - 1;
		while (left <= right) {
			int mid = (left + right) / 2;
			if (arr[mid] > val) right = mid - 1;
			else left = mid + 1;
		}
		if (right >= 0 && arr[right] == val) return right;
		return -1;
	}

#### 有序矩阵中二分查找

如果矩阵中每一行元素都是有序的，而且下一行的元素都大于等于上一行元素，这种情况下
把矩阵抽象成一个一维数组即可用普通的二分完成。

但是，如果矩阵元素满足每一行中右边的比左边的大，每一列下一个比上一个大，那么如何
快速查找呢？

一种方法是与中间对角的两元素比较，从而递归解决，这种方法是比较麻烦的。

另一种方法是从矩阵的右上角元素开始作比较，如果小于，则向左滑动；如果大于则向下滑
动，从而可以减小搜索的空间而解决一个子问题。时间复杂度最坏情况下是`O(m+n)`。


#### 求两个有序数组合并后的第K个元素

