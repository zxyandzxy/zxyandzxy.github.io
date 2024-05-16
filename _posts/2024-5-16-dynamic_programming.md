---
layout: post
title: "动态规划"
date: 2024-5-16
tags: [algorithm]
comments: true
author: zxy
---

### 动态规划理论基础

#### 什么是动态规划

动态规划，英文：Dynamic Programming，简称 DP，如果某一问题有很多重叠子问题，使用动态规划是最有效的。

所以动态规划中每一个状态一定是由上一个状态推导出来的，**这一点就区分于贪心**，贪心没有状态推导，而是从局部直接选最优的，

所以贪心解决不了动态规划的问题。

#### 动态规划的解题步骤

1. 确定 dp 数组（dp table）以及下标的含义
2. 确定递推公式
3. dp 数组如何初始化
4. 确定遍历顺序
5. 举例推导 dp 数组

#### 动态规划应该如何 debug

**找问题的最好方式就是把 dp 数组打印出来，看看究竟是不是按照自己思路推导的！**

**做动规的题目，写代码之前一定要把状态转移在 dp 数组的上具体情况模拟一遍，心中有数，确定最后推出的是想要的结果**。

然后再写代码，如果代码没通过就打印 dp 数组，看看是不是和自己预先推导的哪里不一样。

如果打印出来和自己预先模拟推导是一样的，那么就是自己的递归公式、初始化或者遍历顺序有问题了。

如果和自己预先模拟推导的不一样，那么就是代码实现细节有问题。

### 509. 斐波那契数

```
斐波那契数，通常用 F(n) 表示，形成的序列称为 斐波那契数列 。该数列由 0 和 1 开始，后面的每一项数字都是前面两项数字的和。也就是： F(0) = 0，F(1) = 1 F(n) = F(n - 1) + F(n - 2)，其中 n > 1 给你n ，请计算 F(n) 。

示例 1：
输入：2
输出：1
解释：F(2) = F(1) + F(0) = 1 + 0 = 1

0 <= n <= 30
```

```
动态规划五部曲：
1.确定dp数组以及下标的含义
dp[i]的定义为：第i个数的斐波那契数值是dp[i]

2.确定递推公式
dp[i] = dp[i - 1] + dp[i - 2];

3.dp数组如何初始化
dp[0] = 0;	dp[1] = 1;

4.确定遍历顺序
从递归公式dp[i] = dp[i - 1] + dp[i - 2];中可以看出，dp[i]是依赖 dp[i - 1] 和 dp[i - 2]，那么遍历的顺序一定是从前到后遍历的

5.举例推导dp数组
当N为10的时候，dp数组应该是如下的数列：
0 1 1 2 3 5 8 13 21 34 55
```

```cpp
class Solution {
public:
    int fib(int N) {
        if (N <= 1) return N;
        vector<int> dp(N + 1);
        dp[0] = 0;
        dp[1] = 1;
        for (int i = 2; i <= N; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[N];
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 70. 爬楼梯

```
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？
注意：给定 n 是一个正整数。

示例 1：
输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1 阶 + 1 阶
2 阶
```

```
1.确定dp数组以及下标的含义
dp[i]： 爬到第i层楼梯，有dp[i]种方法

2.确定递推公式
首先是dp[i - 1]，上i-1层楼梯，有dp[i - 1]种方法，那么再一步跳一个台阶不就是dp[i]了么。
还有就是dp[i - 2]，上i-2层楼梯，有dp[i - 2]种方法，那么再一步跳两个台阶不就是dp[i]了么。
那么dp[i]就是 dp[i - 1]与dp[i - 2]之和！

3.dp数组如何初始化
dp[1] = 1，dp[2] = 2

4.确定遍历顺序
dp[i] = dp[i - 1] + dp[i - 2];中可以看出，遍历顺序一定是从前向后遍历的

5.举例推导dp数组
n = 5 dp = {1, 2, 3, 5, 8}
```

```cpp
class Solution {
public:
    int climbStairs(int n) {
        if (n <= 1) return n; // 因为下面直接对dp[2]操作了，防止空指针
        vector<int> dp(n + 1);
        dp[1] = 1;
        dp[2] = 2;
        for (int i = 3; i <= n; i++) { // 注意i是从3开始的
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        return dp[n];
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 746. 使用最小花费爬楼梯

```
给你一个整数数组 cost ，其中 cost[i] 是从楼梯第 i 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。
你可以选择从下标为 0 或下标为 1 的台阶开始爬楼梯。
请你计算并返回达到楼梯顶部的最低花费。

示例 1：

输入：cost = [10, 15, 20]
输出：15
解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。

cost 的长度范围是 [2, 1000]。
cost[i] 将会是一个整型数据，范围为 [0, 999] 。
```

```
1.确定dp数组以及下标的含义
到达第i台阶所花费的最少体力为dp[i]。

2.确定递推公式
可以有两个途径得到dp[i]，一个是dp[i-1] 一个是dp[i-2]。
dp[i - 1] 跳到 dp[i] 需要花费 dp[i - 1] + cost[i - 1]。
dp[i - 2] 跳到 dp[i] 需要花费 dp[i - 2] + cost[i - 2]。
那么究竟是选从dp[i - 1]跳还是从dp[i - 2]跳呢？
一定是选最小的，所以dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);

3.dp数组如何初始化
dp[0] = 0，dp[1] = 0;

4.确定遍历顺序
而且dp[i]由dp[i-1]dp[i-2]推出，所以是从前到后遍历cost数组就可以了。

5.举例推导dp数组
cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1] ，来模拟一下dp数组的状态变化，如下：
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20221026175104.png)

```cpp
class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        vector<int> dp(cost.size() + 1);
        dp[0] = 0; // 默认第一步都是不花费体力的
        dp[1] = 0;
        for (int i = 2; i <= cost.size(); i++) {
            dp[i] = min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
        }
        return dp[cost.size()];
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 62.不同路径

```
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。
问总共有多少条不同的路径？
1 <= m, n <= 100
题目数据保证答案小于等于 2 * 10^9

示例1:
输入：m = 3, n = 7
输出：28
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210110174033215.png)

```
动态规划五部曲：
1.确定dp数组（dp table）以及下标的含义
dp[i][j] ：表示从（0 ，0）出发，到(i, j) 有dp[i][j]条不同的路径。

2.确定递推公式
dp[i][j] = dp[i - 1][j] + dp[i][j - 1]，因为dp[i][j]只有这两个方向过来。

3.dp数组的初始化
首先dp[i][0]一定都是1，因为从(0, 0)的位置到(i, 0)的路径只有一条，那么dp[0][j]也同理。

4.确定遍历顺序
这里要看一下递推公式dp[i][j] = dp[i - 1][j] + dp[i][j - 1]，dp[i][j]都是从其上方和左方推导而来，那么从左到右一层一层遍历就可以了。
这样就可以保证推导dp[i][j]的时候，dp[i - 1][j] 和 dp[i][j - 1]一定是有数值的。

5.举例推导dp数组
```

![62.不同路径1](https://code-thinking-1253855093.file.myqcloud.com/pics/20201209113631392.png)

```cpp
class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n, 0));
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
};
时间复杂度：O(m × n)
空间复杂度：O(m × n)
```

### 63. 不同路径 II

```
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。
现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？
网格中的障碍物和空位置分别用 1 和 0 来表示。
m == obstacleGrid.length
n == obstacleGrid[i].length
1 <= m, n <= 100
obstacleGrid[i][j] 为 0 或 1
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210111204939971.png)

```
输入：obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]]
输出：2 解释：
3x3 网格的正中间有一个障碍物。
从左上角到右下角一共有 2 条不同的路径：
向右 -> 向右 -> 向下 -> 向下
向下 -> 向下 -> 向右 -> 向右
```

```
动规五部曲：
1.确定dp数组（dp table）以及下标的含义
dp[i][j] ：表示从（0 ，0）出发，到(i, j) 有dp[i][j]条不同的路径。

2.确定递推公式
dp[i][j] = dp[i - 1][j] + dp[i][j - 1]。
但这里需要注意一点，因为有了障碍，(i, j)如果就是障碍的话应该就保持初始状态（初始状态为0）。

3.dp数组如何初始化
因为从(0, 0)的位置到(i, 0)的路径只有一条，所以dp[i][0]一定为1，dp[0][j]也同理。
但如果(i, 0) 这条边有了障碍之后，障碍之后（包括障碍）都是走不到的位置了，所以障碍之后的dp[i][0]应该还是初始值0。

4.确定遍历顺序
从递归公式dp[i][j] = dp[i - 1][j] + dp[i][j - 1] 中可以看出，一定是从左到右一层一层遍历，这样保证推导dp[i][j]的时候，dp[i - 1][j] 和 dp[i][j - 1]一定是有数值。

5.举例推导dp数组
```

