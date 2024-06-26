---
layout: post
title: "位运算技巧"
date: 2024-5-23
tags: [algorithm]
comments: true
author: zxy
---

## 前言

集合论：

编程中，使用哈希表表示集合，如C++中的`unordered_set`

集合论中，有交集 ∩、并集 ∪、包含于 ⊆ 等概念，编程实现求「求两个哈希表的交集」，需要一个一个地遍历哈希表中的元素。那么，有没有效率更高的做法呢？

> 有，就用二进制

集合可以用二进制表示，二进制**从低到高**第 𝑖 位为 1 表示 𝑖 在集合中，为 0 表示 𝑖 不在集合中。例如集合 {0,2,3} 可以用二进制数 1101(2) 表示；

正式地说，包含非负整数的集合 𝑆 可以用如下方式「压缩」成一个数字：

![image-20240523170045461](https://zxyandzxy.github.io/images/image-20240523170045461.png)

例如集合 {0,2,3} 可以压缩成 2^0+2^2+2^3=13 ，也就是二进制数 1101(2)。

利用位运算「并行计算」的特点，我们可以高效地做一些和集合有关的运算。按照常见的应用场景，可以分为以下四类：

1. 集合与集合
2. 集合与元素
3. 遍历集合
4. 枚举集合

## 一、集合与集合

其中 & 表示按位与，∣ 表示按位或，⊕ 表示按位异或，∼ 表示按位取反。

两个集合的「对称差」是只属于其中一个集合，而不属于另一个集合的元素组成的集合，也就是不在交集中的元素组成的集合。

![image-20240523170029554](https://zxyandzxy.github.io/images/image-20240523170029554.png)

> 注 1：按位取反的例子中，仅列出最低 4 个比特位取反后的结果，即 0110 取反后是 1001。
>
> 注 2：包含于的两种位运算写法是等价的，在编程时只需判断其中任意一种。
>
> 注 3：编程时，请注意运算符的优先级。例如 `==` 在某些语言中优先级更高。

## 二、集合与元素

通常会用到移位运算。

其中 << 表示左移，>> 表示右移。

> 注：左移 𝑖位相当于乘以 2^𝑖，右移 𝑖 位相当于除以 2^𝑖。

![image-20240523170300215](https://zxyandzxy.github.io/images/image-20240523170300215.png)

```
      s = 101100
    s-1 = 101011 // 最低位的 1 变成 0，同时 1 右边的 0 都取反，变成 1
s&(s-1) = 101000
```

特别地，如果 𝑠 是 2 的幂，那么 𝑠&(𝑠−1) = 0。

此外，编程语言提供了一些和二进制有关的库函数，例如：

- 计算二进制中的 1 的个数，也就是集合大小；

- 计算二进制长度，减一后得到集合最大元素；
- 计算二进制尾零个数，也就是集合最小元素。

调用这些函数的时间复杂度都是 𝑂(1)。

![image-20240523170745384](https://zxyandzxy.github.io/images/image-20240523170745384.png)

请特别注意 𝑠 = 0 的情况。对于 C++ 来说，`__builtin_clz(0)` 和 `__builtin_ctz(0)` 是未定义行为。其它语言请查阅 API 文档。

此外，对于 C++ 的 `long long`，需使用相应的 `__builtin_popcountll` 等函数，即函数名后缀添加 `ll`（两个小写字母 L）。

特别地，只包含最小元素的子集，即二进制最低 1 及其后面的 0，也叫 lowbit，可以用 `s & -s` 算出。举例说明：

```
     s = 101100
    ~s = 010011
(~s)+1 = 010100 // 根据补码的定义，这就是 -s  =>  s 的最低 1 左侧取反，右侧不变
s & -s = 000100 // lowbit
```

## 三、遍历集合

设元素范围从 0 到 𝑛−1，挨个判断每个元素是否在集合 𝑠 中：

```cpp
for (int i = 0; i < n; i++) {
    if ((s >> i) & 1) { // i 在 s 中
        // 处理 i 的逻辑
    }
}
```

## 四、枚举集合

### 枚举所有集合

设元素范围从 0 到 𝑛−1，从空集 ∅ 枚举到全集 𝑈：

```cpp
for (int s = 0; s < (1 << n); s++) {
    // 处理 s 的逻辑
}
```

### 枚举非空子集

设集合为 𝑠，**从大到小**枚举 𝑠 的所有**非空**子集 sub：

```cpp
for (int sub = s; sub; sub = (sub - 1) & s) {
    // 处理 sub 的逻辑
}
```

> 为什么要写成 `sub = (sub - 1) & s` 呢？

![image-20240523171700088](https://zxyandzxy.github.io/images/image-20240523171700088.png)

### 枚举子集（包含空集）

如果要从大到小枚举 𝑠 的所有子集 sub（从 𝑠 枚举到空集 ∅），可以这样写：

```cpp
int sub = s;
do {
    // 处理 sub 的逻辑
    sub = (sub - 1) & s;
} while (sub != s);
```

原理是当 sub=0 时（空集），再减一就得到 −1，对应的二进制为 111⋯1，再 &𝑠 就得到了 𝑠。所以当循环到 sub=𝑠 时，说明最后一次循环的 sub=0（空集），𝑠 的所有子集都枚举到了，退出循环。

## 基础题

### 1486.数组异或操作

**题目：**

给你两个整数，`n` 和 `start` 。

数组 `nums` 定义为：`nums[i] = start + 2*i`（下标从 0 开始）且 `n == nums.length` 。

请返回 `nums` 中所有元素按位异或（**XOR**）后得到的结果。

**示例 1：**

```
输入：n = 5, start = 0
输出：8
解释：数组 nums 为 [0, 2, 4, 6, 8]，其中 (0 ^ 2 ^ 4 ^ 6 ^ 8) = 8 。
     "^" 为按位异或 XOR 运算符。
```

**示例 2：**

```
输入：n = 4, start = 3
输出：8
解释：数组 nums 为 [3, 5, 7, 9]，其中 (3 ^ 5 ^ 7 ^ 9) = 8.
```

**思路：**

```
直接构造出nums数组，然后执行异或操作即可
```

**代码：**

```cpp
class Solution {
public:
    int xorOperation(int n, int start) {
        vector<int> nums(n);
        int ans;
        for (int i = 0; i < n; i++) {
            nums[i] = start + 2 * i;
            if (i == 0) {
                ans = nums[i];
            }else {
                ans = ans ^ nums[i];
            }
        }
        return ans;
    }
};
```

### 2595.奇偶位数

**题目：**

给你一个 **正** 整数 `n` 。

用 `even` 表示在 `n` 的二进制形式（下标从 **0** 开始）中值为 `1` 的偶数下标的个数。

用 `odd` 表示在 `n` 的二进制形式（下标从 **0** 开始）中值为 `1` 的奇数下标的个数。

返回整数数组 `answer` ，其中 `answer = [even, odd]` 。

**示例 1：**

```
输入：n = 17
输出：[2,0]
解释：17 的二进制形式是 10001 。 
下标 0 和 下标 4 对应的值为 1 。 
共有 2 个偶数下标，0 个奇数下标。
```

**示例 2：**

```
输入：n = 2
输出：[0,1]
解释：2 的二进制形式是 10 。 
下标 1 对应的值为 1 。 
共有 0 个偶数下标，1 个奇数下标。
```

**提示：**

- `1 <= n <= 1000`

**思路：**

```
要判断元素s的二进制最后一位是不是1 , s & 1 的结果是1就是，否则不是 
```

**代码：**

```cpp
class Solution {
public:
    vector<int> evenOddBit(int n) {
        int even = 0, odd = 0;
        bool flag = true;
        while (n) {
            if (flag && (n & 1)) {
                even++;
            }else if (!flag && (n & 1)) {
                odd++;
            }
            //奇偶更替
            flag = !flag;
            //删掉最低位
            n = n >> 1;
        }
        return {even, odd};
    }
};
```

### 231.2的幂

**题目：**

给你一个整数 `n`，请你判断该整数是否是 2 的幂次方。如果是，返回 `true` ；否则，返回 `false` 。

如果存在一个整数 `x` 使得 `n == 2x` ，则认为 `n` 是 2 的幂次方。

**示例 1：**

```
输入：n = 1
输出：true
解释：20 = 1
```

**示例 2：**

```
输入：n = 16
输出：true
解释：24 = 16
```

**示例 3：**

```
输入：n = 3
输出：false
```

**提示：**

- `-231 <= n <= 231 - 1`

**思路：**

```
首先，2的幂的二进制形式一定是1后面跟着t个0 （t >= 0）
那么我们使用二进制去判断一个数的二进制形式是否如此即可

如果 (n ^ 1) % 2 == 0 说明n的最低位是1,当 n > 1时，就说明不是2的幂 
```

**代码：**

```cpp
class Solution {
public:
    bool isPowerOfTwo(int n) {
        if (n <= 0) {
            return false;
        }
        while (n > 1) {
            if ((n ^ 1) % 2 == 0) {
                return false;
            }
            n = n >> 1;
        }
        return true;
    }
};
```

### 342.4的幂

**题目：**

给定一个整数，写一个函数来判断它是否是 4 的幂次方。如果是，返回 `true` ；否则，返回 `false` 。

整数 `n` 是 4 的幂次方需满足：存在整数 `x` 使得 `n == 4x`

**示例 1：**

```
输入：n = 16
输出：true
```

**示例 2：**

```
输入：n = 5
输出：false
```

**提示：**

- `-231 <= n <= 231 - 1`

**思路：**

```
如果 n 是 4 的幂，那么 n 的二进制表示中有且仅有一个 1，并且这个 1 出现在从低位开始的第奇数个二进制位上（这是因为这个 1 后面必须有偶数个 0）。这里我们规定最低位为第 0 位，例如 n=16 时，n 的二进制表示为 (10000)2

唯一的 1 出现在第 4 个二进制位上，因此 n 是 4 的幂。
由于题目保证了 n 是一个 32 位的有符号整数，因此我们可以构造一个整数 mask，它的所有偶数二进制位都是 0，所有奇数二进制位都是 1。这样一来，我们将 n 和 mask 进行按位与运算，如果结果为 0，说明 n 二进制表示中的 1 出现在偶数的位置，否则说明其出现在奇数的位置。根据上面的思路，mask 的二进制表示为：
mask=(10101010101010101010101010101010)2
我们也可以将其表示成 161616 进制的形式，使其更加美观：
mask=(AAAAAAAA)16
```

**代码：**

```cpp
class Solution {
public:
    bool isPowerOfFour(int n) {
        return n > 0 && (n & (n - 1)) == 0 && (n & 0xaaaaaaaa) == 0;
    }
};
```

### 476.数字的补数

**题目：**

对整数的二进制表示取反（`0` 变 `1` ，`1` 变 `0`）后，再转换为十进制表示，可以得到这个整数的补数。例如，整数 `5` 的二进制表示是 `"101"` ，取反后得到 `"010"` ，再转回十进制表示得到补数 `2` 。

给你一个整数 `num` ，输出它的补数。

**示例 1：**

```
输入：num = 5
输出：2
解释：5 的二进制表示为 101（没有前导零位），其补数为 010。所以你需要输出 2 。
```

**示例 2：**

```
输入：num = 1
输出：0
解释：1 的二进制表示为 1（没有前导零位），其补数为 0。所以你需要输出 0 。
```

**提示：**

- `1 <= num < 231`

**思路：**

```
直接看代码，我自己也不知道怎么讲思路，反正就是懂
```

**代码：**

```cpp
class Solution {
public:
    int findComplement(int num) {
        long long ans = 0;
        long long base = 1;
        while (num) {
            // 末位是0，取反后是1，就要加上
            if ((num & 1) == 0) {
                ans += base;
            }
            base *= 2;
            num = num >> 1;
        }
        return ans;
    }
};
```

### 461.汉明距离

**题目：**

两个整数之间的 [汉明距离](https://baike.baidu.com/item/汉明距离) 指的是这两个数字对应二进制位不同的位置的数目。

给你两个整数 `x` 和 `y`，计算并返回它们之间的汉明距离。

**示例 1：**

```
输入：x = 1, y = 4
输出：2
解释：
1   (0 0 0 1)
4   (0 1 0 0)
       ↑   ↑
上面的箭头指出了对应二进制位不同的位置。
```

**提示：**

- `0 <= x, y <= 231 - 1`

**思路：**

```
根据异或不同得1的逻辑，将x与y做异或，然后统计结果中有多少个1即可
```

**代码：**

```cpp
class Solution {
public:
    int hammingDistance(int x, int y) {
        int res = x ^ y;
        int ans = 0;
        while (res) {
            if (res & 1) {
                ans++;
            }
            res >>= 1;
        }
        return ans;
    }
};
```

### 868.二进制间距

**题目：**

给定一个正整数 `n`，找到并返回 `n` 的二进制表示中两个 **相邻** 1 之间的 **最长距离** 。如果不存在两个相邻的 1，返回 `0` 。

如果只有 `0` 将两个 `1` 分隔开（可能不存在 `0` ），则认为这两个 1 彼此 **相邻** 。两个 `1` 之间的距离是它们的二进制表示中位置的绝对差。例如，`"1001"` 中的两个 `1` 的距离为 3 。

**示例 1：**

```
输入：n = 22
输出：2
解释：22 的二进制是 "10110" 。
在 22 的二进制表示中，有三个 1，组成两对相邻的 1 。
第一对相邻的 1 中，两个 1 之间的距离为 2 。
第二对相邻的 1 中，两个 1 之间的距离为 1 。
答案取两个距离之中最大的，也就是 2 。
```

**示例 2：**

```
输入：n = 8
输出：0
解释：8 的二进制是 "1000" 。
在 8 的二进制表示中没有相邻的两个 1，所以返回 0 。
```

**提示：**

- `1 <= n <= 109`

**思路：**

```
这题的关键不再二进制，而在于模拟
判断1 就用 n & 1 是否等于1即可
```

**代码：**

```cpp
class Solution {
public:
    int binaryGap(int n) {
        int ans = 0;
        while (n) {
            // 判断到1，就开始找相邻的1
            if (n & 1) {
                int temp = 1;
                n >>= 1;
                while (n && (n & 1) == 0) {
                    n >>= 1;
                    temp++;
                }
                // 如果没有全部移动完，就说明此时间隔有效
                if (n) {
                    ans = max(ans, temp);
                }
            }else{
                //没有1就继续右移
                n >>= 1;
            }
        }
        return ans;
    }
};
```

### 2917.找出数组中的K-or值

**题目：**

给你一个整数数组 `nums` 和一个整数 `k` 。让我们通过扩展标准的按位或来介绍 **K-or** 操作。在 K-or 操作中，如果在 `nums` 中，至少存在 `k` 个元素的第 `i` 位值为 1 ，那么 K-or 中的第 `i` 位的值是 1 。返回 `nums` 的 **K-or** 值。

**示例 1：**

```
输入：nums = [7,12,9,8,9,15], k = 4
输出：9
解释：
用二进制表示 numbers：
```

| **Number**     | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
| -------------- | ----- | ----- | ----- | ----- |
| **7**          | 0     | 1     | 1     | 1     |
| **12**         | 1     | 1     | 0     | 0     |
| **9**          | 1     | 0     | 0     | 1     |
| **8**          | 1     | 0     | 0     | 0     |
| **9**          | 1     | 0     | 0     | 1     |
| **15**         | 1     | 1     | 1     | 1     |
| **Result = 9** | 1     | 0     | 0     | 1     |

```
位 0 在 7, 9, 9, 15 中为 1。位 3 在 12, 9, 8, 9, 15 中为 1。 只有位 0 和 3 满足。结果是 (1001)2 = 9。
```

**提示：**

- `1 <= nums.length <= 50`
- `0 <= nums[i] < 231`
- `1 <= k <= nums.length`

**思路：**

```
从示例来看，就是将数组nums中的每一个元素都不停右移1位，判断位置上为1的个数超没超过k
那么直接模拟即可，终止条件就是每一个nums中的元素都移完位了
```

**代码：**

```cpp
class Solution {
public:
    int findKOr(vector<int>& nums, int k) {
        long long ans = 0;
        long long base = 1;
        while (true) {
            bool flag = false;
            int cnt = 0;
            for (int i = 0; i < nums.size(); i++) {
                if (nums[i] > 0) {
                    flag = true;
                }
                if (nums[i] & 1) cnt++;
                nums[i] >>= 1;
            }
            if (cnt >= k) ans += base;
            base *= 2;
            if (!flag) {
                // 数组nums里面全是0了，就没必要再判断了
                break;
            }
        }
        return ans;
    }
};
```

## 与或（AND/OR）的性质

### 2980.检查按位或是否存在尾随0

**题目：**

给你一个 **正整数** 数组 `nums` 。

你需要检查是否可以从数组中选出 **两个或更多** 元素，满足这些元素的按位或运算（ `OR`）结果的二进制表示中 **至少** 存在一个尾随零。

例如，数字 `5` 的二进制表示是 `"101"`，不存在尾随零，而数字 `4` 的二进制表示是 `"100"`，存在两个尾随零。

如果可以选择两个或更多元素，其按位或运算结果存在尾随零，返回 `true`；否则，返回 `false` 。

**示例 1：**

```
输入：nums = [1,2,3,4,5]
输出：true
解释：如果选择元素 2 和 4，按位或运算结果是 6，二进制表示为 "110" ，存在一个尾随零。
```

**提示：**

- `2 <= nums.length <= 100`
- `1 <= nums[i] <= 100`

**思路：**

```
或运算的性质是：全0才得0
所以按位或要想得到尾随0，那么这些元素的最低位必须全是0
所以题目要判断的是nums中是否存在2个及以上的偶数
```

**代码：**

```cpp
class Solution {
public:
    bool hasTrailingZeros(vector<int>& nums) {
        int cnt_even = 0;
        for (int i = 0; i < nums.size(); i++) {
            if ((nums[i] & 1) == 0) {
                cnt_even++;
            }
        }
        return cnt_even > 1;
    }
};
```

### 1318.或运算的最小翻转次数

**题目：**

给你三个正整数 `a`、`b` 和 `c`。

你可以对 `a` 和 `b` 的二进制表示进行位翻转操作，返回能够使按位或运算  `a` OR `b` == `c` 成立的最小翻转次数。

「位翻转操作」是指将一个数的二进制表示任何单个位上的 1 变成 0 或者 0 变成 1 。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/11/sample_3_1676.png)

```
输入：a = 2, b = 6, c = 5
输出：3
解释：翻转后 a = 1 , b = 4 , c = 5 使得 a OR b == c
```

**思路：**

```
每次取a,b,c的最低位出来，判断即可
c的最低位是1，就看a,b最低位是不是都是0，是就多一次操作
c的最低位是0，就看a,b最低位是不是有1，有1个1就多一次操作
```

**代码：**

```cpp
class Solution {
public:
    int minFlips(int a, int b, int c) {
        int ans = 0;
        while (a || b || c) {
            int x_a = a & 1, x_b = b & 1, x_c = c & 1;
            if (x_c) {
                if (!(x_a || x_b)) ans++; 
            } else {
                if (x_a) ans++;
                if (x_b) ans++;
            }
            a >>= 1;
            b >>= 1;
            c >>= 1;
        }
        return ans;
    }
};
```

### 2419.按位与最大的最长子数组

**题目：**

给你一个长度为 `n` 的整数数组 `nums` 。

考虑 `nums` 中进行 **按位与（bitwise AND）**运算得到的值 **最大** 的 **非空** 子数组。

- 换句话说，令 `k` 是 `nums` **任意** 子数组执行按位与运算所能得到的最大值。那么，只需要考虑那些执行一次按位与运算后等于 `k` 的子数组。

返回满足要求的 **最长** 子数组的长度。

数组的按位与就是对数组中的所有数字进行按位与运算。

**子数组** 是数组中的一个连续元素序列。

**示例 1：**

```
输入：nums = [1,2,3,3,2,2]
输出：2
解释：
子数组按位与运算的最大值是 3 。
能得到此结果的最长子数组是 [3,3]，所以返回 2 。
```

**提示：**

- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 106`

**思路：**

```
这题很像脑筋急转弯，关键要清楚一个点：
	两个元素按位与运算，得到的数字不可能比其中任意一个数字大
所以数组中按位与运算得到的最大值一定就是数组中的最大元素
所以要做的就是统计数组中最大元素的最长有几个挨在一起
```

**代码：**

```cpp
class Solution {
public:
    int longestSubarray(vector<int>& nums) {
        int max_ele = -1;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] > max_ele) {
                max_ele = nums[i];
            }
        }
        int ans = 0;
        int cnt = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] == max_ele) {
                cnt++;
                ans = max(ans, cnt);
            }else {
                cnt = 0;
            }
        }
        return ans;
    }
};
```

### 2871.将数组分割成最多数目的子数组

**题目：**

给你一个只包含 **非负** 整数的数组 `nums` 。

我们定义满足 `l <= r` 的子数组 `nums[l..r]` 的分数为 `nums[l] AND nums[l + 1] AND ... AND nums[r]` ，其中 **AND** 是按位与运算。

请你将数组分割成一个或者更多子数组，满足：

- **每个** 元素都 **只** 属于一个子数组。
- 子数组分数之和尽可能 **小** 。

请你在满足以上要求的条件下，返回 **最多** 可以得到多少个子数组。

一个 **子数组** 是一个数组中一段连续的元素。

**示例 1：**

```
输入：nums = [1,0,2,0,1,2]
输出：3
解释：我们可以将数组分割成以下子数组：
- [1,0] 。子数组分数为 1 AND 0 = 0 。
- [2,0] 。子数组分数为 2 AND 0 = 0 。
- [1,2] 。子数组分数为 1 AND 2 = 0 。
分数之和为 0 + 0 + 0 = 0 ，是我们可以得到的最小分数之和。
在分数之和为 0 的前提下，最多可以将数组分割成 3 个子数组。所以返回 3 。
```

**提示：**

- `1 <= nums.length <= 105`
- `0 <= nums[i] <= 106`

**思路：**

```
提示 1
AND 的性质是，参与 AND 的数越多，结果越小。

