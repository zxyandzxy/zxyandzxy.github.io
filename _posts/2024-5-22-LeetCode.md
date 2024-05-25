---
layout: post
title: "找出输掉零场或一场比赛的玩家"
date: 2024-5-22
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个整数数组 `matches` 其中 `matches[i] = [winneri, loseri]` 表示在一场比赛中 `winneri` 击败了 `loseri` 。

返回一个长度为 2 的列表 `answer` ：

- `answer[0]` 是所有 **没有** 输掉任何比赛的玩家列表。
- `answer[1]` 是所有恰好输掉 **一场** 比赛的玩家列表。

两个列表中的值都应该按 **递增** 顺序返回。

**注意：**

- 只考虑那些参与 **至少一场** 比赛的玩家。
- 生成的测试用例保证 **不存在** 两场比赛结果 **相同** 。

**示例 1：**

```
输入：matches = [[1,3],[2,3],[3,6],[5,6],[5,7],[4,5],[4,8],[4,9],[10,4],[10,9]]
输出：[[1,2,10],[4,5,7,8]]
解释：
玩家 1、2 和 10 都没有输掉任何比赛。
玩家 4、5、7 和 8 每个都输掉一场比赛。
玩家 3、6 和 9 每个都输掉两场比赛。
因此，answer[0] = [1,2,10] 和 answer[1] = [4,5,7,8] 。
```

**提示：**

- `1 <= matches.length <= 105`
- `matches[i].length == 2`
- `1 <= winneri, loseri <= 105`
- `winneri != loseri`
- 所有 `matches[i]` **互不相同**

**思路：**

```
直接用哈希表统计每个玩家输掉的场次数即可
需要注意，对于比赛中获胜的玩家，需要将哈希表对应值设为0，以便统计一场没输的玩家
```

**代码：**

```cpp
class Solution {
public:
    vector<vector<int>> findWinners(vector<vector<int>>& matches) {
        unordered_map<int, int> freq;
        for (const auto& match: matches) {
            int winner = match[0], loser = match[1];
            if (!freq.count(winner)) {
                freq[winner] = 0;
            }
            ++freq[loser];
        }

        vector<vector<int>> ans(2);
        for (const auto& [key, value]: freq) {
            if (value < 2) {
                ans[value].push_back(key);
            }
        }

        sort(ans[0].begin(), ans[0].end());
        sort(ans[1].begin(), ans[1].end());
        return ans;
    }
};
```