![63.不同路径II2](https://code-thinking-1253855093.file.myqcloud.com/pics/20210104114610256.png)

```cpp
class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
	if (obstacleGrid[m - 1][n - 1] == 1 || obstacleGrid[0][0] == 1) //如果在起点或终点出现了障碍，直接返回0
            return 0;
        vector<vector<int>> dp(m, vector<int>(n, 0));
        for (int i = 0; i < m && obstacleGrid[i][0] == 0; i++) dp[i][0] = 1;
        for (int j = 0; j < n && obstacleGrid[0][j] == 0; j++) dp[0][j] = 1;
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (obstacleGrid[i][j] == 1) continue;
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```

### 343. 整数拆分

```
给定一个正整数 n，将其拆分为至少两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积。

示例 1:
输入: 2
输出: 1
解释: 2 = 1 + 1, 1 × 1 = 1。
```

```
动态规划五部曲：
1.确定dp数组（dp table）以及下标的含义
dp[i]：分拆数字i，可以得到的最大乘积为dp[i]。

2.确定递推公式
其实可以从1遍历j，然后有两种渠道得到dp[i].（想象最后一次拆分，总要拆一个数字出来）
一个是j * (i - j) 直接相乘。
一个是j * dp[i - j]，相当于是拆分(i - j)。
递推公式：dp[i] = max(dp[i], max((i - j) * j, dp[i - j] * j));
由于遍历j过程中每次都会计算你dp[i]，所以要取一个最大值

3.dp的初始化
初始化dp[2] = 1，从dp[i]的定义来说，拆分数字2，得到的最大乘积是1

4.确定遍历顺序
dp[i] 是依靠 dp[i - j]的状态，所以遍历i一定是从前向后遍历，先有dp[i - j]再有dp[i]。
对于j来说，要从1开始，到i - 2,因为这样才保证dp[i - j]有确定性意义

5.举例推导dp数组
n = 10时
```

![343.整数拆分](https://code-thinking-1253855093.file.myqcloud.com/pics/20210104173021581.png)

```cpp
class Solution {
public:
    int integerBreak(int n) {
        vector<int> dp(n + 1);
        dp[2] = 1;
        for (int i = 3; i <= n ; i++) {
            for (int j = 1; j <= i / 2; j++) {
                dp[i] = max(dp[i], max((i - j) * j, dp[i - j] * j));
            }
        }
        return dp[n];
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)
```

### 96.不同的二叉搜索树

给定一个整数 n，求以 1 ... n 为节点组成的二叉搜索树有多少种？

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210113161941835.png)

了解了二叉搜索树之后，我们应该先举几个例子，画画图，看看有没有什么规律，如图：

![96.不同的二叉搜索树](https://code-thinking-1253855093.file.myqcloud.com/pics/20210107093106367.png)

![96.不同的二叉搜索树1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210107093129889.png)

n 为 1 的时候有一棵树，n 为 2 有两棵树，这个是很直观的。

来看看 n 为 3 的时候，有哪几种情况。

当 1 为头结点的时候，其右子树有两个节点，看这两个节点的布局，是不是和 n 为 2 的时候两棵树的布局是一样的啊！

**（可能有同学问了，这布局不一样啊，节点数值都不一样。别忘了我们就是求不同树的数量，并不用把搜索树都列出来，所以不用关心其具体数值的差异）**

当 3 为头结点的时候，其左子树有两个节点，看这两个节点的布局，是不是和 n 为 2 的时候两棵树的布局也是一样的啊！

当 2 为头结点的时候，其左右子树都只有一个节点，布局是不是和 n 为 1 的时候只有一棵树的布局也是一样的啊！

发现到这里，其实我们就找到了重叠子问题了，其实也就是发现可以通过 dp[1] 和 dp[2] 来推导出来 dp[3]的某种方式。

思考到这里，这道题目就有眉目了。

dp[3]，就是 元素 1 为头结点搜索树的数量 + 元素 2 为头结点搜索树的数量 + 元素 3 为头结点搜索树的数量

元素 1 为头结点搜索树的数量 = 右子树有 2 个元素的搜索树数量 \* 左子树有 0 个元素的搜索树数量

元素 2 为头结点搜索树的数量 = 右子树有 1 个元素的搜索树数量 \* 左子树有 1 个元素的搜索树数量

元素 3 为头结点搜索树的数量 = 右子树有 0 个元素的搜索树数量 \* 左子树有 2 个元素的搜索树数量

有 2 个元素的搜索树数量就是 dp[2]。

有 1 个元素的搜索树数量就是 dp[1]。

有 0 个元素的搜索树数量就是 dp[0]。

所以 dp[3] = dp[2] _ dp[0] + dp[1] _ dp[1] + dp[0] \* dp[2]

![96.不同的二叉搜索树2](https://code-thinking-1253855093.file.myqcloud.com/pics/20210107093226241.png)

```
动态规划五部曲：
1.确定dp数组（dp table）以及下标的含义
dp[i] ： 1到i为节点组成的二叉搜索树的个数为dp[i]。

2.确定递推公式
在上面的分析中，其实已经看出其递推关系， dp[i] += dp[以j为头结点左子树节点数量] * dp[以j为头结点右子树节点数量]
j相当于是头结点的元素，从1遍历到i为止。
所以递推公式：dp[i] += dp[j - 1] * dp[i - j]; ，j-1 为j为头结点左子树节点数量，i-j 为以j为头结点右子树节点数量

3.dp数组如何初始化
初始化，只需要初始化dp[0]就可以了，推导的基础，都是dp[0]。
那么dp[0]应该是多少呢？
从定义上来讲，空节点也是一棵二叉树，也是一棵二叉搜索树，这是可以说得通的。
从递归公式上来讲，dp[以j为头结点左子树节点数量] * dp[以j为头结点右子树节点数量] 中以j为头结点左子树节点数量为0，也需要dp[以j为头结点左子树节点数量] = 1， 否则乘法的结果就都变成0了。
所以初始化dp[0] = 1

4.确定遍历顺序
首先一定是遍历节点数，从递归公式：dp[i] += dp[j - 1] * dp[i - j]可以看出，节点数为i的状态是依靠 i之前节点数的状态。
那么遍历i里面每一个数作为头结点的状态，用j来遍历。

5.举例推导dp数组
n为5时候的dp数组状态如图：（自己画的话，n = 3就好了）
```

![96.不同的二叉搜索树3](https://code-thinking-1253855093.file.myqcloud.com/pics/20210107093253987.png)

```cpp
class Solution {
public:
    int numTrees(int n) {
        vector<int> dp(n + 1);
        dp[0] = 1;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= i; j++) {
                dp[i] += dp[j - 1] * dp[i - j];
            }
        }
        return dp[n];
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)
```

### 动态规划：01 背包理论基础

对于面试的话，其实掌握 01 背包，和完全背包，就够用了，最多可以再来一个多重背包。

![416.分割等和子集1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210117171307407.png)

而完全背包又是也是 01 背包稍作变化而来，即：完全背包的物品数量是无限的。

**所以背包问题的理论基础重中之重是 01 背包，一定要理解透！**

01 背包有很多的变体，将问题转换成 01 背包后就好解很多了

#### 01 背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，得到的价值是 value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

![动态规划-背包问题](https://code-thinking-1253855093.file.myqcloud.com/pics/20210117175428387.jpg)

```
暴力解法：
每一件物品其实只有两个状态，取或者不取，所以可以使用回溯法搜索出所有的情况，那么时间复杂度就是$o(2^n)$，这里的n表示物品数量。
参数：物品重量数组 + 价值数组 + 当前物品总体积 + 当前物品总价值
```

```
二维dp数组01背包
1.确定dp数组以及下标的含义
dp[i][j] 表示从下标为[0-i]的物品里任意取，放进容量为j的背包，价值总和最大是多少。

2.确定递推公式
第i个物品只有选与不选两种情况：
	不放物品i：由dp[i - 1][j]推出，即背包容量为j，里面不放物品i的最大价值，此时dp[i][j]就是dp[i - 1][j]。(其实就是当物品i的重量大于背包j的重量时，物品i无法放进背包中，所以背包内的价值依然和前面相同。)

	放物品i：由dp[i - 1][j - weight[i]]推出，dp[i - 1][j - weight[i]] 为背包容量为j - weight[i]的时候不放物品i的最大价值，那么dp[i - 1][j - weight[i]] + value[i] （物品i的价值），就是背包放物品i得到的最大价值

dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);

3.dp数组如何初始化
如果背包容量j为0的话，即dp[i][0]，无论是选取哪些物品，背包价值总和一定为0。
dp[0][j]，即：i为0，存放编号0的物品的时候，各个容量的背包所能存放的最大价值。当j < weight[0]时，dp[0][j] = 0. 否则dp[0][j] = value[0]

4.确定遍历顺序
先遍历物品，或者先遍历背包都可以，因为遍历顺序都满足左上角的dp[i][j]被填写完成

5.举例推导dp数组


题目：
小明是一位科学家，他需要参加一场重要的国际科学大会，以展示自己的最新研究成果。他需要带一些研究材料，但是他的行李箱空间有限。这些研究材料包括实验设备、文献资料和实验样本等等，它们各自占据不同的空间，并且具有不同的价值。

小明的行李空间为 N，问小明应该如何抉择，才能携带最大价值的研究材料，每种研究材料只能选择一次，并且只有选与不选两种选择，不能进行切割。

输入描述
第一行包含两个正整数，第一个整数 M 代表研究材料的种类，第二个正整数 N，代表小明的行李空间。
第二行包含 M 个正整数，代表每种研究材料的所占空间。
第三行包含 M 个正整数，代表每种研究材料的价值。

输出描述
输出一个整数，代表小明能够携带的研究材料的最大价值。
输入示例
6 1
2 2 3 1 5 2
2 3 1 5 4 3
输出示例
5

提示信息
小明能够携带 6 种研究材料，但是行李空间只有 1，而占用空间为 1 的研究材料价值为 5，所以最终答案输出 5。

数据范围：
1 <= N <= 5000
1 <= M <= 5000
研究材料占用空间和价值都小于等于 1000
```

```cpp
//二维dp数组实现
#include <bits/stdc++.h>
using namespace std;

int n, bagweight;// bagweight代表行李箱空间
void solve() {
    vector<int> weight(n, 0); // 存储每件物品所占空间
    vector<int> value(n, 0);  // 存储每件物品价值
    for(int i = 0; i < n; ++i) {
        cin >> weight[i];
    }
    for(int j = 0; j < n; ++j) {
        cin >> value[j];
    }
    // dp数组, dp[i][j]代表行李箱空间为j的情况下,从下标为[0, i]的物品里面任意取,能达到的最大价值
    vector<vector<int>> dp(weight.size(), vector<int>(bagweight + 1, 0));

    // 初始化, 因为需要用到dp[i - 1]的值
    // j < weight[0]已在上方被初始化为0
    // j >= weight[0]的值就初始化为value[0]
    for (int j = weight[0]; j <= bagweight; j++) {
        dp[0][j] = value[0];
    }

    for(int i = 1; i < weight.size(); i++) { // 遍历科研物品
        for(int j = 0; j <= bagweight; j++) { // 遍历行李箱容量
            // 如果装不下这个物品,那么就继承dp[i - 1][j]的值
            if (j < weight[i]) dp[i][j] = dp[i - 1][j];
            // 如果能装下,就将值更新为 不装这个物品的最大值 和 装这个物品的最大值 中的 最大值
            // 装这个物品的最大值由容量为j - weight[i]的包任意放入序号为[0, i - 1]的最大值 + 该物品的价值构成
            else dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
        }
    }
    cout << dp[weight.size() - 1][bagweight] << endl;
}

int main() {
    while(cin >> n >> bagweight) {
        solve();
    }
    return 0;
}
```

### 动态规划：01 背包理论基础（滚动数组）

**通过滚动数组，可以将二维 dp 数组降至一维**

```
在使用二维数组的时候，递推公式：dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i]);
其实可以发现如果把dp[i - 1]那一层拷贝到dp[i]上，表达式完全可以是：dp[i][j] = max(dp[i][j], dp[i][j - weight[i]] + value[i]);
与其把dp[i - 1]这一层拷贝到dp[i]上，不如只用一个一维数组了，只用dp[j]（一维数组，也可以理解是一个滚动数组）。
这就是滚动数组的由来，需要满足的条件是上一层可以重复利用，直接拷贝到当前层。
```

```
1.确定dp数组的定义
dp[j]表示：容量为j的背包，所背的物品价值可以最大为dp[j]。

2.一维dp数组的递推公式
dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);

3.一维dp数组如何初始化
dp的每一个位置都应该是0，因为dp[0] = 0,其他位置靠dp[0]去更新，但物品价值不会为负，取最大的情况下，初始化应该为0

4.一维dp数组遍历顺序
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
倒序遍历是为了保证物品i只被放入一次！。从后往前循环，每次取得状态不会和之前取得状态重合，这样每种物品就只取一次了。

不可以先遍历容量，再遍历物品。（可以从二维dp压缩至一维dp的角度考虑，先倒序遍历容量，这样等价于从二维dp考虑，先算右下角，再算左上角，不合理的）
```

### 416. 分割等和子集

```
给定一个只包含正整数的非空数组。是否可以将这个数组分割成两个子集，使得两个子集的元素和相等。
注意: 每个数组中的元素不会超过 100 数组的大小不会超过 200

示例 1:
输入: [1, 5, 11, 5]
输出: true
解释: 数组可以分割成 [1, 5, 5] 和 [11].

提示：
1 <= nums.length <= 200
1 <= nums[i] <= 100
```

```
本题要求集合里能否出现总和为 sum / 2 的子集。

那么来一一对应一下本题，看看背包问题如何来解决。
只有确定了如下四点，才能把01背包问题套到本题上来。

	背包的体积为sum / 2
	背包要放入的商品（集合里的元素）重量为 元素的数值，价值也为元素的数值
	背包如果正好装满，说明找到了总和为 sum / 2 的子集。
	背包中每一个元素是不可重复放入。

以上分析完，我们就可以套用01背包，来解决这个问题了。
```

```
1.确定dp数组以及下标的含义
dp[j] 表示： 容量为j的背包，所背的物品价值最大可以为dp[j]。
本题中每一个元素的数值既是重量，也是价值。
套到本题，dp[j]表示 背包总容量（所能装的总重量）是j，放进物品后，背的最大重量为dp[j]。

2.确定递推公式
dp[j] = max(dp[j], dp[j - nums[i]] + nums[i]);

3.dp数组如何初始化
dp[0]一定是0。其他位置初始也是0.保证最大值更新
数组容量：看一下总和sum最大可能是多少：200 * 100 sum / 2最大为：10000

4.确定遍历顺序
如果使用一维dp数组，物品遍历的for循环放在外层，遍历背包的for循环放在内层，且内层for循环倒序遍历！

5.举例推导dp数组
```

![416.分割等和子集2](https://code-thinking-1253855093.file.myqcloud.com/pics/20210110104240545.png)

```cpp
class Solution {
public:
    bool canPartition(vector<int>& nums) {
        int sum = 0;

        // dp[i]中的i表示背包内总和
        // 题目中说：每个数组中的元素不会超过 100，数组的大小不会超过 200
        // 总和不会大于20000，背包最大只需要其中一半，所以10001大小就可以了
        vector<int> dp(10001, 0);
        for (int i = 0; i < nums.size(); i++) {
            sum += nums[i];
        }
        // 也可以使用库函数一步求和
        // int sum = accumulate(nums.begin(), nums.end(), 0);
        if (sum % 2 == 1) return false;
        int target = sum / 2;

        // 开始 01背包
        for(int i = 0; i < nums.size(); i++) {
            for(int j = target; j >= nums[i]; j--) { // 每一个元素一定是不可重复放入，所以从大到小遍历
                dp[j] = max(dp[j], dp[j - nums[i]] + nums[i]);
            }
        }
        // 集合中的元素正好可以凑成总和target
        if (dp[target] == target) return true;
        return false;
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)，虽然dp数组大小为一个常数，但是大常数
```

### 1049.最后一块石头的重量 II

```
有一堆石头，每块石头的重量都是正整数。
每一回合，从中选出任意两块石头，然后将它们一起粉碎。假设石头的重量分别为 x 和 y，且 x <= y。那么粉碎的可能结果如下：
如果 x == y，那么两块石头都会被完全粉碎；
如果 x != y，那么重量为 x 的石头将会完全粉碎，而重量为 y 的石头新重量为 y-x。
最后，最多只会剩下一块石头。返回此石头最小的可能重量。如果没有石头剩下，就返回 0。

示例：
输入：[2,7,4,1,8,1]
输出：1
解释：
组合 2 和 4，得到 2，所以数组转化为 [2,7,1,8,1]，
组合 7 和 8，得到 1，所以数组转化为 [2,1,1,1]，
组合 2 和 1，得到 1，所以数组转化为 [1,1,1]，
组合 1 和 1，得到 0，所以数组转化为 [1]，这就是最优值。

提示：
1 <= stones.length <= 30
1 <= stones[i] <= 1000
```

```
难点在于怎么将这个问题和01背包结合起来

如果最开始我们就将石头分为两堆，然后从这两堆每次都随便取一块出来粉碎，如果石头还活着，就放回原来的堆。直到其中一堆没有石头了，那么剩下的那个石头就是本次划分两堆的最终粉碎结果。

可以看到，其实最终粉碎结果就是两堆总重量之差。所以要最终粉碎结果最小，那么最开始两堆总重量之差就该越小。那么最理想的情况就是sum / 2.

那么我们的目标就是从原来的堆里选石头，让他努力迫近sum / 2。这就是01背包了
```

```cpp
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        vector<int> dp(15001, 0);
        int sum = 0;
        for (int i = 0; i < stones.size(); i++) sum += stones[i];
        int target = sum / 2;
        for (int i = 0; i < stones.size(); i++) { // 遍历物品
            for (int j = target; j >= stones[i]; j--) { // 遍历背包
                dp[j] = max(dp[j], dp[j - stones[i]] + stones[i]);
            }
        }
        return sum - dp[target] - dp[target];
    }
};
时间复杂度：O(m × n) , m是石头总重量（准确的说是总重量的一半），n为石头块数
空间复杂度：O(m)
```

```cpp
//二维数组版
class Solution {
public:
    int lastStoneWeightII(vector<int>& stones) {
        int sum = 0;
        int n = stones.size();
        for (int i = 0; i < n; i++) {
            sum += stones[i];
        }
        int target = sum / 2;
        vector<vector<int>> dp(n, vector<int>(target + 1, 0));
        for (int j = 0; j <= target; j++) {
            if (stones[0] <= j) {
                dp[0][j] = stones[0];
            }
        }
        for (int i = 1; i < n; i++) {
            for (int j = 1; j <= target; j++) {
                if (j < stones[i]) {
                    dp[i][j] = dp[i - 1][j];
                }else {
                    dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - stones[i]] + stones[i]);
                }
            }
        }
        return sum - dp[n - 1][target] * 2;
    }
};
```

### 494.目标和

```
给定一个非负整数数组，a1, a2, ..., an, 和一个目标数S。现在你有两个符号 + 和 -。对于数组中的任意一个整数，你都可以从 + 或 -中选择一个符号添加在前面。
返回可以使最终数组和为目标数 S 的所有添加符号的方法数。

示例：

输入：nums: [1, 1, 1, 1, 1], S: 3
输出：5
解释：
-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3
一共有5种方法让最终目标和为3。

提示：
数组非空，且长度不会超过 20 。
初始的数组的和不会超过 1000 。
保证返回的最终结果能被 32 位整数存下。
```

```cpp
本题也是01背包的变体
对于每个数组元素只有两种选择，要么是+ 要么是-
所以可以分为两组left right
由于最终要使得left - right = S  并且我们有left + right = sum
所以联立解方程为 left = (S + sum) / 2

现在的目标就是在原数组里面找集合，使他们的和为left的方案数
也就是背包容量为(S + sum) / 2,找出使得背包刚好装满的方法数
```

```
动态规划五部曲：
1.确定dp数组以及下标的含义
dp[j] 表示：填满j（包括j）这么大容积的包，有dp[j]种方法

2.确定递推公式
只要搞到nums[i]，凑成dp[j]就有dp[j - nums[i]] 种方法。
例如：dp[j]，j 为5，
已经有一个1（nums[i]） 的话，有 dp[4]种方法 凑成 容量为5的背包。
已经有一个2（nums[i]） 的话，有 dp[3]种方法 凑成 容量为5的背包。
已经有一个3（nums[i]） 的话，有 dp[2]中方法 凑成 容量为5的背包
已经有一个4（nums[i]） 的话，有 dp[1]中方法 凑成 容量为5的背包
已经有一个5 （nums[i]）的话，有 dp[0]中方法 凑成 容量为5的背包
那么凑整dp[5]有多少方法呢，也就是把 所有的 dp[j - nums[i]] 累加起来。
所以求组合类问题的公式，都是类似这种：
dp[j] += dp[j - nums[i]]

3.dp数组如何初始化
dp[0] 一定要初始化为1，因为dp[0]是在公式中一切递推结果的起源，如果dp[0]是0的话，递推结果将都是0。dp[j]其他下标对应的数值也应该初始化为0，从递推公式也可以看出，dp[j]要保证是0的初始值，才能正确的由dp[j - nums[i]]推导出来。

4.确定遍历顺序
我们讲过对于01背包问题一维dp的遍历，nums放在外循环，target在内循环，且内循环倒序。

5.举例推导dp数组
输入：nums: [1, 1, 1, 1, 1], S: 3
bagSize = (S + sum) / 2 = (3 + 5) / 2 = 4
dp数组状态变化如下：
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210125120743274.jpg)

```cpp
class Solution {
public:
    int findTargetSumWays(vector<int>& nums, int S) {
        int sum = 0;
        for (int i = 0; i < nums.size(); i++) sum += nums[i];
        if (abs(S) > sum) return 0; // 此时没有方案
        if ((S + sum) % 2 == 1) return 0; // 此时没有方案
        int bagSize = (S + sum) / 2;
        vector<int> dp(bagSize + 1, 0);
        dp[0] = 1;
        for (int i = 0; i < nums.size(); i++) {
            for (int j = bagSize; j >= nums[i]; j--) {
                dp[j] += dp[j - nums[i]];
            }
        }
        return dp[bagSize];
    }
};
时间复杂度：O(n × m)，n为正数个数，m为背包容量
空间复杂度：O(m)，m为背包容量
注意两个判断
    abs(S) > sum 说明S已经在所有可组合的情况之外了
    (S + sum) % 2 == 1 背包必须恰好装满，所以必须是偶数
```

### 474.一和零

```
给你一个二进制字符串数组 strs 和两个整数 m 和 n 。
请你找出并返回 strs 的最大子集的大小，该子集中 最多 有 m 个 0 和 n 个 1 。
如果 x 的所有元素也是 y 的元素，集合 x 是集合 y 的 子集 。

示例 1：
输入：strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3
输出：4
解释：最多有 5 个 0 和 3 个 1 的最大子集是 {"10","0001","1","0"} ，因此答案是 4 。 其他满足题意但较小的子集包括 {"0001","1"} 和 {"10","1","0"} 。{"111001"} 不满足题意，因为它含 4 个 1 ，大于 n 的值 3 。

示例 2：
输入：strs = ["10", "0", "1"], m = 1, n = 1
输出：2
解释：最大的子集是 {"0", "1"} ，所以答案是 2 。

提示：
1 <= strs.length <= 600
1 <= strs[i].length <= 100
strs[i] 仅由 '0' 和 '1' 组成
1 <= m, n <= 100
```

```
本题中strs 数组里的元素就是物品，每个物品都是一个！
而m 和 n相当于是一个背包，两个维度的背包。

1.确定dp数组（dp table）以及下标的含义
dp[i][j]：最多有i个0和j个1的strs的最大子集的大小为dp[i][j]。

2.确定递推公式
dp[i][j] 可以由前一个strs里的字符串推导出来，strs里的字符串有zeroNum个0，oneNum个1。
dp[i][j] 就可以是 dp[i - zeroNum][j - oneNum] + 1。
然后我们在遍历的过程中，取dp[i][j]的最大值。
所以递推公式：dp[i][j] = max(dp[i][j], dp[i - zeroNum][j - oneNum] + 1);
此时大家可以回想一下01背包的递推公式：dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
对比一下就会发现，字符串的zeroNum和oneNum相当于物品的重量（weight[i]），字符串本身的个数相当于物品的价值（value[i]）。
这就是一个典型的01背包！ 只不过物品的重量有了两个维度而已。

3.dp数组如何初始化
因为物品价值不会是负数，初始为0，保证递推的时候dp[i][j]不会被初始值覆盖。

4.确定遍历顺序
01背包为什么一定是外层for循环遍历物品，内层for循环遍历背包容量且从后向前遍历！
物品就是strs里的字符串，背包容量就是题目描述中的m和n。

5.举例推导dp数组
以输入：["10","0001","111001","1","0"]，m = 3，n = 3为例
```

![474.一和零](https://code-thinking-1253855093.file.myqcloud.com/pics/20210120111201512.jpg)

```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        vector<vector<int>> dp(m + 1, vector<int> (n + 1, 0)); // 默认初始化0
        for (string str : strs) { // 遍历物品
            int oneNum = 0, zeroNum = 0;
            for (char c : str) {
                if (c == '0') zeroNum++;
                else oneNum++;
            }
            for (int i = m; i >= zeroNum; i--) { // 遍历背包容量且从后向前遍历！
                for (int j = n; j >= oneNum; j--) {
                    dp[i][j] = max(dp[i][j], dp[i - zeroNum][j - oneNum] + 1);
                }
            }
        }
        return dp[m][n];
    }
};
时间复杂度: O(kmn)，k 为strs的长度
空间复杂度: O(mn)
```

### 完全背包理论基础

```cpp
有N件物品和一个最多能背重量为W的背包。第i件物品的重量是weight[i]，得到的价值是value[i] 。每件物品都有无限个（也就是可以放入背包多次），求解将哪些物品装入背包里物品价值总和最大。
完全背包和01背包问题唯一不同的地方就是，每种物品有无限件。
01背包和完全背包唯一不同就是体现在遍历顺序上，所以本文就不去做动规五部曲了，我们直接针对遍历顺序经行分析！

我们知道01背包内嵌的循环是从大到小遍历，为了保证每个物品仅被添加一次。
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
而完全背包的物品是可以添加多次的，所以要从小到大去遍历
for(int i = 0; i < weight.size(); i++) { // 遍历物品
    for(int j = weight[i]; j <= bagWeight ; j++) { // 遍历背包容量
        dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
    }
}
```

```cpp
#include <iostream>
#include <vector>
using namespace std;

// 先遍历背包，再遍历物品
void test_CompletePack(vector<int> weight, vector<int> value, int bagWeight) {

    vector<int> dp(bagWeight + 1, 0);
	for(int i = 0; i < weight.size(); i++) { // 遍历物品
        for(int j = weight[i]; j <= bagWeight; j++) { // 遍历背包容量
            dp[j] = max(dp[j], dp[j - weight[i]] + value[i]);
        }
    }
    cout << dp[bagWeight] << endl;
}
int main() {
    int N, V;
    cin >> N >> V;
    vector<int> weight;
    vector<int> value;
    for (int i = 0; i < N; i++) {
        int w;
        int v;
        cin >> w >> v;
        weight.push_back(w);
        value.push_back(v);
    }
    test_CompletePack(weight, value, V);
    return 0;
}
```

### 518.零钱兑换 II

```
给定不同面额的硬币和一个总金额。写出函数来计算可以凑成总金额的硬币组合数。假设每一种面额的硬币有无限个。

示例 1:
输入: amount = 5, coins = [1, 2, 5]
输出: 4
解释: 有四种方式可以凑成总金额:
5=5
5=2+2+1
5=2+1+1+1
5=1+1+1+1+1

注意，你可以假设：
0 <= amount (总金额) <= 5000
1 <= coin (硬币面额) <= 5000
硬币种类不超过 500 种
结果符合 32 位符号整数
```

```
硬币数量不限制，说明是完全背包问题
问凑成总金额的组合数，说明背包必须刚好满

1.确定dp数组以及下标的含义
dp[j]：凑成总金额j的货币组合数为dp[j]

2.确定递推公式
dp[j] 就是所有的dp[j - coins[i]]（考虑coins[i]的情况）相加。
所以递推公式：dp[j] += dp[j - coins[i]];

3.dp数组如何初始化
dp[0]一定要为1，dp[0] = 1是 递归公式的基础。如果dp[0] = 0 的话，后面所有推导出来的值都是0了。
下标非0的dp[j]初始化为0，这样累计加dp[j - coins[i]]的时候才不会影响真正的dp[j]

4.确定遍历顺序
对于外层for循环遍历物品（钱币），内层for遍历背包（金钱总额）的情况。
for (int i = 0; i < coins.size(); i++) { // 遍历物品
    for (int j = coins[i]; j <= amount; j++) { // 遍历背包容量
        dp[j] += dp[j - coins[i]];
    }
}
假设：coins[0] = 1，coins[1] = 5。
那么就是先把1加入计算，然后再把5加入计算，得到的方法数量只有{1, 5}这种情况。而不会出现{5, 1}的情况。
所以这种遍历顺序中dp[j]里计算的是组合数！

如果反过来
for (int j = 0; j <= amount; j++) { // 遍历背包容量
    for (int i = 0; i < coins.size(); i++) { // 遍历物品
        if (j - coins[i] >= 0) dp[j] += dp[j - coins[i]];
    }
}
背包容量的每一个值，都是经过 1 和 5 的计算，包含了{1, 5} 和 {5, 1}两种情况。
此时dp[j]里算出来的就是排列数！

5.举例推导dp数组
输入: amount = 5, coins = [1, 2, 5] ，dp状态图如下：
```

![518.零钱兑换II](https://code-thinking-1253855093.file.myqcloud.com/pics/20210120181331461.jpg)

```cpp
class Solution {
public:
    int change(int amount, vector<int>& coins) {
        vector<int> dp(amount + 1, 0);
        dp[0] = 1;
        for (int i = 0; i < coins.size(); i++) { // 遍历物品
            for (int j = coins[i]; j <= amount; j++) { // 遍历背包
                dp[j] += dp[j - coins[i]];
            }
        }
        return dp[amount];
    }
};
时间复杂度: O(mn)，其中 m 是amount，n 是 coins 的长度
空间复杂度: O(m)
```

### 377. 组合总和 Ⅳ

```
给定一个由正整数组成且不存在重复数字的数组，找出和为给定目标正整数的组合的个数。

示例:
nums = [1, 2, 3]
target = 4
所有可能的组合为： (1, 1, 1, 1) (1, 1, 2) (1, 2, 1) (1, 3) (2, 1, 1) (2, 2) (3, 1)

请注意，顺序不同的序列被视作不同的组合。
因此输出为 7。
```

```
本题其实就是完全背包问题：背包大小为target,每个物品可以重复选择
同时选择的顺序会对结果造成影响

1.确定dp数组以及下标的含义
dp[i]: 凑成目标正整数为i的排列个数为dp[i]

2.确定递推公式
dp[i]（考虑nums[j]）可以由 dp[i - nums[j]]（不考虑nums[j]） 推导出来。
因为只要得到nums[j]，排列个数dp[i - nums[j]]，就是dp[i]的一部分。
递推公式是dp[i] += dp[i - nums[j]];

3.dp数组如何初始化
因为递推公式dp[i] += dp[i - nums[j]]的缘故，dp[0]要初始化为1，这样递归其他dp[i]的时候才会有数值基础。其他非0下标dp[i] = 0

4.确定遍历顺序
如果求组合数就是外层for循环遍历物品，内层for遍历背包。
如果求排列数就是外层for遍历背包，内层for循环遍历物品。

5.举例来推导dp数组
```

![377.组合总和Ⅳ](https://code-thinking-1253855093.file.myqcloud.com/pics/20230310000625.png)

```cpp
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        vector<int> dp(target + 1, 0);
        dp[0] = 1;
        for (int i = 0; i <= target; i++) { // 遍历背包
            for (int j = 0; j < nums.size(); j++) { // 遍历物品
                if (i - nums[j] >= 0 && dp[i] < INT_MAX - dp[i - nums[j]]) {
                    dp[i] += dp[i - nums[j]];
                }
            }
        }
        return dp[target];
    }
};
时间复杂度: O(target * n)，其中 n 为 nums 的长度
空间复杂度: O(target)
```

### 70. 爬楼梯（进阶版）

```
假设你正在爬楼梯。需要 n 阶你才能到达楼顶。
每次你可以爬至多m (1 <= m < n)个台阶。你有多少种不同的方法可以爬到楼顶呢？
注意：给定 n 是一个正整数。

输入描述：输入共一行，包含两个正整数，分别表示n, m
输出描述：输出一个整数，表示爬到楼顶的方法数。
输入示例：3 2
输出示例：3

提示：
当 m = 2，n = 3 时，n = 3 这表示一共有三个台阶，m = 2 代表你每次可以爬一个台阶或者两个台阶。
此时你有三种方法可以爬到楼顶。
1 阶 + 1 阶 + 1 阶段
1 阶 + 2 阶
2 阶 + 1 阶
```

```
我们之前做的 爬楼梯 是只能至多爬两个台阶。
这次改为：一步一个台阶，两个台阶，三个台阶，.......，直到 m个台阶。问有多少种不同的方法可以爬到楼顶呢？
这又有难度了，这其实是一个完全背包问题。
1阶，2阶，.... m阶就是物品，楼顶就是背包。
每一阶可以重复使用，例如跳了1阶，还可以继续跳1阶。
问跳到楼顶有几种方法其实就是问装满背包有几种方法。
```

```
1.确定dp数组以及下标的含义
dp[i]：爬到有i个台阶的楼顶，有dp[i]种方法。

2.确定递推公式
dp[i] += dp[i - j]

3.dp数组如何初始化
dp[0] 一定为1，dp[0]是递归中一切数值的基础所在，如果dp[0]是0的话，其他数值都是0了。

4.确定遍历顺序
排列问题，外层for循环是背包容量，内层for循环是物品
```

```cpp
#include <iostream>
#include <vector>
using namespace std;
int main() {
    int n, m;
    while (cin >> n >> m) {
        vector<int> dp(n + 1, 0);
        dp[0] = 1;
        for (int i = 1; i <= n; i++) { // 遍历背包
            for (int j = 1; j <= m; j++) { // 遍历物品
                if (i - j >= 0) dp[i] += dp[i - j];
            }
        }
        cout << dp[n] << endl;
    }
}
时间复杂度: O(n * m)
空间复杂度: O(n)
```

### 322. 零钱兑换

```
给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。
你可以认为每种硬币的数量是无限的。

示例 1：
输入：coins = [1, 2, 5], amount = 11
输出：3
解释：11 = 5 + 5 + 1

示例 2：
输入：coins = [2], amount = 3
输出：-1

提示：
1 <= coins.length <= 12
1 <= coins[i] <= 2^31 - 1
0 <= amount <= 10^4
```

```
每种硬币数量都是无限的，所以是完全背包问题
1.确定dp数组以及下标的含义
dp[j]：凑足总额为j所需钱币的最少个数为dp[j]

2.确定递推公式
dp[j] = min(dp[j - coins[i]] + 1, dp[j])

3.dp数组如何初始化
首先凑足总金额为0所需钱币的个数一定是0，那么dp[0] = 0;
考虑到递推公式的特性，dp[j]必须初始化为一个最大的数，否则就会在min(dp[j - coins[i]] + 1, dp[j])比较的过程中被初始值覆盖。

4.确定遍历顺序
外层物品，内层背包，由于是完全背包，所以内层是正序的
这种理解是，我先全用物品0，看结果。然后可用物品多一个物品1，看有没有得减少个数
5.举例推导dp数组
```

![322.零钱兑换](https://code-thinking-1253855093.file.myqcloud.com/pics/20210201111833906.jpg)

```cpp
// 版本一
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 0; i < coins.size(); i++) { // 遍历物品
            for (int j = coins[i]; j <= amount; j++) { // 遍历背包
                if (dp[j - coins[i]] != INT_MAX) { // 如果dp[j - coins[i]]是初始值则跳过
                    dp[j] = min(dp[j - coins[i]] + 1, dp[j]);
                }
            }
        }
        if (dp[amount] == INT_MAX) return -1;
        return dp[amount];
    }
};
时间复杂度: O(n * amount)，其中 n 为 coins 的长度
空间复杂度: O(amount)
```

### 279.完全平方数

```
给定正整数 n，找到若干个完全平方数（比如 1, 4, 9, 16, ...）使得它们的和等于 n。你需要让组成和的完全平方数的个数最少。
给你一个整数 n ，返回和为 n 的完全平方数的 最少数量 。
完全平方数 是一个整数，其值等于另一个整数的平方；换句话说，其值等于一个整数自乘的积。例如，1、4、9 和 16 都是完全平方数，而 3 和 11 不是。