提示 2
子数组的 AND，不会低于整个 nums 数组的 AND。

提示 3
如果 nums 数组的 AND（记作 a）是大于 0 的，那么只能分割出一个数组（即 nums 数组）。根据提示 2，如果分割出超过一个数组，那么得分至少为 2a，而这是大于 a 的，不满足「子数组分数之和尽量小」的要求。所以在 a>0 的情况下，答案为 1。

如果 nums 数组的 AND 是等于 0 的，那么可以分割出尽量多的 AND 等于 0 的子数组。怎么分？从左到右遍历数组，只要发现 AND 等于 0 就立刻分割。如果不立刻分割，由于 AND 的数越多越能为 0，现在多分了一个数，后面就要少分一个数，可能后面就不能为 0 了。注意，如果最后剩下的一段子数组的 AND 大于 0，这一段可以直接合并到前一个 AND 为 0 的子数组中。
```

**代码：**

```cpp
class Solution {
public:
    int maxSubarrays(vector<int>& nums) {
        int test = nums[0];
        for (int i = 1; i < nums.size(); i++) test &=  nums[i];
        if (test) {
            return 1;
        }
        int ans = 0;
        int cur = nums[0];
        for (int i = 1; i < nums.size(); i++) {
            if (cur == 0) {
                ans++;
                cur = nums[i];
                continue;
            }
            cur &= nums[i];
        }
        if (cur == 0) ans++;
        return ans;
    }
};
```

### 2401.最长优雅子数组

**题目：**

给你一个由 **正** 整数组成的数组 `nums` 。

如果 `nums` 的子数组中位于 **不同** 位置的每对元素按位 **与（AND）**运算的结果等于 `0` ，则称该子数组为 **优雅** 子数组。

返回 **最长** 的优雅子数组的长度。

**子数组** 是数组中的一个 **连续** 部分。

**注意：**长度为 `1` 的子数组始终视作优雅子数组。

**示例 1：**

```
输入：nums = [1,3,8,48,10]
输出：3
解释：最长的优雅子数组是 [3,8,48] 。子数组满足题目条件：
- 3 AND 8 = 0
- 3 AND 48 = 0
- 8 AND 48 = 0
可以证明不存在更长的优雅子数组，所以返回 3 。
```

**示例 2：**

```
输入：nums = [3,1,5,11,13]
输出：1
解释：最长的优雅子数组长度为 1 ，任何长度为 1 的子数组都满足题目条件。
```

**提示：**

- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 109`

