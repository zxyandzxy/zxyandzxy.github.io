---
layout: post
title: "价格减免"
date: 2024-6-18
tags: [leetcode]
comments: true
author: zxy
---

**题目：**

**句子** 是由若干个单词组成的字符串，单词之间用单个空格分隔，其中每个单词可以包含数字、小写字母、和美元符号 `'$'` 。如果单词的形式为美元符号后跟着一个非负实数，那么这个单词就表示一个 **价格** 。

- 例如 `"$100"`、`"$23"` 和 `"$6"` 表示价格，而 `"100"`、`"$"` 和 `"$1e5` 不是。

给你一个字符串 `sentence` 表示一个句子和一个整数 `discount` 。对于每个表示价格的单词，都在价格的基础上减免 `discount%` ，并 **更新** 该单词到句子中。所有更新后的价格应该表示为一个 **恰好保留小数点后两位** 的数字。

返回表示修改后句子的字符串。注意：所有价格 **最多** 为 `10` 位数字。

**示例 1：**

```
输入：sentence = "there are $1 $2 and 5$ candies in the shop", discount = 50
输出："there are $0.50 $1.00 and 5$ candies in the shop"
解释：
表示价格的单词是 "$1" 和 "$2" 。 
- "$1" 减免 50% 为 "$0.50" ，所以 "$1" 替换为 "$0.50" 。
- "$2" 减免 50% 为 "$1" ，所以 "$1" 替换为 "$1.00" 。
```

**示例 2：**

```
输入：sentence = "1 2 $3 4 $5 $6 7 8$ $9 $10$", discount = 100
输出："1 2 $0.00 4 $0.00 $0.00 7 8$ $0.00 $10$"
解释：
任何价格减免 100% 都会得到 0 。
表示价格的单词分别是 "$3"、"$5"、"$6" 和 "$9"。
每个单词都替换为 "$0.00"。
```

**提示：**

- `1 <= sentence.length <= 105`
- `sentence` 由小写英文字母、数字、`' '` 和 `'$'` 组成
- `sentence` 不含前导和尾随空格
- `sentence` 的所有单词都用单个空格分隔
- 所有价格都是 **正** 整数且不含前导零
- 所有价格 **最多** 为 `10` 位数字
- `0 <= discount <= 100`

**思路：**

```
我们按照题目中的要求进行模拟即可。

首先我们将给定的字符串 sentence 根据空格进行分割，得到其中的每一个单词。随后我们遍历每个单词，如果该单词：
	以 $ 开头；
	后续至少有一个字符，且均在 [0,9] 中；
那么该单词就表示一个价格。我们提取后续的字符，转换成整数，计算折扣（即乘上 1 − discount/100 ），保留两位小数，再转换回字符串，并添加开头的 $ 即可。

当所有单词遍历完成之后，我们就可以再加上空格，得到最终的字符串。
```

**代码：**

```cpp
class Solution {
public:
    string discountPrices(string sentence, int discount) {
        stringstream sin(sentence), sout;
        sout << fixed << setprecision(2);  // 保留两位小数
        string word;  // 保留按空格分割的单词
        while (sin >> word) {
            if (word[0] == '$' && word.size() > 1 && all_of(word.begin() + 1, word.end(), ::isdigit)) {  //是价格
                double price = stoll(word.substr(1, word.size() - 1)) * (1.0 - discount / 100.0);
                sout << '$' << price;
            }
            else {
                sout << word;
            }
            sout << " ";
        }
        string ans = sout.str();
        ans.pop_back();  // 删除末尾空格
        return ans;
    }
};
```



