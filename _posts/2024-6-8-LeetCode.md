---
layout: post
title: "相同分数的最大操作数目II"
date: 2024-6-8
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个整数数组 `nums` ，如果 `nums` **至少** 包含 `2` 个元素，你可以执行以下操作中的 **任意** 一个：

- 选择 `nums` 中最前面两个元素并且删除它们。
- 选择 `nums` 中最后两个元素并且删除它们。
- 选择 `nums` 中第一个和最后一个元素并且删除它们。

一次操作的 **分数** 是被删除元素的和。在确保 **所有操作分数相同** 的前提下，请你求出 **最多** 能进行多少次操作。请你返回按照上述要求 **最多** 可以进行的操作次数。

**示例 1：**

```
输入：nums = [3,2,1,2,3,4]
输出：3
解释：我们执行以下操作：
- 删除前两个元素，分数为 3 + 2 = 5 ，nums = [1,2,3,4] 。
- 删除第一个元素和最后一个元素，分数为 1 + 4 = 5 ，nums = [2,3] 。
- 删除第一个元素和最后一个元素，分数为 2 + 3 = 5 ，nums = [] 。
由于 nums 为空，我们无法继续进行任何操作。
```

**示例 2：**

```
输入：nums = [3,2,6,1,4]
输出：2
解释：我们执行以下操作：
- 删除前两个元素，分数为 3 + 2 = 5 ，nums = [6,1,4] 。
- 删除最后两个元素，分数为 1 + 4 = 5 ，nums = [6] 。
至多进行 2 次操作。
```

**提示：**

- `2 <= nums.length <= 2000`
- `1 <= nums[i] <= 1000`

**思路：**

```
这题咋一看很难，是因为首先它的操作有三个（不只一个就说明要记录结果并比较了）；其次是因为确定了分数后，每一次删减元素都有三种选择，必须记录结果并取最大值。所以这么一分析，做法肯定是动态规划。

首先我们要抽象问题，要求取得相同分数的最大操作次数，我们第一步就是确定分数，这里分数只有三种可能。第一：nums[0] + nums[1] 第二：nums[0] + nums[n - 1] 第三：nums[n - 1] + nums[n - 2]
确定好了分数target之后，我们再去动态规划求取在该分数下的最大操作次数

1.确定dp数组含义
memo[i][j]代表从区间[i, j]中进行删减元素可以进行的最大操作次数

2.确定递推公式
我们用函数dfs(i, j)来获取区间[i, j]的最大操作次数
对于区间[i, j]来说，它的结果可以由三种可能性确定：
	nums[i] + nums[i + 1] == target, memo[i][j] = dfs(i + 2, j) + 1
	nums[j - 1] + nums[j - 2] == target, memo[i][j] = dfs(i, j - 2) + 1
	nums[i] + nums[j] == target, memo[i][j] = dfs(i + 1, j - 1) + 1
我们取三个的最大值即是memo[i][j]

3.确定遍历顺序
其实没有遍历顺序，最初的入口是dfs(0, n - 1)

4.确定初始化条件
由于要取最大值，所以memo[i][j] = -1

说一个点，这题看似动态规划，但其实核心点还是在于深度优先搜索。
在确定好分数之后，其实就是构建树的过程。每次从三棵子树中选择一个最大的进行搜索
```

**代码：**

```cpp
class Solution {
public:
    int maxOperations(vector<int>& nums) {
        int n = nums.size();
        int memo[n][n];
        auto helper = [&](int i, int j, int target) -> int{
            memset(memo, -1, sizeof(memo));
            function<int(int, int)> dfs = [&](int i, int j) -> int{
                //区间不合理
                if (i >= j) { return 0; }
                //已经访问过，没必要再访问
                if (memo[i][j] != -1) { return memo[i][j]; }
                int ans = 0;
                if (nums[i] + nums[i + 1] == target) {
                    ans = max(ans, 1 + dfs(i + 2, j));
                }
                if (nums[i] + nums[j] == target) {
                    ans = max(ans, 1 + dfs(i + 1, j - 1));
                }
                if (nums[j - 1] + nums[j] == target) {
                    ans = max(ans, 1 + dfs(i, j - 2));
                }
                memo[i][j] = ans;
                return ans;
            };
            return dfs(i, j);
        };
        int res = 0;
        res = max(res, helper(0, n - 1, nums[0] + nums[n - 1]));
        res = max(res, helper(0, n - 1, nums[0] +nums[1]));
        res = max(res, helper(0, n - 1, nums[n - 2] + nums[n - 1]));
        return res;
    }
};
```



