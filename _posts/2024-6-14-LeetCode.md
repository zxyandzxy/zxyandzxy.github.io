---
layout: post
title: "子序列最大优雅度"
date: 2024-6-14
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个下标从 **0** 开始的整数数组 `nums` 和一个正整数 `x` 。你 **一开始** 在数组的位置 `0` 处，你可以按照下述规则访问数组中的其他位置：

- 如果你当前在位置 `i` ，那么你可以移动到满足 `i < j` 的 **任意** 位置 `j` 。
- 对于你访问的位置 `i` ，你可以获得分数 `nums[i]` 。
- 如果你从位置 `i` 移动到位置 `j` 且 `nums[i]` 和 `nums[j]` 的 **奇偶性** 不同，那么你将失去分数 `x` 。

请你返回你能得到的 **最大** 得分之和。**注意** ，你一开始的分数为 `nums[0]` 。

**示例 1：**

```
输入：nums = [2,3,6,1,9,2], x = 5
输出：13
解释：我们可以按顺序访问数组中的位置：0 -> 2 -> 3 -> 4 。
对应位置的值为 2 ，6 ，1 和 9 。因为 6 和 1 的奇偶性不同，所以下标从 2 -> 3 让你失去 x = 5 分。
总得分为：2 + 6 + 1 + 9 - 5 = 13 。
```

**示例 2：**

```
输入：nums = [2,4,6,8], x = 3
输出：20
解释：数组中的所有元素奇偶性都一样，所以我们可以将每个元素都访问一次，而且不会失去任何分数。
总得分为：2 + 4 + 6 + 8 = 20 。
```

**提示：**

- `2 <= nums.length <= 105`
- `1 <= nums[i], x <= 106`

> 暴力动态规划

**思路：**

```
从题目分析出来，我们要的是移动的最大分数，没说一定要移动到最后一个元素。所以我们要用res记录过程中的最大分数。

从结果反推，一定是移动到某一个位置获取的最大分数；这个位置一定是从它前面的某个位置移动而来，且保证这个移动让分数最大。由于我们不知道从哪个位置移动而来分数最大，所以从0开始判断所有位置。

这么想其实就是动态规划了。
1.dp数组含义
dp[i]表示从0移动到索引i可以获得的最大分数

2.确定递推公式
if nums[i] 与 nums[j] 奇偶性不同 then dp[i] = dp[j] + nums[i] - x
else dp[i] = dp[j] + nums[i]
对于j ∈ [0, i),我们取dp[i]的最大值

3.确定递推顺序 ： 从0 到 n
4.确定初始化：由于要取dp[i]最大值，所以初始化为INT_MIN
```

**代码：**

```cpp
class Solution {
public:
    long long maxScore(vector<int>& nums, int x) {
        int n = nums.size();
        vector<int> dp(n, INT_MIN);
        int ans = nums[0];
        dp[0] = nums[0];
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if ((nums[i]%2 == 0 && nums[j]%2 == 1) || (nums[i]%2 == 1 && nums[j]%2 == 0)) {
                    dp[i] = max(dp[i], dp[j] + nums[i] - x);
                }else {
                    dp[i] = max(dp[i], dp[j] + nums[i]);
                }
                ans = max(dp[i], ans);
            }
        }
        return ans;
    }
};
```

> 精细化动态规划

**思路：**

```
之前我们说，不知道从哪个位置转移到dp[i]是得分最大的，其实可以确定：
	要想dp[i]得分最大，那么移动到它的前一个元素要么是奇数，要么是偶数。其次移动到这个元素的得分一定是[0, i)最大的
	所以其实我们可以保存这么两个元素的得分，然后进行转移
```

**代码：**

```cpp
class Solution {
public:
    long long maxScore(vector<int>& nums, int x) {
        long long res = nums[0];
        vector<long long> dp(2, INT_MIN);
        dp[nums[0] % 2] = nums[0];
        for (int i = 1; i < nums.size(); i++) {
            int parity = nums[i] % 2;
            long long cur = max(dp[parity] + nums[i], dp[1 - parity] - x + nums[i]);
            res = max(res, cur);
            dp[parity] = max(dp[parity], cur);
        }
        return res;
    }
};
```
