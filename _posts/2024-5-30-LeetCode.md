---
layout: post
title: "找出出现至少三次的最长特殊子字符串II"
date: 2024-5-30
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

给你一个仅由小写英文字母组成的字符串 `s` 。

如果一个字符串仅由单一字符组成，那么它被称为 **特殊** 字符串。例如，字符串 `"abc"` 不是特殊字符串，而字符串 `"ddd"`、`"zz"` 和 `"f"` 是特殊字符串。

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

- `3 <= s.length <= 5 * 105`
- `s` 仅由小写英文字母组成。

**思路：**

```
由于特殊子串只包含单一字母，我们按照相同字母分组，每组统计相同字母连续出现的长度。例如字符串 aaaabbbabb 分成 aaaa+bbb+a+bb 四组，字母 a 有长度 4 和长度 1，字母 b 有长度 3 和长度 2。所以字母 a 的长度列表为 [4,1]，字母 b 的长度列表为 [3,2]。

遍历每个字母对应的长度列表 a，把 a 从大到小排序。

有哪些取出三个特殊子串的方法呢？
	从最长的特殊子串（a[0]）中取三个长度均为 a[0]−2 的特殊子串。例如示例 1 的 aaaa 可以取三个 aa。
	或者，从最长和次长的特殊子串（a[0],a[1]）中取三个长度一样的特殊子串：
		如果 a[0]=a[1]，那么可以取三个长度均为 a[0]−1 的特殊子串。
		如果 a[0]>a[1]，那么可以取三个长度均为 a[1] 的特殊子串：从最长中取两个，从次长中取一个。
		这两种情况合并成 min⁡(a[0]−1,a[1])。
	又或者，从最长、次长、第三长的的特殊子串（a[0],a[1],a[2]）中各取一个长为 a[2] 的特殊子串。

这三种情况取最大值，即max⁡(a[0]−2,min⁡(a[0]−1,a[1]),a[2])
对每个长度列表计算上式，取最大值即为答案。
如果答案是 0，返回 −1。
代码实现时，在数组末尾加两个 0，就无需特判 a 长度小于 3 的情况了。
```

**代码 ：**

```cpp
class Solution {
public:
    static bool cmp(int a, int b) {
        return a > b;
    }
    int maximumLength(string s) {
        vector<vector<int>> cnt(26);
        for (int i = 0; i < s.size(); i++) {
            int j = i;
            while (j < s.size() && s[j] == s[i]) j++;
            cnt[s[i] - 'a'].push_back(j - i);
            i = j - 1;
        } 
        int ans = -1;
        for (int i = 0; i < 26; i++) {
            if (cnt[i].size() == 0) continue;
            sort(cnt[i].begin(), cnt[i].end(), cmp);
            cnt[i].push_back(0);
            cnt[i].push_back(0);
            int cur = max(cnt[i][0] - 2, max(cnt[i][2], min(cnt[i][0] - 1, cnt[i][1])));
            if (cur) ans = max(ans, cur); 
        }
        return ans;
    }
};
```

