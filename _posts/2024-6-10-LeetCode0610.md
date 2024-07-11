---
layout: post
title: "救生艇"
date: 2024-6-10
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给定数组 `people` 。`people[i]`表示第 `i` 个人的体重 ，**船的数量不限**，每艘船可以承载的最大重量为 `limit`。每艘船最多可同时载两人，但条件是这些人的重量之和最多为 `limit`。返回 *承载所有人所需的最小船数* 。

**示例 1：**

```
输入：people = [1,2], limit = 3
输出：1
解释：1 艘船载 (1, 2)
```

**示例 2：**

```
输入：people = [3,2,2,1], limit = 3
输出：3
解释：3 艘船分别载 (1, 2), (2) 和 (3)
```

**提示：**

- `1 <= people.length <= 5 * 104`
- `1 <= people[i] <= limit <= 3 * 104`

**思路：**

```
要想要船的数量最少，那么我们应该让每艘船尽可能都载两个人。
那么限制是什么呢？	people[i]的重量有点大，导致船的剩余承重没法载两个人
所以思路就要转换成让每一艘船的承重都尽量逼近limit。

那么一艘船上的一个人一定是重量大的，另一个重量小的
那么排序 + 贪心就是思路

先将people数组排序，然后利用双指针指向首i尾j，如果limit - people[j] >= people[i]那么这两个人就可以一艘船。否则就people[j]单独一艘船（没有其他办法了，people[i]已经是最小的了）
```

**代码：**

```cpp
class Solution {
public:
    int numRescueBoats(vector<int>& people, int limit) {
        sort(people.begin(), people.end());
        int n = people.size();
        int i = 0, j = n - 1;
        int ans = 0;
        while (i < j) {
            int remain = limit - people[j];
            if (remain >= people[i]) {
                i++;
            }
            j--;ans++;
        }
        if (i == j) ans++;
        return ans;
    }
};
```
