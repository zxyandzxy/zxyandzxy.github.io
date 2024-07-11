---
layout: post
title: "在带权树网络中统计可连接服务器对数目"
date: 2024-6-4
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一棵无根带权树，树中总共有 `n` 个节点，分别表示 `n` 个服务器，服务器从 `0` 到 `n - 1` 编号。同时给你一个数组 `edges` ，其中 `edges[i] = [ai, bi, weighti]` 表示节点 `ai` 和 `bi` 之间有一条双向边，边的权值为 `weighti` 。再给你一个整数 `signalSpeed` 。

如果两个服务器 `a` ，`b` 和 `c` 满足以下条件，那么我们称服务器 `a` 和 `b` 是通过服务器 `c` **可连接的** ：

- `a < b` ，`a != c` 且 `b != c` 。
- 从 `c` 到 `a` 的距离是可以被 `signalSpeed` 整除的。
- 从 `c` 到 `b` 的距离是可以被 `signalSpeed` 整除的。
- 从 `c` 到 `b` 的路径与从 `c` 到 `a` 的路径没有任何公共边。

请你返回一个长度为 `n` 的整数数组 `count` ，其中 `count[i]` 表示通过服务器 `i` **可连接** 的服务器对的 **数目** 。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2024/01/21/example22.png)

```
输入：edges = [[0,1,1],[1,2,5],[2,3,13],[3,4,9],[4,5,2]], signalSpeed = 1
输出：[0,4,6,6,4,0]
解释：由于 signalSpeed 等于 1 ，count[c] 等于所有从 c 开始且没有公共边的路径对数目。
在输入图中，count[c] 等于服务器 c 左边服务器数目乘以右边服务器数目。
```

**示例 2：**

![img](https://assets.leetcode.com/uploads/2024/01/21/example11.png)

```
输入：edges = [[0,6,3],[6,5,3],[0,3,1],[3,2,7],[3,1,6],[3,4,2]], signalSpeed = 3
输出：[2,0,0,0,0,0,2]
解释：通过服务器 0 ，有 2 个可连接服务器对(4, 5) 和 (4, 6) 。
通过服务器 6 ，有 2 个可连接服务器对 (4, 5) 和 (0, 5) 。
所有服务器对都必须通过服务器 0 或 6 才可连接，所以其他服务器对应的可连接服务器对数目都为 0 。
```

**提示：**

- `2 <= n <= 1000`
- `edges.length == n - 1`
- `edges[i].length == 3`
- `0 <= ai, bi < n`
- `edges[i] = [ai, bi, weighti]`
- `1 <= weighti <= 106`
- `1 <= signalSpeed <= 106`
- 输入保证 `edges` 构成一棵合法的树。

**思路：**

```
我们可以将每个服务器等价于树中的节点，根据题意可知，如果两个节点 a, b 和 节点 c 满足以下条件，那么节点 a 和 b 是通过 c 可连接。
	a < b ，a ≠ c 且 b ≠ c
	从 c 到 a 的距离是可以被 signalSpeed 整除的；
	从 c 到 b 的距离是可以被 signalSpeed 整除的；
	从 c 到 b 的路径与从 c 到 a 的路径没有任何公共边；

由于 a 与 b 可以互换，a 与 b 只需满足 a ≠ b 即可。题目要求求出通过每个节点 x 的可连接对数，此时我们可以枚举以 x 为根结点的树，以 x 为根节点的子树一定满足：
	如果 a 与 b 属于 x 的不同子树，从 x 到 b 的路径与从 x 到 a 的路径一定没有公共边；如果 a 与 b 属于 x 的同一个子树，从 x 到 b 的路径与从 i 到 a 的路径一定存在公共边；
	如果 a 与 b 属于 x 的不同子树，且满足 x 到 b 的距离与 x 到 a 的距离都能被 signalSpeed 整除，则 a 与 b 可以构成连接对；

根据以上分析可知，通过 x 节点的连接对数即等于以 x 为根节点的任意两个子树中满足距离被 signalSpeed 整除的节点数目的乘积之和。我们枚举以 x 为根节点的树，此时分别计算不同子树中存在满足到根节点 x 的距离被 signalSpeed 整除的节点数目，假设此时含有 k 个子树，每个子树中含有满足距离被整数的节点数目为 c0, c1, c2, ⋯ , ck−1。

实际计算过程中，枚举每个节点 x 为根节点：
	用 pre 表示当前已经遍历过的到结点 i 的距离可以被 signalSpeed 整除的结点数目，初始时 pre=0；
	枚举节点 x 的每一个相邻的节点 v，计算以 v 为根节点的子树中含有到 i 的距离可以被整除的节点数目 cnt，此时可以通过深度优先搜索来实现，每次递归时会传递节点 i 到当前节点的距离对 signalSpeed 取模后的结果 curr，如果 curr = 0 则计数加 1；
	每次计算完时，根据上述公式可以知道节点对数会增加 pre timescnt，当前可被整除的节点数目增加 cnt；
	遍历完成后，保存结果返回即可；
```

**代码：**

```cpp
class Solution {
public:
    vector<int> countPairsOfConnectableServers(vector<vector<int>>& edges, int signalSpeed) {
        int n = edges.size() + 1;
        vector<vector<pair<int, int>>> graph(n);
        //领接表
        for (auto e : edges) {
            graph[e[0]].emplace_back(e[1], e[2]);
            graph[e[1]].emplace_back(e[0], e[2]);
        }
        function<int(int, int, int)> dfs = [&](int p, int root, int curr) -> int {
            int res = 0;
            if (curr == 0) res++;
            for (auto &[v, cost] : graph[p]) {
                if (v != root) {
                    res += dfs(v, p, (curr + cost) % signalSpeed);
                }
            }
            return res;
        };

        vector<int> count(n);
        for (int i = 0; i < n; i++) {
            int pre = 0;
            vector<int> temp;
            for (auto &[v, cost] : graph[i]) {
                temp.push_back(dfs(v, i, cost % signalSpeed));
            }
            int res = 0;
            for (int j = 0; j < temp.size(); j++) {
                for (int k = j + 1; k < temp.size(); k++) {
                    res += temp[j] * temp[k];
                }
            }
            count[i] = res;
        }
        return count;
    }
};
```

