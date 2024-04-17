---
layout: post
title: "编程素养"
date: 2024-4-17
tags: [algorithm]
comments: true
author: zxy
---

### 代码规范

C++语言的命名规范按照 Google 为主

**变量命名**

| 命名方法   | 举例                                      | 适用语言                          |
| ---------- | ----------------------------------------- | --------------------------------- |
| 小驼峰命名 | int myAge                                 | java,go,C++                       |
| 大驼峰     | 类名，函数名，属性名 public class MyAge   | java,go,C++的函数和结构体         |
| 下划线     | int my_age                                | Python 和 Linux 下的 C/C++        |
| 匈牙利命名 | int iMyAge 变量名= 属性 + 类型 + 对象描述 | Windows 下 C/C++，没有 IDE 时好用 |

**水平留白（空格）**

```c++
(1)操作符左右一定有空格
	i = i + 1;
(2)分隔符（,  ;）前一位没有空格，后一位保持空格.
	int i, j;
	for (int fastIndex = 0; fastIndex < nums.size(); fastIndex++)
(3)大括号和函数保持同一行，并有一个空格
    while (n) {
      n--;
	}
(4)控制语句（while，if，for）后都有一个空格
    while (n) {
    	if (k > 0) return 9;
    	n--;
	}
```

代码规范没有统一的标准，C++一般就遵循 Google 规范。

工作中最重要的是融入团队

### 使用库函数的时机

如果题目的**关键部分**可以有**库函数直接解决**，那么就不要使用库函数，不然这题目就没有训练意义了

使用库函数最重要的是清楚库函数内部的**实现原理**还有它的**时间复杂度**

### LeetCode 代码的本地编译运行

调试 LeetCode 代码，一般来说 debug 就直接的方法就是打印 log，把关键的变量打印出来看看就知道什么问题

```c++
#include <iostream>
#include <vector>
using namespace std;

class Solution {
public:
    int minCostClimbingStairs(vector<int>& cost) {
        vector<int> dp(cost.size());
        dp[0] = cost[0];
        dp[1] = cost[1];
        for (int i = 2; i < cost.size(); i++) {
            dp[i] = min(dp[i - 1], dp[i - 2]) + cost[i];
        }
        return min(dp[cost.size() - 1], dp[cost.size() - 2]);
    }
};

int main() {
    int a[] = {1, 100, 1, 1, 1, 100, 1, 1, 100, 1};
    vector<int> cost(a, a + sizeof(a) / sizeof(int));
    Solution solution;
    cout << solution.minCostClimbingStairs(cost) << endl;
}
```

### 核心代码模式和 ACM 模式

**核心代码模式**：就是 LeetCode 的模式，直接实现关键函数即可，不涉及主函数，库函数的编写与调用

**ACM 模式**：从 0 开始，头文件，main 函数，关键函数，数据的输入和输出都要自己处理

#### ACM 模式的输入输出练习（满足 Google 代码规范）

##### （1）A + B 问题 I

###### 题目描述

你的任务是计算 a+b。

###### 输入描述

输入包含一系列的 a 和 b 对，通过空格隔开。一对 a 和 b 占一行。

###### 输出描述

对于输入的每对 a 和 b，你需要依次输出 a、b 的和。

如对于输入中的第二对 a 和 b，在输出中它们的和应该也在第二行。

###### 输入示例

```
3 4
11 40
```

###### 输出示例

```
7
51
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int a, b;
    while (cin >> a >> b) {
        cout << a + b << endl;
    }
    return 0;
}
```

##### （2） A+B 问题 II

###### 题目描述

计算 a+b，但输入方式有所改变。

###### 输入描述

第一行是一个整数 N，表示后面会有 N 行 a 和 b，通过空格隔开。

###### 输出描述

对于输入的每对 a 和 b，你需要在相应的行输出 a、b 的和。

如第二对 a 和 b，对应的和也输出在第二行。

###### 输入示例

```
2
2 4
9 21
```

###### 输出示例

```
6
30
```

###### 提示信息

注意，测试数据不仅仅一组。也就是说，会持续输入 N 以及后面的 a 和 b

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int N, a, b;
    while (cin >> N) {
        for (int i = 0; i < N; i++) {
            cin >> a >> b;
            cout << a + b << endl;
        }
    }
    return 0;
}
```

##### （3）A+B 问题 III

###### 题目描述

你的任务依然是计算 a+b。

###### 输入描述

输入中每行是一对 a 和 b。其中会有一对是 0 和 0 标志着输入结束，且这一对不要计算。

###### 输出描述

对于输入的每对 a 和 b，你需要在相应的行输出 a、b 的和。

如第二对 a 和 b，他们的和也输出在第二行。

###### 输入示例

```
2 4
11 19
0 0
```

###### 输出示例

```
6
30
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int a, b;
    while (cin >> a >> b) {
        if (a == 0 && b == 0) {
            break;
        }
        cout << a + b << endl;
    }
    return 0;
}
```

##### （4）A+B 问题 IV

###### 题目描述

你的任务是计算若干整数的和。

###### 输入描述

每行的第一个数 N，表示本行后面有 N 个数。

如果 N=0 时，表示输入结束，且这一行不要计算。

###### 输出描述

对于每一行数据需要在相应的行输出和。

###### 输入示例

```
4 1 2 3 4
5 1 2 3 4 5
0
```

###### 输出示例

```
10
15
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int N;
    while (cin >> N) {
        if (!N) {
            break;
        }
        int tmp, sum = 0;
        while (N--) {
            cin >> tmp;
            sum += tmp;
        }
        cout << sum << endl;
    }
    return 0;
}
```

##### （5）A+B 问题 VII

###### 题目描述

你的任务是计算两个整数的和。

###### 输入描述

输入包含若干行，每行输入两个整数 a 和 b，由空格分隔。

###### 输出描述

对于每组输入，输出 a 和 b 的和，每行输出后接一个空行。

###### 输入示例

```
2 4
11 19
```

###### 输出示例

```
6