示例 1：
输入：n = 12
输出：3
解释：12 = 4 + 4 + 4

示例 2：
输入：n = 13
输出：2
解释：13 = 4 + 9

提示：
1 <= n <= 10^4
```

```
完全平方数就是物品（可以无限件使用），凑个正整数n就是背包，问凑满这个背包最少有多少物品？

1.确定dp数组（dp table）以及下标的含义
dp[j]：和为j的完全平方数的最少数量为dp[j]

2.确定递推公式
dp[j] = min(dp[j - i * i] + 1, dp[j])

3.dp数组如何初始化
dp[0]表示 和为0的完全平方数的最小数量，那么dp[0]一定是0。
非0索引，其他dp初始化就是最大值

4.确定遍历顺序
非排列，也非组合问题。没有顺序要求

5.举例推导dp数组
```

![279.完全平方数](https://code-thinking-1253855093.file.myqcloud.com/pics/20210202112617341.jpg)

```cpp
class Solution {
public:
    int numSquares(int n) {
        vector<int> dp(n + 1, INT_MAX);
        dp[0] = 0;
        for (int i = 0; i <= n; i++) { // 遍历背包
            for (int j = 1; j * j <= i; j++) { // 遍历物品
                dp[i] = min(dp[i - j * j] + 1, dp[i]);
            }
        }
        return dp[n];
    }
};
时间复杂度: O(n * √n)
空间复杂度: O(n)
```

### 139.单词拆分

```
给定一个非空字符串 s 和一个包含非空单词的列表 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。

