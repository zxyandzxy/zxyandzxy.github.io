---
layout: post
title: "将元素分配到两个数组中II"
date: 2024-6-5
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个下标从 **1** 开始、长度为 `n` 的整数数组 `nums` 。

现定义函数 `greaterCount` ，使得 `greaterCount(arr, val)` 返回数组 `arr` 中 **严格大于** `val` 的元素数量。

你需要使用 `n` 次操作，将 `nums` 的所有元素分配到两个数组 `arr1` 和 `arr2` 中。在第一次操作中，将 `nums[1]` 追加到 `arr1` 。在第二次操作中，将 `nums[2]` 追加到 `arr2` 。之后，在第 `i` 次操作中：

- 如果 `greaterCount(arr1, nums[i]) > greaterCount(arr2, nums[i])` ，将 `nums[i]` 追加到 `arr1` 。
- 如果 `greaterCount(arr1, nums[i]) < greaterCount(arr2, nums[i])` ，将 `nums[i]` 追加到 `arr2` 。
- 如果 `greaterCount(arr1, nums[i]) == greaterCount(arr2, nums[i])` ，将 `nums[i]` 追加到元素数量较少的数组中。
- 如果仍然相等，那么将 `nums[i]` 追加到 `arr1` 。

连接数组 `arr1` 和 `arr2` 形成数组 `result` 。例如，如果 `arr1 == [1,2,3]` 且 `arr2 == [4,5,6]` ，那么 `result = [1,2,3,4,5,6]` 。

返回整数数组 `result` 。

**示例 1：**

```
输入：nums = [2,1,3,3]
输出：[2,3,1,3]
解释：在前两次操作后，arr1 = [2] ，arr2 = [1] 。
在第 3 次操作中，两个数组中大于 3 的元素数量都是零，并且长度相等，因此，将 nums[3] 追加到 arr1 。
在第 4 次操作中，两个数组中大于 3 的元素数量都是零，但 arr2 的长度较小，因此，将 nums[4] 追加到 arr2 。
在 4 次操作后，arr1 = [2,3] ，arr2 = [1,3] 。
因此，连接形成的数组 result 是 [2,3,1,3] 。
```

**提示：**

- `3 <= n <= 105`
- `1 <= nums[i] <= 109`

**思路：**

```
本题核心就是如何优化greaterCount函数，如果采用暴力的方法，对arr中的元素一个一个判断的话，那么总时间复杂度就是O(n^2)
可以采用树状数组。现提供暴力解法
```

**代码：**

```cpp
class Solution {
public:
    int greaterCount(vector<int> arr, int val) {
        int ans = 0;
        for (int num : arr) {
            if (num > val) {
                ans++;
            }
        }
        return ans;
    }
    vector<int> resultArray(vector<int>& nums) {
        vector<int> arr1;
        vector<int> arr2;
        int n = nums.size();
        arr1.push_back(nums[0]);
        arr2.push_back(nums[1]);
        for (int i = 2; i < nums.size(); i++) {
            if (greaterCount(arr1, nums[i]) > greaterCount(arr2, nums[i])) {
                arr1.push_back(nums[i]);
            }else if (greaterCount(arr1, nums[i]) < greaterCount(arr2, nums[i])) {
                arr2.push_back(nums[i]);
            }else {
                if (arr1.size() <= arr2.size()) {
                    arr1.push_back(nums[i]);
                }else {
                    arr2.push_back(nums[i]);
                }
            }
        }
        vector<int> ans(n);
        for (int i = 0; i < n; i++) {
            if (i < arr1.size()) { ans[i] = arr1[i]; }
            else { ans[i] = arr2[i - arr1.size()]; }
        }
        return ans;
    }
};
```

