---
layout: post
title: "移除字符串中的尾随零"
date: 2024-6-29
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个用字符串表示的正整数 `num` ，请你以字符串形式返回不含尾随零的整数 `num` 。

**示例 1：**

```
输入：num = "51230100"
输出："512301"
解释：整数 "51230100" 有 2 个尾随零，移除并返回整数 "512301" 。
```

**示例 2：**

```
输入：num = "123"
输出："123"
解释：整数 "123" 不含尾随零，返回整数 "123" 。
```

**提示：**

- `1 <= num.length <= 1000`
- `num` 仅由数字 `0` 到 `9` 组成
- `num` 不含前导零

**思路：**

```
从num尾部开始遍历，找到第一个非0索引，然后截取字符串即可
```

**代码：**

```cpp
class Solution {
public:
    string removeTrailingZeros(string num) {
        int n = num.size() - 1;
        while (n >= 0 && num[n] == '0') {
            n--;
        }
        return num.substr(0, n + 1);
    }
};
```