30
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int a, b;
    while (cin >> a >> b) {
        cout << a + b << endl;
        cout << endl;
    }
    return 0;
}
```

##### （6）A+B 问题 VIII

###### 题目描述

你的任务是计算若干整数的和。

###### 输入描述

输入的第一行为一个整数 N，接下来 N 行每行先输入一个整数 M，然后在同一行内输入 M 个整数。

###### 输出描述

对于每组输入，输出 M 个数的和，每组输出之间输出一个空行。

###### 输入示例

```
3
4 1 2 3 4
5 1 2 3 4 5
3 1 2 3
3
4 1 2 3 4
5 1 2 3 4 5
3 1 2 3
```

###### 输出示例

```
10

15

6
10

15

6
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int N, M;
    while (cin >> N) {
        for (int i = 0; i < N; i++) {
            cin >> M;
            int tmp, sum = 0;
            while (M--) {
                cin >> tmp;
                sum += tmp;
            }
            cout << sum <<endl;
            if (i == N - 1) {
                continue;
            }
            cout << endl;
        }
    }
    return 0;
}
```

##### （7）平均绩点

###### 题目描述

每门课的成绩分为 A、B、C、D、F 五个等级，为了计算平均绩点，规定 A、B、C、D、F 分别代表 4 分、3 分、2 分、1 分、0 分。

###### 输入描述

有多组测试样例。每组输入数据占一行，由一个或多个大写字母组成，字母之间由空格分隔。

###### 输出描述

每组输出结果占一行。如果输入的大写字母都在集合｛A,B,C,D,F｝中，则输出对应的平均绩点，结果保留两位小数。否则，输出“Unknown”。

###### 输入示例

```
A B C D F
B F F C C A
D C E F
```

###### 输出示例

```
2.00
1.83
Unknown
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;
#include <iomanip>

int main() {
    string s;
    while (getline(cin, s)) {
        int flag = 1;
        double sum = 0;
        double cnt = 0;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == 'A') {
                cnt++;
                sum += 4;
            }else if (s[i] == 'B') {
                cnt++;
                sum += 3;
            }else if (s[i] == 'C') {
                cnt++;
                sum += 2;
            }else if (s[i] == 'D') {
                cnt++;
                sum += 1;
            }else if (s[i] == 'F') {
                cnt++;
                sum += 0;
            }else if (s[i] == ' ') {
                continue;
            }else{
                flag = 0;
                cout << "Unknown" << endl;
                break;
            }
        }
        if (flag) {
            cout << fixed << setprecision(2) << sum / cnt << endl;
        }
    }
    return 0;
}
```

##### （8）摆平积木

###### 题目描述

小明很喜欢玩积木。一天，他把许多积木块组成了好多高度不同的堆，每一堆都是一个摞一个的形式。然而此时，他又想把这些积木堆变成高度相同的。但是他很懒，他想移动最少的积木块来实现这一目标，你能帮助他吗？

![img](https://kamacoder.com/upload/kamacoder.com/20230718//1007_1_20230718172742_56975.bmp)

###### 输入描述

输入包含多组测试样例。每组测试样例包含一个正整数 n，表示小明已经堆好的积木堆的个数。
接着下一行是 n 个正整数，表示每一个积木堆的高度 h，每块积木高度为 1。其中 1<=n<=50,1<=h<=100。测试数据保证积木总数能被积木堆数整除。当 n=0 时，输入结束。

###### 输出描述

对于每一组数据，输出将积木堆变成相同高度需要移动的最少积木块的数量。
在每组输出结果的下面都输出一个空行。

###### 输入示例

```
6
5 2 4 1 7 5
0
```

###### 输出示例

```
5
```

###### AC 代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    int n;
    while (cin >> n) {
        if (!n) {
            break;
        }
        vector<int> height(n);
        int i = 0, sum = 0;
        while (i < n) {
            cin >> height[i];
            sum += height[i];
            i++;
        }
        int average = sum / n;
        int ans = 0;
        for (int j = 0; j < n; j++){
            if (height[j] > average) {
                ans += height[j] - average;
            }
        }
        cout << ans << endl;
        cout << endl;
    }
    return 0;
}
```

##### （9）奇怪的信

###### 题目描述

有一天, 小明收到一张奇怪的信, 信上要小明计算出给定数各个位上数字为偶数的和。
例如：5548，结果为 12，等于 4 + 8 。小明很苦恼，想请你帮忙解决这个问题。

###### 输入描述

输入数据有多组。每组占一行，只有一个整整数，保证数字在 32 位整型范围内。

###### 输出描述

对于每组输入数据，输出一行，每组数据下方有一个空行。

###### 输入示例

```
415326
3262
```

###### 输出示例

```
12

10
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int i;
    while (cin >> i) {
        int ans = 0;
        while (i) {
            int tmp = i%10;
            if (tmp%2 == 0) {
                ans += tmp;
            }
            i /= 10;
        }
        cout << ans << endl << endl;
    }
}
```

##### （10）运营商活动

###### 题目描述

小明每天的话费是 1 元，运营商做活动，手机每充值 K 元就可以获赠 1 元话费，一开始小明充值 M 元，问最多可以用多少天？ 注意赠送的话费也可以**全部**参与到奖励规则中

###### 输入描述

输入包括多个测试实例。每个测试实例包括 2 个整数 M，K（2<=k<=M<=1000)。M=0，K=0 代表输入结束。

###### 输出描述

对于每个测试实例输出一个整数，表示 M 元可以用的天数。

###### 输入示例

```
2 2
4 3
13 3
0 0
```

###### 输出示例

```
3
5
19
```

###### 提示信息

