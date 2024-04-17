---
layout: post
title: "算法性能分析"
date: 2024-4-17
tags: [algorithm]
comments: true
author: zxy
---

## 时间复杂度

时间复杂度是**一个函数**，定性描述算法的运行时间。

时间复杂度估算程序运行的时间，在默认 CPU 的每个单元运行消耗的时间相同的情况下，用操作的**单元数量**来代表程序消耗的时间

假设问题规模是 n，操作的单元数量为 f(n)，随着 n 增大，程序运行时间和 f(n)的增长率相同，记为 O(f(n))

### 什么是大 O

算法导论：大 O 用来表示上界，也即算法最坏情况下的运行时间

一般情况：大部分数据情况下，也就是平均情况下的运行时间

### 不同数据规模的差异

在决定使用哪个算法时，不能**单纯看时间复杂度**（简化后的时间复杂度忽略常数项），而是要考虑**数据规模**。在数据规模很小时，使用 n^2 的算法有时比使用 n + C 的算法更快

在我们使用大 O 记法时，这时候我们考虑的**数据量级已经非常大，常数项系数已经不起作用了**(除非常数项也很大)

一般情况下默认数据规模巨大，此时算法复杂度排序为：
O(1) < O(logn) < O(n) < O(nlogn) < O(n^2) < O(n^3) < O(2^n)

### 复杂表达式简化

一般我们根据算法思路计算一个程序的时间复杂度会得到一个复杂的函数表达式，简化这个表达式是需要的：

去掉常数项 —— 去掉常数系数 ———— 只保留最高项

### O(logn)以什么为底

logn 可以以任意数字为底数，我们统一称为 logn 就是为了忽略底数的作用

举个例子：`以2为底n的对数 = 以2为底10的对数 * 以10为底n的对数`

所以 O(log`2`n) = log`2` 10 \* O(log`10`n) 一个常数系数直接忽略就行

### 举个例子看一下时间复杂度

题目描述：找出 n 个字符串中相同的两个字符串（假设这里只有两个相同的字符串）。

```
方法1：暴力枚举
举出两个字符串：双层for循环，要 n * n
比较两个字符串是否相同：逐一比较，要 m
所以总的复杂度为：O(m*n*n)
方法2：先排序，再比较
将每一个字符串按照字典序进行排序，那么相同的两个字符串排序后肯定相邻
排序时间复杂度：快排 n * logn * m
两两比较：n * m
总的时间复杂度是 n * m * (logn + 1)
大O记法：O(n*m*logn)
所以先排序更优
```

## 算法为什么会超时

算法超时：在规定的时间内，输入数据后，计算机没有计算出结果。或者换种思路，算法要操作的步骤 > 计算机在这个规定时间内能做的最大步骤

### 计算机性能

我们假设 1s 是规定的时间，那么 1s 内计算机能做的最大步骤是多少？

做实验计算 1s 内计算机**大致能进行多少运行时**有三点注意：

- CPU 执行每条指令所需的时间实际上并不相同，例如 CPU 执行加法和乘法操作的耗时实际上都是不一样的。
- 现在大多计算机系统的内存管理都有缓存技术，所以频繁访问相同地址的数据和访问不相邻元素所需的时间也是不同的。
- 计算机同时运行多个程序，每个程序里还有不同的进程线程在抢占资源。

我们以加法计算为例，考虑 O(n) , O(n^2),O(nlog n)的计算次数

测试代码：

```c++
#include <iostream>
#include <chrono>
#include <thread>
using namespace std;
using namespace chrono;
// O(n)
void function1(long long n) {
    long long k = 0;
    for (long long i = 0; i < n; i++) {
        k++;
    }
}

// O(n^2)
void function2(long long n) {
    long long k = 0;
    for (long long i = 0; i < n; i++) {
        for (long j = 0; j < n; j++) {
            k++;
        }
    }

}
// O(nlogn)
void function3(long long n) {
    long long k = 0;
    for (long long i = 0; i < n; i++) {
        for (long long j = 1; j < n; j = j*2) { // 注意这里j=1
            k++;
        }
    }
}
int main() {
    long long n; // 数据规模
    while (1) {
        cout << "输入n：";
        cin >> n;
        milliseconds start_time = duration_cast<milliseconds >(
            system_clock::now().time_since_epoch()
        );
        function1(n);
//        function2(n);
//        function3(n);
        milliseconds end_time = duration_cast<milliseconds >(
            system_clock::now().time_since_epoch()
        );
        cout << "耗时:" << milliseconds(end_time).count() - milliseconds(start_time).count()
            <<" ms"<< endl;
    }
}

```

| 时间复杂度 | 数据规模     |
| ---------- | ------------ |
| O(n)       | 5 \* 10^8    |
| O(n^2)     | 2.25 \* 10^4 |
| O(nlogn)   | 2 \* 10^7    |

不同进程数，不同电脑算出来的肯定都不同的，只是数量级大差不差

### 递归算法的时间复杂度