说明：
拆分时可以重复使用字典中的单词。
你可以假设字典中没有重复的单词。

示例 1：
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以被拆分成 "leet code"。

示例 2：
输入: s = "applepenapple", wordDict = ["apple", "pen"]
输出: true
解释: 返回 true 因为 "applepenapple" 可以被拆分成 "apple pen apple"。
注意你可以重复使用字典中的单词。
```

```
背包大小就是s的长度，物品就是wordDict的元素，可以无限使用每个元素，所以是完全背包

1.确定dp数组以及下标的含义
dp[i] : 字符串长度为i的话，dp[i]为true，表示可以拆分为一个或多个在字典中出现的单词。

2.确定递推公式
如果确定dp[j] 是true，且 [j, i] 这个区间的子串出现在字典里，那么dp[i]一定是true。（j < i ）。
所以递推公式是 if([j, i] 这个区间的子串出现在字典里 && dp[j]是true) 那么 dp[i] = true。

3.dp数组如何初始化
dp[i] 的状态依靠 dp[j]是否为true，那么dp[0]就是递推的根基，dp[0]一定要为true，否则递推下去后面都都是false了。
下标非0的dp[i]初始化为false，只要没有被覆盖说明都是不可拆分为一个或多个在字典中出现的单词。

4.确定遍历顺序
本题其实我们求的是排列数，为什么呢。 拿 s = "applepenapple", wordDict = ["apple", "pen"] 举例。
"apple", "pen" 是物品，那么我们要求 物品的组合一定是 "apple" + "pen" + "apple" 才能组成 "applepenapple"。
"apple" + "apple" + "pen" 或者 "pen" + "apple" + "apple" 是不可以的，那么我们就是强调物品之间顺序。
所以说，本题一定是 先遍历 背包，再遍历物品。