注意第三组数据「13 3」结果为什么是 19 呢， 13/3=4，获得 4 元奖励。 13%3=1，还剩下 1 元，4+1=5，5 元继续参加奖励规则。 5/3=1，获得 1 元奖励。 5%3=2，剩下 2 元，1+2=3，3 元继续参与奖励规则。 3/3=1，获得 1 元奖励。 3%3=0，剩下 0 元，1+0=1。 1 元不能参与剩下奖励。所以一共可以使用的天数是 13+4+1+1=19

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int M, K;
    while (cin >> M >> K) {
        if (!M && !K) {
            break;
        }
        int ans = 0;
        while (M >= K) {
            int left = M%K, add = M/K;
            ans += M - left;
            M = left + add;
        }
        cout << ans + M << endl;
    }
    return 0;
}
```

##### （11）共同祖先

###### 题目描述

小明发现和小宇有共同祖先！现在小明想知道小宇是他的长辈，晚辈，还是兄弟。

###### 输入描述

输入包含多组测试数据。每组首先输入一个整数 N（N<=10），接下来 N 行，每行输入两个整数 a 和 b，表示 a 的父亲是 b（1<=a,b<=20）。小明的编号为 1，小宇的编号为 2。
输入数据保证每个人只有一个父亲。

###### 输出描述

对于每组输入，如果小宇是小明的晚辈，则输出“You are my younger”，如果小宇是小明的长辈，则输出“You are my elder”，如果是同辈则输出“You are my brother”。

###### 输入示例

```
5
1 3
2 4
3 5
4 6
5 6
6
1 3
2 4
3 5
4 6
5 7
6 7
```

###### 输出示例

```
You are my elder
You are my brother
```

###### AC 代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    int N, a, b;
    while (cin >> N) {
        vector<int> tree(21);
        for (int i = 1; i < 21; i++) {
            tree[i] = i;
        }
        while (N--) {
            cin >> a >> b;
            tree[a] = b;
        }
        int MingToAnc = 0, YuToAnc = 0;
        int Ming = 1, Yu = 2;
        while (tree[Ming] != Ming) {
            MingToAnc++;
            Ming = tree[Ming];
        }
        while (tree[Yu] != Yu) {
            YuToAnc++;
            Yu = tree[Yu];
        }
        if (MingToAnc > YuToAnc) {
            cout << "You are my elder" << endl;
        }else if (MingToAnc == YuToAnc) {
            cout << "You are my brother" << endl;
        }else {
            cout << "You are my younger" << endl;
        }
    }
    return 0;
}
```

##### （12）打印数字图形

###### 题目描述

先要求你从键盘输入一个整数 n（1<=n<=9），打印出指定的数字图形。

###### 输入描述

输入包含多组测试数据。每组输入一个整数 n（1<=n<=9）。

###### 输出描述

对于每组输入，输出指定的数字图形。
注意：每行最后一个数字后没有任何字符。

###### 输入示例

```
5
```

###### 输出示例

```
    1
   121
  12321
 1234321
123454321
 1234321
  12321
   121
    1
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int n;
    while (cin >> n){
        int i = 1;
        int maxLen = 2 * n - 1;
        while (i <= n) {
            //空格
            int k = n - i;
            while (k--) {
                cout << " ";
            }
            //数字
            int j = 1;
            while (j <= i) {
                cout << j;
                j++;
            }
            j -= 2;
            while (j >= 1) {
                cout << j;
                j--;
            }
            cout << endl;
            i++;
        }
        i -= 2;
        while (i >= 1) {
            //空格
            int k = n - i;
            while (k--) {
                cout << " ";
            }
            //数字
            int j = 1;
            while (j <= i) {
                cout << j;
                j++;
            }
            j -= 2;
            while (j >= 1) {
                cout << j;
                j--;
            }
            cout <<endl;
            i--;
        }
    }
    return 0;
}
```

##### （13）镂空三角形

###### 题目描述

把一个字符三角形掏空，就能节省材料成本，减轻重量，但关键是为了追求另一种视觉效果。在设计的过程中，需要给出各种花纹的材料和大小尺寸的三角形样板，通过电脑临时做出来，以便看看效果。

###### 输入描述

每行包含一个字符和一个整数 n(0<n<41)，不同的字符表示不同的花纹，整数 n 表示等腰三角形的高。显然其底边长为 2n-1。如果遇到@字符，则表示所做出来的样板三角形已经够了。

###### 输出描述

每个样板三角形之间应空上一行，三角形的中间为空。行末没有多余的空格。每条结果后需要再多输出一个空行。

###### 输入示例

```
X 2
A 7
@
```

###### 输出示例

```
 X
XXX

      A
     A A
    A   A
   A     A
  A       A
 A         A
AAAAAAAAAAAAA
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    char pattern;
    int n;
    while (cin >> pattern) {
        if (pattern == '@') {
            break;
        }
        cin >> n;
        //行数
        int i = 1;
        while (i <= n) {
            //前置空格数
            int preK = n - i;
            int j = 1;
            while (j <= preK) {
                cout << " ";
                j++;
            }
            if (i == 1) {
                cout << pattern << endl;
            }else if (i == n) {
                int j = 1;
                while (j <= 2 * n - 1) {
                    cout << pattern;
                    j++;
                }
                cout << endl;
            }else {
                cout << pattern;
                int m = 1;
                while (m <= 2 * (i - 1) - 1) {
                    cout << " ";
                    m++;
                }
                cout << pattern << endl;
            }
            i++;
        }
        cout << endl;
    }
}
```

##### （14）句子缩写

###### 题目描述

输出一个词组中每个单词的首字母的大写组合。

###### 输入描述

输入的第一行是一个整数 n，表示一共有 n 组测试数据。（输入只有一个 n，没有多组 n 的输入）
接下来有 n 行，每组测试数据占一行，每行有一个词组，每个词组由一个或多个单词组成；每组的单词个数不超过 10 个，每个单词有一个或多个大写或小写字母组成；
单词长度不超过 10，由一个或多个空格分隔这些单词。

###### 输出描述

请为每组测试数据输出规定的缩写，每组输出占一行。

###### 输入示例

```
1
ad dfa     fgs
```

###### 输出示例

```
ADF
```

###### 提示信息

注意：单词之间可能有多个空格

###### AC 代码：

```c++
#include <iostream>
#include <string>
using namespace std;

int main() {
    int n;
    string s;
    cin >> n;
    getchar(); //度过换行符
    while (n--) {
        getline(cin, s);
        string ans;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] != ' '){
                if (s[i] >= 'a' && s[i] <= 'z') {
                    ans += 'A' + s[i] - 'a';
                }else {
                    ans += s[i];
                }
                while (i < s.size() && s[i] != ' ') {
                    i++;
                }
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```

##### （15）神秘字符

###### 题目描述

考古学家发现墓碑上有神秘的字符。经过仔细研究，发现原来这是开启古墓入口的方法。
墓碑上有 2 行字符串，其中第一个串的长度为偶数，现在要求把第 2 个串插入到第一个串的正中央，如此便能开启墓碑进入墓中。

###### 输入描述

