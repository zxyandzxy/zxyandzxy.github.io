---
layout: post
title: "给墙壁刷油漆"
date: 2024-6-28
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你两个长度为 `n` 下标从 **0** 开始的整数数组 `cost` 和 `time` ，分别表示给 `n` 堵不同的墙刷油漆需要的开销和时间。你有两名油漆匠：

- 一位需要 **付费** 的油漆匠，刷第 `i` 堵墙需要花费 `time[i]` 单位的时间，开销为 `cost[i]` 单位的钱。
- 一位 **免费** 的油漆匠，刷 **任意** 一堵墙的时间为 `1` 单位，开销为 `0` 。但是必须在付费油漆匠 **工作** 时，免费油漆匠才会工作。

请你返回刷完 `n` 堵墙最少开销为多少。

**示例 1：**

```
输入：cost = [1,2,3,2], time = [1,2,3,2]
输出：3
解释：下标为 0 和 1 的墙由付费油漆匠来刷，需要 3 单位时间。同时，免费油漆匠刷下标为 2 和 3 的墙，需要 2 单位时间，开销为 0 。总开销为 1 + 2 = 3 。
```

**示例 2：**

```
输入：cost = [2,3,4,2], time = [1,1,1,1]
输出：4
解释：下标为 0 和 3 的墙由付费油漆匠来刷，需要 2 单位时间。同时，免费油漆匠刷下标为 1 和 2 的墙，需要 2 单位时间，开销为 0 。总开销为 2 + 2 = 4 。
```

**提示：**

- `1 <= cost.length <= 500`
- `cost.length == time.length`
- `1 <= cost[i] <= 106`
- `1 <= time[i] <= 500`

> 错误思路：贪心算法

**思路：**

```
将cost和time数组按照cost递增顺序进行排序，然后每次取cost的最低位idx_pay,消耗的时间time[idx_pay]用于消耗cost的高位。这样理论上总花费是最低的

反例：
cost = [26,53,10,24,25,20,63,51]
time = [1,1,1,1,2,2,2,1]
预期结果是55, 按照贪心算法输出是79。
这里是因为贪心最后一步选择了(24,1) 只消耗了高位的(26, 1)，所以仍然要选择(25,2)

但是更好的选择其实是选(25,2)，这样就直接消耗掉(24,1)和(26,1)
```

**代码：**

```cpp
class Solution {
public:
    static bool cmp(const pair<int, int>& a, const pair<int, int>& b) {
        if (a.first == b.first) {
            return a.second > b.second;
        }
        return a.first < b.first;
    }
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        vector<pair<int, int>> cost_time(n);
        for (int i = 0; i < n; i++) {
            cost_time[i] = make_pair(cost[i], time[i]);
        }
        sort(cost_time.begin(), cost_time.end(), cmp);
        int pay = 0;
        int idx_pay = 0, idx_free = n - 1;
        int remain_time = 0;
        while (idx_pay <= idx_free) {
            pay += cost_time[idx_pay].first;
            remain_time += cost_time[idx_pay].second;
            while (remain_time && idx_free > idx_pay) {
                idx_free--;
                remain_time--;
            }
            idx_pay++;
        }
        return pay;
    }
};
```

> 正确思路：动态规划

**思路：**

```
我们可以考虑每一堵墙是给付费油漆匠刷还是给免费油漆匠刷，设计一个函数 dfs(i,j)，表示从第 i 堵墙开始，且当前剩余的免费油漆匠工作时间为 j 时，刷完剩余所有墙壁的最小开销。那么答案为 dfs(0,0)。

函数 dfs(i,j) 的计算过程如下：
	如果 n − i ≤ j，表示剩余的墙壁不超过免费油漆匠的工作时间，那么剩余的墙壁都由免费油漆匠刷，开销为 0；
	如果 i ≥ n，返回 +∞；
	否则，如果第 i 堵墙由付费油漆匠刷，开销为 cost[i]，那么 dfs(i,j) = dfs(i + 1,j + time[i]) + cost[i]；如果第 i 堵墙由免费油漆匠刷，开销为 0，那么 dfs(i,j) = dfs(i + 1,j − 1)。

	注意，参数 j 可能小于 0，因此，在实际编码过程中，除了 Python 语言外，我们对 j 加上一个偏移量 n，使得 j 的取值范围为 [0,2n]。
```

**代码：**

```cpp
class Solution {
public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        int f[n][n << 1 | 1];
        memset(f, -1, sizeof(f));
        function<int(int, int)> dfs = [&](int i, int j) -> int {
            if (n - i <= j - n) {
                return 0;
            }
            if (i >= n) {
                return 1 << 30;
            }
            if (f[i][j] == -1) {
                f[i][j] = min(dfs(i + 1, j + time[i]) + cost[i], dfs(i + 1, j - 1));
            }
            return f[i][j];
        };
        return dfs(0, n);
    }
};
```