**思路：**

```
子数组中任意两个数按位与均为 0，意味着任意两个数对应的集合的交集为空

这意味着子数组中的从低到高的第 i 个比特位上，至多有一个比特 1，其余均为比特 0。例如子数组不可能有两个奇数（最低位为 1)，否则这两个数按位与是大于 0 的。

根据鸽巢原理（抽屉原理），在本题数据范围下，优雅子数组的长度不会超过 30。例如子数组为 [2^0,2^1,2^2,⋯ ,2^29]，我们无法再加入一个数 x，使 x 与子数组中的任何一个数按位与均为 0。
既然长度不会超过 30，直接暴力枚举子数组的右端点 i 即可。

代码实现时，可以把在子数组中的元素按位或起来（求并集），这样可以 O(1) 判断当前元素是否与前面的元素按位与的结果为 0（交集为空）。
```

**代码：**

```cpp
class Solution {
public:
    int longestNiceSubarray(vector<int> &nums) {
        int ans = 0;
        for (int i = 0; i < nums.size(); i++) { // 枚举子数组右端点 i
            int or_ = 0, j = i;
            while (j >= 0 && (or_ & nums[j]) == 0) // nums[j] 与子数组中的任意元素 AND 均为 0
                or_ |= nums[j--]; // 加到子数组中
            ans = max(ans, i - j);
        }
        return ans;
    }
};
```