输入数据首先给出一个整数 n，表示测试数据的组数。（整个输入中，只有一个 n）
然后是 n 组数据，每组数据 2 行，每行一个字符串，长度大于 0，小于 50，并且第一个串的长度必为偶数。

###### 输出描述

请为每组数据输出一个能开启古墓的字符串，每组输出占一行。

###### 输入示例

```
2
asdf
yu
rtyu
HJK
```

###### 输出示例

```
asyudf
rtHJKyu
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    string first, second;
    int n;
    cin >> n;
    getchar();
    while (n--) {
        getline(cin, first);
        getline(cin, second);
        int pos = first.size() / 2;
        string ans;
        for (int i = 0; i < pos; i++) {
            ans += first[i];
        }
        ans += second;
        for (int i = pos; i < first.size(); i++) {
            ans += first[i];
        }
        cout << ans <<endl;
    }
}
```

##### （16）位置互换

###### 题目描述

给定一个长度为偶数位的字符串，请编程实现字符串的奇偶位互换。

###### 输入描述

输入包含多组测试数据。
输入的第一行是一个整数 n，表示有测试数据。（整个输入中，只有一个 n）
接下来是 n 组测试数据，保证串长为偶数位(串长<=50)。

###### 输出描述

请为每组测试数据输出奇偶位互换后的结果，每组输出占一行。

###### 输入示例

```
2
0aa0
bb00
```

###### 输出示例

```
a00a
bb00
```

###### AC 代码：

```c++
#include <iostream>
using namespace std;

int main() {
    int n;
    string s;
    cin >> n;
    getchar();
    while (n--) {
        getline(cin, s);
        for (int i = 0; i < s.size() - 1; i += 2) {
            char tmp = s[i+1];
            s[i+1] = s[i];
            s[i] = tmp;
        }
        cout << s << endl;
    }
}
```

##### （17）出栈合法性

###### 题目描述

已知自然数 1，2，...，N（1<=N<=100）依次入栈，请问序列 C1，C2，...，CN 是否为合法的出栈序列。

###### 输入描述

输入包含多组测试数据。
每组测试数据的第一行为整数 N（1<=N<=100），当 N=0 时，输入结束。
第二行为 N 个正整数，以空格隔开，为出栈序列。

###### 输出描述

对于每组输入，输出结果为一行字符串。
如给出的序列是合法的出栈序列，则输出 Yes，否则输出 No。

###### 输入示例

```
5
3 4 2 1 5
5
3 5 1 4 2
0
```

###### 输出示例

```
Yes
No
```

###### AC 代码：

```c++
#include <iostream>
#include <stack>
using namespace std;

int main() {
    int N;
    while (cin >> N) {
        if (!N) {
            break;
        }
        stack<int> stk;
        int flag = 1;
        //当前进栈数字
        int i = 1;
        while (N--) {
            //当前出栈数字
            int cur;
            cin >> cur;
            while (i <= cur) {
                stk.push(i);
                i++;
            }
            if (stk.empty()) {
                flag = 0;
                continue;
            }
            if (stk.top() == cur) {
                stk.pop();
            }else {
                flag = 0;
                continue;
            }
        }
        if (flag) {
            cout << "Yes" << endl;
        }else {
            cout << "No" <<endl;
        }
    }
    return 0;
}
```

##### （18）链表的基本操作

###### 题目描述

本题考察链表的基本操作。温馨提示：本题较为复杂，需要细心多花一些时间。

###### 输入描述

输入数据只有一组，第一行有 n+1 个整数，第一个整数是这行余下的整数数目 n，后面是 n 个整数。

这一行整数是用来初始化列表的，并且输入的顺序与列表中的顺序相反，也就是说如果列表中是 1、2、3 那么输入的顺序是 3、2、1。

第二行有一个整数 m，代表下面还有 m 行。每行有一个字符串，字符串是“get”，“insert”，“delete”，“show”中的一种。

- 如果是“get”，代表获得第 a 个元素。（a 从 1 开始计数）
- 如果是“delete”，代表删除第 a 个元素。（a 从 1 开始计数）
- 如果是“insert”，则其后跟着两个整数 a 和 e，代表在第 a 个位置前面插入 e。（a 从 1 开始计数）
- “show”之后没有正数，直接打印链表全部内容。

###### 输出描述

- 如果获取成功，则输出该元素；
- 如果删除成功，则输出“delete OK”；
- 如果获取失败，则输出“get fail”
- 如果删除失败，则输出“delete fail”
- 如果插入成功，则输出“insert OK”，否则输出“insert fail”。
- 如果是“show”，则输出列表中的所有元素，如果列表是空的，则输出“Link list is empty”

注：所有的双引号均不输出。

###### 输入示例

```
3 3 2 1
21
show
delete 1
show
delete 2
show
delete 1
show
delete 2
insert 2 5
show
insert 1 5
show
insert 1 7
show
insert 2 5
show
insert 3 6
show
insert 1 8
show
get 2
```

###### 输出示例

```
1 2 3
delete OK
2 3
delete OK
2
delete OK
Link list is empty
delete fail
insert fail
Link list is empty
insert OK
5
insert OK
7 5
insert OK
7 5 5
insert OK
7 5 6 5
insert OK
8 7 5 6 5
7
```

###### 提示信息

初始化链表的元素是倒序的，这个使用题目中创建列表的方法（从头部插入）就可以了。