5.举例推导dp[i]
```

![139.单词拆分](https://code-thinking-1253855093.file.myqcloud.com/pics/20210202162652727.jpg)

```cpp
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> wordSet(wordDict.begin(), wordDict.end());
        vector<bool> dp(s.size() + 1, false);
        dp[0] = true;
        for (int i = 1; i <= s.size(); i++) {   // 遍历背包
            for (int j = 0; j < i; j++) {       // 遍历物品
                string word = s.substr(j, i - j); //substr(起始位置，截取的个数)
                if (wordSet.find(word) != wordSet.end() && dp[j]) {
                    dp[i] = true;
                }
            }
        }
        return dp[s.size()];
    }
};
时间复杂度：O(n^3)，因为substr返回子串的副本是O(n)的复杂度（这里的n是substring的长度）
空间复杂度：O(n)
```

### 多重背包理论基础

```
有N种物品和一个容量为V 的背包。第i种物品最多有Mi件可用，每件耗费的空间是Ci ，价值是Wi 。求解将哪些物品装入背包可使这些物品的耗费的空间 总和不超过背包容量，且价值总和最大。
多重背包和01背包是非常像的， 为什么和01背包像呢？
每件物品最多有Mi件可用，把Mi件摊开，其实就是一个01背包问题了。
```

```cpp
#include<iostream>
#include<vector>
using namespace std;
int main() {
    int bagWeight,n;
    cin >> bagWeight >> n;
    vector<int> weight(n, 0);
    vector<int> value(n, 0);
    vector<int> nums(n, 0);
    for (int i = 0; i < n; i++) cin >> weight[i];
    for (int i = 0; i < n; i++) cin >> value[i];
    for (int i = 0; i < n; i++) cin >> nums[i];

    vector<int> dp(bagWeight + 1, 0);

    for(int i = 0; i < n; i++) { // 遍历物品
        for(int j = bagWeight; j >= weight[i]; j--) { // 遍历背包容量
            // 以上为01背包，然后加一个遍历个数
            for (int k = 1; k <= nums[i] && (j - k * weight[i]) >= 0; k++) { // 遍历个数
                dp[j] = max(dp[j], dp[j - k * weight[i]] + k * value[i]);
            }
        }
    }

    cout << dp[bagWeight] << endl;
}
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/%E8%83%8C%E5%8C%85%E9%97%AE%E9%A2%981.jpeg)

### 198.打家劫舍

```
你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
给定一个代表每个房屋存放金额的非负整数数组，计算你 不触动警报装置的情况下 ，一夜之内能够偷窃到的最高金额。

示例 1：
输入：[1,2,3,1]
输出：4
解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。   偷窃到的最高金额 = 1 + 3 = 4 。

示例 2：
输入：[2,7,9,3,1]
输出：12 解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。   偷窃到的最高金额 = 2 + 9 + 1 = 12 。

提示：
0 <= nums.length <= 100
0 <= nums[i] <= 400
```

```
1.确定dp数组（dp table）以及下标的含义
dp[i]：考虑下标i（包括i）以内的房屋，最多可以偷窃的金额为dp[i]。

2.确定递推公式
dp[i] = max(dp[i - 2] + nums[i], dp[i - 1]);

3.dp数组如何初始化
dp[0] 一定是 nums[0]，dp[1]就是nums[0]和nums[1]的最大值即：dp[1] = max(nums[0], nums[1]);

4.确定遍历顺序
dp[i] 是根据dp[i - 2] 和 dp[i - 1] 推导出来的，那么一定是从前到后遍历！

5.举例推导dp数组
```

![198.打家劫舍](https://code-thinking-1253855093.file.myqcloud.com/pics/20210221170954115.jpg)

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        if (nums.size() == 1) return nums[0];
        vector<int> dp(nums.size());
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < nums.size(); i++) {
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[nums.size() - 1];
    }
};
时间复杂度: O(n)
空间复杂度: O(n)
```

### 213.打家劫舍 II

```
你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。
给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，能够偷窃到的最高金额。

示例 1：
输入：nums = [2,3,2]
输出：3
解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。

示例 2：
输入：nums = [1,2,3,1]
输出：4
解释：你可以先偷窃 1 号房屋（金额 = 1），然后偷窃 3 号房屋（金额 = 3）。偷窃到的最高金额 = 1 + 3 = 4 。

示例 3：
输入：nums = [0]
输出：0

提示：
1 <= nums.length <= 100
0 <= nums[i] <= 1000
```

由于首尾相接了，所以我们分为两种情况考虑：

![213.打家劫舍II1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210129160821374-20230310134003961.jpg)

![213.打家劫舍II2](https://code-thinking-1253855093.file.myqcloud.com/pics/20210129160842491-20230310134008133.jpg)

```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        if (nums.size() == 1) return nums[0];
        int result1 = robRange(nums, 0, nums.size() - 2); // 情况二
        int result2 = robRange(nums, 1, nums.size() - 1); // 情况三
        return max(result1, result2);
    }
    // 198.打家劫舍的逻辑
    int robRange(vector<int>& nums, int start, int end) {
        if (end == start) return nums[start];
        vector<int> dp(nums.size());
        dp[start] = nums[start];
        dp[start + 1] = max(nums[start], nums[start + 1]);
        for (int i = start + 2; i <= end; i++) {
            dp[i] = max(dp[i - 2] + nums[i], dp[i - 1]);
        }
        return dp[end];
    }
};
时间复杂度: O(n)
空间复杂度: O(n)
```

### 337.打家劫舍 III

```
在上次打劫完一条街道之后和一圈房屋后，小偷又发现了一个新的可行窃的地区。这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。
```

![337.打家劫舍III](https://code-thinking-1253855093.file.myqcloud.com/pics/20210223173849619.png)

```cpp
暴力递归：
class Solution {
public:
    unordered_map<TreeNode* , int> umap; // 记录计算过的结果
    int rob(TreeNode* root) {
        if (root == NULL) return 0;
        if (root->left == NULL && root->right == NULL) return root->val;
        if (umap[root]) return umap[root]; // 如果umap里已经有记录则直接返回
        // 偷父节点
        int val1 = root->val;
        if (root->left) val1 += rob(root->left->left) + rob(root->left->right); // 跳过root->left
        if (root->right) val1 += rob(root->right->left) + rob(root->right->right); // 跳过root->right
        // 不偷父节点
        int val2 = rob(root->left) + rob(root->right); // 考虑root的左右孩子
        umap[root] = max(val1, val2); // umap记录一下结果
        return max(val1, val2);
    }
};
时间复杂度：O(n)
空间复杂度：O(log n)，算上递推系统栈的空间
```

```
暴力递归对一个节点 偷与不偷得到的最大金钱都没有做记录，而是需要实时计算。

而动态规划其实就是使用状态转移容器来记录状态的变化，这里可以使用一个长度为2的数组，记录当前节点偷与不偷所得到的的最大金钱。

1.确定递归函数的参数和返回值
这里我们要求一个节点 偷与不偷的两个状态所得到的金钱，那么返回值就是一个长度为2的数组。
参数为当前节点
这里的dp数组就是返回的数组

2.确定终止条件
在遍历的过程中，如果遇到空节点的话，很明显，无论偷还是不偷都是0，所以就返回
这里相当于dp数组的初始化

3.确定遍历顺序
通过递归左节点，得到左节点偷与不偷的金钱。
通过递归右节点，得到右节点偷与不偷的金钱。

4.确定单层递归的逻辑
如果是偷当前节点，那么左右孩子就不能偷，val1 = cur->val + left[0] + right[0]; （如果对下标含义不理解就再回顾一下dp数组的含义）
如果不偷当前节点，那么左右孩子就可以偷，至于到底偷不偷一定是选一个最大的，所以：val2 = max(left[0], left[1]) + max(right[0], right[1]);
最后当前节点的状态就是{val2, val1}; 即：{不偷当前节点得到的最大金钱，偷当前节点得到的最大金钱}

5.举例推导dp数组
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20230203110031.png)

```cpp
class Solution {
public:
    int rob(TreeNode* root) {
        vector<int> result = robTree(root);
        return max(result[0], result[1]);
    }
    // 长度为2的数组，0：不偷，1：偷
    vector<int> robTree(TreeNode* cur) {
        if (cur == NULL) return vector<int>{0, 0};
        vector<int> left = robTree(cur->left);
        vector<int> right = robTree(cur->right);
        // 偷cur，那么就不能偷左右节点。
        int val1 = cur->val + left[0] + right[0];
        // 不偷cur，那么可以偷也可以不偷左右节点，则取较大的情况
        int val2 = max(left[0], left[1]) + max(right[0], right[1]);
        return {val2, val1};
    }
};
时间复杂度：O(n)，每个节点只遍历了一次
空间复杂度：O(log n)，算上递推系统栈的空间
```

### 121. 买卖股票的最佳时机

```
给定一个数组 prices ，它的第 i 个元素 prices[i] 表示一支给定股票第 i 天的价格。
你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。
返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

示例 1：
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。

示例 2：
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```

```cpp
暴力解法：
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int result = 0;
        for (int i = 0; i < prices.size(); i++) {
            for (int j = i + 1; j < prices.size(); j++){
                result = max(result, prices[j] - prices[i]);
            }
        }
        return result;
    }
};
时间复杂度：O(n^2)
空间复杂度：O(1)
```

```cpp
贪心算法
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int low = INT_MAX;
        int result = 0;
        for (int i = 0; i < prices.size(); i++) {
            low = min(low, prices[i]);  // 取最左最小价格
            result = max(result, prices[i] - low); // 直接取最大区间利润
        }
        return result;
    }
};
时间复杂度：O(n)
空间复杂度：O(1)
```

```
动态规划：
1.确定dp数组（dp table）以及下标的含义
dp[i][0] 表示第i天持有股票所得最多现金 ，这里可能有同学疑惑，本题中只能买卖一次，持有股票之后哪还有现金呢？其实一开始现金是0，那么加入第i天买入股票现金就是 -prices[i]， 这是一个负数。

dp[i][1] 表示第i天不持有股票所得最多现金
注意这里说的是“持有”，“持有”不代表就是当天“买入”！也有可能是昨天就买入了，今天保持持有的状态

2.确定递推公式
如果第i天持有股票即dp[i][0]， 那么可以由两个状态推出来
	第i-1天就持有股票，那么就保持现状，所得现金就是昨天持有股票的所得现金 即：dp[i - 1][0]
	第i天买入股票，所得现金就是买入今天的股票后所得现金即：-prices[i]
那么dp[i][0]应该选所得现金最大的，所以dp[i][0] = max(dp[i - 1][0], -prices[i])

如果第i天不持有股票即dp[i][1]， 也可以由两个状态推出来
	第i-1天就不持有股票，那么就保持现状，所得现金就是昨天不持有股票的所得现金 即：dp[i - 1][1]
	第i天卖出股票，所得现金就是按照今天股票价格卖出后所得现金即：prices[i] + dp[i - 1][0]
同样dp[i][1]取最大的，dp[i][1] = max(dp[i - 1][1], prices[i] + dp[i - 1][0]);

3.dp数组如何初始化
dp[0][0]表示第0天持有股票，此时的持有股票就一定是买入股票了，因为不可能有前一天推出来，所以dp[0][0] -= prices[0];
dp[0][1]表示第0天不持有股票，不持有股票那么现金就是0，所以dp[0][1] = 0;

4.确定遍历顺序
从递推公式可以看出dp[i]都是由dp[i - 1]推导出来的，那么一定是从前向后遍历。

5.举例推导dp数组
```

![121.买卖股票的最佳时机](https://code-thinking-1253855093.file.myqcloud.com/pics/20210224225642465.png)

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int len = prices.size();
        if (len == 0) return 0;
        vector<vector<int>> dp(len, vector<int>(2));
        dp[0][0] -= prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < len; i++) {
            dp[i][0] = max(dp[i - 1][0], -prices[i]);
            dp[i][1] = max(dp[i - 1][1], prices[i] + dp[i - 1][0]);
        }
        return dp[len - 1][1];
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 122.买卖股票的最佳时机 II

```
给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4。随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。

示例 2:
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。

示例 3:
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

提示：
1 <= prices.length <= 3 * 10 ^ 4
0 <= prices[i] <= 10 ^ 4
```

```
思路1：贪心算法
本题可以多次买卖股票
price[i] - price[i-2] = price[i] - price[i-1] + price[i-1] - price[i-2]
从上面的公式中我们可以看出：
第i-2天买入股票，第i天出售获得的利润 = 第i-1天买入，第i天出售 + 第i-2天买入，第i-1天出售的利润和。依照此思路递推，就有下面的公式
price[m] - price[n] = price[m] - price[m - 1] + .... + price[n + 1] - price[n]
由于我们要最大的利润，那么我们就两天两天判断，取(price[i] - price[i-1], 0)的最大值。逐步累加就是答案
```

```c++
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int ans = 0;
        for(int i = 1; i < prices.size(); i++){
            ans += max(0, prices[i] - prices[i-1]);
        }
        return ans;
    }
};
```

