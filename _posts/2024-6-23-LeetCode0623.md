---
layout: post
title: "检测大写字母"
date: 2024-6-23
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

我们定义，在以下情况时，单词的大写用法是正确的：

- 全部字母都是大写，比如 `"USA"` 。
- 单词中所有字母都不是大写，比如 `"leetcode"` 。
- 如果单词不只含有一个字母，只有首字母大写， 比如 `"Google"` 。

给你一个字符串 `word` 。如果大写用法正确，返回 `true` ；否则，返回 `false` 。

**示例 1：**

```
输入：word = "USA"
输出：true
```

**示例 2：**

```
输入：word = "FlaG"
输出：false
```

**提示：**

- `1 <= word.length <= 100`
- `word` 由小写和大写英文字母组成

**思路：**

```
根据三种情况一一判别就可以了
```

**代码：**

```cpp
class Solution {
public:
    bool detectCapitalUse(string word) {
        if (word.size() == 1) return true;
        if (word[0] >= 'a' && word[0] <= 'z') {
            //全都要小写
            for (int i = 1; i < word.size(); i++) {
                if (word[i] >= 'A' && word[i] <= 'Z') {
                    return false;
                }
            }
            return true;
        }
        if (word[1] >= 'A' && word[1] <= 'Z') {
            //全大写
            for (int i = 2; i < word.size(); i++) {
                if (word[i] >= 'a' && word[i] <= 'z') {
                    return false;
                }
            }
            return true;
        }
        for (int i = 2; i < word.size(); i++) {
            if (word[i] >= 'A' && word[i] <= 'Z') {
                return false;
            }
        }
        return true;
    }
};
```