###### AC 代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    int n, m;
    cin >> n;
    vector<int> list(n);
    while (n--) {
        cin >> list[n];
    }
    getchar();
    cin >> m;
    getchar();
    string s;
    while (m--) {
        getline(cin, s);
        if (s[0] == 'g') {
            //get语句
            int idx = s[4] - '0' - 1;
            if (idx < 0 || idx >= list.size()) {
                cout << "get fail" << endl;
            }else {
                cout << list[idx] << endl;
            }
        }else if (s[0] == 'i') {
            //insert语句
            int idx = s[7] - '0' - 1;
            int num = s[9] - '0';
            if (idx < 0 || idx > list.size()) {
                cout << "insert fail" << endl;
            }else {
                list.insert(list.begin() + idx, num);
                cout << "insert OK" << endl;
            }
        }else if (s[0] == 'd') {
            //delete语句
             int idx = s[7] - '0' - 1;
             if (idx < 0 || idx >= list.size()) {
                 cout << "delete fail" << endl;
             }else {
                 list.erase(list.begin() + idx);
                 cout << "delete OK" << endl;
             }
        }else if (s[0] == 's') {
            //show语句
            if (!list.size()) {
                cout << "Link list is empty" << endl;
            }else {
                for (int i = 0; i < list.size(); i++) {
                    cout << list[i] << " ";
                }
                cout << endl;
            }
        }
    }
}
```

##### （19）单链表反转

###### 题目描述

根据一个整数序列构造一个单链表，然后将其反转。

例如：原单链表为 2 3 4 5 ，反转之后为 5 4 3 2

###### 输入描述

输入包括多组测试数据，每组测试数据占一行，第一个为大于等于 0 的整数 n，表示该单链表的长度，后面跟着 n 个整数，表示链表的每一个元素。整数之间用空格隔开

###### 输出描述

针对每组测试数据，输出包括两行，分别是反转前和反转后的链表元素，用空格隔开

如果链表为空，则只输出一行，list is empty

###### 输入示例

```
5 1 2 3 4 5
0
```

###### 输出示例

```
1 2 3 4 5
5 4 3 2 1
list is empty
```

###### 提示信息

本题用数组，也是可有过的，输入一遍数组，然后倒叙输出。不过建议大家用本题来链表操作

###### AC 代码：

```c++
#include <iostream>
using namespace std;

struct listNode{
  int val;
  listNode* next;
  listNode(int _val) {
      val = _val;
      next = NULL;
  }
};

listNode* reverse(listNode* head){
    listNode* prev;
    listNode* cur;
    listNode* last;
    prev = NULL;
    cur = head;
    while(cur){
        last = cur->next;
        cur->next = prev;
        prev = cur;
        cur = last;
    }
    return prev;
}

int main() {
    int n;
    while (cin >> n) {
        if (!n) {
            cout << "list is empty" << endl;
            getchar();
            continue;
        }
        listNode* head;
        listNode* cur;
        int i = 0;
        while (i < n) {
            int tmp;
            cin >> tmp;
            listNode* newnode = new listNode(tmp);
            if (i == 0) {
                head = newnode;
                cur = head;
                i++;
                continue;
            }
            i++;
            cur -> next = newnode;
            cur = newnode;
        }
        getchar();
        cur = head;
        while (cur) {
            cout << cur->val << " ";
            cur = cur -> next;
        }
        cout << endl;
        head = reverse(head);
        cur = head;
        while (cur) {
            cout << cur->val << " ";
            cur = cur -> next;
        }
        cout << endl;
    }
   return 0;
}
```

##### （20）删除重复元素

###### 题目描述

根据一个递增的整数序列构造有序单链表，删除其中的重复元素

###### 输入描述

输入包括多组测试数据，每组测试数据占一行，第一个为大于等于 0 的整数 n，表示该单链表的长度，后面跟着 n 个整数，表示链表的每一个元素。整数之间用空格隔开

###### 输出描述

针对每组测试数据，输出包括两行，分别是删除前和删除后的链表元素，用空格隔开

如果链表为空，则只输出一行，list is empty

###### 输入示例

```
5 1 2 3 4 5
5 1 1 2 2 3
0
```

###### 输出示例

```
1 2 3 4 5
1 2 3 4 5
1 1 2 2 3
1 2 3
list is empty
```

###### AC 代码：

```c++
#include <iostream>
#include <set>
#include <vector>
using namespace std;

int main() {
    int n;
    while (cin >> n) {
        if (!n) {
            cout << "list is empty" <<endl;
            continue;
        }
        vector<int> origin(n);
        set<int> m;
        for (int i = 0; i < n; i++) {
            cin >> origin[i];
            m.insert(origin[i]);
        }
        for (int i = 0; i < n; i++) {
            cout << origin[i] << " ";
        }
        cout << endl;
        for (auto i : m) {
            cout << i << " ";
        }
        cout << endl;
    }
}
```

##### （21）构造二叉树

###### 题目描述

给你一棵二叉树的前序遍历和中序遍历结果，要求你写出这棵二叉树的后序遍历结果。

###### 输入描述

输入包含多组测试数据。每组输入包含两个字符串，分别表示二叉树的前序遍历和中序遍历结果。每个字符串由不重复的大写字母组成。

###### 输出描述

对于每组输入，输出对应的二叉树的后续遍历结果。

###### 输入示例

```
DBACEGF ABCDEFG
BCAD CBAD
```

###### 输出示例

```
ACBFGED
CDAB
```

###### AC 代码：

```c++
#include <iostream>
#include <unordered_map>
using namespace std;

struct treeNode {
    char val;
    treeNode* left;
    treeNode* right;
    treeNode(char _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }
};

treeNode* reBuild(string& pre, string& mid, int preL, int preR, int midL, int midR, unordered_map<char, int>& hash) {
    if (preL > preR) {
        return NULL;
    }
    char rootVal = pre[preL];
    int midIdx = hash[rootVal];
    int leftTreeLen = midIdx - midL;
    treeNode* root = new treeNode(rootVal);
    root->left = reBuild(pre, mid, preL + 1, preL + leftTreeLen, midL, midIdx - 1, hash);
    root->right = reBuild(pre, mid, preL + leftTreeLen + 1, preR, midIdx + 1, midR, hash);
    return root;
}

void postOrder(treeNode* root) {
    if (!root) {
        return;
    }
    postOrder(root->left);
    postOrder(root->right);
    cout << root->val;
}

