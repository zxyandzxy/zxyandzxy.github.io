---
layout: post
title: "子序列最大优雅度"
date: 2024-6-13
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个长度为 `n` 的二维整数数组 `items` 和一个整数 `k` 。`items[i] = [profiti, categoryi]`，其中 `profiti` 和 `categoryi` 分别表示第 `i` 个项目的利润和类别。

现定义 `items` 的 **子序列** 的 **优雅度** 可以用 `total_profit + distinct_categories2` 计算，其中 `total_profit` 是子序列中所有项目的利润总和，`distinct_categories` 是所选子序列所含的所有类别中不同类别的数量。

你的任务是从 `items` 所有长度为 `k` 的子序列中，找出 **最大优雅度** 。用整数形式表示并返回 `items` 中所有长度恰好为 `k` 的子序列的最大优雅度。

**示例 1：**

```
输入：items = [[3,2],[5,1],[10,1]], k = 2
输出：17
解释：
在这个例子中，我们需要选出长度为 2 的子序列。
其中一种方案是 items[0] = [3,2] 和 items[2] = [10,1] 。
子序列的总利润为 3 + 10 = 13 ，子序列包含 2 种不同类别 [2,1] 。
因此，优雅度为 13 + 22 = 17 ，可以证明 17 是可以获得的最大优雅度。 
```

**示例 2：**

```
输入：items = [[3,1],[3,1],[2,2],[5,3]], k = 3
输出：19
解释：
在这个例子中，我们需要选出长度为 3 的子序列。 
其中一种方案是 items[0] = [3,1] ，items[2] = [2,2] 和 items[3] = [5,3] 。
子序列的总利润为 3 + 2 + 5 = 10 ，子序列包含 3 种不同类别 [1, 2, 3] 。 
因此，优雅度为 10 + 32 = 19 ，可以证明 19 是可以获得的最大优雅度。
```

**提示：**

- `1 <= items.length == n <= 105`
- `items[i].length == 2`
- `items[i][0] == profiti`
- `items[i][1] == categoryi`
- `1 <= profiti <= 109`
- `1 <= categoryi <= n `
- `1 <= k <= n`

> 暴力求解法

**思路：**

```
我们直接求出所有的子序列，然后对于每个子序列求它的优雅度，记录最大优雅度即可

通过回溯求所有子序列，通过哈希表求取子序列中不同的类别数

由于涉及到回溯算法，所以时间复杂度肯定是比较高的
```

**代码：**

```cpp
class Solution {
public:
    vector<vector<vector<int>>> result;
    vector<vector<int>> path;
    long long ans = 0;
    void backTracking(vector<vector<int>>& items, int start, int k) {
        if (path.size() == k) {
            result.push_back(path);
            ans = max(ans, getElegance(path));
            return;
        }
        for (int i = start; i < items.size(); i++) {
            path.push_back(items[i]);
            backTracking(items, i + 1, k);
            path.pop_back();
        }
    }
    long long getElegance(vector<vector<int>>& path) {
        unordered_set<int> hash;
        long long total_profit = 0;
        for (int i = 0; i < path.size(); i++) {
            total_profit += path[i][0];
            hash.insert(path[i][1]);
        }
        return total_profit + hash.size() * hash.size();
    }
    long long findMaximumElegance(vector<vector<int>>& items, int k) {
        //两步：先求所有子序列，对每个子序列求优雅度
        backTracking(items, 0, k);
        return ans;
    }
};
```

> 反悔贪心

**思路：**

```
按照利润从大到小排序。先把前 k 个项目选上。

考虑第 k+1 个项目，如果要选它，我们必须从前 k 个项目中移除一个项目。

由于已经按照利润从大到小排序，选这个项目不会让 totalProfit 变大，所以重点考虑能否让 distinctCategories 变大。

分类讨论：
	如果第 k+1 个项目和前面某个已选项目的类别相同，那么无论怎么移除都不会让 distinctCategories 变大，所以无需选择这个项目。
	如果第 k+1 个项目和前面任何已选项目的类别都不一样，考虑移除前面已选项目中的哪一个：
		如果移除的项目的类别只出现一次，那么选第 k+1 个项目后，distinctCategories 一减一增，保持不变，所以不考虑这种情况。
		如果移除的项目的类别重复出现多次，那么选第 k+1k+1k+1 个项目后，distinctCategories 会增加一，此时有可能会让优雅度变大，一定要选择这个项目。为什么说「一定」呢？因为 totalProfit 只会变小，我们现在的目标就是让 totalProfit 保持尽量大，同时让 distinctCategories 增加，那么能让 distinctCategories 增加就立刻选上！因为后面的利润更小，现在不选的话将来 totalProfit 只会更小。

按照这个过程，继续考虑选择后面的项目。计算优雅度，取最大值，即为答案。
代码实现时，我们应当移除已选项目中类别和前面重复且利润最小的项目，这可以用一个栈 duplicate 来维护，由于利润从大到小排序，所以栈顶就是最小的利润。入栈前判断 category 之前是否遇到过，没遇到才入栈。
```

**代码：**

```cpp
class Solution {
public:
    long long findMaximumElegance(vector<vector<int>>& items, int k) {
        // 把利润从大到小排序
        ranges::sort(items, [](const auto &a, const auto &b) { return a[0] > b[0]; });
        long long ans = 0, total_profit = 0;
        unordered_set<int> vis;
        stack<int> duplicate; // 重复类别的利润
        for (int i = 0; i < items.size(); i++) {
            int profit = items[i][0], category = items[i][1];
            if (i < k) {
                total_profit += profit; // 累加前 k 个项目的利润
                if (!vis.insert(category).second) { // 重复类别
                    duplicate.push(profit);
                }
            } else if (!duplicate.empty() && vis.insert(category).second) { // 之前没有的类别
                total_profit += profit - duplicate.top(); // 选一个重复类别中的最小利润替换
                duplicate.pop();
            } // else：比前面的利润小，而且类别还重复了，选它只会让 total_profit 变小，vis.size() 不变，优雅度不会变大
            ans = max(ans, total_profit + (long long) vis.size() * (long long) vis.size());
        }
        return ans;
    }
};
```

