---
layout: post
title: "质数的最大距离"
date: 2024-7-2
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个整数数组 `nums`。返回两个（不一定不同的）质数在 `nums` 中 **下标** 的 **最大距离**。

**示例 1：**

> **输入：** nums = [4,2,9,5,3]
>
> **输出：** 3
>
> **解释：** `nums[1]`、`nums[3]` 和 `nums[4]` 是质数。因此答案是 `|4 - 1| = 3`。

**示例 2：**

> **输入：** nums = [4,8,2,8]
>
> **输出：** 0
>
> **解释：** `nums[2]` 是质数。因为只有一个质数，所以答案是 `|2 - 2| = 0`。

**提示：**

- `1 <= nums.length <= 3 * 105`
- `1 <= nums[i] <= 100`
- 输入保证 `nums` 中至少有一个质数。

**思路：**

```
其实核心点在于判断是不是质数，看nums[i] ∈ [1, 100]：
	要么使用打表法直接写出 1 - 100 中所有的质数
	要么使用质筛法求出 1 - 100 中所有的质数

主函数中，判断每个nums中的每个数是不是质数即可，记录最左边和最右边的质数索引，相减即是答案。
```

**代码：**

```cpp
class Solution {
public:
    unordered_set<int> hash;
    // 质筛法求质数
    void getPrimeNumber() {
        vector<bool> pass(101, true);
        for (int i = 2; i < 101; i++) {
            if (pass[i]) {
                hash.insert(i);
                int j = 2;
                while (i * j < 101) {
                    pass[i * j] = false;
                    j++;
                }
            }
        }
    }
    int maximumPrimeDifference(vector<int>& nums) {
        int l = -1, r = -1;
        getPrimeNumber();
        for (int i = 0; i < nums.size(); i++) {
            // 记录最左边和最右边的质数坐标
            if (hash.find(nums[i]) != hash.end()) {
                if (l == -1) {
                    l = i;
                }else {
                    r = i;
                }
            }
        }
        if (r == -1) {
            return 0;
        }
        return r - l;
    }
};
```

