---
layout: post
title: "找出第K大的异或坐标值"
date: 2024-5-26
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个二维矩阵 `matrix` 和一个整数 `k` ，矩阵大小为 `m x n` 由非负整数组成。

矩阵中坐标 `(a, b)` 的 **值** 可由对所有满足 `0 <= i <= a < m` 且 `0 <= j <= b < n` 的元素 `matrix[i][j]`（**下标从 0 开始计数**）执行异或运算得到。

请你找出 `matrix` 的所有坐标中第 `k` 大的值（**`k` 的值从 1 开始计数**）。

**示例 1：**

```
输入：matrix = [[5,2],[1,6]], k = 1
输出：7
解释：坐标 (0,1) 的值是 5 XOR 2 = 7 ，为最大的值。
```

**提示：**

- `m == matrix.length`
- `n == matrix[i].length`
- `1 <= m, n <= 1000`
- `0 <= matrix[i][j] <= 106`
- `1 <= k <= m * n`

**思路：**

```
我们直接构造出matrix中坐标对应的值，然后将值都放到优先级队列中，
优先级队列中弹出的第K个元素就是答案
```

**代码：**

```cpp
class Solution {
public:
    int kthLargestValue(vector<vector<int>>& matrix, int k) {
        priority_queue<int> que;
        int m = matrix.size(), n = matrix[0].size();
        vector<vector<int>> res(m, vector<int>(n));
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 && j == 0) {
                    res[i][j] = matrix[i][j];
                }else if (i == 0) {
                    res[i][j] = res[i][j - 1] ^ matrix[i][j];
                }else if (j == 0) {
                    res[i][j] = res[i - 1][j] ^ matrix[i][j];
                }else {
                    int cur = res[i - 1][j];
                    for (int k = 0; k <= j; k++) {
                        cur ^= matrix[i][k];
                    }
                    res[i][j] = cur;
                }
                que.push(res[i][j]);
            }
        }
        while (k > 1) {
            que.pop();
            k--;
        }
        return que.top();
    }
};
```