### 3097.或值至少为K的最短子数组

**题目：**

给你一个 **非负** 整数数组 `nums` 和一个整数 `k` 。如果一个数组中所有元素的按位或运算 `OR` 的值 **至少** 为 `k` ，那么我们称这个数组是 **特别的** 。请你返回 `nums` 中 **最短特别非空** 子数组的长度，如果特别子数组不存在，那么返回 `-1` 。

**示例 1：**

**输入：**nums = [1,2,3], k = 2

**输出：**1

**解释：**

子数组 `[3]` 的按位 `OR` 值为 `3` ，所以我们返回 `1` 。

**提示：**

- `1 <= nums.length <= 2 * 105`
- `0 <= nums[i] <= 109`
- `0 <= k <= 109`

**思路：**

```
暴力枚举右端点
```

**代码：**

```cpp
class Solution {
public:
    int minimumSubarrayLength(vector<int>& nums, int k) {
        int ans = INT_MAX;
        for (int i = 0; i < nums.size(); i++) {
            int temp = nums[i];
            if (temp >= k) return 1;
            for (int j = i - 1; j >= 0; j--) {
                temp |= nums[j];
                if (temp >= k) ans = min(ans, i - j + 1);
            }
        }
        if (ans == INT_MAX) {return -1;}
        return ans;
    }
};
```

### 3133.数组最后一个元素的最小值

**题目：**

给你两个整数 `n` 和 `x` 。你需要构造一个长度为 `n` 的 **正整数** 数组 `nums` ，对于所有 `0 <= i < n - 1` ，满足 `nums[i + 1]` **大于** `nums[i]` ，并且数组 `nums` 中所有元素的按位 `AND` 运算结果为 `x` 。

返回 `nums[n - 1]` 可能的 **最小** 值。

**示例 1：**

**输入：**n = 3, x = 4

**输出：**6

**解释：**

数组 `nums` 可以是 `[4,5,6]` ，最后一个元素为 `6` 。

**提示：**

- `1 <= n, x <= 108`

**思路：**

