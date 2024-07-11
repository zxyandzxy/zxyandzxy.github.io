---
layout: post
title: "下一个更大元素II"
date: 2024-6-24
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给定一个循环数组 `nums` （ `nums[nums.length - 1]` 的下一个元素是 `nums[0]` ），返回 *`nums` 中每个元素的 **下一个更大元素*** 。

数字 `x` 的 **下一个更大的元素** 是按数组遍历顺序，这个数字之后的第一个比它更大的数，这意味着你应该循环地搜索它的下一个更大的数。如果不存在，则输出 `-1` 。

**示例 1:**

```
输入: nums = [1,2,1]
输出: [2,-1,2]
解释: 第一个 1 的下一个更大的数是 2；
数字 2 找不到下一个更大的数； 
第二个 1 的下一个最大的数需要循环搜索，结果也是 2。
```

**示例 2:**

```
输入: nums = [1,2,3,4,3]
输出: [2,3,4,-1,4]
```

**提示:**

- `1 <= nums.length <= 104`
- `0 <= nums[i] <= 109`

**思路：**

```
这题LeetCode上给定nums数值范围有问题.
按照题目所要求的的如果没有比nums[i]更大的元素，那么答案数组i处应该为-1
那么nums的每个元素应该都要大于0

由于要寻找比本元素大的右侧第一个元素，如果是暴力解法就直接双重for循环

但是我们其实可以利用单调栈，按照栈从底到顶元素大小递减的规则将元素进栈
那么当某个元素i比栈顶元素大时，就说明它是比栈顶元素大的右侧第一个元素

由于数组是循环的，所以我们复制原数组拼接在原数组之后，索引就对数组长度取模，这样就不用考虑循环带来的影响
```

**代码：**

```cpp
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        stack<int> stk;
        int n = nums.size();
        //int maxNum = INT_MIN;
        vector<int> ans(n, -1);  // 初始化为-1，就不用考虑最大值
        // 单调栈内存索引，索引对n取模
        for (int i = 0; i < n * 2; i++) {
            while (stk.size() > 0 && nums[stk.top() % n] < nums[i % n]) {
                ans[stk.top() % n] = nums[i % n];
                stk.pop();
            }
            //maxNum = max(maxNum, nums[i % n]);
            stk.push(i % n);
        }
        /*
        for (int i = 0; i < n; i++) {
            if (nums[i] == maxNum) {
                ans[i] = -1;
            }
        }
        */
        return ans;
    }
};
```