int main() {
    string s;
    while (getline(cin, s)) {
        int flag = 0;
        string pre, mid;
        unordered_map<char, int> hash;
        for (int i = 0; i < s.size(); i++) {
            while (flag == 0 && s[i] != ' ') {
                pre += s[i];
                i++;
            }
            if (s[i] == ' ') {
                flag = 1;
            }
            while (flag == 1 && i < s.size() && s[i] != ' ') {
                mid += s[i];
                i++;
            }
        }
        for (int i = 0; i < mid.size(); i++) {
            hash[mid[i]] = i;
        }
        treeNode* root = reBuild(pre, mid, 0, pre.size() - 1, 0, mid.size() - 1, hash);
        postOrder(root);
        cout << endl;
    }
}
```

##### （22）二叉树的遍历

###### 题目描述

给出一个 n 个节点的二叉树，请求出二叉树的前序遍历，中序遍历和后序遍历。

###### 输入描述

题目包含多行输入，第一行为一个整数 N，表示二叉树的节点总数，后面将有 N 行对应于二叉树的 N 个节点的信息。

**每行的数据格式遵循如下规则：**

- 每行开始是该节点的标识符，代表二叉树中具体的一个节点。
- 节点标识符之后是两个数字，分别表示该节点的左孩子和右孩子的序号。序号根据输入顺序从 1 开始自动分配，**即第一个输入的节点序号为 1，第二个为 2，以此类推。**
- 若某个节点的左孩子或右孩子序号为 0，则表示该节点不存在相应的左孩子或右孩子。

###### 输出描述

共三行，第一行为二叉树的前序遍历，第二行为中序遍历，第三行为后序遍历

###### 输入示例

```
7
F 2 3
C 4 5
E 0 6
A 0 0
D 7 0
G 0 0
B 0 0
```

###### 输出示例

```
FCADBEG
ACBDFEG
ABDCGEF
```

###### 提示信息

样例输入解释如下： 首行数字 7 表示二叉树共有 7 个节点。接下来的行按序提供每个节点的信息，

格式为：标识符 左孩子序号 右孩子序号。**节点序号自动从 1 依次递增，对应输入的行顺序。**

例如：

- F 2 3 表示标识符为 F 的节点（序号为 1）有左孩子（序号为 2）和右孩子（序号为 3）；
- E 0 6 表示标识符为 E 的节点（序号为 3）无左孩子，有右孩子（序号为 6）。
- 由于 F 的右孩子是序号为 3 的节点，而 E 的序号刚好为 3，所以 F 的右孩子就是 E。

###### AC 代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

struct treeNode {
    char val;
    treeNode* left;
    treeNode* right;
    treeNode(char _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }
};

void preOrder(treeNode* root) {
    if (!root) {
        return;
    }
    cout << root->val;
    preOrder(root->left);
    preOrder(root->right);
}

void midOrder(treeNode* root) {
    if (!root) {
        return;
    }
    midOrder(root->left);
    cout << root->val;
    midOrder(root->right);
}

void postOrder(treeNode* root) {
    if (!root) {
        return;
    }
    postOrder(root->left);
    postOrder(root->right);
    cout << root->val;
}

int main() {
    int N;
    cin >> N;
    vector<treeNode*> tree(N + 1);
    vector<pair<int, int>> child(N + 1);
    int i = 1, flag = N;
    while (N--) {
        char val;
        int leftChild, rightChild;
        cin >> val >> leftChild >> rightChild;
        treeNode* newnode = new treeNode(val);
        tree[i] = newnode;
        child[i++] = make_pair(leftChild, rightChild);
    }
    for (int i = 1; i <= flag; i++) {
        int leftChild, rightChild;
        leftChild = child[i].first;
        rightChild = child[i].second;
        if (leftChild) {
            tree[i]->left = tree[leftChild];
        }
        if (rightChild) {
            tree[i]->right = tree[rightChild];
        }
    }
    preOrder(tree[1]);
    cout << endl;
    midOrder(tree[1]);
    cout << endl;
    postOrder(tree[1]);
    return 0;
}
```

##### （23）二叉树的高度

###### 题目描述

现给定一棵二叉树的先序遍历序列和中序遍历序列，要求你计算该二叉树的高度。

###### 输入描述

输入包含多组测试数据，每组输入首先给出正整数 N（<=50），为树中结点总数。下面 2 行先后给出先序和中序遍历序列，均是长度为 N 的不包含重复英文字母（区别大小写）的字符串。

###### 输出描述

对于每组输入，输出一个整数，即该二叉树的高度。

###### 输入示例

```
9
ABDFGHIEC
FDHGIBEAC
7
Abcdefg
gfedcbA
```

###### 输出示例

```
5
7
```

###### AC 代码：

```c++
#include <iostream>
#include <unordered_map>
using namespace std;

struct treeNode {
    char val;
    treeNode* left;
    treeNode* right;
    treeNode(char _val) {
        val = _val;
        left = NULL;
        right = NULL;
    }
};

treeNode* reBuild(string& pre, string& mid, int preL, int preR, int midL, int midR, unordered_map<char, int>& hash) {
    if (preL > preR) {
        return NULL;
    }
    char rootVal = pre[preL];
    int midIdx = hash[rootVal];
    int leftTreeLen = midIdx - midL;
    treeNode* root = new treeNode(rootVal);
    root->left = reBuild(pre, mid, preL + 1, preL + leftTreeLen, midL, midIdx - 1, hash);
    root->right = reBuild(pre, mid, preL + leftTreeLen + 1, preR, midIdx + 1, midR, hash);
    return root;
}

int height(treeNode* root) {
    if (!root) {
        return 0;
    }
    if (!root->left && !root->right) {
        return 1;
    }
    return max(height(root->left), height(root->right)) + 1;
}

int main() {
    int N;
    while (cin >> N) {
        getchar();
        string pre, mid;
        getline(cin, pre);
        getline(cin, mid);
        unordered_map<char, int> hash;
        for (int i = 0; i < mid.size(); i++) {
            hash[mid[i]] = i;
        }
        treeNode* root = reBuild(pre, mid, 0, pre.size() - 1, 0, mid.size() - 1, hash);
        cout << height(root) << endl;
    }
}
```

##### （24）最长公共子序列

###### 题目描述

给你一个序列 X 和另一个序列 Z，当 Z 中的所有元素都在 X 中存在，并且在 X 中的下标顺序是严格递增的，那么就把 Z 叫做 X 的子序列。
例如：Z=是序列 X=的一个子序列，Z 中的元素在 X 中的下标序列为<1,2,4,6>。
现给你两个序列 X 和 Y，请问它们的最长公共子序列的长度是多少？

###### 输入描述

输入包含多组测试数据。每组输入占一行，为两个字符串，由若干个空格分隔。每个字符串的长度不超过 100。

###### 输出描述

对于每组输入，输出两个字符串的最长公共子序列的长度。

###### 输入示例

```
abcfbc abfcab
programming contest
abcd mnp
```

###### 输出示例

```
4
2
0
```

###### AC 代码：

