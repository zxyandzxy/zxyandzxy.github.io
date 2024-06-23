---
layout: post
title: "最长特殊序列II"
date: 2024-6-17
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给定字符串列表 `strs` ，返回其中 **最长的特殊序列** 的长度。如果最长特殊序列不存在，返回 `-1` 。**特殊序列** 定义如下：该序列为某字符串 **独有的子序列（即不能是其他字符串的子序列）**。

**示例 1：**

```
输入: strs = ["aba","cdc","eae"]
输出: 3
```

**思路：**

```
这道题翻译成人话就是：判断某个字符串strs[i]是不是其他字符串数组元素的子序列，如果不是那它就可以成为最长特殊序列的候选人。

我们选候选人中最长的就可
```

**代码：**

```cpp
class Solution {
    // 判断 s 是否为 t 的子序列
    bool isSubseq(string& s, string& t) {
        int i = 0;
        for (char c : t) {
            if (s[i] == c && ++i == s.length()) { // 所有字符匹配完毕
                return true; // s 是 t 的子序列
            }
        }
        return false;
    }

public:
    int findLUSlength(vector<string>& strs) {
        int ans = -1;
        for (int i = 0; i < strs.size(); i++) {
            if ((int) strs[i].length() <= ans) { // 不会让 ans 变大
                continue;
            }
            for (int j = 0; j < strs.size(); j++) {
                if (j != i && isSubseq(strs[i], strs[j])) {
                    goto next;
                }
            }
            ans = strs[i].length();
            next:;
        }
        return ans;
    }
};
```



