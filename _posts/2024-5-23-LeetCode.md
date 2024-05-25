---
layout: post
title: "找出最长等值子数组"
date: 2024-5-23
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个下标从 **0** 开始的整数数组 `nums` 和一个整数 `k` 。

如果子数组中所有元素都相等，则认为子数组是一个 **等值子数组** 。注意，空数组是 **等值子数组** 。

从 `nums` 中删除最多 `k` 个元素后，返回可能的最长等值子数组的长度。

**子数组** 是数组中一个连续且可能为空的元素序列。

**示例 1：**

```
输入：nums = [1,3,2,3,1,3], k = 3
输出：3
解释：最优的方案是删除下标 2 和下标 4 的元素。
删除后，nums 等于 [1, 3, 3, 3] 。
最长等值子数组从 i = 1 开始到 j = 3 结束，长度等于 3 。
可以证明无法创建更长的等值子数组。
```

**示例 2：**

```
输入：nums = [1,1,2,2,1,1], k = 2
输出：4
解释：最优的方案是删除下标 2 和下标 3 的元素。 
删除后，nums 等于 [1, 1, 1, 1] 。 
数组自身就是等值子数组，长度等于 4 。 
可以证明无法创建更长的等值子数组。
```

**提示：**

- `1 <= nums.length <= 105`
- `1 <= nums[i] <= nums.length`
- `0 <= k <= nums.length`

**思路：**

```
没有思路就从答案开始想，最后一定是某个元素组成的最长等值子数组，那么我们就需要确定这个元素，即需要为每一个不同的元素记录它们出现的位置（因为子数组要连续，所以要看位置来确定是否可以达成连续）

把相同元素分组，相同元素的下标记录到哈希表（或者数组）posLists 中。例如示例 1，元素 3 在 nums 中的下标有 1,3,5 那么 posLists[3]=[1,3,5]。遍历 posLists 中的每个下标列表 pos，例如遍历 pos=[1,3,5]。请记住，pos 中保存的是下标，这些下标在 nums 中的对应元素都相同。然后用滑动窗口计算。设窗口左右端点为 left 和 right。

假设 nums 的等值子数组的元素下标从 pos[left] 到 pos[right]，那么在删除前，子数组的长度为 pos[right]−pos[left]+1 这个子数组有 right−left+1 个数都是相同的，无需删除，其余元素都需要删除，那么需要删除的元素个数就是 pos[right]−pos[left]−(right−left) 如果上式大于 k，说明要删除的数太多了，那么移动左指针 left，直到上式小于等于 k，此时用 right−left+1 更新答案的最大值。

代码实现时，为简化上式，pos 实际保存的是 pos[i]−i，也就是把上面的每个 pos[i] 都减去其在 pos 中的下标 i，于是需要删除的元素个数简化为 pos[right]−pos[left]
```

**代码：**

```cpp
class Solution {
public:
    int longestEqualSubarray(vector<int> &nums, int k) {
        int n = nums.size();
        vector<vector<int>> pos_lists(n + 1);
        for (int i = 0; i < n; i++) {
            int x = nums[i];
            pos_lists[x].push_back(i - pos_lists[x].size());
        }

        int ans = 0;
        for (auto& pos : pos_lists) {
            int left = 0;
            for (int right = 0; right < pos.size(); right++) {
                while (pos[right] - pos[left] > k) { // 要删除的数太多了
                    left++;
                }
                ans = max(ans, right - left + 1);
            }
        }
        return ans;
    }
};
```