```c++
面试题：求x的n次方 （重点关注时间复杂度，超不超int范围就先不考虑）

（1）for循环，O(n)
int function1(int x, int n) {
    int result = 1;  // 注意 任何数的0次方等于1
    for (int i = 0; i < n; i++) {
        result = result * x;
    }
    return result;
}

（2）递归算法，时间复杂度O(n)
int function2(int x, int n) {
    if (n == 0) {
        return 1; // return 1 同样是因为0次方是等于1的
    }
    return function2(x, n - 1) * x;
}
递归n次，每次操作一个乘法 n * 1

（3）递归算法，时间复杂度O(n)
int function3(int x, int n) {
    if (n == 0) return 1;
    if (n == 1) return x;

    if (n % 2 == 1) {
    	//还剩奇数次
        return function3(x, n / 2) * function3(x, n / 2)*x;
    }
    //还剩偶数次
    return function3(x, n / 2) * function3(x, n / 2);
}
这个算法其实就是二分，每次将要计算的指数对半砍，可以用二叉树模拟
由于每次递归都是只做1个乘法，所以时间复杂度就是递归次数（二叉树的节点数）
如果二叉树的深度为m，从0开始，那么节点数为： 2^m + 2^m-1 + ... + 2^0 = 2^m+1 - 1
由于要计算n次方，所有 m = log2n - 1，代入上式，节点数为n-1

（4）递归算法，时间复杂度O(logn)
int function4(int x, int n) {
    if (n == 0) return 1;
    if (n == 1) return x;
    int t = function4(x, n / 2);// 这里相对于function3，是把这个递归操作抽取出来
    if (n % 2 == 1) {
        return t * t * x;
    }
    return t * t;
}
每次二分，时间复杂度为O(logn)
```

递归算法时间复杂度：**递归的次数 \* 每次递归中的操作次数**

## 空间复杂度

空间复杂度不是程序本身的文件大小，而是程序运行时占用的内存大小

空间复杂度只是大致评估程序运行时会占用的内存，精确的内存占用受太多因素影响

```c++
O(1)的空间复杂度
int j = 0;
for (int i = 0; i < n; i++) {
    j++;
}

O(n)的空间复杂度
int* a = new int(n);
for (int i = 0; i < n; i++) {
    a[i] = i;
}

O(logn)的空间复杂度
递归次数为logn时会出现
```

### 递归算法的空间，时间复杂度

```c++
递归求斐波那契数列的性能分析
(1)递归
int fibonacci(int i) {
       if(i <= 0) return 0;
       if(i == 1) return 1;
       return fibonacci(i-1) + fibonacci(i-2);
}
时间复杂度：每次递归都是O(1)操作（加法而已），主要看递归次数
由于这个函数逻辑是将i用i-1和i-2的递归调用来计算，那么其实就是二叉树
递归次数 = 二叉树节点 一颗高度为k（根节点为1）的二叉树最多有 2^k - 1个节点
所以时间复杂度为O(2^n)，空间复杂度O(n)

(2)记录前两次的值
int fibonacci(int first, int second, int n) {
    if (n <= 0) {
        return 0;
    }
    if (n < 3) {
        return 1;
    }
    else if (n == 3) {
        return first + second;
    }
    else {
        return fibonacci(second, first + second, n - 1);
    }
}
时间复杂度O(n),空间复杂度O(n) n次递归，每次都要传递两个数字
```

**递归算法的空间复杂度 = 每次递归的空间复杂度 \* 递归深度**

```c++
int binary_search( int arr[], int l, int r, int x) {
    if (r >= l) {
        int mid = l + (r - l) / 2;
        if (arr[mid] == x)
            return mid;
        if (arr[mid] > x)
            return binary_search(arr, l, mid - 1, x);
        return binary_search(arr, mid + 1, r, x);
    }
    return -1;
}
时间复杂度：O(logn) 每次递归只做判断 O(1) 二分递归O(logn)
空间复杂度：O(logn) 递归深度O(logn) 每次传参：O(1)
```

## 代码的内存消耗

### C++内存管理

一个 C++程序占用的内存分为下面几个部分：

- 栈区：编译器自动分配释放，存放函数参数值，局部变量值等
- 堆区：由程序员分配释放，人不主动释放，程序运行结束由 OS 收回
- 未初始化数据区：存放未初始化的全局变量和静态变量
- 初始化数据区：存放初始化后的全局变量和静态变量
- 程序代码区：函数体的二进制代码

最主要的内存消耗还是在堆区和栈区

### 如何计算程序占用多大内存

先要了解数据类型的大小，以 64 位编译器为例：

| char | short | int | long | float | double | 指针 | 单位 |
| ---- | ----- | --- | ---- | ----- | ------ | ---- | ---- |
| 1    | 2     | 4   | 8    | 4     | 8      | 8    | byte |

注意指针占用大小的确定：8byte 就是 64 位，那么就可以支持 2^64 的内存寻址，覆盖了 64 位 OS 的内存大小

### 内存对齐

**不要以为只有 C/C++才会有内存对齐，只要可以跨平台的编程语言都需要做内存对齐，Java、Python 都是一样的**。

内存对齐主要有两个原因：

1. 平台原因：不是所有的硬件平台都能访问任意内存地址上的任意数据，某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。为了同一个程序可以在多平台运行，需要内存对齐。
2. 硬件原因：经过内存对齐后，CPU 访问内存的速度大大提升。

```
struct node{
   int num;
   char cha;
}st;
sizeof(st)的输出为8 > 4 + 1 这就是因为内存对齐

    CPU读取内存不是一次读取单个字节，而是一块一块的来读取内存，块的大小可以是2，4，8，16个字节，具体取多少个字节取决于硬件。
    假设CPU把内存划分为4字节大小的块，要读取一个4字节大小的int型数据，来看一下这两种情况下CPU的工作量：
    内存对齐：
    0	1	2	3	4	5	6	7
    char             int int int int
                   一次寻址就可以读4、5、6、7byte，获取数据
     内存不对齐：
    0	1	2	3	4	5	6	7
    char int int int int
    因为CPU是四个字节四个字节来寻址，首先CPU读取0，1，2，3处的四个字节数据
    CPU读取4，5，6，7处的四个字节数据
    合并地址1，2，3，4处四个字节的数据才是本次操作需要的int数据
```

虽然内存对齐会浪费一定的内存资源，但是一般来说内存资源是充足的，我们更希望提高速度