```c++
#include <iostream>
#include <vector>
using namespace std;

int main() {
    string s;
    while (getline(cin, s)) {
        int flag = 0;
        string pre, mid;
        for (int i = 0; i < s.size(); i++) {
            while (flag == 0 && s[i] != ' ') {
                pre += s[i];
                i++;
            }
            if (s[i] == ' ') {
                flag = 1;
            }
            while (flag == 1 && i < s.size() && s[i] != ' ') {
                mid += s[i];
                i++;
            }
        }
        int m = pre.size(), n = mid.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        for (int i = 1; i < m + 1; i++) {
            for (int j = 1; j < n + 1; j++) {
                if (pre[i - 1] == mid[j - 1]) {
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else {
                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
                }
            }
        }
        cout << dp[m][n] << endl;
    }
}
```

##### （25）最爱的城市

###### 题目描述

一天小明捧着一本世界地图在看，突然小明拿起笔，将他最爱的那些城市标记出来，并且随机的将这些城市中的某些用线段两两连接起来。
小明量出了每条线段的长度，现在小明想知道在这些线段组成的图中任意两个城市之间的最短距离是多少。

###### 输入描述

输入包含多组测试数据。
每组输入第一行为两个正整数 n（n<=10）和 m（m<=n\*(n-1)/2），n 表示城市个数，m 表示线段个数。
接下来 m 行，每行输入三个整数 a，b 和 l，表示 a 市与 b 市之间存在一条线段，线段长度为 l。（a 与 b 不同）
每组最后一行输入两个整数 x 和 y，表示问题：x 市与 y 市之间的最短距离是多少。（x 与 y 不同）
城市标号为 1~n，l<=20。

###### 输出描述

对于每组输入，输出 x 市与 y 市之间的最短距离，如果 x 市与 y 市之间非连通，则输出“No path”。

###### 输入示例

```
4 4
1 2 4
1 3 1
1 4 1
2 3 1
2 4
```

###### 输出示例

```
3
```

###### AC 代码：

```c++
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

int main() {
    int n, m;
    while (cin >> n >> m) {
        vector<vector<pair<int, int>>> edges(n + 1);
        while (m--) {
            int a, b, l;
            cin >> a >> b >> l;
            edges[a].push_back(make_pair(b, l));
            edges[b].push_back(make_pair(a, l));
        }
        int x, y;
        cin >> x >> y;
        vector<int> dist(n + 1, 250);
        vector<bool> visit(n + 1, false);
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> que;
        que.push({0, x});
        dist[x] = 0;
        while (!que.empty()) {
            pair<int, int> top = que.top();
            int from = top.second;
            int cost = top.first;
            que.pop();
            if (visit[from]) {
                continue;
            }
            visit[from] = true;
            for (int i = 0; i < edges[from].size(); i++) {
                int to = edges[from][i].first;
                int cost = edges[from][i].second;
                if (dist[to] > dist[from] + cost) {
                    dist[to] = dist[from] + cost;
                    que.push({dist[to], to});
                }
            }
        }
        if (dist[y] == 250) {
            cout << "No path" << endl;
        }else {
            cout << dist[y] << endl;
        }
    }
}
```

### 从数组构造二叉树（LeetCode 中的示例如何转化为二叉树）

传入的数组是：{4,1,6,0,2,5,7,-1,-1,-1,3,-1,-1,-1,8} ， 这里是用 -1 来表示 null

从这个数组构造一颗二叉树

```c++
#include <iostream>
#include <vector>
#include <queue>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

// 根据数组构造二叉树
TreeNode* construct_binary_tree(const vector<int>& vec) {
    vector<TreeNode*> vecTree (vec.size(), NULL);
    TreeNode* root = NULL;
    for (int i = 0; i < vec.size(); i++) {
        TreeNode* node = NULL;
        if (vec[i] != -1) node = new TreeNode(vec[i]);
        vecTree[i] = node;
        if (i == 0) root = node;
    }
    for (int i = 0; i * 2 + 1 < vec.size(); i++) {
        if (vecTree[i] != NULL) {
            vecTree[i]->left = vecTree[i * 2 + 1];
            if(i * 2 + 2 < vec.size())
            vecTree[i]->right = vecTree[i * 2 + 2];
        }
    }
    return root;
}

// 层序打印打印二叉树
void print_binary_tree(TreeNode* root) {
    queue<TreeNode*> que;
    if (root != NULL) que.push(root);
    vector<vector<int>> result;
    while (!que.empty()) {
        int size = que.size();
        vector<int> vec;
        for (int i = 0; i < size; i++) {
            TreeNode* node = que.front();
            que.pop();
            if (node != NULL) {
                vec.push_back(node->val);
                que.push(node->left);
                que.push(node->right);
            }
            // 这里的处理逻辑是为了把null节点打印出来，用-1 表示null
            else vec.push_back(-1);
        }
        result.push_back(vec);
    }
    for (int i = 0; i < result.size(); i++) {
        for (int j = 0; j < result[i].size(); j++) {
            cout << result[i][j] << " ";
        }
        cout << endl;
    }
}

int main() {
    // 注意本代码没有考虑输入异常数据的情况
    // 用 -1 来表示null
    vector<int> vec = {4,1,6,0,2,5,7,-1,-1,-1,3,-1,-1,-1,8};
    TreeNode* root = construct_binary_tree(vec);
    print_binary_tree(root);
}
```

### 互联网项目研发流程

#### （1）需求文档

作用：

- 一方面是**倒逼产品经理去系统性的思考这个需求究竟有哪些功能**，用来满足哪些用户的需求。
- 另一方面是**保证我们在研发的时候，研发出来的功能是满足需求文档里所描述的**。

#### （2）这个需求包含哪些功能

从代码的角度上来讲，可能要增添很多功能才能满足 用户看起来好像微不足道的小功能。

例如点击登录，点击下单，后台都有很多模块协同运行的。

我们要把产品经理角度上的这个功能，拆解为我们代码角度上的具体应该开发的那些功能。

#### （3）确定有哪些难点

给大家举一个例子，给你一个需求文档。

你说你一周的时间就能把它开发完，那为什么是一周呢，为什么不是两天，为什么不是两周呢。

其实 和上面的领导汇报你的工作的时候 **都要把自己的工作进行量化**。

那么这个功能有哪些难点，我们要克服这个难点，所需要花费的时间，都要有一个大体的量化。