```
思路2：动态规划
1.确定dp数组的含义
dp[i][0] 表示第i天持有股票所得最多现金
dp[i][1] 表示第i天不持有股票所得最多现金

2.确定递推公式
dp[i][0]可有两种情况推导出来：
	dp[i-1][0]
	dp[i-1][1] - price[i]
dp[i][1]可有两种情况推导出来：
	dp[i-1][1]
	dp[i-1][0] + price[i]

3.确定初始化条件
dp[0][0] = -price[0]  dp[0][1] = 0

4.确定递归方向
从前向后即可，dp[i-1] 才可以退出dp[i]
```

```
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int len = prices.size();
        vector<vector<int>> dp(len, vector<int>(2, 0));
        dp[0][0] -= prices[0];
        dp[0][1] = 0;
        for (int i = 1; i < len; i++) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i]);
        }
        return dp[len - 1][1];
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 123.买卖股票的最佳时机 III

```
给定一个数组，它的第 i 个元素是一支给定的股票在第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:
输入：prices = [3,3,5,0,0,3,1,4]
输出：6 解释：在第 4 天（股票价格 = 0）的时候买入，在第 6 天（股票价格 = 3）的时候卖出，这笔交易所能获得利润 = 3-0 = 3 。随后，在第 7 天（股票价格 = 1）的时候买入，在第 8 天 （股票价格 = 4）的时候卖出，这笔交易所能获得利润 = 4-1 = 3。

示例 2：
输入：prices = [1,2,3,4,5]
输出：4 解释：在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4。注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。

示例 3：
输入：prices = [7,6,4,3,1]
输出：0 解释：在这个情况下, 没有交易完成, 所以最大利润为0。

示例 4：
输入：prices = [1] 输出：0

提示：
1 <= prices.length <= 10^5
0 <= prices[i] <= 10^5
```

```
1.确定dp数组以及下标的含义
一天一共就有五个状态，
	没有操作 （其实我们也可以不设置这个状态）
	第一次持有股票
	第一次不持有股票
	第二次持有股票
	第二次不持有股票
dp[i][j]中 i表示第i天，j为 [0 - 4] 五个状态，dp[i][j]表示第i天状态j所剩最大现金。
需要注意：dp[i][1]，表示的是第i天，买入股票的状态，并不是说一定要第i天买入股票.
例如 dp[i][1] ，并不是说 第i天一定买入股票，有可能 第 i-1天 就买入了，那么 dp[i][1] 延续买入股票的这个状态。

2.确定递推公式
达到dp[i][1]状态，有两个具体操作：
	第i天买入股票了，那么dp[i][1] = dp[i-1][0] - prices[i]
	第i天没有操作，而是沿用前一天买入的状态，即：dp[i][1] = dp[i - 1][1]
一定是选最大的，所以 dp[i][1] = max(dp[i-1][0] - prices[i], dp[i - 1][1]);

dp[i][2]也有两个操作：
	第i天卖出股票了，那么dp[i][2] = dp[i - 1][1] + prices[i]
	第i天没有操作，沿用前一天卖出股票的状态，即：dp[i][2] = dp[i - 1][2]
所以dp[i][2] = max(dp[i - 1][1] + prices[i], dp[i - 1][2])

dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]);

dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]);

3.dp数组的初始化
第0天没有操作，这个最容易想到，就是0，即：dp[0][0] = 0;
第0天做第一次买入的操作，dp[0][1] = -prices[0];
第0天做第一次卖出的操作，这个初始值应该是多少呢？
此时还没有买入，怎么就卖出呢？ 其实大家可以理解当天买入，当天卖出，所以dp[0][2] = 0;
第0天第二次买入操作，初始值应该是多少呢？应该不少同学疑惑，第一次还没买入呢，怎么初始化第二次买入呢？
第二次买入依赖于第一次卖出的状态，其实相当于第0天第一次买入了，第一次卖出了，然后再买入一次（第二次买入），那么现在手头上没有现金，只要买入，现金就做相应的减少。
所以第二次买入操作，初始化为：dp[0][3] = -prices[0];
同理第二次卖出初始化dp[0][4] = 0;

4.确定遍历顺序
从递归公式其实已经可以看出，一定是从前向后遍历，因为dp[i]，依靠dp[i - 1]的数值。

5.举例推导dp数组，以输入[1,2,3,4,5]为例
最大的利润一定出现在dp[4][4]，因为我们的递推公式一直保存着最大值，再加上遍历完所有股票，把所有情况都找寻完成了，再加上最后一天手上没股票肯定比有股票获得钱更多
```

![123.买卖股票的最佳时机III](https://code-thinking-1253855093.file.myqcloud.com/pics/20201228181724295-20230310134201291.png)

```cpp
// 版本一
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() == 0) return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(5, 0));
        dp[0][1] = -prices[0];
        dp[0][3] = -prices[0];
        for (int i = 1; i < prices.size(); i++) {
            dp[i][0] = dp[i - 1][0];
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i]);
            dp[i][2] = max(dp[i - 1][2], dp[i - 1][1] + prices[i]);
            dp[i][3] = max(dp[i - 1][3], dp[i - 1][2] - prices[i]);
            dp[i][4] = max(dp[i - 1][4], dp[i - 1][3] + prices[i]);
        }
        return dp[prices.size() - 1][4];
    }
};
时间复杂度：O(n)
空间复杂度：O(n × 5)
```

### 188.买卖股票的最佳时机 IV

```
给定一个整数数组 prices ，它的第 i 个元素 prices[i] 是一支给定的股票在第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你最多可以完成 k 笔交易。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1：
输入：k = 2, prices = [2,4,1]
输出：2 解释：在第 1 天 (股票价格 = 2) 的时候买入，在第 2 天 (股票价格 = 4) 的时候卖出，这笔交易所能获得利润 = 4-2 = 2。

示例 2：
输入：k = 2, prices = [3,2,6,5,0,3]
输出：7 解释：在第 2 天 (股票价格 = 2) 的时候买入，在第 3 天 (股票价格 = 6) 的时候卖出, 这笔交易所能获得利润 = 6-2 = 4。随后，在第 5 天 (股票价格 = 0) 的时候买入，在第 6 天 (股票价格 = 3) 的时候卖出, 这笔交易所能获得利润 = 3-0 = 3 。

提示：
0 <= k <= 100
0 <= prices.length <= 1000
0 <= prices[i] <= 1000
```

```
1.确定dp数组以及下标的含义
使用二维数组 dp[i][j] ：第i天的状态为j，所剩下的最大现金是dp[i][j]
状态就是:
	0:不操作	1：第一次买入   2：第一次卖出
	3：第二次买入	   4：第二次卖出   ....
因为最多k次交易，那么状态就有2 * k + 1个，二维数组dp[n][2*k + 1]，奇数状态是买入，偶数状态是卖出

2.确定递推公式
本次递推公式和之前的是一模一样的推导思路
j = 0; j < 2*k - 1; j+=2
dp[i][j + 1] = max(dp[i - 1][j + 1], dp[i - 1][j] - prices[i]);
dp[i][j + 2] = max(dp[i - 1][j + 2], dp[i - 1][j + 1] + prices[i]);

3.dp数组如何初始化
dp[0][0] = 0;
剩下的奇数状态就是-price[0],买入就要扣钱
剩下的偶数状态就是0,买入又卖出就恢复为0

4.确定遍历顺序
从前向后遍历

5.举例推导dp数组
最大利润一定出现在dp[n-1][2*k],因为我们一直保存着最大状态，把数组右下角一定是保存着最大值
```

```cpp
class Solution {
public:
    int maxProfit(int k, vector<int>& prices) {

        if (prices.size() == 0) return 0;
        vector<vector<int>> dp(prices.size(), vector<int>(2 * k + 1, 0));
        for (int j = 1; j < 2 * k; j += 2) {
            dp[0][j] = -prices[0];
        }
        for (int i = 1;i < prices.size(); i++) {
            for (int j = 0; j < 2 * k - 1; j += 2) {
                dp[i][j + 1] = max(dp[i - 1][j + 1], dp[i - 1][j] - prices[i]);
                dp[i][j + 2] = max(dp[i - 1][j + 2], dp[i - 1][j + 1] + prices[i]);
            }
        }
        return dp[prices.size() - 1][2 * k];
    }
};
时间复杂度: O(n * k)，其中 n 为 prices 的长度
空间复杂度: O(n * k)
```

### 309.最佳买卖股票时机含冷冻期

```
给定一个整数数组，其中第 i 个元素代表了第 i 天的股票价格 。
设计一个算法计算出最大利润。在满足以下约束条件下，你可以尽可能地完成更多的交易（多次买卖一支股票）:
你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。
卖出股票后，你无法在第二天买入股票 (即冷冻期为 1 天)。

示例:
输入: [1,2,3,0,2]
输出: 3
解释: 对应的交易状态为: [买入, 卖出, 冷冻期, 买入, 卖出]
```

```
1.确定dp数组以及下标的含义
dp[i][j]，第i天状态为j，所剩的最多现金为dp[i][j]。
状态一：持有股票状态（今天买入股票，或者是之前就买入了股票然后没有操作，一直持有）
不持有股票状态，这里就有两种卖出股票状态
	状态二：保持卖出股票的状态（两天前就卖出了股票，度过一天冷冻期。或者是前一天就是卖出股票状态，一直没操作）
	状态三：今天卖出股票
状态四：今天为冷冻期状态，但冷冻期状态不可持续，只有一天！

2.确定递推公式
达到买入股票状态（状态一）即：dp[i][0]，有两个具体操作：
	前一天就是持有股票状态（状态一），dp[i][0] = dp[i - 1][0]
	今天买入了，有两种情况
		前一天是冷冻期（状态四），dp[i - 1][3] - prices[i]
		前一天是保持卖出股票的状态（状态二），dp[i - 1][1] - prices[i]
那么dp[i][0] = max(dp[i - 1][0], dp[i - 1][3] - prices[i], dp[i - 1][1] - prices[i]);

达到保持卖出股票状态（状态二）即：dp[i][1]，有两个具体操作：
	前一天就是状态二
	前一天是冷冻期（状态四）
dp[i][1] = max(dp[i - 1][1], dp[i - 1][3]);

达到今天就卖出股票状态（状态三），即：dp[i][2] ，只有一个操作：
昨天一定是持有股票状态（状态一），今天卖出
即：dp[i][2] = dp[i - 1][0] + prices[i];

达到冷冻期状态（状态四），即：dp[i][3]，只有一个操作：
昨天卖出了股票（状态三）
dp[i][3] = dp[i - 1][2];

3.dp数组如何初始化
dp[0][0] = -prices[0]，一定是当天买入股票。
dp[0][1],dp[0][2],dp[0][3]都是0

4.确定遍历顺序
从前向后即可

5.举例推导dp数组
以 [1,2,3,0,2] 为例，dp数组如下：
最大利润一定出现在dp[n-1][2],dp[n-1][3],dp[n-1][4]上面，此时手中没有股票
```

![309.最佳买卖股票时机含冷冻期](https://code-thinking-1253855093.file.myqcloud.com/pics/2021032317451040.png)

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) return 0;
        vector<vector<int>> dp(n, vector<int>(4, 0));
        dp[0][0] -= prices[0]; // 持股票
        for (int i = 1; i < n; i++) {
            dp[i][0] = max(dp[i - 1][0], max(dp[i - 1][3] - prices[i], dp[i - 1][1] - prices[i]));
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][3]);
            dp[i][2] = dp[i - 1][0] + prices[i];
            dp[i][3] = dp[i - 1][2];
        }
        return max(dp[n - 1][3], max(dp[n - 1][1], dp[n - 1][2]));
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 714.买卖股票的最佳时机含手续费

```
给定一个整数数组 prices，其中第 i 个元素代表了第 i 天的股票价格 ；非负整数 fee 代表了交易股票的手续费用。
你可以无限次地完成交易，但是你每笔交易都需要付手续费。如果你已经购买了一个股票，在卖出它之前你就不能再继续购买股票了。
返回获得利润的最大值。
注意：这里的一笔交易指买入持有并卖出股票的整个过程，每笔交易你只需要为支付一次手续费。

示例 1:
输入: prices = [1, 3, 2, 8, 4, 9], fee = 2
输出: 8
解释: 能够达到的最大利润:
在此处买入 prices[0] = 1
在此处卖出 prices[3] = 8
在此处买入 prices[4] = 4
在此处卖出 prices[5] = 9
总利润: ((8 - 1) - 2) + ((9 - 4) - 2) = 8.

