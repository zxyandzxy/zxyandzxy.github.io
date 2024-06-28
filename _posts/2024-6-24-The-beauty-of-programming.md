---
layout: post
title: "编程之美"
date: 2024-6-24
tags: [books]
comments: true
author: zxy
---

> 编程之美阅读笔记
>
> 书本笔记链接：[read_books: 一些书本的阅读 (gitee.com)](https://gitee.com/zou-xinyu6/read_books)

## 第一章：游戏之乐

### 让CPU占用率曲线听你指挥

- [编程之美1 让CPU占用率曲线听你指挥_cpu能耗曲线-CSDN博客](https://blog.csdn.net/qq_41380950/article/details/127970317)
- 任务管理器CPU显示原理

- 占用率本质 -> 忙 / 闲时间比




### 中国象棋将帅问题

- [编程之美 1.2中国象棋将帅问题 - SeeKHit - 博客园 (cnblogs.com)](https://www.cnblogs.com/SeekHit/p/5105864.html)
- 位运算技巧




### 一摞烙饼的排序（难）

- [【编程之美】一摞烙饼的排序_烙饼问题 编程-CSDN博客](https://blog.csdn.net/tianshuai1111/article/details/7659673)
- 递归思考，局部有序问题




### 买书问题

- [《编程之美》买书问题——动态规划_买书问题 三本书-CSDN博客](https://blog.csdn.net/q623702748/article/details/51592427)
- 动态规划递推公式寻找思路，存在已知信息枚举贪心




### 快速找出故障的机器

- [编程之美(五)快速找出故障机器_编程之美 快速找出机器中的故障-CSDN博客](https://blog.csdn.net/qq_38790716/article/details/90379156)
- [编程之美_1.5_快速找出机器故障_算法 n台机器死机-CSDN博客](https://blog.csdn.net/insistgogo/article/details/7687936)
- 异位位运算， 从不变量解析变量




### 饮料供货

- [编程之美 - 1.6 饮料供货 ☞ 【动态规划】_编程之美1.6-CSDN博客](https://blog.csdn.net/peerslee/article/details/77418065)
- [编程之美 - 1.6 饮料供货 ☞ 【记忆化搜索】_寻找以往记忆饮料-CSDN博客](https://blog.csdn.net/peerslee/article/details/77450586)
- 多重背包动态规划 + 存在已知信息枚举贪心 




### 光影切割问题

- [编程之美(七)光影切割问题-CSDN博客](https://blog.csdn.net/qq_38790716/article/details/90439620)
- 将分块问题通过数学逻辑转化成求交点问题




### 小飞的电梯调度算法

- [编程之美：小飞的电梯调度算法-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/288991)
- 数学建模找思路+暴力求解

- 利用选项有限优化算法




### 高效率安排见面会

- [【编程之美】高效率的安排见面会_编程之美 高效率地安排见面会 java-CSDN博客](https://blog.csdn.net/tianshuai1111/article/details/7692200)
- 抽象出图结构转化成图染色问题

- 数据结构优化遍历过程




### 双线程高效下载

- [1.10 编程之美-双线程下载[double threads to download\] - hellogiser - 博客园 (cnblogs.com)](https://www.cnblogs.com/hellogiser/p/double-threads-to-download-and-write.html)
- 处理网络文件下载 -- 内存 --- 本地磁盘 这两个过程分别用一个线程实现

- 通过信号量机制保证安全，达到加速下载效果




### NIM（1）一排石头的游戏

- [第1章 游戏之乐——NIM(1)一排石子的游戏 - ~风轻云淡~ - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaopeng527/p/4608419.html)
- 先取必胜策略

- 从特殊情况出发，举例子分析，找出规律




### NIM（2）"拈"游戏分析

- [第1章 游戏之乐——NIM（2）“拈”游戏分析 - ~风轻云淡~ - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaopeng527/p/4609456.html)
- 从特殊情况出发，举例子分析

- 分类讨论，转成数学问题分析（异或）




### NIM（3）两堆石头的游戏

- [第1章 游戏之乐——NIM（3）两堆石头的游戏 - ~风轻云淡~ - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaopeng527/p/4610195.html)
- 质数筛法思路分析寻找规律

- 数学证明递推公式降低复杂度

> Nim游戏及各种变形
>
> [Nim游戏及各种变形 - BigSmall_En - 博客园 (cnblogs.com)](https://www.cnblogs.com/BigSmall-En/p/nim.html)



### 连连看游戏设计

- [编程之美一------连连看游戏设计 - Champion Lai - 博客园 (cnblogs.com)](https://www.cnblogs.com/championlai/articles/3707349.html)
- 数学建模游戏问题，根据游戏规则从游戏状态和规则进行描述

- 广度优先搜索求取最短路径

- 死锁判定规则




### 构造数独

- [第1章 游戏之乐——构造数独 - ~风轻云淡~ - 博客园 (cnblogs.com)](https://www.cnblogs.com/gaopeng527/p/4611497.html)
- 深度优先搜索，回溯解决数独初始化

- 通过数独需要满足的特征进行考虑，通过列，行变换初始化




### 24点游戏

- [【编程之美】24点游戏 - Ryan in C++ - 博客园 (cnblogs.com)](https://www.cnblogs.com/forcheryl/p/3976002.html)
- 暴力枚举每种合理表达式（回溯实现），判断是不是24点

- 分治考虑非空真子集，真子集之间四则运算，之内四则运算实现判定24点




### 俄罗斯方块游戏

- [编程之美----俄罗斯方块_int clearline()-CSDN博客](https://blog.csdn.net/chdhust/article/details/8365844)
- [编程之美-俄罗斯方块游戏方法整理_《编程之美》俄罗斯方块算法-CSDN博客](https://blog.csdn.net/GarfieldEr007/article/details/49736995)
- 游戏设计建模：分析有什么对象需要建模，分析可不可以通过存储减少运算

- 写出伪代码，剪枝优化




### 扫雷游戏

- [解答《编程之美》1.18问题1：给所有未标识方块标注有地雷概率 - 五岳 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wuyuegb2312/p/3163833.html)
- 游戏思路提出




## 第二章：数字之魅

### 求二进制中1的个数

[【编程之美学习笔记】2.1求二进制数中1的个数 - GraceSkyer - 博客园 (cnblogs.com)](https://www.cnblogs.com/GraceSkyer/p/5836718.html)

- 除2取余
- 与运算右移
- 位运算消除最低位1
- 直接查表



### 不要怕阶乘

- [[编程之美\][2.2] 不要被阶乘吓倒 - 菜鸟加贝的爬升 - 博客园 (cnblogs.com)](https://www.cnblogs.com/jiabei521/p/3355974.html)
- 正整数唯一分解定理

- 不要求阶乘，因为非常可能溢出

- 通过分解出质数，然后根据特性取值




### 寻找发帖的水王

- [【编程之美学习笔记】2.3寻找发帖水王 - GraceSkyer - 博客园 (cnblogs.com)](https://www.cnblogs.com/GraceSkyer/p/5843555.html)
- 利用大家一起删，那么排序不变的思想，将多余的东西全部删掉




### 1的数目

- [编程之美-1的个数 - 小江。 - 博客园 (cnblogs.com)](https://www.cnblogs.com/c-c-c-c/p/7044174.html)
- 从特殊到一般；通过按照位数举例，判断1的增量情况，减小时间复杂度

- 先找上界，然后逐一递减确定下界




### 寻找最大的K个数

[编程之美之2.5 寻找最大的K个数-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/3798)

- 暴力排序
- 快速排序修改版
- 二分法求第K大的元素
- 堆排序保留前K个元素
- 如果N不太大，直接统计[0, Vmax]中各个元素出现的次数，统计最大K个即可



### 精确表达浮点数

- [【编程之美】2.6 精确表达浮点数 - 匡子语 - 博客园 (cnblogs.com)](https://www.cnblogs.com/dplearning/p/4064786.html)
- 用分数表达浮点数

- 通过数学推导，将小数位除去，利用整数重新表达




### 最大公约数

- [最大公约数问题－－－编程之美 - cjmandlulu - 博客园 (cnblogs.com)](https://www.cnblogs.com/cjmlovelulu/articles/3784557.html)
- 辗转相除法（不适用大整数）

- 相减法

- 结合移位、相减、奇偶判定优化




### 找出符合条件的整数

- [编程之美-2.8 找到符合条件的整数 - jfcspring - 博客园 (cnblogs.com)](https://www.cnblogs.com/jfcspring/p/3776388.html)
- 哈哈，看不懂一点




### 斐波那契数列

[编程之美2.9 斐波那契数列_2419: 斐波那契数列-CSDN博客](https://blog.csdn.net/u011438605/article/details/72845199)

- 空间换时间，记录已知信息
- 递归变循环
- 求解通项公式
- 转换成矩阵乘法，分治求解



### 寻找数组中的最大值和最小值

- [《编程之美》学习笔记——2.10寻找数组中的最大值和最小值_(2)求最大最小值,1.5n-2,是最快的方法吗?-CSDN博客](https://blog.csdn.net/chensilly8888/article/details/42672123)
- 分治算法减少比较次数




### 寻找最近点对

- [编程之美2.11：寻找最近的点对 - higirle - 博客园 (cnblogs.com)](https://www.cnblogs.com/Jessy/p/3512076.html)
- 先从一维考虑，暴力为N^2，通过归并排序可以压缩到N * logN

- 在二维角度，也进行从暴力到归并排序的优化




### 快速寻找满足条件的两个数

- [编程之美2.12 快速寻找满足条件的两个数 - panpannju - 博客园 (cnblogs.com)](https://www.cnblogs.com/panpannju/p/3732012.html)
- 两数之和：暴力 + 排序双指针 + 哈希




### 子数组的最大乘积

[一文看懂《子数组的最大乘积问题》-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1494743)

- 暴力求解
- 前缀积（从左边/右边开始统计）
- 分类讨论N个数乘积的情况



### 求数组的子数组之和的最大值

[编程之美8：求数组的子数组之和的最大值_int maxsum-CSDN博客](https://blog.csdn.net/ndzjx/article/details/84404589)

- 暴力求解：三重for循环 -> 两重for循环
- 分治求解
- 动态规划



### 子数组之和的最大值（二维）

[编程之美2.15 子数组之和的最大值（二维） - panpannju - 博客园 (cnblogs.com)](https://www.cnblogs.com/panpannju/p/3734024.html)

- 暴力解法
- 动态规划，先行求解出Sum(i,j)



### 求数组中的最长递增子序列

- https://blog.csdn.net/linyunzju/article/details/7727254
- [编程之美--2.16求数组中最长递增子序列 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4592400.html)
- 动态规划求取之前子序列的长度最大值进行比较，看能不能往后加

- 通过定义MaxV这样一个递增数组来实现二分查找长度固定的子序列中的最优子序列




### 数组循环移位

- [编程之美--2.17数组循环移位 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4592315.html)
- 没啥意思，LeetCode原题




### 数组分割

- [编程之美——数组分割 - Jessica程序猿 - 博客园 (cnblogs.com)](https://www.cnblogs.com/wuchanming/p/4457358.html)
- 什么东西？直接动态规划求n个元素的和为SUM/2

- 重点是解法2的思路，解法3着实看不懂




### 区间重合判断

[编程之美--2.19区间重合判断 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4594989.html)

从源区间判断：所有覆盖区间是否能覆盖完源区间

从无序区间判断：预处理无序区间：排序 + 合并



### 程序理解和时间分析

[编程之美——2.20 程序理解和时间分析-CSDN博客](https://blog.csdn.net/a2796749/article/details/46464595)

- 第一：我直接就是读懂了程序
- 第二：我把自己当成电脑，去执行这段程序，看输出什么



### 只考加法的面试题

[《编程之美》2.21 只考加法的面试题 - 风言枫语 - 博客园 (cnblogs.com)](https://www.cnblogs.com/keanuyaoo/p/3281251.html)

[编程之美：2.21只考加法的面试题 - higirle - 博客园 (cnblogs.com)](https://www.cnblogs.com/Jessy/p/3479783.html)

- 好难好难
- 数学好难



## 第三章：结构之法

### 字符串移位包含的问题

- [编程之美3.1：字符串移位包含的问题_编程之美3.1字符串移位包含 java-CSDN博客](https://blog.csdn.net/zjwcdd/article/details/52635133)
- 空间换时间思路

- 暴力解法就是循环移位源字符串，看一下匹不匹配的上新字符串

- 优化法就是复制源字符串拼接到自身末尾，然后KMP找目标串




### 电话号码对应英语单词

- [编程之美--3.2电话号码对应英语单词（含多重循环变单重循环的思路） - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4603337.html)
- 回溯算法经典问题

- 回溯问题其实就是树的遍历问题，我们可以通过循环来解决，也可以利用递归实现




### 计算字符串的相似度

- [计算字符串的相似度--编程之美3.3 - tzc_yujunyong - 博客园 (cnblogs.com)](https://www.cnblogs.com/yujunyong/articles/2004724.html)
- 其实转换到最后就是动态规划求最长公共子序列问题

- 不过这之后还是要处理如何转化到相同字符串




### 从无头单链表中删除节点

- [编程之美--3.4从无头单链表中删除节点 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4605674.html)
- 非常简单的链表问题而已




### 最短摘要的生成

- [《编程之美》读书笔记24： 3.5 最短摘要的生成 - flyinghearts - 博客园 (cnblogs.com)](https://www.cnblogs.com/flyinghearts/archive/2011/03/24/1994453.html)
- 最主要还是理解这个最短摘要的意思

- 其实就是最小化目标字符串数组在搜索文本中的间距




### 编程判断两个链表是否相交

- [编程之美：编程判断两个链表是否相交 - youxin - 博客园 (cnblogs.com)](https://www.cnblogs.com/youxin/p/3303172.html)
- 链表1的每个节点在不在链表2中

- 利用哈希表存储某个链表的全部节点

- 如果两个无环链表相交，那么他们的最后一个节点肯定是相同的




### 队列中取最大值操作问题

- [编程之美--3.7队列中取最大值操作问题 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4605764.html)
- 用堆实现，不过这样就涉及到任一堆中元素的删除操作

- 用栈维护一个最大值序列，记录上一个最大值索引




### 求二叉树中节点的最大距离

- [编程之美--3.8求二叉树中节点的最大距离 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4606990.html)
- 本质上就是深搜找子树的高度，然后看距离

- 需要用全局变量记录最大距离，因为子树中的最大距离不一定经过根节点




### 重建二叉树

- [编程之美--3.9重建二叉树 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4608335.html)
- 递归调用原方法，通过一步步构建子树，搭建完整二叉树

- 必须有中序遍历结果




### 分层遍历二叉树

- [编程之美--3.10分层遍历二叉树 - 流了个火 - 博客园 (cnblogs.com)](https://www.cnblogs.com/gnivor/articles/4608524.html)
- 就层序遍历而已




### 程序改错

- [编程之美：第三章 结构之法 3.11程序改错_程序改错:求第三边的长度-CSDN博客](https://blog.csdn.net/qingyuanluofeng/article/details/47187737)
- 二分查找必须注意：循环退出条件，左右边界更新条件




## 第四章：数学之趣

### 金刚坐飞机问题

- [编程之美-金刚坐飞机问题-CSDN博客](https://blog.csdn.net/taoqick/article/details/103794829)
- 全概率公式基础下，分类讨论各个情况求概率问题




### 瓷砖覆盖地板

- [状态压缩动态规划 POJ 2411 （编程之美-瓷砖覆盖地板）_用a*a的瓷砖给m*n的地方铺路-CSDN博客](https://blog.csdn.net/hopeztm/article/details/7841917)
- 分类讨论 + 动态规划




### 买票找零

- [编程之美 4.3买票找零 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3357662.html)
- 转化成括号匹配问题，求解括号匹配的所有情况数




### 点是否在三角形内

- [编程之美 4.4点是否在三角形内 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3357768.html)
- 转化成面积判断 + 通过向量判断




### 磁带文件存放优化

- [编程之美 4.5磁带文件存放优化 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3358351.html)
- 关键就是求出最终的平均访问长度公式，然后以降低其为目标讨论放置要求




### 桶内取黑白球

- [编程之美：第四章 数字之趣 4.6桶中取黑白球_用计算机点击一个小桶出来一个小球-CSDN博客](https://blog.csdn.net/qingyuanluofeng/article/details/47187921)
- 数学建模能力，通过符号化题目中的操作，再进行数学分析




### 蚂蚁爬杆

- 化繁为简，其实碰头 = 擦肩而过

- [编程之美 4.7蚂蚁爬杆 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3359828.html)
- [编程之美:蚂蚁爬杆问题的扩展_蚂蚁爬竿问题扩展2-CSDN博客](https://blog.csdn.net/fangjian1204/article/details/38680879)



### 三角形测试用例

[编程之美：第四章 数字之趣 4.8三角形测试用例_一个byte有255种可能是什么-CSDN博客](https://blog.csdn.net/qingyuanluofeng/article/details/47187955)

测试用例的设计考虑：

- 基于问题的特性出发
- 正常的输入数据
- 异常的输入数据
- 中间值，边界值的处理情况



### 数独知多少

- [编程之美 4.9数独知多少 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3361014.html)
- 看不懂，哈哈




### 数字哑谜和回文

- [《编程之美》4.10数字哑谜和回文_编程之美 神奇的9位数-CSDN博客](https://blog.csdn.net/cjc211322/article/details/39137197)
- 程序能解的问题，尝试一下逻辑推理，也能处理
- [编程之美 4.10数学哑谜和回文 - Linka - 博客园 (cnblogs.com)](https://www.cnblogs.com/Linkabox/p/3361437.html)



### 挖雷游戏的概率

[4.11 挖雷游戏的概率 - BOTAK - 博客园 (cnblogs.com)](https://www.cnblogs.com/botak/p/14067477.html)

概率推理