这样才能量化我们自己的工作，**领导其实不知道你的工作是简单 还是困难, 领导只在意最终结果**，所以你需要展现给领导你的工作是有难度的是有挑战的。

而且**这些也是我们年底用来晋升或者评职称的素材**。

如果这些东西你自己都不在乎的话，谁还会帮你在乎呢。

而且**确定了自己的工作难点，把这些难点都记录下来，对以后跳槽也很有帮助**。

面试官最喜欢问的问题，就是：**你做的项目中有哪些难点？以及你是如何克服的**。

所以这一步对自己个人成长来说也是很有重要的。 对于项目组来说也是记录下来，每一个迭代版本有哪些难点，这些难点团队是如何克服的。

这也是项目组往上一级去邀功的资料。

#### （4）画架构图

一般来说，大厂项目的架构都是比较复杂的，特别是后端架构。

如果添加一个模块连个文档都没有，连个图也没有，上来就添加的话，后面的人是很难维护的。

而且每个模块和每一个模块之间的依赖关系，如果没有画出一个架构图的话，直接看代码很容易直接就看懵了。

为什么你可以快速开发一个新的模块，也是因为之前团队有人把这个架构图画清楚了，你只需要看一眼这个架构图，就知道你的模块应该添加在哪里。

那么你去添加模块的时候，也应该把这个架构图相应的位置 完善一下。

同时呢，在画架构图的过程中，也增添了自己对整个系统架构的掌握程度。

这个图也会让你确定，你的模块在整个项目中扮演一个什么样的角色。

#### （5）定协议

后台模块之间进行通讯需要协议，后台和前端通讯也需要协议。

所以只要有交互，就要确定协议的数据格式。

**定协议要考虑到兼容，要考虑易于维护**。

#### （6）设计数据结构与算法

其实设计数据结构更多一些，因为我们要选择使用什么容器，什么格式来处理我们的数据。

什么快排，二叉树，动态规划，最短路啥的，在实际开发中，都不需要我们自己去写，**直接调包！**

#### （7）预估容量

特别是后端开发，要估计出 我们自己模块大体需要多大磁盘，多大内存，多大带宽，多少核 CPU。

这也是没有做过研发工作的同学经常忽略的，**因为大家好像默认 磁盘、内存、带宽、cpu 是无穷的**。

其实我们在设计一个模块的时候，这些都要评估的，不能模块一上线，把机器直接打爆了。

例如 直接把带宽打满了，不仅影响自己的模块功能，还影响了机器上其他模块的运行。

#### （8）考虑部署

要考虑如果一台机器挂了（可能是硬件原因），那么我们的模块还能不能正常提供服务。

这就是考虑模块的容灾性，一般都是采用分布式，服务部署在三台机器上，一台如果挂了，还有其他两台提供服务。

还有就是要弹性可伸缩，即我们的模块可不可以直接 部署多台机器来提高承载能力。

如果用户量突然上来了，或者流量突然上来了，可以通过快速部署多台机器来抗住流量。

而不是模块只能在单机上跑，多部署几台就发生各种问题。

**这才能说明是有足够强的风险意识的**。

#### （9）设计评审

前八的阶段其实都是设计阶段，那么你的设计需要让组里的同学一起来评审一下，看看有没有什么问题。

大家主要也是看看你的模块 会不会给其他模块或者整个系统带来什么问题 以及 设计的是否合理。

#### （10）写代码

#### （11）自测功能

#### （12）联调

自己的模块可能会涉及到其他模块之间的一个交互，或者和前端的一个交互。

所以需要其他同学配合一起来测试。

这里就有很多沟通工作了，因为其他同学可能手头有自己的活，那么就要协调一个时间来一起测试。

这一步也是很费时间的，**其费时关键是要等，要等其他同学有空和你联调或者是别人等你**，而且往往不是联调一次就能解决问题的。

所以 在评估开发时间的时候 也要考虑到联调的时间。

这也是大厂研发效率低的地方，但上百人开发的项目，**这种沟通上消耗也是在所难免的**。

#### （13）交给测试

自己的代码，自己测 一般都测不出什么问题，需要交给测试同学来给你测一测。

这里如果测试同学测出问题，你就要判断确实有问题还是 测试方式不对，如果有问题就要修改，再提给测试，反反复复这么几轮，直到测试同学测试没问题了。

**这个过程也是累心的**。

#### （14）code review

代码合入主干之前，需要 项目组的同学一起来评审一下你的代码。

之前是评审设计，看设计上有没有什么缺失，这次是大家来看看你代码写的怎么样。

例如合入主干会不会有什么问题，代码兼容性做的好不好，接口设计的好不好，甚至字段，函数，变量名，命名合不合理。

都要经过大家的评审，如果有问题就还是要改。

如果没有问题一般 大家会给+2（就是通过的意思），这样就可以合入主干了。

#### （15）合入主干

合入主干为什么会单独列出来呢。

其实合入主干是很重要的，经常是自己的代码没问题，但合入主干之后就有问题了。

一般就是合入主干的时候有冲突，例如你从主干拉出一个分支，另一个同学从主干拉出一个分支，而且两个分支修改了同一个模块，如果另一个同学提前合入主干，你再合入主干的时候就会有代码冲突。

在解决代码冲突的时候 就会修改别人的代码，这个过程很容易产生新的 bug。

**一般合入主干之后，测试同学还要重新跑一个全量测试，才能放心发布**。

如果跑全量测试没有问题的话，才会松一口气（懂的人都懂）。

#### （16）发布

最后一步就是发布了。

发布其实就是把主干的代码更新到线上的服务器上。

一些还没有工作的同学，可能不太理解究竟什么是发布。

用大白话来讲，就是把 本地的代码或者某台机器的代码，编译成可执行文件，然后更新到 线上的服务器（一个独立的集群，专门处理线上的流量）并运行起来。

上线是最重要的一步了，也很容易出问题，因为一个大型项目，其上线的过程都非常复杂（要更新上百台机器的集群），而且**要考虑线上新版和旧版本的兼容问题**。

这也是为什么大厂项目都选择深夜上线，**因为深夜在线用户最少，如果出问题，影响的用户会比较少，可以快速修复**。

所以大家经常看到 某大厂程序员深夜上线发布版本之类的。