注意:
0 < prices.length <= 50000.
0 < prices[i] < 50000.
0 <= fee < 50000.
```

```
贪心算法：最低值买，最高值（如果算上手续费还盈利）就卖。
此时无非就是要找到两个点，买入日期，和卖出日期。
	买入日期：其实很好想，遇到更低点就记录一下。
	卖出日期：这个就不好算了，但也没有必要算出准确的卖出日期，只要当前价格大于（最低价格+手续费），就可以收获利润，至于准确的卖出日期，就是连续收获利润区间里的最后一天（并不需要计算是具体哪一天）。
所以我们在做收获利润操作的时候其实有三种情况：
	情况一：收获利润的这一天并不是收获利润区间里的最后一天（不是真正的卖出，相当于持有股票），所以后面要继续收获利润。
	情况二：前一天是收获利润区间里的最后一天（相当于真正的卖出了），今天要重新记录最小价格了。
	情况三：不作操作，保持原有状态（买入，卖出，不买不卖）
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int result = 0;
        int minPrice = prices[0]; // 记录最低价格
        for (int i = 1; i < prices.size(); i++) {
            // 情况二：相当于买入
            if (prices[i] < minPrice) minPrice = prices[i];

            // 情况三：保持原有状态（因为此时买则不便宜，卖则亏本）
            if (prices[i] >= minPrice && prices[i] <= minPrice + fee) {
                continue;
            }

            // 计算利润，可能有多次计算利润，最后一次计算利润才是真正意义的卖出
            if (prices[i] > minPrice + fee) {
                result += prices[i] - minPrice - fee;
                minPrice = prices[i] - fee; // 情况一，这一步很关键，避免重复扣手续费
            }
        }
        return result;
    }
};
时间复杂度：O(n)
空间复杂度：O(1)
```

```
动态规划：
1.确定dp数组的含义
dp[i][0] 表示第i天持有股票所省最多现金。 dp[i][1] 表示第i天不持有股票所得最多现金

2.确定递推公式
如果第i天持有股票即dp[i][0]， 那么可以由两个状态推出来
	第i-1天就持有股票，那么就保持现状，所得现金就是昨天持有股票的所得现金 即：dp[i - 1][0]
	第i天买入股票，所得现金就是昨天不持有股票的所得现金减去 今天的股票价格 即：dp[i - 1][1] - prices[i]
所以：dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);

如果第i天不持有股票即dp[i][1]的情况， 依然可以由两个状态推出来
	第i-1天就不持有股票，那么就保持现状，所得现金就是昨天不持有股票的所得现金 即：dp[i - 1][1]
	第i天卖出股票，所得现金就是按照今天股票价格卖出后所得现金，注意这里需要有手续费了即：dp[i - 1][0] + prices[i] - fee
所以：dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i] - fee);

3.确定初始化条件
dp[0][0] = -prices[0];   dp[0][1]  = 0;

4.确定遍历方向
从前向后即可
```

```cpp
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int n = prices.size();
        vector<vector<int>> dp(n, vector<int>(2, 0));
        dp[0][0] -= prices[0]; // 持股票
        for (int i = 1; i < n; i++) {
            dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] - prices[i]);
            dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] + prices[i] - fee);
        }
        return max(dp[n - 1][0], dp[n - 1][1]);
    }
};
```

### 300.最长递增子序列

```
给你一个整数数组 nums ，找到其中最长严格递增子序列的长度。
子序列是由数组派生而来的序列，删除（或不删除）数组中的元素而不改变其余元素的顺序。例如，[3,6,2,7] 是数组 [0,3,1,6,2,2,7] 的子序列。

示例 1：
输入：nums = [10,9,2,5,3,7,101,18]
输出：4
解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。

示例 2：
输入：nums = [7,7,7,7,7,7,7]
输出：1

提示：
1 <= nums.length <= 2500
-10^4 <= nums[i] <= 104
```

```
1.确定dp数组含义
dp[i]表示以第i个元素结尾的最长子序列长度

2.确定递推公式
对于每个小于i的索引j（从i - 1 到 0）
如果nums[i] > nums[j]，说明nums[i]可以纳入这个子序列中
dp[i] = max(dp[i], dp[j] + 1)
注意不能找到第一个nums[i] > nums[j]就停下，因为nums[k](k < j)可能子序列更长，因为nums[k] > nums[j]

3.确定初始化条件
dp[0] = 1;

4.确定递推方向
外层从0 到 n - 1
内层从i 到 0
最后的答案是dp中的最大值，因为每一个元素都有可能成为最长子序列的末尾元素

5举例推导dp数组
```

![300.最长上升子序列](https://code-thinking-1253855093.file.myqcloud.com/pics/20210110170945618.jpg)

```cpp
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        if (nums.size() <= 1) return nums.size();
        vector<int> dp(nums.size(), 1);
        int result = 0;
        for (int i = 1; i < nums.size(); i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) dp[i] = max(dp[i], dp[j] + 1);
            }
            if (dp[i] > result) result = dp[i]; // 取长的子序列
        }
        return result;
    }
};
时间复杂度: O(n^2)
空间复杂度: O(n)
```

### 674. 最长连续递增序列

```
给定一个未经排序的整数数组，找到最长且 连续递增的子序列，并返回该序列的长度。
连续递增的子序列 可以由两个下标 l 和 r（l < r）确定，如果对于每个 l <= i < r，都有 nums[i] < nums[i + 1] ，那么子序列 [nums[l], nums[l + 1], ..., nums[r - 1], nums[r]] 就是连续递增子序列。

示例 1：
输入：nums = [1,3,5,4,7]
输出：3
解释：最长连续递增序列是 [1,3,5], 长度为3。尽管 [1,3,5,7] 也是升序的子序列, 但它不是连续的，因为 5 和 7 在原数组里被 4 隔开。

示例 2：
输入：nums = [2,2,2,2,2]
输出：1
解释：最长连续递增序列是 [2], 长度为1。

提示：
0 <= nums.length <= 10^4
-10^9 <= nums[i] <= 10^9
```

```
1.确定dp数组的含义
dp[i]表示以第i个元素结尾的最长连续递增子序列长度

2.确定递推公式
if (nums[i] > nums[i - 1]) dp[i] = dp[i - 1] + 1
else dp[i] = 1

3.确定初始化
dp[0] = 1

4.确定递推顺序
从前到后

5.举例推导
```

![674.最长连续递增序列](https://code-thinking-1253855093.file.myqcloud.com/pics/20210204103529742.jpg)

```cpp
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        int result = 1;
        vector<int> dp(nums.size() ,1);
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] > nums[i - 1]) { // 连续记录
                dp[i] = dp[i - 1] + 1;
            }
            if (dp[i] > result) result = dp[i];
        }
        return result;
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 718. 最长重复子数组

```
给两个整数数组 A 和 B ，返回两个数组中公共的、长度最长的子数组的长度。

示例：
输入：
A: [1,2,3,2,1]
B: [3,2,1,4,7]
输出：3
解释：长度最长的公共子数组是 [3, 2, 1] 。

提示：
1 <= len(A), len(B) <= 1000
0 <= A[i], B[i] < 100
```

```
1.确定dp数组的含义
dp[i][j] ：以下标i - 1为结尾的A，和以下标j - 1为结尾的B，最长重复子数组长度为dp[i][j]。 （特别注意： “以下标i - 1为结尾的A” 标明一定是 以A[i-1]为结尾的字符串 ）

2.确定递推公式
dp[i][j]的状态只能由dp[i - 1][j - 1]推导出来。
即当A[i - 1] 和B[j - 1]相等的时候，dp[i][j] = dp[i - 1][j - 1] + 1;

3.dp数组如何初始化
为了方便递归公式,dp[i][0] 和dp[0][j]初始化为0。
举个例子A[0]如果和B[0]相同的话，dp[1][1] = dp[0][0] + 1，只有dp[0][0]初始为0，正好符合递推公式逐步累加起来。

4.确定遍历顺序
从递推公式可以看出，正常二维数组遍历即可
由于最长子数组可以出现在任意索引位置上，所以要保存最大值

5.举例推导dp数组
```

![718.最长重复子数组](https://code-thinking-1253855093.file.myqcloud.com/pics/2021011215282060.jpg)

```cpp
// 版本一
class Solution {
public:
    int findLength(vector<int>& nums1, vector<int>& nums2) {
        vector<vector<int>> dp (nums1.size() + 1, vector<int>(nums2.size() + 1, 0));
        int result = 0;
        for (int i = 1; i <= nums1.size(); i++) {
            for (int j = 1; j <= nums2.size(); j++) {
                if (nums1[i - 1] == nums2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                }
                if (dp[i][j] > result) result = dp[i][j];
            }
        }
        return result;
    }
};
时间复杂度：O(n × m)，n 为A长度，m为B长度
空间复杂度：O(n × m)
```

### 1143.最长公共子序列

```
给定两个字符串 text1 和 text2，返回这两个字符串的最长公共子序列的长度。
若这两个字符串没有公共子序列，则返回 0。

示例 1:
输入：text1 = "abcde", text2 = "ace"
输出：3
解释：最长公共子序列是 "ace"，它的长度为 3。

示例 2:
输入：text1 = "abc", text2 = "abc"
输出：3
解释：最长公共子序列是 "abc"，它的长度为 3。

示例 3:
输入：text1 = "abc", text2 = "def"
输出：0
解释：两个字符串没有公共子序列，返回 0。

提示:
1 <= text1.length <= 1000
1 <= text2.length <= 1000 输入的字符串只含有小写英文字符。
```

```
1.确定dp数组含义
dp[i][j]:长度为[0, i - 1]的字符串text1与长度为[0, j - 1]的字符串text2的最长公共子序列为dp[i][j]

2.确定递推公式
如果text1[i - 1] 与 text2[j - 1]相同，那么找到了一个公共元素，所以dp[i][j] = dp[i - 1][j - 1] + 1;
如果text1[i - 1] 与 text2[j - 1]不相同，那就看看text1[0, i - 2]与text2[0, j - 1]的最长公共子序列 和 text1[0, i - 1]与text2[0, j - 2]的最长公共子序列，取最大的。
即：dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);

3.dp数组如何初始化
test1[0, i-1]和空串的最长公共子序列自然是0，所以dp[i][0] = 0;同理dp[0][j]也是0。
其他下标都是随着递推公式逐步覆盖，初始为多少都可以，那么就统一初始为0。

4.确定遍历顺序
从递推公式，可以看出，有三个方向可以推出dp[i][j]，所以要从左到右，从上到下

5.举例推导dp数组
```

![1143.最长公共子序列1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210210150215918.jpg)

```cpp
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        vector<vector<int>> dp(text1.size() + 1, vector<int>(text2.size() + 1, 0));
        for (int i = 1; i <= text1.size(); i++) {
            for (int j = 1; j <= text2.size(); j++) {
                if (text1[i - 1] == text2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[text1.size()][text2.size()];
    }
};
时间复杂度: O(n * m)，其中 n 和 m 分别为 text1 和 text2 的长度
空间复杂度: O(n * m)
```

### 1035.不相交的线

```
我们在两条独立的水平线上按给定的顺序写下 A 和 B 中的整数。
现在，我们可以绘制一些连接两个数字 A[i] 和 B[j] 的直线，只要 A[i] == B[j]，且我们绘制的直线不与任何其他连线（非水平线）相交。
以这种方法绘制线条，并返回我们可以绘制的最大连线数。
```

![1035.不相交的线](https://code-thinking-1253855093.file.myqcloud.com/pics/2021032116363533.png)

```
想一想，怎么样两个数组之间相同数字的连线会不相交
当没有思路时，我们从答案数组出发，想象最后的答案数组，一定是A，B两个数组的子序列。最长公共子序列相连就是答案。（子序列，所以连线一定不相交）
```

```cpp
class Solution {
public:
    int maxUncrossedLines(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size(), n = nums2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        int ans = 0;
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (nums1[i - 1] == nums2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                }else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
                ans = max(ans, dp[i][j]);
            }
        }
        return ans;
    }
};
时间复杂度: O(n * m)
空间复杂度: O(n * m)
```

### 53. 最大子序和

```
给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
输入: [-2,1,-3,4,-1,2,1,-5,4]
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```

```
1.确定dp数组的含义
dp[i]表示以第i个元素为结尾的具有最大和的连续子数组的长度

2.确定递推公式
如果dp[i - 1] <= 0 那么为了保证和最大 且 连续 第i个元素就应该自己存在dp[i] = nums[i]。否则就应该接上前面的数组:dp[i] = dp[i - 1] + nums[i]

3.确定dp数组的初始化
dp[0] = nums[0]

4.确定遍历方向
从前到后遍历，从递推公式就可以看出来
答案是dp中的最大值

5.举例推导dp数组
```

![53.最大子序和（动态规划）](https://code-thinking-1253855093.file.myqcloud.com/pics/20210303104129101.png)

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        if (nums.size() == 0) return 0;
        vector<int> dp(nums.size());
        dp[0] = nums[0];
        int result = dp[0];
        for (int i = 1; i < nums.size(); i++) {
            dp[i] = max(dp[i - 1] + nums[i], nums[i]); // 状态转移公式
            if (dp[i] > result) result = dp[i]; // result 保存dp[i]的最大值
        }
        return result;
    }
};
时间复杂度：O(n)
空间复杂度：O(n)
```

### 115.不同的子序列

```
给定一个字符串 s 和一个字符串 t ，计算在 s 的子序列中 t 出现的个数。
题目数据保证答案符合 32 位带符号整数范围。

提示：
0 <= s.length, t.length <= 1000
s 和 t 由英文字母组成
```

![115.不同的子序列示例](https://code-thinking.cdn.bcebos.com/pics/115.%E4%B8%8D%E5%90%8C%E7%9A%84%E5%AD%90%E5%BA%8F%E5%88%97%E7%A4%BA%E4%BE%8B.jpg)

```
1.确定dp数组的含义
dp[i][j]为s[0 : i - 1]中出现t[0-j-1]的子序列方案数

2.确定递推公式
s[i - 1] == t[j - 1]时，说明要么s[i - 1]纳入子序列考察范围(t[j - 1]和s[i - 1]匹配)，要么不纳入，所以dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j]
如果s[i - 1] != t[j - 1]，那么dp[i][j] = dp[i - 1][j]只能让前i - 1个去匹配j个了

