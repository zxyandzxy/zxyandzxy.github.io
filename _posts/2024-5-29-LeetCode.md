---
layout: post
title: "找出出现至少三次的最长特殊子字符串I"
date: 2024-5-29
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个仅由小写英文字母组成的字符串 `s` 。如果一个字符串仅由单一字符组成，那么它被称为 **特殊** 字符串。例如，字符串 `"abc"` 不是特殊字符串，而字符串 `"ddd"`、`"zz"` 和 `"f"` 是特殊字符串。

返回在 `s` 中出现 **至少三次** 的 **最长特殊子字符串** 的长度，如果不存在出现至少三次的特殊子字符串，则返回 `-1` 。**子字符串** 是字符串中的一个连续 **非空** 字符序列。

**示例 1：**

```
输入：s = "aaaa"
输出：2
解释：出现三次的最长特殊子字符串是 "aa" ：子字符串 "aaaa"、"aaaa" 和 "aaaa"。
可以证明最大长度是 2 。
```

**示例 2：**

```
输入：s = "abcdef"
输出：-1
解释：不存在出现至少三次的特殊子字符串。因此返回 -1 。
```

**提示：**

- `3 <= s.length <= 50`
- `s` 仅由小写英文字母组成。

**思路：**

```
我们直接暴力统计，每一个特殊子字符串的长度，全部记录在哈希表中。
最后输出最长的特殊子字符串即可。

暴力统计是有说法的，我们就从头到尾遍历源字符串s，对于每一个元素s[i]，统计以s[i]开头的特殊字符串，只要是就进入哈希表。
这样的做法，我们天生就不会有重复的特殊子字符串遭到统计
```

**代码：**

```c++
class Solution {
public:
    int maximumLength(string s) {
        unordered_map<string, int> u_m;
        int ans = -1;
        int len = s.size();
        for (int i = 0; i < len; i++) {
            for (int j = i; j < len; j++) {
                if (s[j] != s[i]) break;
                string sub = s.substr(i, j - i + 1);
                u_m[sub]++;
                if (u_m[sub] >= 3 && (int)sub.size() > ans) {
                    ans = (int)sub.size();
                }
            }
        }
        return ans;
    }
};
```