![image-20240524113533191](https://zxyandzxy.github.io/images/image-20240524113533191.png)

**代码：**

```cpp
class Solution {
public:
    long long minEnd(int n, int x) {
        n--; // 先把 n 减一，这样下面讨论的 n 就是原来的 n-1
        long long ans = x;
        int i = 0, j = 0;
        while (n >> j) {
            // x 的第 i 个比特值是 0，即「空位」
            if ((ans >> i & 1) == 0) {
                // 空位填入 n 的第 j 个比特值
                ans |= (long long) (n >> j & 1) << i;
                j++;
            }
            i++;
        }
        return ans;
    }
};
```

## 异或（XOR）的性质

### 1720.解码异或后的数组

**题目：**

**未知** 整数数组 `arr` 由 `n` 个非负整数组成。

经编码后变为长度为 `n - 1` 的另一个整数数组 `encoded` ，其中 `encoded[i] = arr[i] XOR arr[i + 1]` 。例如，`arr = [1,0,2,1]` 经编码后得到 `encoded = [1,2,3]` 。

给你编码后的数组 `encoded` 和原数组 `arr` 的第一个元素 `first`（`arr[0]`）。

请解码返回原数组 `arr` 。可以证明答案存在并且是唯一的。

**示例 1：**

```
输入：encoded = [1,2,3], first = 1
输出：[1,0,2,1]
解释：若 arr = [1,0,2,1] ，那么 first = 1 且 encoded = [1 XOR 0, 0 XOR 2, 2 XOR 1] = [1,2,3]
```

**提示**

- `2 <= n <= 104`
- `encoded.length == n - 1`
- `0 <= encoded[i] <= 105`
- `0 <= first <= 105`

**思路：**

![image-20240524205212682](https://zxyandzxy.github.io/images/image-20240524205212682.png)

**代码：**

```cpp
class Solution {
public:
    vector<int> decode(vector<int>& encoded, int first) {
        int n = encoded.size() + 1;
        vector<int> arr(n);
        arr[0] = first;
        for (int i = 1; i < n; i++) {
            arr[i] = arr[i - 1] ^ encoded[i - 1];
        }
        return arr;
    }
};
```

### 2433.找出前缀异或的原始数组

**题目：**

给你一个长度为 `n` 的 **整数** 数组 `pref` 。找出并返回满足下述条件且长度为 `n` 的数组 `arr` ：

- `pref[i] = arr[0] ^ arr[1] ^ ... ^ arr[i]`.

注意 `^` 表示 **按位异或**（bitwise-xor）运算。可以证明答案是 **唯一** 的。

**示例 1：**

```
输入：pref = [5,2,0,3,1]
输出：[5,7,2,3,2]
解释：从数组 [5,7,2,3,2] 可以得到如下结果：
- pref[0] = 5
- pref[1] = 5 ^ 7 = 2
- pref[2] = 5 ^ 7 ^ 2 = 0
- pref[3] = 5 ^ 7 ^ 2 ^ 3 = 3
- pref[4] = 5 ^ 7 ^ 2 ^ 3 ^ 2 = 1
```

**示例 2：**

```
输入：pref = [13]
输出：[13]
解释：pref[0] = arr[0] = 13
```

**提示：**

- `1 <= pref.length <= 105`
- `0 <= pref[i] <= 106`

**思路：**

```
这题和1720题本质上是一样的，只不过我们需要记录ans前缀异或
```

**代码：**

```cpp
class Solution {
public:
    vector<int> findArray(vector<int>& pref) {
        int n = pref.size();
        vector<int> ans(n);
        ans[0] = pref[0];
        int a = pref[0];
        for (int i = 1; i < n; i++) {
            ans[i] = a ^ pref[i];
            a ^= ans[i];
        }
        return ans;
    }
};
```

### 1310.子数组异或查询

**题目：**

有一个正整数数组 `arr`，现给你一个对应的查询数组 `queries`，其中 `queries[i] = [Li, Ri]`。对于每个查询 `i`，请你计算从 `Li` 到 `Ri` 的 **XOR** 值（即 `arr[Li] xor arr[Li+1] xor ... xor arr[Ri]`）作为本次查询的结果。并返回一个包含给定查询 `queries` 所有结果的数组。

**示例 1：**

```
输入：arr = [1,3,4,8], queries = [[0,1],[1,2],[0,3],[3,3]]
输出：[2,7,14,8] 
解释：
数组中元素的二进制表示形式是：
1 = 0001 
3 = 0011 
4 = 0100 
8 = 1000 
查询的 XOR 值为：
[0,1] = 1 xor 3 = 2 
[1,2] = 3 xor 4 = 7 
[0,3] = 1 xor 3 xor 4 xor 8 = 14 
[3,3] = 8
```

**提示：**

- `1 <= arr.length <= 3 * 10^4`
- `1 <= arr[i] <= 10^9`
- `1 <= queries.length <= 3 * 10^4`
- `queries[i].length == 2`
- `0 <= queries[i][0] <= queries[i][1] < arr.length`

**思路：**

```
本题更加关心优化，我们可以先求出数组的前缀异或数组pre，然后对于[l, r]间元素的异或值，直接就是pre[l - 1] ^ pre[r]
```

**代码：**

```cpp
class Solution {
public:
    vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
        int n = arr.size();
        vector<int> pre(n);
        pre[0] = arr[0];
        int cur = pre[0];
        for (int i = 1; i < n; i++) {
            cur ^= arr[i];
            pre[i] = cur;
        }
        int m = queries.size();
        vector<int> ans(m);
        for (int i = 0; i < m; i++) {
            int l = queries[i][0], r = queries[i][1];
            if (l == 0) {
                ans[i] = pre[r];
                continue;
            }
            ans[i] = pre[r] ^ pre[l - 1];
        }
        return ans;
    }
};
```

### 1829.每个查询的最大异或值

**题目：**

给你一个 **有序** 数组 `nums` ，它由 `n` 个非负整数组成，同时给你一个整数 `maximumBit` 。你需要执行以下查询 `n` 次：

1. 找到一个非负整数 `k < 2maximumBit` ，使得 `nums[0] XOR nums[1] XOR ... XOR nums[nums.length-1] XOR k` 的结果 **最大化** 。`k` 是第 `i` 个查询的答案。
2. 从当前数组 `nums` 删除 **最后** 一个元素。

请你返回一个数组 `answer` ，其中 `answer[i]`是第 `i` 个查询的结果。

**示例 1：**

```
输入：nums = [0,1,1,3], maximumBit = 2
输出：[0,3,2,3]
解释：查询的答案如下：
第一个查询：nums = [0,1,1,3]，k = 0，因为 0 XOR 1 XOR 1 XOR 3 XOR 0 = 3 。
第二个查询：nums = [0,1,1]，k = 3，因为 0 XOR 1 XOR 1 XOR 3 = 3 。
第三个查询：nums = [0,1]，k = 2，因为 0 XOR 1 XOR 2 = 3 。
第四个查询：nums = [0]，k = 3，因为 0 XOR 3 = 3 。
```

**提示：**

- `nums.length == n`
- `1 <= n <= 105`
- `1 <= maximumBit <= 20`
- `0 <= nums[i] < 2maximumBit`
- `nums` 中的数字已经按 **升序** 排好序。

**思路：**

```
暴力枚举每个k即可
```

**代码：**

```cpp
class Solution {
public:
    vector<int> getMaximumXor(vector<int>& nums, int maximumBit) {
        int n = nums.size();
        vector<int> pre(n);
        for (int i = 0; i < n; i++) {
            if (i == 0) {
                pre[i] = nums[i];
                continue;
            }
            pre[i] = pre[i - 1] ^ nums[i];
        }
        vector<int> ans(n);
        int base = 1;
        while (maximumBit) {
            base *= 2;
            maximumBit--;
        }
        for (int i = 0; i < n; i++) {
            int max_ans = -1;
            for (int j = 0; j < base; j++) {
                if ((pre[n - 1 - i] ^ j) > max_ans) {
                    max_ans = pre[n - 1 - i] ^ j;
                    ans[i] = j; 
                }
            }
        }
        return ans;
    }
};
```

### 2997.使数组异或和等于K的最少操作次数

**题目：**

给你一个下标从 **0** 开始的整数数组 `nums` 和一个正整数 `k` 。你可以对数组执行以下操作 **任意次** ：

- 选择数组里的 **任意** 一个元素，并将它的 **二进制** 表示 **翻转** 一个数位，翻转数位表示将 `0` 变成 `1` 或者将 `1` 变成 `0` 。

你的目标是让数组里 **所有** 元素的按位异或和得到 `k` ，请你返回达成这一目标的 **最少** 操作次数。

**注意**，你也可以将一个数的前导 0 翻转。比方说，数字 `(101)2` 翻转第四个数位，得到 `(1101)2` 。

**示例 1：**

```
输入：nums = [2,1,3,4], k = 1
输出：2
解释：我们可以执行以下操作：
- 选择下标为 2 的元素，也就是 3 == (011)2 ，我们翻转第一个数位得到 (010)2 == 2 。数组变为 [2,1,2,4] 。
- 选择下标为 0 的元素，也就是 2 == (010)2 ，我们翻转第三个数位得到 (110)2 == 6 。数组变为 [6,1,2,4] 。
最终数组的所有元素异或和为 (6 XOR 1 XOR 2 XOR 4) == 1 == k 。
无法用少于 2 次操作得到异或和等于 k 。
```

**示例 2：**

```
输入：nums = [2,0,2,0], k = 0
输出：0
解释：数组所有元素的异或和为 (2 XOR 0 XOR 2 XOR 0) == 0 == k 。所以不需要进行任何操作。
```

**提示：**

- `1 <= nums.length <= 105`
- `0 <= nums[i] <= 106`
- `0 <= k <= 106`

**思路：**

```
设 nums 的异或和为 s。

s=k 等价于 s⊕k=0，其中 ⊕ 表示异或。

设 x=s⊕k，我们把 nums 中的任意数字的某个比特位翻转，那么 x 的这个比特位也会翻转。要让 x=0，就必须把 x 中的每个 1 都翻转，所以 x 中的 1 的个数就是我们的操作次数。
```

**代码：**

```cpp
class Solution {
public:
    int minOperations(vector<int> &nums, int k) {
        for (int x : nums) {
            k ^= x;
        }
        return __builtin_popcount(k);
    }
};
```

### 1442.形成两个异或相等数组的三元组数目

**题目：**

给你一个整数数组 `arr` 。现需要从数组中取三个下标 `i`、`j` 和 `k` ，其中 `(0 <= i < j <= k < arr.length)` 。`a` 和 `b` 定义如下：

- `a = arr[i] ^ arr[i + 1] ^ ... ^ arr[j - 1]`
- `b = arr[j] ^ arr[j + 1] ^ ... ^ arr[k]`

注意：**^** 表示 **按位异或** 操作。请返回能够令 `a == b` 成立的三元组 (`i`, `j` , `k`) 的数目。

**示例 1：**

```
输入：arr = [2,3,1,6,7]
输出：4
解释：满足题意的三元组分别是 (0,1,2), (0,2,2), (2,3,4) 以及 (2,4,4)
```

**提示：**

- `1 <= arr.length <= 300`
- `1 <= arr[i] <= 10^8`

**思路：**

```
使用数组的异或前缀和，这样我们就可以暴力枚举i,j,k
时间复杂度：O(n^3)
```

**代码：**

```cpp
class Solution {
public:
    int countTriplets(vector<int>& arr) {
        vector<int> pre(arr.size());
        for (int i = 0; i < arr.size(); i++) {
            if (i == 0) {
                pre[i] = arr[i];
                continue;
            }
            pre[i] = pre[i - 1] ^ arr[i];
        }
        int ans = 0;
        int a, b;
        for (int i = 0; i < arr.size() - 1; i++) {
            for (int j = i + 1; j < arr.size(); j++) {
                for (int k = j; k < arr.size(); k++) {
                    if (i == 0) {
                        a = pre[j - 1];
                    }else {
                        a = pre[j - 1] ^ pre[i - 1];
                    }
                    b = pre[k] ^ pre[j - 1];
                    if (a == b) ans++;
                }
            }
        }
        return ans;
    }
};
```

### 2429.最小异或

**题目：**

给你两个正整数 `num1` 和 `num2` ，找出满足下述条件的正整数 `x` ：

- `x` 的置位数和 `num2` 相同，且
- `x XOR num1` 的值 **最小**

注意 `XOR` 是按位异或运算。返回整数 `x` 。题目保证，对于生成的测试用例， `x` 是 **唯一确定** 的。整数的 **置位数** 是其二进制表示中 `1` 的数目。

**示例 1：**

```
输入：num1 = 3, num2 = 5
输出：3
解释：
num1 和 num2 的二进制表示分别是 0011 和 0101 。
整数 3 的置位数与 num2 相同，且 3 XOR 3 = 0 是最小的。
```

**示例 2：**

```
输入：num1 = 1, num2 = 12
输出：3
解释：
num1 和 num2 的二进制表示分别是 0001 和 1100 。
整数 3 的置位数与 num2 相同，且 3 XOR 1 = 2 是最小的。
```

**提示：**

- `1 <= num1, num2 <= 109`

**思路：**

```
我们首先统计num1和num2中置位数
if cnt_num1 == cnt_num2 说明 x = num1,这样 x xor num2 = 0最小
if cnt_num1 > cnt_num2 说明 x中1的位置是num1中高位1的位置，共cnt_num2个
if cnt_num1 < cnt_num2 说明 x中1的位置是num1中所有1的位置 + num1中低位0的位置(共cnt_num2 - cnt_num1个)
```

**代码：**

```cpp
class Solution {
public:
    int minimizeXor(int num1, int num2) {
        int copy_num1 = num1, copy_num2 = num2;
        int c_num1 = num1;
        int cnt_num1 = 0, cnt_num2;
        while (num1) {
            if (num1 & 1) {
                cnt_num1++;
            }
            num1 >>= 1;
        }
        while (num2) {
            if (num2 & 1) {
                cnt_num2++;
            }
            num2 >>= 1;
        }
        if (cnt_num1 == cnt_num2) return copy_num1;
        int ans = 0, base = 1;
        if (cnt_num1 > cnt_num2) {
            int diff = cnt_num1 - cnt_num2;
            while (diff) {
                if (copy_num1 & 1) {
                    ans += base;
                    diff--;
                }
                base *= 2;
                copy_num1 >>= 1;
            }
            ans = c_num1 - ans;
        } else {
            int diff = cnt_num2 - cnt_num1;
            while (diff) {
                if (!(copy_num1 & 1)) {
                    ans += base;
                    diff--;
                }
                base *= 2;
                copy_num1 >>= 1;
            }
            ans += c_num1;
        }
        return ans;
    }
};
```

## 拆位 / 贡献法

### 477.汉明距离总和

**题目：**

两个整数的 [汉明距离](https://baike.baidu.com/item/汉明距离/475174?fr=aladdin) 指的是这两个数字的二进制数对应位不同的数量。

给你一个整数数组 `nums`，请你计算并返回 `nums` 中任意两个数之间 **汉明距离的总和** 。

**示例 1：**

```
输入：nums = [4,14,2]
输出：6
解释：在二进制表示中，4 表示为 0100 ，14 表示为 1110 ，2表示为 0010 。（这样表示是为了体现后四位之间关系）
所以答案为：
HammingDistance(4, 14) + HammingDistance(4, 2) + HammingDistance(14, 2) = 2 + 2 + 2 = 6
```

**示例 2：**

```
输入：nums = [4,14,4]
输出：4
```

**提示：**

- `1 <= nums.length <= 104`
- `0 <= nums[i] <= 109`
- 给定输入的对应答案符合 **32-bit** 整数范围

**思路：**

```
暴力枚举nums中的任意两个数字，然后统计他们异或后结果中1的个数即可
```

**代码：**

```cpp
class Solution {
public:
    int totalHammingDistance(vector<int>& nums) {
        int ans = 0;
        for (int i = 0; i < nums.size(); i++) {
            for (int j = i + 1; j < nums.size(); j++) {
                ans += __builtin_popcount(nums[i] ^ nums[j]);
            }
        }
        return ans;
    }
};
```

### 1863.找出所有子集的异或总和再求和

**题目：**

一个数组的 **异或总和** 定义为数组中所有元素按位 `XOR` 的结果；如果数组为 **空** ，则异或总和为 `0` 。

- 例如，数组 `[2,5,6]` 的 **异或总和** 为 `2 XOR 5 XOR 6 = 1` 。

给你一个数组 `nums` ，请你求出 `nums` 中每个 **子集** 的 **异或总和** ，计算并返回这些值相加之 **和** 。**注意：**在本题中，元素 **相同** 的不同子集应 **多次** 计数。数组 `a` 是数组 `b` 的一个 **子集** 的前提条件是：从 `b` 删除几个（也可能不删除）元素能够得到 `a` 。

**示例 1：**

```
输入：nums = [1,3]
输出：6
解释：[1,3] 共有 4 个子集：
- 空子集的异或总和是 0 。
- [1] 的异或总和为 1 。
- [3] 的异或总和为 3 。
- [1,3] 的异或总和为 1 XOR 3 = 2 。
0 + 1 + 3 + 2 = 6
```

**提示：**

- `1 <= nums.length <= 12`
- `1 <= nums[i] <= 20`

**思路：**

```
先求子集，再对每个子集进行异或和相加
```

**代码：**

```cpp
class Solution {
public:
    vector<vector<int>> result;
    vector<int> path;
    void backtracking(vector<int>& nums, int start) {
        result.push_back(path);
        for (int i = start; i < nums.size(); i++) {
            path.push_back(nums[i]);
            backtracking(nums, i + 1);
            path.pop_back();
        }
    }
    int subsetXORSum(vector<int>& nums) {
        backtracking(nums, 0);
        int ans = 0;
        for (int i = 0; i < result.size(); i++) {
            if (result[i].size() == 0) continue;
            int cur = result[i][0];
            for (int j = 1; j < result[i].size(); j++) {
                cur ^= result[i][j];
            }
            ans += cur;
        }
        return ans;
    }
};
```

### 2425.所有数对的异或和

**题目：**

给你两个下标从 **0** 开始的数组 `nums1` 和 `nums2` ，两个数组都只包含非负整数。请你求出另外一个数组 `nums3` ，包含 `nums1` 和 `nums2` 中 **所有数对** 的异或和（`nums1` 中每个整数都跟 `nums2` 中每个整数 **恰好** 匹配一次）。

请你返回 `nums3` 中所有整数的 **异或和** 。

**示例 1：**

```
输入：nums1 = [2,1,3], nums2 = [10,2,5,0]
输出：13
解释：
一个可能的 nums3 数组是 [8,0,7,2,11,3,4,1,9,1,6,3] 。
所有这些数字的异或和是 13 ，所以我们返回 13 。
```

**示例 2：**

```
输入：nums1 = [1,2], nums2 = [3,4]
输出：0
解释：
所有数对异或和的结果分别为 nums1[0] ^ nums2[0] ，nums1[0] ^ nums2[1] ，nums1[1] ^ nums2[0] 和 nums1[1] ^ nums2[1] 。
所以，一个可能的 nums3 数组是 [2,5,1,6] 。
2 ^ 5 ^ 1 ^ 6 = 0 ，所以我们返回 0 。
```

**提示：**

- `1 <= nums1.length, nums2.length <= 105`
- `0 <= nums1[i], nums2[j] <= 109`

**思路：**

```
暴力枚举每个数对，然后进行异或运算
```

**代码：**

```cpp
class Solution {
public:
    int xorAllNums(vector<int>& nums1, vector<int>& nums2) {
        int n = nums1.size(), m = nums2.size();
        vector<int> result(n * m);
        int ans;
        int idx = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                result[idx++] = nums1[i] ^ nums2[j];
                if (idx == 1) ans = result[0];
                else ans ^= result[idx - 1];
            }
        }
        return ans;
    }
};
```

### 2275.按位与结果大于0的最长组合

**题目：**

对数组 `nums` 执行 **按位与** 相当于对数组 `nums` 中的所有整数执行 **按位与** 。

- 例如，对 `nums = [1, 5, 3]` 来说，按位与等于 `1 & 5 & 3 = 1` 。
- 同样，对 `nums = [7]` 而言，按位与等于 `7` 。

给你一个正整数数组 `candidates` 。计算 `candidates` 中的数字每种组合下 **按位与** 的结果。 `candidates` 中的每个数字在每种组合中只能使用 **一次** 。

返回按位与结果大于 `0` 的 **最长** 组合的长度*。*

**示例 1：**

```
输入：candidates = [16,17,71,62,12,24,14]
输出：4
解释：组合 [16,17,62,24] 的按位与结果是 16 & 17 & 62 & 24 = 16 > 0 。
组合长度是 4 。
可以证明不存在按位与结果大于 0 且长度大于 4 的组合。
注意，符合长度最大的组合可能不止一种。
例如，组合 [62,12,24,14] 的按位与结果是 62 & 12 & 24 & 14 = 8 > 0 。
```

**示例 2：**

```
输入：candidates = [8,8]
输出：2
解释：最长组合是 [8,8] ，按位与结果 8 & 8 = 8 > 0 。
组合长度是 2 ，所以返回 2 。
```

**提示：**

- `1 <= candidates.length <= 105`
- `1 <= candidates[i] <= 107`

**思路：**

```
先回溯法求出所有子集
对每个子集进行按位与，找最长即可
```

**代码：**

```cpp
class Solution {
public:
    vector<vector<int>> result;
    vector<int> path;
    void backtracking(vector<int>& candidates, int start) {
        result.push_back(path);
        for (int i = start; i < candidates.size(); i++) {
            path.push_back(candidates[i]);
            backtracking(candidates, i + 1);
            path.pop_back();
        }
    }
    int largestCombination(vector<int>& candidates) {
        backtracking(candidates, 0);
        int ans = 0;
        for (int i = 0; i < result.size(); i++) {
            if (result[i].size() == 0) continue;
            int cur = result[i][0];
            for (int j = 1; j < result[i].size(); j++) {
                cur &= result[i][j];
            }
            if (cur) {
                if (ans < result[i].size()) {
                    ans = result[i].size();
                }
            }
        }
        return ans;
    }
};
```

## 试填法

### 3007.价值和小于等于K的最大数字

**题目：**

给你一个整数 `k` 和一个整数 `x` 。整数 `num` 的价值是它的二进制表示中在 `x`，`2x`，`3x` 等位置处 **设置位**的数目（从最低有效位开始）。下面的表格包含了如何计算价值的例子。

| x    | num  | Binary Representation | Price |
| ---- | ---- | --------------------- | ----- |
| 1    | 13   | 00000**11**0**1**     | 3     |
| 2    | 13   | 00000**1**101         | 1     |
| 2    | 233  | 0**1**1**1**0**1**001 | 3     |
| 3    | 13   | 000001**1**01         | 1     |
| 3    | 362  | **1**01**1**01010     | 2     |

`num` 的 **累加价值** 是从 `1` 到 `num` 的数字的 **总** 价值。如果 `num` 的累加价值小于或等于 `k` 则被认为是 **廉价** 的。请你返回 **最大** 的廉价数字。

**示例 1：**

```
输入：k = 9, x = 1
输出：6
解释：由下表所示，6 是最大的廉价数字。
```

| x    | num  | Binary Representation | Price | Accumulated Price |
| ---- | ---- | --------------------- | ----- | ----------------- |
| 1    | 1    | 00**1**               | 1     | 1                 |
| 1    | 2    | 0**1**0               | 1     | 2                 |
| 1    | 3    | 0**11**               | 2     | 4                 |
| 1    | 4    | **1**00               | 1     | 5                 |
| 1    | 5    | **1**0**1**           | 2     | 7                 |
| 1    | 6    | **11**0               | 2     | 9                 |
| 1    | 7    | **111**               | 3     | 12                |

**思路：**

```
关键就是模拟求一个数字价值的过程
```

**代码：**

```cpp
class Solution {
public:
    long long getPrice(long long num, int x) {
        int ans = 0;
        int cnt = 1;
        while (num) {
            if (cnt == x) {
                ans += (num & 1);
                cnt = 0;
            }
            num >>= 1;
            cnt++;
        }
        return ans;
    }
    long long findMaximumNumber(long long k, int x) {
        long long s = 0;
        long long idx = 1;
        while (s <= k) {
            s += getPrice(idx++, x);
        }
        return idx - 2;
    }
};
```

### 2935.找出强数对的最大异或值II

**题目：**

给你一个下标从 **0** 开始的整数数组 `nums` 。如果一对整数 `x` 和 `y` 满足以下条件，则称其为 **强数对** ：

- `|x - y| <= min(x, y)`

你需要从 `nums` 中选出两个整数，且满足：这两个整数可以形成一个强数对，并且它们的按位异或（`XOR`）值是在该数组所有强数对中的 **最大值** 。

返回数组 `nums` 所有可能的强数对中的 **最大** 异或值。**注意**，你可以选择同一个整数两次来形成一个强数对。

**示例 1：**

```
输入：nums = [1,2,3,4,5]
输出：7
解释：数组 nums 中有 11 个强数对：(1, 1), (1, 2), (2, 2), (2, 3), (2, 4), (3, 3), (3, 4), (3, 5), (4, 4), (4, 5) 和 (5, 5) 。
这些强数对中的最大异或值是 3 XOR 4 = 7 。
```

**示例 2：**

```
输入：nums = [10,100]
输出：0
解释：数组 nums 中有 2 个强数对：(10, 10) 和 (100, 100) 。
这些强数对中的最大异或值是 10 XOR 10 = 0 ，数对 (100, 100) 的异或值也是 100 XOR 100 = 0 。
```

**提示：**

- `1 <= nums.length <= 5 * 104`
- `1 <= nums[i] <= 220 - 1`

**思路：**

```
暴力求解所有强数对，然后取强数对最大值
```

**代码：**

```cpp
class Solution {
public:
    int maximumStrongPairXor(vector<int>& nums) {
        int ans = 0;
        for (int i = 0; i < nums.size(); i++) {
            for (int j = i + 1; j < nums.size(); j++) {
                if (abs(nums[i] - nums[j]) <= min(nums[i], nums[j])) {
                    ans = max(ans, nums[i] ^ nums[j]);
                }
            }
        }
        return ans;
    }
};
```

## 恒等式

### 2354.优质数对的数目

**题目：**

给你一个下标从 **0** 开始的正整数数组 `nums` 和一个正整数 `k` 。

如果满足下述条件，则数对 `(num1, num2)` 是 **优质数对** ：

- `num1` 和 `num2` **都** 在数组 `nums` 中存在。
- `num1 OR num2` 和 `num1 AND num2` 的二进制表示中值为 **1** 的位数之和大于等于 `k` ，其中 `OR` 是按位 **或** 操作，而 `AND` 是按位 **与** 操作。

返回 **不同** 优质数对的数目。如果 `a != c` 或者 `b != d` ，则认为 `(a, b)` 和 `(c, d)` 是不同的两个数对。例如，`(1, 2)` 和 `(2, 1)` 不同。

**注意：**如果 `num1` 在数组中至少出现 **一次** ，则满足 `num1 == num2` 的数对 `(num1, num2)` 也可以是优质数对。

**示例 1：**

```
输入：nums = [1,2,3,1], k = 3
输出：5
解释：有如下几个优质数对：
- (3, 3)：(3 AND 3) 和 (3 OR 3) 的二进制表示都等于 (11) 。值为 1 的位数和等于 2 + 2 = 4 ，大于等于 k = 3 。
- (2, 3) 和 (3, 2)： (2 AND 3) 的二进制表示等于 (10) ，(2 OR 3) 的二进制表示等于 (11) 。值为 1 的位数和等于 1 + 2 = 3 。
- (1, 3) 和 (3, 1)： (1 AND 3) 的二进制表示等于 (01) ，(1 OR 3) 的二进制表示等于 (11) 。值为 1 的位数和等于 1 + 2 = 3 。
所以优质数对的数目是 5 。
```

**示例 2：**

```
输入：nums = [5,1,1], k = 10
输出：0
解释：该数组中不存在优质数对。
```

**提示：**

- `1 <= nums.length <= 105`
- `1 <= nums[i] <= 109`
- `1 <= k <= 60`

**思路：**

```
暴力枚举所有数对，利用set去重，统计所有优质数对个数
```

**代码：**

```cpp
class Solution {
public:
    long long countExcellentPairs(vector<int>& nums, int k) {
        set<pair<int, int>> hash;
        for (int i = 0; i < nums.size(); i++) {
            for (int j = 0; j < nums.size(); j++) {
                if (hash.find({nums[i], nums[j]}) == hash.end()) {
                    if (__builtin_popcount(nums[i] & nums[j]) + __builtin_popcount(nums[i] | nums[j]) >= k) {
                        hash.insert({nums[i], nums[j]});
                    }
                }
            }
        }
        return hash.size();
    }
};
```

### 1835.所有数对按位与结果的异或和

**题目：**

列表的 **异或和**（**XOR sum**）指对所有元素进行按位 `XOR` 运算的结果。如果列表中仅有一个元素，那么其 **异或和** 就等于该元素。

- 例如，`[1,2,3,4]` 的 **异或和** 等于 `1 XOR 2 XOR 3 XOR 4 = 4` ，而 `[3]` 的 **异或和** 等于 `3` 。

给你两个下标 **从 0 开始** 计数的数组 `arr1` 和 `arr2` ，两数组均由非负整数组成。根据每个 `(i, j)` 数对，构造一个由 `arr1[i] AND arr2[j]`（按位 `AND` 运算）结果组成的列表。其中 `0 <= i < arr1.length` 且 `0 <= j < arr2.length` 。返回上述列表的 **异或和** 。

**示例 1：**

```
输入：arr1 = [1,2,3], arr2 = [6,5]
输出：0
解释：列表 = [1 AND 6, 1 AND 5, 2 AND 6, 2 AND 5, 3 AND 6, 3 AND 5] = [0,1,2,0,2,1] ，
异或和 = 0 XOR 1 XOR 2 XOR 0 XOR 2 XOR 1 = 0 。
```

**示例 2：**

```
输入：arr1 = [12], arr2 = [4]
输出：4
解释：列表 = [12 AND 4] = [4] ，异或和 = 4 。
```

**提示：**

- `1 <= arr1.length, arr2.length <= 105`
- `0 <= arr1[i], arr2[j] <= 109`

**思路：**

```
暴力构造所有数对，再进行异或和
```

**代码：**

```cpp
class Solution {
public:
    int getXORSum(vector<int>& arr1, vector<int>& arr2) {
        int n = arr1.size(), m = arr2.size();
        vector<int> result(n * m);
        int idx = 0;
        int ans;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                result[idx++] = arr1[i] & arr2[j];
            }
        }
        for (int i = 0; i < n * m; i++) {
            if (i == 0) ans = result[i];
            else ans ^= result[i];
        }
        return ans;
    }
};
```













## 