3.确定初始化
我们从递推公式可以知道，左上角是一定要初始化的部分
dp[0][j]表示s取空，跟t[0]做匹配的方案数，一定是0
dp[i][0]表示s取0 - i - 1取匹配空，方案数是1（s前i个字符全删了）
dp[0][0]表示空匹配空，方案数为1

4.确定递推方向
从上到下，从左到右即可

5.举例推导dp数组
```

![115.不同的子序列](https://code-thinking.cdn.bcebos.com/pics/115.%E4%B8%8D%E5%90%8C%E7%9A%84%E5%AD%90%E5%BA%8F%E5%88%97.jpg)

```cpp
class Solution {
public:
    int numDistinct(string s, string t) {
        int m = s.size(), n = t.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        int mod = 1e9 + 7;
        for (int i = 0; i <= m; i++) {
            dp[i][0] = 1;
        }
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s[i - 1] == t[j - 1]) {
                    dp[i][j] = (dp[i - 1][j - 1] + dp[i - 1][j]) % mod;
                }else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return dp[m][n];
    }
};
时间复杂度: O(n * m)
空间复杂度: O(n * m)
```

### 583. 两个字符串的删除操作

```
给定两个单词 word1 和 word2，找到使得 word1 和 word2 相同所需的最小步数，每步可以删除任意一个字符串中的一个字符。

示例：
输入: "sea", "eat"
输出: 2
解释: 第一步将"sea"变为"ea"，第二步将"eat"变为"ea"
```

```
动态规划：
本题其实就是最长公共子序列，求出两个单词的最长公共子序列，然后让两个单词都变成最长公共子序列的次数就是答案
```

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size()+1, vector<int>(word2.size()+1, 0));
        for (int i=1; i<=word1.size(); i++){
            for (int j=1; j<=word2.size(); j++){
                if (word1[i-1] == word2[j-1]) dp[i][j] = dp[i-1][j-1] + 1;
                else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
        return word1.size()+word2.size()-dp[word1.size()][word2.size()]*2;
    }
};
```

### 72. 编辑距离

```
给你两个单词 word1 和 word2，请你计算出将 word1 转换成 word2 所使用的最少操作数 。
你可以对一个单词进行如下三种操作：
	插入一个字符
	删除一个字符
	替换一个字符

示例 1：
输入：word1 = "horse", word2 = "ros"
输出：3
解释： horse -> rorse (将 'h' 替换为 'r') rorse -> rose (删除 'r') rose -> ros (删除 'e')

示例 2：
输入：word1 = "intention", word2 = "execution"
输出：5
解释： intention -> inention (删除 't') inention -> enention (将 'i' 替换为 'e') enention -> exention (将 'n' 替换为 'x') exention -> exection (将 'n' 替换为 'c') exection -> execution (插入 'u')

提示：
0 <= word1.length, word2.length <= 500
word1 和 word2 由小写英文字母组成
```

```
1.确定dp数组的含义
dp[i][j]表示word1的前i - 1个字符，变成word2的前j - 1个字符需要的最小操作次数

2.确定递推公式
题目提供了三种操作，删添换。
dp[i][j] 可以由
	dp[i - 1][j]次数，再删除一个word1字符达成
	dp[i - 1][j - 1]次数，再更换word1[i - 1]变成word2[j - 1]
	dp[i][j - 1]次数，再添加一个word2字符完成
三种操作取最小值即可

3.确定初始化条件
dp[0][j] = j	dp[i][0] = i

4.确定递推顺序
从递推公式可以看出，从上到下，从左到右

5.举例推导dp数组
```

![72.编辑距离1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210114162132300.jpg)

```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        vector<vector<int>> dp(word1.size() + 1, vector<int>(word2.size() + 1, 0));
        for (int i = 0; i <= word1.size(); i++) dp[i][0] = i;
        for (int j = 0; j <= word2.size(); j++) dp[0][j] = j;
        for (int i = 1; i <= word1.size(); i++) {
            for (int j = 1; j <= word2.size(); j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                }
                else {
                    dp[i][j] = min({dp[i - 1][j - 1], dp[i - 1][j], dp[i][j - 1]}) + 1;
                }
            }
        }
        return dp[word1.size()][word2.size()];
    }
};
时间复杂度: O(n * m)
空间复杂度: O(n * m)
```

### 647. 回文子串

```
给定一个字符串，你的任务是计算这个字符串中有多少个回文子串。
具有不同开始位置或结束位置的子串，即使是由相同的字符组成，也会被视作不同的子串。

示例 1：
输入："abc"
输出：3
解释：三个回文子串: "a", "b", "c"

示例 2：
输入："aaa"
输出：6
解释：6个回文子串: "a", "a", "a", "aa", "aa", "aaa"

提示：输入的字符串长度不会超过 1000 。
```

1. 确定 dp 数组（dp table）以及下标的含义

如果大家做了很多这种子序列相关的题目，在定义 dp 数组的时候 很自然就会想题目求什么，我们就如何定义 dp 数组。

绝大多数题目确实是这样，不过本题如果我们定义，dp[i] 为 下标 i 结尾的字符串有 dp[i]个回文串的话，我们会发现很难找到递归关系。

dp[i] 和 dp[i-1] ，dp[i + 1] 看上去都没啥关系。

所以我们要看回文串的性质。 如图：

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20230102170752.png)

我们在判断字符串 S 是否是回文，那么如果我们知道 s[1]，s[2]，s[3] 这个子串是回文的，那么只需要比较 s[0]和 s[4]这两个元素是否相同，如果相同的话，这个字符串 s 就是回文串。

那么此时我们是不是能找到一种递归关系，也就是判断一个子字符串（字符串的下表范围[i,j]）是否回文，依赖于，子字符串（下表范围[i + 1, j - 1]）） 是否是回文。

所以为了明确这种递归关系，我们的 dp 数组是要定义成一位二维 dp 数组。

布尔类型的 dp[i] [j]：表示区间范围[i,j] （注意是左闭右闭）的子串是否是回文子串，如果是 dp[i] [j]为 true，否则为 false。

**确定递推公式**

在确定递推公式时，就要分析如下几种情况。

整体上是两种，就是 s[i]与 s[j]相等，s[i]与 s[j]不相等这两种。

当 s[i]与 s[j]不相等，那没啥好说的了，dp[i][j]一定是 false。

当 s[i]与 s[j]相等时，这就复杂一些了，有如下三种情况

- 情况一：下标 i 与 j 相同，同一个字符例如 a，当然是回文子串
- 情况二：下标 i 与 j 相差为 1，例如 aa，也是回文子串
- 情况三：下标：i 与 j 相差大于 1 的时候，例如 cabac，此时 s[i]与 s[j]已经相同了，我们看 i 到 j 区间是不是回文子串就看 aba 是不是回文就可以了，那么 aba 的区间就是 i+1 与 j-1 区间，这个区间是不是回文就看 dp[i + 1] [j - 1]是否为 true。

以上三种情况分析完了，那么递归公式如下：

```cpp
if (s[i] == s[j]) {
    if (j - i <= 1) { // 情况一 和 情况二
        result++;
        dp[i][j] = true;
    } else if (dp[i + 1][j - 1]) { // 情况三
        result++;
        dp[i][j] = true;
    }
}
```

result 就是统计回文子串的数量。

注意这里我没有列出当 s[i]与 s[j]不相等的时候，因为在下面 dp[i][j]初始化的时候，就初始为 false。

**dp 数组如何初始化**

dp[i] [j]可以初始化为 true 么？ 当然不行，怎能刚开始就全都匹配上了。

所以 dp[i] [j]初始化为 false。

**确定遍历顺序**

首先从递推公式中可以看出，情况三是根据 dp[i + 1] [j - 1]是否为 true，在对 dp[i] [j]进行赋值 true 的。

dp[i + 1] [j - 1] 在 dp[i] [j]的左下角，如图：

![647.回文子串](https://code-thinking-1253855093.file.myqcloud.com/pics/20210121171032473-20230310132134822.jpg)

**一定要从下到上，从左到右遍历，这样保证 dp[i + 1] [j - 1]都是经过计算的**。

**举例推导 dp 数组**

举例，输入："aaa"，dp[i] [j]状态如下：

![647.回文子串1](https://code-thinking-1253855093.file.myqcloud.com/pics/20210121171059951-20230310132153163.jpg)

```cpp
class Solution {
public:
    int countSubstrings(string s) {
        vector<vector<bool>> dp(s.size(), vector<bool>(s.size(), false));
        int result = 0;
        for (int i = s.size() - 1; i >= 0; i--) {  // 注意遍历顺序
            for (int j = i; j < s.size(); j++) {
                if (s[i] == s[j]) {
                    if (j - i <= 1) { // 情况一 和 情况二
                        result++;
                        dp[i][j] = true;
                    } else if (dp[i + 1][j - 1]) { // 情况三
                        result++;
                        dp[i][j] = true;
                    }
                }
            }
        }
        return result;
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n^2)
```

### 516.最长回文子序列

```
给定一个字符串 s ，找到其中最长的回文子序列，并返回该序列的长度。可以假设 s 的最大长度为 1000 。

示例 1: 输入: "bbbab" 输出: 4 一个可能的最长回文子序列为 "bbbb"。

示例 2: 输入:"cbbd" 输出: 2 一个可能的最长回文子序列为 "bb"。

提示：
1 <= s.length <= 1000
s 只包含小写英文字母
```

```
1.确定dp数组的含义
dp[i][j]表示s[i - j]范围内最长回文子序列长度

2.确定递推公式
回文的判定一定是两头往中间
if (s[i] == s[j]) 那么dp[i][j] = dp[i + 1][j - 1] + 2
else dp[i][j] = max(dp[i + 1][j], dp[i][j - 1])

3.确定初始化
由于j >= i，所以数组左下角都是0，对角线是1

4.确定递推顺序
从下到上，从左到右

5.举例推导dp数组
```

![516.最长回文子序列3](https://code-thinking-1253855093.file.myqcloud.com/pics/20210127151521432.jpg)

```cpp
class Solution {
public:
    int longestPalindromeSubseq(string s) {
        vector<vector<int>> dp(s.size(), vector<int>(s.size(), 0));
        for (int i = 0; i < s.size(); i++) dp[i][i] = 1;
        for (int i = s.size() - 1; i >= 0; i--) {
            for (int j = i + 1; j < s.size(); j++) {
                if (s[i] == s[j]) {
                    dp[i][j] = dp[i + 1][j - 1] + 2;
                } else {
                    dp[i][j] = max(dp[i + 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[0][s.size() - 1];
    }
};
时间复杂度: O(n^2)
空间复杂度: O(n^2)
```

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE_%E9%9D%92.png)
