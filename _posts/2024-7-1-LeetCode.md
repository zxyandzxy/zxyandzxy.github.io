---
layout: post
title: "最大化一张图中的路径价值"
date: 2024-7-1
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一张 **无向** 图，图中有 `n` 个节点，节点编号从 `0` 到 `n - 1` （**都包括**）。同时给你一个下标从 **0** 开始的整数数组 `values` ，其中 `values[i]` 是第 `i` 个节点的 **价值** 。同时给你一个下标从 **0** 开始的二维整数数组 `edges` ，其中 `edges[j] = [uj, vj, timej]` 表示节点 `uj` 和 `vj` 之间有一条需要 `timej` 秒才能通过的无向边。最后，给你一个整数 `maxTime` 。

**合法路径** 指的是图中任意一条从节点 `0` 开始，最终回到节点 `0` ，且花费的总时间 **不超过** `maxTime` 秒的一条路径。你可以访问一个节点任意次。一条合法路径的 **价值** 定义为路径中 **不同节点** 的价值 **之和** （每个节点的价值 **至多** 算入价值总和中一次）。

请你返回一条合法路径的 **最大** 价值。**注意：**每个节点 **至多** 有 **四条** 边与之相连。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/10/19/ex1drawio.png)

```
输入：values = [0,32,10,43], edges = [[0,1,10],[1,2,15],[0,3,10]], maxTime = 49
输出：75
解释：
一条可能的路径为：0 -> 1 -> 0 -> 3 -> 0 。总花费时间为 10 + 10 + 10 + 10 = 40 <= 49 。
访问过的节点为 0 ，1 和 3 ，最大路径价值为 0 + 32 + 43 = 75 。
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2021/10/19/ex2drawio.png)

```
输入：values = [5,10,15,20], edges = [[0,1,10],[1,2,10],[0,3,10]], maxTime = 30
输出：25
解释：
一条可能的路径为：0 -> 3 -> 0 。总花费时间为 10 + 10 = 20 <= 30 。
访问过的节点为 0 和 3 ，最大路径价值为 5 + 20 = 25 。
```

**示例 3：**

![img](https://assets.leetcode.com/uploads/2021/10/19/ex31drawio.png)

```
输入：values = [1,2,3,4], edges = [[0,1,10],[1,2,11],[2,3,12],[1,3,13]], maxTime = 50
输出：7
解释：
一条可能的路径为：0 -> 1 -> 3 -> 1 -> 0 。总花费时间为 10 + 13 + 13 + 10 = 46 <= 50 。
访问过的节点为 0 ，1 和 3 ，最大路径价值为 1 + 2 + 4 = 7 。
```

**示例 4：**

**![img](https://assets.leetcode.com/uploads/2021/10/21/ex4drawio.png)**

```
输入：values = [0,1,2], edges = [[1,2,10]], maxTime = 10
输出：0
解释：
唯一一条路径为 0 。总花费时间为 0 。
唯一访问过的节点为 0 ，最大路径价值为 0 。
```

**提示：**

- `n == values.length`
- `1 <= n <= 1000`
- `0 <= values[i] <= 108`
- `0 <= edges.length <= 2000`
- `edges[j].length == 3 `
- `0 <= uj < vj <= n - 1`
- `10 <= timej, maxTime <= 100`
- `[uj, vj]` 所有节点对 **互不相同** 。
- 每个节点 **至多有四条** 边。
- 图可能不连通。

**思路：**

```
仔细阅读题目描述我们可以发现，timeⱼ 的最小值为 10，而 maxTime 的最大值为 100。这说明我们至多只会经过图上的 10 条边。由于图中每个节点的度数都不超过 4，因此我们可以枚举所有从节点 0 开始的路径。

我们可以使用递归 + 回溯的方法进行枚举。递归函数记录当前所在的节点编号，已经过的路径的总时间以及节点的价值之和。如果当前在节点 u，我们可以枚举与 u 直接相连的节点 v 进行递归搜索。在搜索的过程中，如果我们回到了节点 0，就可以对答案进行更新；如果总时间超过了 maxTime，我们需要停止搜索，进行回溯。
```

**代码：**

```cpp
class Solution {
public:
    int maximalPathQuality(vector<int>& values, vector<vector<int>>& edges, int maxTime) {
        // n 是节点数
        int n= values.size();
        // 建立邻接表
        vector<vector<pair<int, int>>> g(n);
        for (const auto& edge :edges) {
            g[edge[0]].emplace_back(edge[1], edge[2]);
            g[edge[1]].emplace_back(edge[0], edge[2]);
        }
        // visited数组表明是否访问过
        vector<int> visited(n);
        visited[0] = true;
        int ans = 0;
        // u 是当前访问节点， time是已经过的时间 value是当前获得总价值
        function<void(int, int, int)> dfs = [&](int u, int time, int value) {
            // 回到原点，说明找到一个可能路径，记录最大值
            if (u == 0) {
                ans = max(ans, value);
            }
            // 从当前节点出发，进行深搜
            for (const auto& [v, dist] : g[u]) {
                // 合法路径
                if (time + dist <= maxTime) {
                    // 没访问过的节点，它的价值就可以增加
                    if (!visited[v]) {
                        visited[v] = true;
                        dfs(v, time + dist, value + values[v]);
                        visited[v] = false;
                    }
                    // 访问过的节点，其价值不用再加
                    else {
                        dfs(v, time + dist, value);
                    }
                }
            }
        };

        dfs(0, 0, values[0]);
        return ans;
    }
};
```

