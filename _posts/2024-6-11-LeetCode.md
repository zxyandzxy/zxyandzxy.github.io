---
layout: post
title: "甲板上的战舰"
date: 2024-6-11
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个大小为 `m x n` 的矩阵 `board` 表示甲板，其中，每个单元格可以是一艘战舰 `'X'` 或者是一个空位 `'.'` ，返回在甲板 `board` 上放置的 **舰队** 的数量。

**舰队** 只能水平或者垂直放置在 `board` 上。换句话说，舰队只能按 `1 x k`（`1` 行，`k` 列）或 `k x 1`（`k` 行，`1` 列）的形状建造，其中 `k` 可以是任意大小。两个舰队之间至少有一个水平或垂直的空位分隔 （即没有相邻的舰队）。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/04/10/battelship-grid.jpg)

```
输入：board = [["X",".",".","X"],[".",".",".","X"],[".",".",".","X"]]
输出：2
```

**示例 2：**

```
输入：board = [["."]]
输出：0
```

**提示：**

- `m == board.length`
- `n == board[i].length`
- `1 <= m, n <= 200`
- `board[i][j]` 是 `'.'` 或 `'X'`

**思路：**

```
LeetCode中文网站上的描述极致抽象，翻译都翻译错了，真的服了

这题要求舰队数量，那么其实就是深度优先搜索，遇到一个board[i][j] == 'X',就将它横向，纵向连着的'X'变成'.'，表示舰队数 + 1。最后统计有多少个舰队即可
```

**代码：**

```cpp
class Solution {
public:
    int countBattleships(vector<vector<char>>& board) {
        int m = board.size(), n = board[0].size();
        int ans = 0;
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (board[i][j] == 'X') {
                    board[i][j] = '.';
                    for (int k = j + 1; k < n && board[i][k] == 'X'; k++) {
                        board[i][k] = '.';
                    }
                    for (int k = i + 1; k < m && board[k][j] == 'X'; k++) {
                        board[k][j] = '.';
                    }
                    ans++;
                }
            }
        }
        return ans;
    }
};
```
