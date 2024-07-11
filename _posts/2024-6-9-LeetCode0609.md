---
layout: post
title: "相同分数的最大操作数目II"
date: 2024-6-9
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

有 `n` 个气球，编号为`0` 到 `n - 1`，每个气球上都标有一个数字，这些数字存在数组 `nums` 中。

现在要求你戳破所有的气球。戳破第 `i` 个气球，你可以获得 `nums[i - 1] * nums[i] * nums[i + 1]` 枚硬币。 这里的 `i - 1` 和 `i + 1` 代表和 `i` 相邻的两个气球的序号。如果 `i - 1`或 `i + 1` 超出了数组的边界，那么就当它是一个数字为 `1` 的气球。

求所能获得硬币的最大数量。

**示例 1：**

```
输入：nums = [3,1,5,8]
输出：167
解释：
nums = [3,1,5,8] --> [3,5,8] --> [3,8] --> [8] --> []
coins =  3*1*5    +   3*5*8   +  1*3*8  + 1*8*1 = 167
```

**示例 2：**

```
输入：nums = [1,5]
输出：10
```

**提示：**

- `n == nums.length`
- `1 <= n <= 300`
- `0 <= nums[i] <= 100`

**思路：**

```
这道题和昨天的题其实很像，采用的思路都是动态规划 + 深度优先搜索

为了方便处理，我们将nums数组加上两端的nums[-1]和nums[n]得到数组val。
即val[i] = nums[i - 1];下文所说区间都是val的区间

我们观察戳气球的操作，会将不相邻的气球变成相邻的气球。在数组中删元素然后处理新数据是很困难的，所以我们反过来想。将全过程当成添加气球。举个例子：示例1中，如果是加气球的操作，那么过程就是从右到左。最终得到的结果相同。

我们定义solve(i, j)为开区间(i, j)全部填满气球能获得的最高分数，此时两端的气球为val[i]和val[j]。
	当i >= j - 1时，开区间没有元素，solve(i, j) = 0;
	当i < j - 1时，我们要往区间里面添加元素，由于要添加哪一个是不知道的，所以我们枚举所有情况，比如添加mid索引上的元素；那么solve(i, j) = val[i] * val[mid] * val[j] + solve(i, mid) + solve(mid, j)。所有mid取最大结果就是solve(i, j)最后答案
	为了防止重复计算，我们存储solve(i, j)的结果
```

**代码：**

```cpp
class Solution {
public:
    vector<vector<int>> rec;
    vector<int> val;

public:
    int solve(int left, int right) {
        if (left >= right - 1) {
            return 0;
        }
        if (rec[left][right] != -1) {
            return rec[left][right];
        }
        for (int i = left + 1; i < right; i++) {
            int sum = val[left] * val[i] * val[right];
            sum += solve(left, i) + solve(i, right);
            rec[left][right] = max(rec[left][right], sum);
        }
        return rec[left][right];
    }

    int maxCoins(vector<int>& nums) {
        int n = nums.size();
        val.resize(n + 2);
        for (int i = 1; i <= n; i++) {
            val[i] = nums[i - 1];
        }
        val[0] = val[n + 1] = 1;
        rec.resize(n + 2, vector<int>(n + 2, -1));
        return solve(0, n + 1);
    }
};
```
