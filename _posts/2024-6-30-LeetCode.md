---
layout: post
title: "目标和"
date: 2024-6-30
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个非负整数数组 `nums` 和一个整数 `target` 。

向数组中的每个整数前添加 `'+'` 或 `'-'` ，然后串联起所有整数，可以构造一个 **表达式** ：

- 例如，`nums = [2, 1]` ，可以在 `2` 之前添加 `'+'` ，在 `1` 之前添加 `'-'` ，然后串联起来得到表达式 `"+2-1"` 。

返回可以通过上述方法构造的、运算结果等于 `target` 的不同 **表达式** 的数目。

**示例 1：**

```
输入：nums = [1,1,1,1,1], target = 3
输出：5
解释：一共有 5 种方法让最终目标和为 3 。
-1 + 1 + 1 + 1 + 1 = 3
+1 - 1 + 1 + 1 + 1 = 3
+1 + 1 - 1 + 1 + 1 = 3
+1 + 1 + 1 - 1 + 1 = 3
+1 + 1 + 1 + 1 - 1 = 3
```

**示例 2：**

```
输入：nums = [1], target = 1
输出：1
```

**提示：**

- `1 <= nums.length <= 20`
- `0 <= nums[i] <= 1000`
- `0 <= sum(nums[i]) <= 1000`
- `-1000 <= target <= 1000`

> 回溯算法

**思路：**

```
数组 nums 的每个元素都可以添加符号 + 或 -，因此每个元素有 2 种添加符号的方法，n 个数共有 2^n 种添加符号的方法，对应 2^n 种不同的表达式。当 n 个元素都添加符号之后，即得到一种表达式，如果表达式的结果等于目标数 target，则该表达式即为符合要求的表达式。

可以使用回溯的方法遍历所有的表达式，回溯过程中维护一个计数器 count，当遇到一种表达式的结果等于目标数 target 时，将 count 的值加 1。遍历完所有的表达式之后，即可得到结果等于目标数 target 的表达式的数目。

时间复杂度：O(2^n)，其中 n 是数组 nums 的长度。回溯需要遍历所有不同的表达式，共有 2 ^n 种不同的表达式，每种表达式计算结果需要 O(1) 的时间，因此总时间复杂度是 O(2^n)。

空间复杂度：O(n)，其中 n 是数组 nums 的长度。空间复杂度主要取决于递归调用的栈空间，栈的深度不超过 n。
```

**代码：**

```cpp
class Solution {
public:
    int count = 0;

    int findTargetSumWays(vector<int>& nums, int target) {
        backtrack(nums, target, 0, 0);
        return count;
    }

    void backtrack(vector<int>& nums, int target, int index, int sum) {
        if (index == nums.size()) {
            if (sum == target) {
                count++;
            }
        } else {
            backtrack(nums, target, index + 1, sum + nums[index]);
            backtrack(nums, target, index + 1, sum - nums[index]);
        }
    }
};

注意注意：这里不能用for循环取下面的每个元素，因为题目要求的是所有元素都必须用上。
```

> 动态规划

**思路：**

```
记数组的元素和为 sum，添加 - 号的元素之和为 neg，则其余添加 + 的元素之和为 sum − neg，得到的表达式的结果为 (sum − neg) − neg = sum − 2 * neg = target。即 neg = 
(sum - target) / 2。由于数组 nums 中的元素都是非负整数，neg 也必须是非负整数，所以上式成立的前提是 sum−target 是非负偶数。若不符合该条件可直接返回 0。

若上式成立，问题转化成在数组 nums 中选取若干元素，使得这些元素之和等于 neg，计算选取元素的方案数。我们可以使用动态规划的方法求解。

定义二维数组 dp，其中 dp[i][j] 表示在数组 nums 的前 i 个数中选取元素，使得这些元素之和等于 j 的方案数。假设数组 nums 的长度为 n，则最终答案为 dp[n][neg]。

当没有任何元素可以选取时，元素和只能是 0，对应的方案数是 1，因此动态规划的边界条件是：
  dp[0][j] = 1 while j = 0;
  dp[0][j] = 0 while j >= 1;
  
当 1 ≤ i ≤ n 时，对于数组 nums 中的第 i 个元素 num（i 的计数从 1 开始），遍历 0 ≤ j ≤ neg，计算 dp[i][j] 的值：
	如果 j < num，则不能选 num，此时有 dp[i][j] = dp[i−1][j]；
	如果 j ≥ num，则如果不选 num，方案数是 dp[i−1][j]，如果选 num，方案数是 dp[i−1][j−num]，此时有 dp[i][j] = dp[i−1][j] + dp[i−1][j−num]。

最终得到 dp[n][neg] 的值即为答案。
由此可以得到空间复杂度为 O(n × neg) 的实现。
```

**代码：**

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int target) {
        int sum = 0;
        for (int& num : nums) {
            sum += num;
        }
        int diff = sum - target;
        if (diff < 0 || diff % 2 != 0) {
            return 0;
        }
        int n = nums.size(), neg = diff / 2;
        vector<vector<int>> dp(n + 1, vector<int>(neg + 1));
        dp[0][0] = 1;
        for (int i = 1; i <= n; i++) {
            int num = nums[i - 1];
            for (int j = 0; j <= neg; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= num) {
                    dp[i][j] += dp[i - 1][j - num];
                }
            }
        }
        return dp[n][neg];
    }
};
```

