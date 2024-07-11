---
layout: post
title: "矩阵中严格递增的单元格数"
date: 2024-6-19
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个下标从 **1** 开始、大小为 `m x n` 的整数矩阵 `mat`，你可以选择任一单元格作为 **起始单元格** 。从起始单元格出发，你可以移动到 **同一行或同一列** 中的任何其他单元格，但前提是目标单元格的值 **严格大于** 当前单元格的值。

你可以多次重复这一过程，从一个单元格移动到另一个单元格，直到无法再进行任何移动。请你找出从某个单元开始访问矩阵所能访问的 **单元格的最大数量** 。返回一个表示可访问单元格最大数量的整数。

**示例 1：**

**![img](https://assets.leetcode.com/uploads/2023/04/23/diag1drawio.png)**

```
输入：mat = [[3,1],[3,4]]
输出：2
解释：上图展示了从第 1 行、第 2 列的单元格开始，可以访问 2 个单元格。可以证明，无论从哪个单元格开始，最多只能访问 2 个单元格，因此答案是 2 。 
```

**示例 2：**

**![img](https://assets.leetcode.com/uploads/2023/04/23/diag4drawio.png)**

```
输入：mat = [[3,1,6],[-9,5,7]]
输出：4
解释：上图展示了从第 2 行、第 1 列的单元格开始，可以访问 4 个单元格。可以证明，无论从哪个单元格开始，最多只能访问 4 个单元格，因此答案是 4 。  
```

**提示：**

- `m == mat.length `
- `n == mat[i].length `
- `1 <= m, n <= 105`
- `1 <= m * n <= 105`
- `-105 <= mat[i][j] <= 105`

**思路：**

```
提示 1
按元素值从小到大计算。

提示 2
定义 f[i][j] 表示到达 mat[i][j] 时，访问过的单元格的最大数量（包含 mat[i][j]）。那么答案就是所有 f[i][j] 的最大值。

如何计算 f[i][j] 从哪转移过来？
请注意，我们不需要知道具体从哪个单元格转移过来，只需要知道转移来源的 f 的最大值是多少。

提示 3
按照元素值从小到大计算，那么第 i 行的比 mat[i][j] 小的 f 值都算出来了，大于等于 mat[i][j] 的尚未计算，视作 0。

所以对于第 i 行，相当于取这一行的 f 值的最大值，作为转移来源的值。我们用一个长为 m 的数组 rowMax 维护每一行的 f 值的最大值。
对于每一列，也同理，用一个长为 n 的数组 colMax 维护。

所以有 f[i][j] = max⁡(rowMax[i], colMax[j]) + 1 其中 + 1 是把 mat[i][j] 算上。

细节
代码实现时 f[i][j] 可以省略，因为只需要知道每行每列的 f 值的最大值。
对于相同元素，先计算出 (i,j) 处的最大值，再更新到 rowMax 和 colMax 中。
最后答案为 rowMax 的最大值。（或者 colMax 的最大值，由于这两个最大值相等，计算其一即可。）
```

**代码：**

```cpp
//其实思路我也没怎么看懂，给我的一种直观感觉就是暴力更新，通过对mat中的每个元素进行dp[i][j]的更新，在这个过程中更新row_max和col_max。
class Solution {
public:
    int maxIncreasingCells(vector<vector<int>>& mat) {
        int m = mat.size(), n = mat[0].size();
        map<int, vector<pair<int, int>>> g;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                g[mat[i][j]].emplace_back(i, j); // 相同元素放在同一组，统计位置
            }
        }

        vector<int> row_max(m), col_max(n);
        for (auto& [_, pos] : g) {
            vector<int> mx; // 先把最大值算出来，再更新 row_max 和 col_max
            for (auto& [i, j] : pos) {
                mx.push_back(max(row_max[i], col_max[j]) + 1);
            }
            for (int k = 0; k < pos.size(); k++) {
                auto& [i, j] = pos[k];
                row_max[i] = max(row_max[i], mx[k]); // 更新第 i 行的最大 f 值
                col_max[j] = max(col_max[j], mx[k]); // 更新第 j 列的最大 f 值
            }
        }
        return ranges::max(row_max);
    }
};
```



