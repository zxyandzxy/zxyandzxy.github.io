---
layout: post
title: "算法核心框架"
date: 2024-7-23
tags: [labuladong]
comments: true
author: zxy
---

## 学习算法和数据结构的框架思维

### 数据结构的存储方式

**数据结构的存储方式只有两种：数组（顺序存储）和链表（链式存储）**。

多样化的数据结构，究其源头，都是在链表或者数组上的特殊操作，API 不同而已。

- 比如说「队列」、「栈」这两种数据结构既可以使用链表也可以使用数组实现。用数组实现，就要**处理扩容缩容**的问题；用链表实现，没有这个问题，但需要更多的**内存空间存储节点**指针。
- 「图」的两种表示方法，邻接表就是链表，邻接矩阵就是二维数组。邻接矩阵判断连通性迅速，并可以进行**矩阵运算解决一些问题**，但是如果图比较稀疏的话很耗费空间。邻接表比较节省空间，但是很多**操作的效率**上肯定比不过邻接矩阵。
- 「散列表」就是通过散列函数把键**映射到一个大数组**里。而且对于解决散列冲突的方法，拉链法需要链表特性，操作简单，但需要额外的空间存储指针；线性探查法就需要数组特性，以便连续寻址，不需要指针的存储空间，但操作稍微复杂些。
- 「树」，用**数组实现就是「堆」**，因为「堆」是一个完全二叉树，用数组存储不需要节点指针，操作也比较简单；用链表实现就是很常见的那种「树」，因为不一定是完全二叉树，所以不适合用数组存储。为此，在这种链表「树」结构之上，又衍生出各种巧妙的设计，比如二叉搜索树、AVL 树、红黑树、区间树、B 树等等，以应对不同的问题。

Redis 提供列表、字符串、集合等等几种常用数据结构，但是对于每种数据结构，底层的存储方式都至少有两种，以便于根据存储数据的实际情况使用合适的存储方式。

综上，数据结构种类很多，甚至你也可以发明自己的数据结构，但是底层存储无非数组或者链表，**二者的优缺点如下**：

**数组**

- 是紧凑连续存储,可以随机访问，通过索引快速找到对应元素，而且相对节约存储空间。
- 但正因为连续存储，内存空间必须一次性分配够，所以说数组如果要扩容，需要重新分配一块更大的空间，再把数据全部复制过去，时间复杂度 O(N)；
- 而且你如果想在数组中间进行插入和删除，每次必须搬移后面的所有数据以保持连续，时间复杂度 O(N)。

**链表**

- 因为元素不连续，而是靠指针指向下一个元素的位置，所以不存在数组的扩容问题；
- 如果知道某一元素的前驱和后驱，操作指针即可删除该元素或者插入新元素，时间复杂度 O(1)。
- 但是正因为存储空间不连续，你无法根据一个索引算出对应元素的地址，所以不能随机访问；
- 而且由于每个元素必须存储指向前后元素位置的指针，会消耗相对更多的储存空间。

### 数据结构的基本操作

> 为什么有那么多种数据结构？

现实中有各种各样的场景，他们需要不同的数据存储和访问方式。只有**对问题做适配的数据结构**，才能高效地在不同应用场景中，尽可能**高效地增删改查。**

> 数据结构的基本操作是什么?

就两个：**遍历 + 访问**。 具体到应用场景就是增删改查。

要进行遍历 + 访问，到编程语言上有两种方式：**线性 + 非线性**

线性以`for循环`和`while循环`为代表。 非线性以`递归`为代表

```cpp
// 数组遍历框架

// 线性for循环为代表
void traverse(vector<int>& arr) {
    for (int i = 0; i < arr.size(); i++) {
        // 访问 arr[i]
    }
}

// 非线性递归为代表
void traverse(vector<int>& arr, int idx) {
    if (idx >= arr.size()) {
        return;
    }
    // 访问 arr[i]
    // 递归访问
    traverse(arr, idx + 1);
}
```

```cpp
// 链表遍历框架
class listNode {
	public:
    	int val;
    	listNode* next;
};

// 线性for循环
void travsere(listNode* head) {
    for (listNode* cur = head; cur != nullptr; cur = cur->next) {
        // 访问 cur->val
    }
}

// 递归非线性
void travserse(listNode* head) {
    if (head == nullptr) {
        return;
    }
    // 访问 head->val
    // 递归下推
    traverse(head->next);
}
```

```cpp
// 二叉树遍历框架 
// 递归非线性 (线性的二叉树用数组存储，不过这种一般是完全二叉树，不具备一般性)
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
};

void preTraverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 访问 root->val
    preTraverse(root->left);
    preTraverse(root->right);
}

void inTraverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    preTraverse(root->left);
    // 访问 root->val
    preTraverse(root->right);
}

void postTraverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    preTraverse(root->left);
    preTraverse(root->right);
    // 访问 root->val
}
```

```cpp
// 可以观察到，二叉树和链表 用内存指针节点存储的方式是非常像的。
// 抽象出来其实就是 一个节点的子节点数量
// N 叉树 存储和遍历模板
class TreeNode {
    public:
    	int val;
    	vector<TreeNode*> children;
};

void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 访问节点 root->val
    // 访问子节点
    for (int i = 0; i < root->children.size(); i++) {
        traverse(root->children[i]);
    }
}

/*
N叉树 进一步拓展就是 图（图可以理解为多个N叉树的结合体）
如果有环，用visited做下标记
*/
class GraphNode {
    public:
    	int val;
    	vector<GraphNode*> neighbor;
}

unordered_set<GraphNode*> visited;

void traverse(GraphNode* cur) {
    // 访问当前节点 cur->val
    for (GraphNode* neighbor : cur->neighbor) {
        if (visited.find(neighbor) == visited.end()) {
            visited.insert(neighbor);
            traverse(neighbor);
        }
    }
}
```

### 算法刷题指南

- 数组、链表这种基本数据结构及其常用算法
- 二叉树培养框架思维，所有递归问题都是树的遍历问题
- 回溯（N叉树的后序遍历问题），动态规划（遍历树问题），分治等算法

## 算法的本质

在CS领域内，有很多种算法。对于编程来说，可以概括为三种：

- 数据结构与算法：这类算法的本质是穷举。通过用数据结构抽象，化简问题，枚举所有情况，找到问题答案。
- 脑筋急转弯算法：这类算法的本质是规律。通过观察题目，找寻规律，找到最优解法
- 数学算法：这类算法的本质是数学。通过数学原理解决问题，再用编程语言描述数学解法

> 两种工程师

- 算法工程师：重点在于数学建模和调参经验
- 开发工程师：重点在于站在计算机视角，抽象简化实际问题，再用合理的数据结构解决

> 两种思维

- 数学思维：观察问题，找到几何关系，列方程，求解答案
- 计算机思维：在复杂度允许的情况下，枚举所有情况，找到答案

> 穷举的关键

穷举的关键在于：**无冗余 + 无遗漏**

- 冗余：代码如果将完全相同的计算流程重复十遍，那么运行速度就非常低下了
- 遗漏：代码如果遗漏了某种情况，那么答案是极其有可能出错的
- 无遗漏：穷举所有可能的解
- 无冗余：避免所有冗余的计算，消耗尽可能少的资源

> 什么算法难在 如何穷举？（无遗漏）

**递归类问题：比如回溯算法，动态规划系列**

回溯算法：人**手动穷举的过程抽象成程序化的规律** 这个过程是非常难的。我们解决的方式是将回溯问题抽象为一颗多叉树，然后通过**遍历**这棵树的所有节点来穷举所有可能。

动态规划算法：动态规划的思维模式是`分解问题`，`分解问题`的思维就是**树上只有一片叶子和剩下的叶子**。我们对于动态规划的解法思维是用数学归纳法，明确函数的定义，分解问题，然后利用这个定义递归求解子问题。

> 什么算法难在 如何聪明地穷举？（无冗余）

这里就没有什么思维模式了，主要就是针对具体问题，有大神想出来高效穷举方式，咱直接学

- 数组中找元素 - 二分查找
- 高效计算连通分量 - 并查集
- 贪心算法 - 发现贪心选择性质（不用枚举所有情况就能找到答案）
- 字符串匹配 - KMP算法

### 数组/单链表系列算法

- 双指针
- 二分搜索
- 滑动窗口
- 回文串
- 前缀和
- 差分数组

### 二叉树系列算法

核心就是两个思维模式：**遍历树 + 分解问题**

具体到题目解法就是：回溯算法 + 动态规划 + 分治算法 + BFS + 图论算法

## 双指针技巧解决链表题目

### 合并两个有序链表

> 题目

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/03/merge_ex1.jpg)

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

**示例 2：**

```
输入：l1 = [], l2 = []
输出：[]
```

**示例 3：**

```
输入：l1 = [], l2 = [0]
输出：[0]
```

**提示：**

- 两个链表的节点数目范围是 `[0, 50]`
- `-100 <= Node.val <= 100`
- `l1` 和 `l2` 均按 **非递减顺序** 排列

> 思路

![img](https://labuladong.online/algo/images/%E9%93%BE%E8%A1%A8%E6%8A%80%E5%B7%A7/1.gif)

> 技巧

虚拟头节点(dummy)：用虚拟头节点可以**避免处理链表为空**的情况，同时将l1,l2的起始节点**统一纳入了处理过程**，这就极大降低了代码复杂性

技巧核心：**当你需要创造一条新链表的时候，可以使用虚拟头结点简化边界情况的处理**。

> 代码

```cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    // 虚拟头结点
    ListNode* dummy = new ListNode(-1), *p = dummy;
    ListNode* p1 = l1, *p2 = l2;
    
    while (p1 != NULL && p2 != NULL) {
        // 比较 p1 和 p2 两个指针
        // 将值较小的的节点接到 p 指针
        if (p1->val > p2->val) {
            p->next = p2;
            p2 = p2->next;
        } else {
            p->next = p1;
            p1 = p1->next;
        }
        // p 指针不断前进
        p = p->next;
    }
    // 一条链表到底了， 那么剩余的肯定就是另一条链表
    if (p1 != NULL) {
        p->next = p1;
    }
    
    if (p2 != NULL) {
        p->next = p2;
    }
    
    return dummy->next;
}
```

### 单链表分解

> 题目

给你一个链表的头节点 `head` 和一个特定值 `x` ，请你对链表进行分隔，使得所有 **小于** `x` 的节点都出现在 **大于或等于** `x` 的节点之前。你应当 **保留** 两个分区中每个节点的初始相对位置。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/01/04/partition.jpg)

```
输入：head = [1,4,3,2,5,2], x = 3
输出：[1,2,2,4,3,5]
```

**示例 2：**

```
输入：head = [2,1], x = 2
输出：[1,2]
```

**提示：**

- 链表中节点的数目在范围 `[0, 200]` 内
- `-100 <= Node.val <= 100`
- `-200 <= x <= 200`

> 思路

我们可以把原链表分成两个小链表，一个链表中的元素大小都小于 `x`，另一个链表中的元素都大于等于 `x`，最后再把这两条链表接到一起，就得到了题目想要的结果。

这里也是构造新链表，可以使用虚拟头节点

> 代码

```cpp
ListNode* partition(ListNode* head, int x) {
    // 存放小于 x 的链表的虚拟头结点
    ListNode *dummy1 = new ListNode(-1);
    // 存放大于等于 x 的链表的虚拟头结点
    ListNode *dummy2 = new ListNode(-1);
    // p1, p2 指针负责生成结果链表
    ListNode *p1 = dummy1, *p2 = dummy2;
    // p 负责遍历原链表，类似合并两个有序链表的逻辑
    // 这里是将一个链表分解成两个链表
    ListNode *p = head;
    while (p != NULL) {
        if (p->val >= x) {
            p2->next = p;
            p2 = p2->next;
        } else {
            p1->next = p;
            p1 = p1->next;
        }
        // 不能直接让 p 指针前进，
        // p = p.next
        // 断开原链表中的每个节点的 next 指针
        ListNode *temp = p->next;
        p->next = NULL;
        p = temp;
    }
    // 连接两个链表
    p1->next = dummy2->next;

    return dummy1->next;
}
```

```cpp
// 不能直接让 p 指针前进，
// p = p.next
// 断开原链表中的每个节点的 next 指针
ListNode temp = p.next;
p.next = null;
p = temp;

如果不断开链表中每个节点的next指针，那么最终结果中会包含一个环。
因为最终是需要拼接两个新链表的。
```

> 技巧

总的来说，**如果我们需要把原链表的节点接到新链表上，而不是 new 新节点来组成新链表的话，那么断开节点和原链表之间的链接可能是必要的。**那其实我们可以养成一个好习惯，但凡遇到这种情况，就把原链表的节点断开，这样就不会出错了。

### 合并k个有序链表

> 题目

给你一个链表数组，每个链表都已经按升序排列。请你将所有链表合并到一个升序链表中，返回合并后的链表。

**示例 1：**

```
输入：lists = [[1,4,5],[1,3,4],[2,6]]
输出：[1,1,2,3,4,4,5,6]
解释：链表数组如下：
[
  1->4->5,
  1->3->4,
  2->6
]
将它们合并到一个有序链表中得到。
1->1->2->3->4->4->5->6
```

**示例 2：**

```
输入：lists = []
输出：[]
```

**示例 3：**

```
输入：lists = [[]]
输出：[]
```

**提示：**

- `k == lists.length`
- `0 <= k <= 10^4`
- `0 <= lists[i].length <= 500`
- `-10^4 <= lists[i][j] <= 10^4`
- `lists[i]` 按 **升序** 排列
- `lists[i].length` 的总和不超过 `10^4`

> 思路

合并 `k` 个有序链表的逻辑类似合并两个有序链表，**难点在于，如何快速得到 `k` 个节点中的最小节点**，接到结果链表上？

这里我们就要用到 **优先级队列** 这种数据结构，把链表节点放入一个最小堆，就可以每次获得 `k` 个节点中的最小节点：

> 代码

```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    if (lists.empty()) return nullptr;
    // 虚拟头结点
    ListNode* dummy = new ListNode(-1);
    ListNode* p = dummy;
    // 优先级队列，最小堆
    priority_queue<ListNode*, vector<ListNode*>, function<bool(ListNode*, ListNode*)>> pq(
        [] (ListNode* a, ListNode* b) { return a->val > b->val; });
    // 将 k 个链表的头结点加入最小堆
    for (auto head : lists) {
        if (head != nullptr) {
            pq.push(head);
        }
    }

    while (!pq.empty()) {
        // 获取最小节点，接到结果链表中
        ListNode* node = pq.top();
        pq.pop();
        p->next = node;
        if (node->next != nullptr) {
            pq.push(node->next);
        }
        // p 指针不断前进
        p = p->next;
    }
    return dummy->next;
}
```

优先队列 `pq` 中的元素个数最多是 `k`，所以一次 `poll` 或者 `add` 方法的时间复杂度是 `O(logk)`；所有的链表节点都会被加入和弹出 `pq`，**所以算法整体的时间复杂度是 `O(Nlogk)`，其中 `k` 是链表的条数，`N` 是这些链表的节点总数**。

### 单链表的倒数第k个节点

> 题目

从前往后寻找单链表的第 `k` 个节点很简单，一个 for 循环遍历过去就找到了，但是如何寻找从后往前数的第 `k` 个节点呢？

> 思路

![img](https://labuladong.online/algo/images/%E9%93%BE%E8%A1%A8%E6%8A%80%E5%B7%A7/1.jpeg)

首先，我们先让一个指针 `p1` 指向链表的头节点 `head`，然后走 `k` 步，现在的 `p1`，只要再走 `n - k` 步，就能走到链表末尾的空指针了对吧？趁这个时候，再用一个指针 `p2` 指向链表头节点 `head`，接下来就很显然了，让 `p1` 和 `p2` 同时向前走，`p1` 走到链表末尾的空指针时前进了 `n - k` 步，`p2` 也从 `head` 开始前进了 `n - k` 步，停留在第 `n - k + 1` 个节点上，即恰好停链表的倒数第 `k` 个节点上。

> 代码

```cpp
// 返回链表的倒数第 k 个节点
ListNode* findFromEnd(ListNode* head, int k) {
    ListNode* p1 = head;
    // p1 先走 k 步
    for (int i = 0; i < k; i++) {
        p1 = p1 -> next;
    }
    ListNode* p2 = head;
    // p1 和 p2 同时走 n - k 步
    while (p1 != nullptr) {
        p2 = p2 -> next;
        p1 = p1 -> next;
    }
    // p2 现在指向第 n - k + 1 个节点，即倒数第 k 个节点
    return p2;
}

时间复杂度: O (N)
```

### 单链表的中点

> 题目

给你一个链表，找寻链表的中间节点，一趟完成

> 思路

我们让两个指针 `slow` 和 `fast` 分别指向链表头结点 `head`。

**每当慢指针 `slow` 前进一步，快指针 `fast` 就前进两步，这样，当 `fast` 走到链表末尾时，`slow` 就指向了链表中点**。

> 代码

```cpp
ListNode* middleNode(ListNode* head) {
    // 快慢指针初始化指向 head
    ListNode* slow = head;
    ListNode* fast = head;
    // 快指针走到末尾时停止
    while (fast != nullptr && fast->next != nullptr) {
        // 慢指针走一步，快指针走两步
        slow = slow->next;
        fast = fast->next->next;
    }
    // 慢指针指向中点
    return slow;
}
```

### 判断链表是否包含环

> 题目

给你一个链表，判断链表中是否有环存在

> 思路

每当慢指针 `slow` 前进一步，快指针 `fast` 就前进两步。

如果 `fast` 最终能正常走到链表末尾，说明链表中没有环；如果 `fast` 走着走着竟然和 `slow` 相遇了，那肯定是 `fast` 在链表中转圈了，说明链表中含有环。

> 代码

```cpp
bool hasCycle(ListNode* head) {
    // 初始化快慢指针，指向头结点
    ListNode* slow = head;
    ListNode* fast = head;
    // 快指针到尾部时停止
    while (fast && fast->next) {
        // 慢指针走一步，快指针走两步
        slow = slow->next;
        fast = fast->next->next;
        // 快慢指针相遇，说明含有环
        if (slow == fast) {
            return true;
        }
    }
    // 不包含环
    return false;
}
```

```cpp
// 进阶：如果存在环，怎么找环的起点
ListNode* detectCycle(ListNode* head) {
    ListNode* fast = head;
    ListNode* slow = head;
    while (fast != nullptr && fast->next != nullptr) {
        fast = fast->next->next;
        slow = slow->next;
        if (fast == slow) break;

    }
    // 上面的代码类似 hasCycle 函数
    if (fast == nullptr || fast->next == nullptr) {
        // fast 遇到空指针说明没有环
        return nullptr;
    }

    // 重新指向头结点
    slow = head;

    // 快慢指针同步前进，相交点就是环起点
    /*
    慢指针：k 步  快指针：2k 步
    假设相遇点距环起点 m 步， 那么相遇点距起点 k - m 步
    快指针距离再次到达相遇点 2k - k - m 步 = k - m 步
    */
    while (slow != fast) {
        fast = fast->next;
        slow = slow->next;
    }
    return slow;
}
```

![img](https://labuladong.online/algo/images/%E5%8F%8C%E6%8C%87%E9%92%88/2.jpeg)

### 两个链表是否相加

> 题目

给你输入两个链表的头结点 `headA` 和 `headB`，这两个链表可能存在相交。

如果相交，你的算法应该返回相交的那个节点；如果没相交，则返回 null。

> 思路

用哈希表存储某个链表的指针，然后在另外一个链表上判断节点是否在哈希表内即可

但是这样空间复杂度就上去了，要不使用额外空间，必须**通过某些方式，让 `p1` 和 `p2` 能够同时到达相交节点 `c1`**。

所以，我们可以让 `p1` 遍历完链表 `A` 之后开始遍历链表 `B`，让 `p2` 遍历完链表 `B` 之后开始遍历链表 `A`，这样相当于「逻辑上」两条链表接在了一起。

如果这样进行拼接，就可以让 `p1` 和 `p2` 同时进入公共部分，也就是同时到达相交节点 `c1`：

![img](https://labuladong.online/algo/images/%E9%93%BE%E8%A1%A8%E6%8A%80%E5%B7%A7/6.jpeg)

那你可能会问，如果说两个链表没有相交点，是否能够正确的返回 null 呢？

这个逻辑可以覆盖这种情况的，相当于 `c1` 节点是 null 空指针嘛，可以正确返回 null。

> 代码

```
// 求链表的交点
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    // p1 指向 A 链表头结点，p2 指向 B 链表头结点
    ListNode *p1 = headA, *p2 = headB;
    while (p1 != p2) {
        // p1 走一步，如果走到 A 链表末尾，转到 B 链表
        if (p1 == nullptr) p1 = headB;
        else               p1 = p1->next;
        // p2 走一步，如果走到 B 链表末尾，转到 A 链表
        if (p2 == nullptr) p2 = headA;
        else               p2 = p2->next;
    }
    return p1; // 返回交点
}
```

### 总结

```
链表问题其实对于C++来说，就是指针遍历与访问问题。

技巧1：链表存于数组，在数组上进行操作。
	这样就结合了数组可以随机访问的优点，适合对链表节点进行各种各样的操作
	
技巧2：虚拟头节点，需要构造新链表时，通过虚拟头节点
	既防止了空链表造成的额外判断，又实现了将头结点纳为普通节点统一处理
	
技巧3：快慢指针，通过题目所需数学规则，调整快慢指针的移动速度

技巧4：左右指针，通过题目要求，设置两个指针的行为轨迹
```

## 双指针技巧解决数组题目

### 快慢指针技巧

> 数组中的快慢指针，一般都是用来原地修改数组

#### 删除有序数组中的重复项

> 题目

给你一个 **升序排列** 的数组 `nums` ，请你**[ 原地](http://baike.baidu.com/item/原地算法)** 删除重复出现的元素，使每个元素 **只出现一次** ，返回删除后数组的新长度。元素的 **相对顺序** 应该保持 **一致** 。然后返回 `nums` 中唯一元素的个数。

考虑 `nums` 的唯一元素的数量为 `k` ，你需要做以下事情确保你的题解可以被通过：

- 更改数组 `nums` ，使 `nums` 的前 `k` 个元素包含唯一元素，并按照它们最初在 `nums` 中出现的顺序排列。`nums` 的其余元素与 `nums` 的大小不重要。
- 返回 `k` 。

**判题标准:**

系统会用下面的代码来测试你的题解:

```
int[] nums = [...]; // 输入数组
int[] expectedNums = [...]; // 长度正确的期望答案

int k = removeDuplicates(nums); // 调用

assert k == expectedNums.length;
for (int i = 0; i < k; i++) {
    assert nums[i] == expectedNums[i];
}
```

如果所有断言都通过，那么您的题解将被 **通过**。

**示例 1：**

```
输入：nums = [1,1,2]
输出：2, nums = [1,2,_]
解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
```

**示例 2：**

```
输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
```

**提示：**

- `1 <= nums.length <= 3 * 104`
- `-104 <= nums[i] <= 104`
- `nums` 已按 **升序** 排列

> 思路

原地修改：只能在原数组上进行操作

我们让慢指针 `slow` 走在后面，快指针 `fast` 走在前面探路，找到一个不重复的元素就赋值给 `slow` 并让 `slow` 前进一步。

这样，就保证了 `nums[0..slow]` 都是无重复的元素，当 `fast` 指针遍历完整个数组 `nums` 后，`nums[0..slow]` 就是整个数组去重之后的结果。

> 代码

```cpp
int removeDuplicates(vector<int>& nums) {
    if (nums.size() == 0) {
        return 0;
    }
    int slow = 0, fast = 0;
    while (fast < nums.size()) {
        if (nums[fast] != nums[slow]) {
            slow++;
            // 维护 nums[0..slow] 无重复
            nums[slow] = nums[fast];
        }
        fast++;
    }
    // 数组长度为索引 + 1
    return slow + 1;
}
```

![img](https://labuladong.online/algo/images/%E6%95%B0%E7%BB%84%E5%8E%BB%E9%87%8D/1.gif)

#### 移除元素

> 题目

给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/原地算法)** 移除所有数值等于 `val` 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 `O(1)` 额外空间并 **[原地 ](https://baike.baidu.com/item/原地算法)修改输入数组**。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

**说明:**

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以**「引用」**方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```
// nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

**示例 1：**

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2]
解释：函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。你不需要考虑数组中超出新长度后面的元素。例如，函数返回的新长度为 2 ，而 nums = [2,2,3,3] 或 nums = [2,2,0,0]，也会被视作正确答案。
```

**示例 2：**

```
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3]
解释：函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。注意这五个元素可为任意顺序。你不需要考虑数组中超出新长度后面的元素。
```

**提示：**

- `0 <= nums.length <= 100`
- `0 <= nums[i] <= 50`
- `0 <= val <= 100`

> 思路

题目要求我们把 `nums` 中所有值为 `val` 的元素原地删除，依然需要使用快慢指针技巧：

如果 `fast` 遇到值为 `val` 的元素，则直接跳过，否则就赋值给 `slow` 指针，并让 `slow` 前进一步。

> 代码

```cpp
int removeElement(vector<int>& nums, int val) {
    int fast = 0, slow = 0;
    while (fast < nums.size()) {
        if (nums[fast] != val) {
            nums[slow] = nums[fast];
            slow++;
        }
        fast++;
    }
    return slow;
}
```

#### 移动零

> 题目

给你输入一个数组 `nums`，请你**原地修改**，将数组中的所有值为 0 的元素移到数组末尾。比如说给你输入 `nums = [0,1,4,0,2]`，你的算法没有返回值，但是会把 `nums` 数组原地修改成 `[1,4,2,0,0]`。

> 思路

题目让我们将所有 0 移到最后，其实就相当于移除 `nums` 中的所有 0，然后再把后面的元素都赋值为 0 即可。

> 代码

```cpp
void moveZeroes(vector<int>& nums) {
    // 去除 nums 中的所有 0，返回不含 0 的数组长度
    int p = removeElement(nums, 0);
    // 将 nums[p..] 的元素赋值为 0
    for (; p < nums.size(); p++) {
        nums[p] = 0;
    }
}

// 见上文代码实现
int removeElement(vector<int>& nums, int val);
```

### 左右指针常用算法

#### 二分查找

```cpp
// 只看双指针特性即可
int binarySearch(vector<int>& nums, int target) {
    // 双指针，从左右到中间
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        }
     	else if (nums[mid] < target) {
            left = mid + 1;
        }   
        else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return -1;
}
```

#### 两数之和

> 题目

给你一个下标从 **1** 开始的整数数组 `numbers` ，该数组已按 **非递减顺序排列** ，请你从数组中找出满足相加之和等于目标数 `target` 的两个数。如果设这两个数分别是 `numbers[index1]` 和 `numbers[index2]` ，则 `1 <= index1 < index2 <= numbers.length` 。

以长度为 2 的整数数组 `[index1, index2]` 的形式返回这两个整数的下标 `index1` 和 `index2`。

你可以假设每个输入 **只对应唯一的答案** ，而且你 **不可以** 重复使用相同的元素。你所设计的解决方案必须只使用常量级的额外空间。

**示例 1：**

```
输入：numbers = [2,7,11,15], target = 9
输出：[1,2]
解释：2 与 7 之和等于目标数 9 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。
```

**示例 2：**

```
输入：numbers = [2,3,4], target = 6
输出：[1,3]
解释：2 与 4 之和等于目标数 6 。因此 index1 = 1, index2 = 3 。返回 [1, 3] 。
```

**示例 3：**

```
输入：numbers = [-1,0], target = -1
输出：[1,2]
解释：-1 与 0 之和等于目标数 -1 。因此 index1 = 1, index2 = 2 。返回 [1, 2] 。
```

**提示：**

- `2 <= numbers.length <= 3 * 104`
- `-1000 <= numbers[i] <= 1000`
- `numbers` 按 **非递减顺序** 排列
- `-1000 <= target <= 1000`
- **仅存在一个有效答案**

> 思路

由于数组已经有序，就应该使用双指针技巧，从两边往中间压缩，找到合适的位置

> 代码

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    // 一左一右两个指针相向而行
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            // 题目要求的索引是从 1 开始的
            return {left + 1, right + 1};
        } else if (sum < target) {
            left++; // 让 sum 大一点
        } else if (sum > target) {
            right--; // 让 sum 小一点
        }
    }
    return {-1, -1};
}
```

#### 反转数组

> 题目

反转一个 `char[]` 类型的字符数组

> 思路

每次交换left和right索引的元素即可

> 代码

```cpp
void reverseString(vector<char>& s) {
    // 一左一右两个指针相向而行
    int left = 0, right = s.size() - 1;
    while (left < right) {
        // 交换 s[left] 和 s[right]
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        left++;
        right--;
    }
}
```

#### 回文判断

> 题目

判断一个字符串是不是回文串

> 思路

左右指针向中压缩，一旦左右指针指向内容不同，就不是回文串

> 代码

```cpp
bool isPalindrome(string s) {
    // 一左一右两个指针相向而行
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s[left] != s[right]) { // 如果不相同，就不是回文串
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

#### 最长回文子串

> 题目

给你一个字符串 `s`，找到 `s` 中最长的回文子串。如果字符串的反序与原始字符串相同，则该字符串称为回文字符串。

**示例 1：**

```
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。
```

**示例 2：**

```
输入：s = "cbbd"
输出："bb"
```

**提示：**

- `1 <= s.length <= 1000`
- `s` 仅由数字和英文字母组成

> 思路

找回文串的难点在于，回文串的的长度可能是奇数也可能是偶数，解决该问题的核心是**从中心向两端扩散的双指针技巧**。

如果回文串的长度为奇数，则它有一个中心字符；如果回文串的长度为偶数，则可以认为它有两个中心字符。所以我们可以先实现这样一个函数：

```cpp
// 在 s 中寻找以 s[l] 和 s[r] 为中心的最长回文串
string palindrome(string s, int l, int r) {
    // 防止索引越界
    while (l >= 0 && r < s.length()
            && s[l] == s[r]) {
        // 双指针，向两边展开
        l--; r++;
    }
    // 返回以 s[l] 和 s[r] 为中心的最长回文串
    return s.substr(l + 1, r - l - 1);
}
```

这样，如果输入相同的 `l` 和 `r`，就相当于寻找长度为奇数的回文串，如果输入相邻的 `l` 和 `r`，则相当于寻找长度为偶数的回文串。

那么回到最长回文串的问题，解法的大致思路就是：

```
for 0 <= i < len(s):
    找到以 s[i] 为中心的回文串
    找到以 s[i] 和 s[i+1] 为中心的回文串
    更新答案
```

> 代码

```cpp
string longestPalindrome(string s) {
    string res = "";
    for (int i = 0; i < s.length(); i++) {
        // 以 s[i] 为中心的最长回文子串
        string s1 = palindrome(s, i, i);
        // 以 s[i] 和 s[i+1] 为中心的最长回文子串
        string s2 = palindrome(s, i, i + 1);
        // res = longest(res, s1, s2)
        res = res.length() > s1.length() ? res : s1;
        res = res.length() > s2.length() ? res : s2;
    }
    return res;
}
```

### 总结

```
数组类双指针问题：
	快慢指针：两个指针移动方向相同
	左右指针：两个指针移动方向相反（从两端往中心 / 从中心往两端）
	
	快慢指针：一般是根据题目条件判断，快指针一直移动，慢指针满足条件则移动
	左右指针：同时移动，然后操作数组数据
```

## 滑动窗口框架

### 概览

> 基本思想

维护两个指针，他们夹住的就是窗口。不停滑动，更新答案

```cpp
int left = 0, right = 0;

while (right < nums.size()) {
    // 增大窗口
    window.add(nums[right]);
    right++;
    
    while (window needs shrink) {
        // 缩小窗口
        window.remove(nums[left]);
        left++;
    }
}

// 滑动窗口 的等价暴力写法就是 双重for循环
// 时间复杂度上 滑动窗口O(N)  暴力解法O(N^2)
// 窗口的指针都是不回退的，那么最多就是N + N 
// 滑动窗口没办法枚举所有子数组，因为它的移动规则是有限定的
```

> 通用算法框架

```cpp
// 以求解子串问题为例
void slidingWindow(string s) {
    // 用合适的数据结构记录窗口中的数据，根据具体场景变通
    // such as 记录窗口中元素出现的次数，用map
    // 记录窗口中元素和，就用int
    unordered_map<char, int> window;
    
    int left = 0, right = 0;
    while (right < s.size()) {
        // c 就是将移入窗口的字符
        char c = s[right];
        window.add(c);
        //增大窗口
        right++;
        // 对窗口中数据进行更新
        ...
            
        // debug的位置
        printf("window: [%d, %d]\n", left, right);
        
        // 判断左侧窗口是否要压缩
        while (left < right && window needs shrink) {
            // d 就是要移出窗口的字符
            char d = s[left];
            window.remove(d);
            // 缩小窗口
            left++;
            // 对窗口内数据进行更新
            ...
        }
    }
}
```

### 最小覆盖子串

> 题目

给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。

**注意：**

- 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
- 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。

**示例 1：**

```
输入：s = "ADOBECODEBANC", t = "ABC"
输出："BANC"
解释：最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
```

**示例 2：**

```
输入：s = "a", t = "a"
输出："a"
解释：整个字符串 s 是最小覆盖子串。
```

**示例 3:**

```
输入: s = "a", t = "aa"
输出: ""
解释: t 中两个字符 'a' 均应包含在 s 的子串中，
因此没有符合条件的子字符串，返回空字符串。
```

**提示：**

- `m == s.length`
- `n == t.length`
- `1 <= m, n <= 105`
- `s` 和 `t` 由英文字母组成

**进阶：**你能设计一个在 `o(m+n)` 时间内解决此问题的算法吗？

> 思路

如果使用**暴力解法**

```java
for (int i = 0; i < s.size(); i++)
    for (int j = i + 1; j < s.size(); j++)
        if s[i:j] 包含 t 的所有字母:
            更新答案
```

复杂度必然比O(N^2)大

如果使用**滑动窗口**

1. 我们在字符串 `S` 中使用双指针中的左右指针技巧，初始化 `left = right = 0`，把索引**左闭右开**区间 `[left, right)` 称为一个「窗口」。
2. 我们先不断地增加 `right` 指针扩大窗口 `[left, right)`，直到窗口中的字符串符合要求（包含了 `T` 中的所有字符）。
3. 此时，我们停止增加 `right`，转而不断增加 `left` 指针缩小窗口 `[left, right)`，直到窗口中的字符串不再符合要求（不包含 `T` 中的所有字符了）。同时，每次增加 `left`，我们都要更新一轮结果。
4. 重复第 2 和第 3 步，直到 `right` 到达字符串 `S` 的尽头。

> 为什么滑动窗口是左闭右开区间

**理论上你可以设计两端都开或者两端都闭的区间，但设计为左闭右开区间是最方便处理的**。

因为这样初始化 `left = right = 0` 时区间 `[0, 0)` 中没有元素，但只要让 `right` 向右移动（扩大）一位，区间 `[0, 1)` 就包含一个元素 `0` 了。

如果你设置为两端都开的区间，那么让 `right` 向右移动一位后开区间 `(0, 1)` 仍然没有元素；如果你设置为两端都闭的区间，那么初始区间 `[0, 0]` 就包含了一个元素。这两种情况都会给边界处理带来不必要的麻烦。

> 代码

```cpp
string minWindow(string s, string t) {
    // need用于记录t，window用于记录窗口内容
	unordered_map<char, int> need, window;
    for (char c : t) need[c]++;
    
    int left = 0, right = 0;
    int valid = 0;
    // 记录最小覆盖子串的起始索引和长度
    int start = 0, len = INT_MAX;
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        // 扩大窗口
        right++;
        // 进行窗口内数据更新
        if (need.count(c)) {
            window[c]++;
            if (window[c] == need[c]) {
                valid++;
            }
        }
        
        // 判断左侧窗口是否要压缩
        while (valid == need.size()) {
            // 在这里更新最小覆盖子串
            if (right - left < len) {
                start = left;
                len = right - left;
            }
            // d 是将移出窗口的字符
            char d = s[left];
            // 缩小窗口
            left++;
            // 进行窗口内数据更新
            if (need.count(d)) {
                if (window[d] == need[d]) {
                    valid--;
                }
                window[d]--;
            }
        }
    }
    return len == INT_MAX ? "" : s.substr(start, len);
}
```

需要注意的是，当我们发现某个字符在 `window` 的数量满足了 `need` 的需要，就要更新 `valid`，表示有一个字符已经满足要求。而且，你能发现，两次对窗口内数据的更新操作是完全对称的。

当 `valid == need.size()` 时，说明 `T` 中所有字符已经被覆盖，已经得到一个可行的覆盖子串，现在应该开始收缩窗口了，以便得到「最小覆盖子串」。

移动 `left` 收缩窗口时，窗口内的字符都是可行解，所以应该在收缩窗口的阶段进行最小覆盖子串的更新，以便从可行解中找到长度最短的最终结果。

### 字符串排列

> 题目

给你两个字符串 `s1` 和 `s2` ，写一个函数来判断 `s2` 是否包含 `s1` 的排列。如果是，返回 `true` ；否则，返回 `false` 。换句话说，`s1` 的排列之一是 `s2` 的 **子串** 。

**示例 1：**

```
输入：s1 = "ab" s2 = "eidbaooo"
输出：true
解释：s2 包含 s1 的排列之一 ("ba").
```

**示例 2：**

```
输入：s1= "ab" s2 = "eidboaoo"
输出：false
```

**提示：**

- `1 <= s1.length, s2.length <= 104`
- `s1` 和 `s2` 仅包含小写字母

> 思路

其实就是利用滑动窗口，然后看窗口构成的子串是不是s1的排列

> 代码

```cpp
// 判断 s 中是否存在 t 的排列
bool checkInclusion(string t, string s) {
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;

    int left = 0, right = 0;
    int valid = 0;
    while (right < s.size()) {
        char c = s[right];
        right++;
        // 进行窗口内数据的一系列更新
        if (need.count(c)) {
            window[c]++;
            if (window[c] == need[c])
                valid++;
        }

        // 判断左侧窗口是否要收缩
        while (right - left >= t.size()) {
            // 在这里判断是否找到了合法的子串
            if (valid == need.size())
                return true;
            char d = s[left];
            left++;
            // 进行窗口内数据的一系列更新
            if (need.count(d)) {
                if (window[d] == need[d])
                    valid--;
                window[d]--;
            }
        }
    }
    // 未找到符合条件的子串
    return false;
}

由于这道题中 [left, right) 其实维护的是一个定长的窗口，窗口大小为 t.size()。因为定长窗口每次向前滑动时只会移出一个字符，所以可以把内层的 while 改成 if，效果是一样的。
```

### 找到所有的字母异位词

> 题目

给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。**异位词** 指由相同字母重排列形成的字符串（包括相同的字符串）。

**示例 1:**

```
输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
```

 **示例 2:**

```
输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
```

**提示:**

- `1 <= s.length, p.length <= 3 * 104`
- `s` 和 `p` 仅包含小写字母

> 思路

异位词就上上一题的排列

> 代码

```cpp
vector<int> findAnagrams(string s, string t) {
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;

    int left = 0, right = 0;
    int valid = 0;
    vector<int> res; // 记录结果
    while (right < s.size()) {
        char c = s[right];
        right++;
        // 进行窗口内数据的一系列更新
        if (need.count(c)) {
            window[c]++;
            if (window[c] == need[c]) 
                valid++;
        }
        // 判断左侧窗口是否要收缩
        while (right - left >= t.size()) {
            // 当窗口符合条件时，把起始索引加入 res
            if (valid == need.size())
                res.push_back(left);
            char d = s[left];
            left++;
            // 进行窗口内数据的一系列更新
            if (need.count(d)) {
                if (window[d] == need[d])
                    valid--;
                window[d]--;
            }
        }
    }
    return res;
}
```

### 最长无重复子串

> 题目

给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。

**示例 1:**

```
输入: s = "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

**示例 2:**

```
输入: s = "bbbbb"
输出: 1
解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
```

**示例 3:**

```
输入: s = "pwwkew"
输出: 3
解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
     请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
```

**提示：**

- `0 <= s.length <= 5 * 104`
- `s` 由英文字母、数字、符号和空格组成

> 思路

窗口内就是要判断的子串，保证这个子串有没有重复字符即可

所以每次往右扩展窗口就要更新答案

当不满足子串字符唯一时就要缩小窗口到满足唯一

> 代码

```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> window;

    int left = 0, right = 0;
    int res = 0; // 记录结果
    while (right < s.size()) {
        char c = s[right];
        right++;
        // 进行窗口内数据的一系列更新
        window[c]++;
        // 判断左侧窗口是否要收缩
        while (window[c] > 1) {
            char d = s[left];
            left++;
            // 进行窗口内数据的一系列更新
            window[d]--;
        }
        // 在这里更新答案
        res = max(res, right - left);
    }
    return res;
}
```

## 二分搜索算法框架

二分搜索的思想非常简单，但是细节是魔鬼

while里面是 < 还是 <= ，mid + 1 还是 mid  ....

这些细节就是容易触发bug的关键

### 二分查找框架

```cpp
int binarySearch(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;

    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        }
    }
    return ...;
}
```

**分析二分查找的一个技巧是：不要出现 else，而是把所有情况用 else if 写清楚，这样可以清楚地展现所有细节**。

其中 `...` 标记的部分，就是可能出现细节问题的地方，当见到一个二分查找的代码时，首先注意这几个地方。

**计算 `mid` 时需要防止溢出**，代码中 `left + (right - left) / 2` 就和 `(left + right) / 2` 的结果相同，但是有效防止了 `left` 和 `right` 太大，直接相加导致溢出的情况。

### 寻找一个数（基本的二分搜索）

这个场景是最简单的，即搜索一个数，如果存在，返回其索引，否则返回 -1。

```cpp
int binarySearch(vector<int>& nums, int target) {
    int left = 0; 
    int right = nums.size() - 1; // 注意

    while(left <= right) {
        int mid = left + (right - left) / 2;
        if(nums[mid] == target)
            return mid; 
        else if (nums[mid] < target)
            left = mid + 1; // 注意
        else if (nums[mid] > target)
            right = mid - 1; // 注意
    }
    return -1;
}
```

> **为什么 while 循环的条件中是 <=，而不是 <**？

答：因为初始化 `right` 的赋值是 `nums.length - 1`，即最后一个元素的索引，而不是 `nums.length`。

这二者可能出现在不同功能的二分查找中，区别是：前者相当于两端都闭区间 `[left, right]`，后者相当于左闭右开区间 `[left, right)`。因为索引大小为 `nums.length` 是越界的，所以我们把 `right` 这一边视为开区间。

这个算法中使用的是前者 `[left, right]` 两端都闭的区间。**这个区间其实就是每次进行搜索的区间**。什么时候应该停止搜索呢？当然，找到了目标值的时候可以终止：

```cpp
    if(nums[mid] == target)
        return mid;
```

但如果没找到，就需要 while 循环终止，然后返回 -1。那 while 循环什么时候应该终止？**搜索区间为空的时候应该终止**，意味着你没得找了，就等于没找到嘛。

`while(left <= right)` 的终止条件是 `left == right + 1`，写成区间的形式就是 `[right + 1, right]`，或者带个具体的数字进去 `[3, 2]`，可见**这时候区间为空**，因为没有数字既大于等于 3 又小于等于 2 的吧。所以这时候 while 循环终止是正确的，直接返回 -1 即可。

`while(left < right)` 的终止条件是 `left == right`，写成区间的形式就是 `[right, right]`，或者带个具体的数字进去 `[2, 2]`，**这时候区间非空**，还有一个数 2，但此时 while 循环终止了。也就是说区间 `[2, 2]` 被漏掉了，索引 2 没有被搜索，如果这时候直接返回 -1 就是错误的。

当然，如果非要用 `while(left < right)` 也可以，就打个补丁好了：

```cpp
    //...
    while(left < right) {
        // ...
    }
    return nums[left] == target ? left : -1;
```

> **为什么 `left = mid + 1`，`right = mid - 1`？有的代码是 `right = mid` 或者 `left = mid`，没有这些加加减减，到底怎么回事，怎么判断**？

答：刚才明确了「搜索区间」这个概念，而且本算法的搜索区间是两端都闭的，即 `[left, right]`。那么当我们发现索引 `mid` 不是要找的 `target` 时，下一步应该去搜索哪里呢？

当然是去搜索区间 `[left, mid-1]` 或者区间 `[mid+1, right]` 对不对？**因为 `mid` 已经搜索过，应该从搜索区间中去除**。

> **此算法有什么缺陷**？

答：至此，你应该已经掌握了该算法的所有细节，以及这样处理的原因。但是，这个算法存在局限性。

比如说给你有序数组 `nums = [1,2,2,2,3]`，`target` 为 2，此算法返回的索引是 2，没错。但是如果我想得到 `target` 的左侧边界，即索引 1，或者我想得到 `target` 的右侧边界，即索引 3，这样的话此算法是无法处理的。

这样的需求很常见，**你也许会说，找到一个 `target`，然后向左或向右线性搜索不行吗？可以，但是不好，因为这样难以保证二分查找对数级的复杂度了**。

### 寻找左侧边界的二分搜索

```cpp
int left_bound(vector<int>& nums, int target) {
    int left = 0;
    int right = nums.size(); // 注意
    
    while (left < right) { // 注意
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            right = mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    return left;
}

// 函数返回 nums 数组中最左侧的 target的索引
```

> **为什么 while 中是 `<` 而不是 `<=`**?

答：用相同的方法分析，因为 `right = nums.length` 而不是 `nums.length - 1`。因此每次循环的「搜索区间」是 `[left, right)` 左闭右开。

`while(left < right)` 终止的条件是 `left == right`，此时搜索区间 `[left, left)` 为空，所以可以正确终止。

```
这里先要说一个搜索左右边界和上面这个算法的一个区别：刚才的 right 不是 nums.length - 1 吗，为啥这里非要写成 nums.length 使得「搜索区间」变成左闭右开呢？

因为对于搜索左右侧边界的二分查找，这种写法比较普遍。
非要用两端都闭的写法反而更简单，后面相关的代码会展示，把三种二分搜索都用一种两端都闭的写法统一起来。
```

> **如果 `nums` 中不存在 `target` 这个值，计算出来的这个索引含义是什么？如果我想让它返回 -1，怎么做**？

直接说结论：**如果 `target` 不存在，搜索左侧边界的二分搜索返回的索引是大于 `target` 的最小索引**。

举个例子，`nums = [2,3,5,7], target = 4`，`left_bound` 函数返回值是 2，因为元素 5 是大于 4 的最小元素。

有点绕晕了是吧？这个 `left_bound` 函数明明是搜索左边界的，但是当 `target` 不存在的时候，却返回的是大于 `target` 的最小索引。这个结论不用死记，你要是拿不准，**简单举个例子就能得到这个结论了。**

所以跟你说二分搜索这个东西思路很简单，细节是魔鬼嘛，里面的坑太多了。

话说回来，`left_bound` 的这个行为有一个好处。比方说现在让你写一个 `floor` 函数：

```java
// 在一个有序数组中，找到「小于 target 的最大元素的索引」
// 比如说输入 nums = [1,2,2,2,3]，target = 2，函数返回 0，因为 1 是小于 2 的最大元素。
// 再比如输入 nums = [1,2,3,5,6]，target = 4，函数返回 2，因为 3 是小于 4 的最大元素。
int floor(int[] nums, int target);
```

那么就可以直接用 `left_bound` 函数来实现：

```java
int floor(int[] nums, int target) {
    // 当 target 不存在，比如输入 [4,6,8,10], target = 7
    // left_bound 返回 2，减一就是 1，元素 6 就是小于 7 的最大元素
    // 当 target 存在，比如输入 [4,6,8,8,8,10], target = 8
    // left_bound 返回 2，减一就是 1，元素 6 就是小于 8 的最大元素
    return left_bound(nums, target) - 1;
}
```

如果想让 `target` 不存在时返回 -1 其实很简单，在返回的时候额外判断一下 `nums[left]` 是否等于 `target` 就行了，如果不等于，就说明 `target` 不存在。需要注意的是，访问数组索引之前要保证索引不越界：

```cpp
while (left < right) {
    //...
}
// 如果索引越界，说明数组中无目标元素，返回 -1
if (left < 0 || left >= nums.length) {
    return -1;
}
// 提示：其实上面的 if 中 left < 0 这个判断可以省略，因为对于这个算法，left 不可能小于 0
// 你看这个算法执行的逻辑，left 初始化就是 0，且只可能一直往右走，那么只可能在右侧越界
// 不过我这里就同时判断了，因为在访问数组索引之前保证索引在左右两端都不越界是一个好习惯，没有坏处
// 另一个好处是让二分的模板更统一，降低你的记忆成本，因为等会儿寻找右边界的时候也有类似的出界判断


// 判断一下 nums[left] 是不是 target
return nums[left] == target ? left : -1;
```

> **为什么 `left = mid + 1`，`right = mid` ？和之前的算法不一样**？

答：这个很好解释，因为我们的「搜索区间」是 `[left, right)` 左闭右开，所以当 `nums[mid]` 被检测之后，下一步应该去 `mid` 的左侧或者右侧区间搜索，即 `[left, mid)` 或 `[mid + 1, right)`。

> **为什么该算法能够搜索左侧边界**？

关键在于对于 `nums[mid] == target` 这种情况的处理：

```java
    if (nums[mid] == target)
        right = mid;
```

可见，找到 target 时不要立即返回，而是缩小「搜索区间」的上界 `right`，在区间 `[left, mid)` 中继续搜索，即不断向左收缩，达到锁定左侧边界的目的。

> **为什么返回 `left` 而不是 `right`**？

答：都是一样的，因为 while 终止的条件是 `left == right`。

>**能不能想办法把 `right` 变成 `nums.length - 1`，也就是继续使用两边都闭的「搜索区间」？这样就可以和第一种二分搜索在某种程度上统一起来了**。

```cpp
int left_bound(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    // 搜索区间为 [left, right]
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            // 搜索区间变为 [mid+1, right]
            left = mid + 1;
        } else if (nums[mid] > target) {
            // 搜索区间变为 [left, mid-1]
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 收缩右侧边界
            right = mid - 1;
        }
    }
    // 判断 target 是否存在于 nums 中
    // 如果越界，target 肯定不存在，返回 -1
    if (left < 0 || left >= nums.size()) {
        return -1;
    }
    // 判断一下 nums[left] 是不是 target
    return nums[left] == target ? left : -1;
}
```

### 寻找右侧边界的二分查找

```cpp
int right_bound(vector<int>& nums, int target) {
    int left = 0, right = nums.size();

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            left = mid + 1; // 注意
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    return left - 1; // 注意
}
```

>**为什么这个算法能够找到右侧边界**？

答：类似地，关键点还是这里：

```cpp
if (nums[mid] == target) {
    left = mid + 1;
```

当 `nums[mid] == target` 时，不要立即返回，而是增大「搜索区间」的左边界 `left`，使得区间不断向右靠拢，达到锁定右侧边界的目的。

> **为什么最后返回 `left - 1` 而不像左侧边界的函数，返回 `left`？而且我觉得这里既然是搜索右侧边界，应该返回 `right` 才对**。

答：首先，while 循环的终止条件是 `left == right`，所以 `left` 和 `right` 是一样的，你非要体现右侧的特点，返回 `right - 1` 好了。

至于为什么要减一，这是搜索右侧边界的一个特殊点，关键在锁定右边界时的这个条件判断：

```cpp
// 增大 left，锁定右侧边界
if (nums[mid] == target) {
    left = mid + 1;
    // 这样想: mid = left - 1
```

![img](https://labuladong.online/algo/images/%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE/3.jpg)

因为我们对 `left` 的更新必须是 `left = mid + 1`，就是说 while 循环结束时，`nums[left]` 一定不等于 `target` 了，而 `nums[left-1]` 可能是 `target`。

至于为什么 `left` 的更新必须是 `left = mid + 1`，当然是为了把 `nums[mid]` 排除出搜索区间，这里就不再赘述。

> **如果 `nums` 中不存在 `target` 这个值，计算出来的这个索引含义是什么？如果我想让它返回 -1，怎么做**？

直接说结论，和前面讲的 `left_bound` 相反：**如果 `target` 不存在，搜索右侧边界的二分搜索返回的索引是小于 `target` 的最大索引**。

这个结论不用死记，你要是拿不准，**简单举个例子就能得到这个结论了**。比如 `nums = [2,3,5,7], target = 4`，`right_bound` 函数返回值是 1，因为元素 3 是小于 4 的最大元素。

如果你想在 `target` 不存在时返回 -1，很简单，只要在最后判断一下 `nums[left-1]` 是不是 `target` 就行了，类似之前的左侧边界搜索，做一点额外的判断即可：

```cpp
while (left < right) {
    // ...
}
// 判断 target 是否存在于 nums 中
// left - 1 索引越界的话 target 肯定不存在
if (left - 1 < 0 || left - 1 >= nums.size()) {
    return -1;
}
// 判断一下 nums[left - 1] 是不是 target
return nums[left - 1] == target ? (left - 1) : -1;
```

> **是否也可以把这个算法的「搜索区间」也统一成两端都闭的形式呢？这样这三个写法就完全统一了，以后就可以闭着眼睛写出来了**。

```cpp
int right_bound(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1;
        } else if (nums[mid] == target) {
            // 这里改成收缩左侧边界即可
            left = mid + 1;
        }
    }
    // 最后改成返回 left - 1
    if (left - 1 < 0 || left - 1 >= nums.size()) {
        return -1;
    }
    return nums[left - 1] == target ? (left - 1) : -1;
}
```

### 逻辑统一

**第一个，最基本的二分查找算法**：

```cpp
因为我们初始化 right = nums.length - 1
所以决定了我们的「搜索区间」是 [left, right]
所以决定了 while (left <= right)
同时也决定了 left = mid+1 和 right = mid-1

因为我们只需找到一个 target 的索引即可
所以当 nums[mid] == target 时可以立即返回
```

**第二个，寻找左侧边界的二分查找**：

```cpp
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最左侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧右侧边界以锁定左侧边界 right = mid
```

**第三个，寻找右侧边界的二分查找**：

```cpp
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最右侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧左侧边界以锁定右侧边界

又因为收紧左侧边界时必须 left = mid + 1
所以最后无论返回 left 还是 right，必须减一
```

以上二分搜索的框架属于「术」的范畴，如果上升到「道」的层面，**二分思维的精髓就是：通过已知信息尽可能多地收缩（折半）搜索空间**，从而增加穷举效率，快速找到目标。

## 二叉树纲领

> 二叉树的思维模式

**1、是否可以通过遍历一遍二叉树得到答案**？如果可以，用一个 `traverse` 函数配合外部变量来实现，这叫「遍历」的思维模式。

**2、是否可以定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案**？如果可以，写出这个递归函数的定义，并充分利用这个函数的返回值，这叫「分解问题」的思维模式。

无论使用哪种思维模式，都需要思考：

**如果单独抽出一个二叉树节点，它需要做什么事情？需要在什么时候（前/中/后序位置）做**？其他的节点不用你操心，递归函数会帮你在所有节点上执行相同的操作。

### 二叉树的重要性

**快速排序就是个二叉树的前序遍历，归并排序就是个二叉树的后序遍历，那么我就知道你是个算法高手了**。

快速排序的逻辑是，若要对 `nums[lo..hi]` 进行排序，我们先找一个分界点 `p`，通过交换元素使得 `nums[lo..p-1]` 都小于等于 `nums[p]`，且 `nums[p+1..hi]` 都大于 `nums[p]`，然后递归地去 `nums[lo..p-1]` 和 `nums[p+1..hi]` 中寻找新的分界点，最后整个数组就被排序了。

快速排序的代码框架如下：

```cpp
void sort(vector<int>& nums, int lo, int hi) {
    // ****** 前序遍历位置 ******
    // 通过交换元素构建分界点 p
    int p = partition(nums, lo, hi);
    // ************************

    sort(nums, lo, p - 1);
    sort(nums, p + 1, hi);
}
// 先构造分界点，然后去左右子数组构造分界点，你看这不就是一个二叉树的前序遍历吗？
```

归并排序的逻辑，若要对 `nums[lo..hi]` 进行排序，我们先对 `nums[lo..mid]` 排序，再对 `nums[mid+1..hi]` 排序，最后把这两个有序的子数组合并，整个数组就排好序了。

归并排序的代码框架如下：

```cpp
// 定义：排序 nums[lo..hi]
void sort(vector<int>& nums, int lo, int hi) {
    if (hi <= lo) return;
    int mid = lo + (hi - lo) / 2;
    // 排序 nums[lo..mid]
    sort(nums, lo, mid);
    // 排序 nums[mid+1..hi]
    sort(nums, mid + 1, hi);

    // ****** 后序位置 ******
    // 合并 nums[lo..mid] 和 nums[mid+1..hi]
    merge(nums, lo, mid, hi);
    // *********************
}

// 先对左右子数组排序，然后合并（类似合并有序链表的逻辑），你看这是不是二叉树的后序遍历框架？另外，这不就是传说中的分治算法嘛。
```

### 深入理解前中后序

**先想三个问题**

1、你理解的二叉树的前中后序遍历是什么，仅仅是三个顺序不同的 List 吗？

2、请分析，后序遍历有什么特殊之处？

3、请分析，为什么多叉树没有中序遍历？

首先前中后序都是二叉树的遍历方式，也就是就使用递归进行每个节点的访问。

```cpp
// 迭代遍历数组
void traverse(vector<int>& arr) {
    for (int i = 0; i < arr.size(); i++) {

    }
}

// 递归遍历数组
void traverse(vector<int>& arr, int i) {
    if (i == arr.size()) {
        return;
    }
    // 前序位置
    traverse(arr, i + 1);
    // 后序位置
}

struct ListNode {
    int val;
    ListNode *next;
};

// 迭代遍历单链表
void traverse(ListNode* head) {
    for (ListNode* p = head; p != nullptr; p = p->next) {

    }
}

// 递归遍历单链表
void traverse(ListNode* head) {
    if (head == nullptr) {
        return;
    }
    // 前序位置
    traverse(head->next);
    // 后序位置
}

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode * right;
};

// 递归遍历二叉树
void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    traverse(root->left);
    // 中序位置
    traverse(root->right);
    // 后序位置
}
```

单链表和数组的遍历可以是迭代的，也可以是递归的，**二叉树这种结构无非就是二叉链表**，由于没办法简单改写成迭代形式，所以一般说二叉树的遍历框架都是指递归的形式。

你也注意到了，**只要是递归形式的遍历，都可以有前序位置和后序位置**，分别在递归之前和递归之后。

**所谓前序位置，就是刚进入一个节点（元素）的时候，后序位置就是即将离开一个节点（元素）的时候**，那么进一步，你把代码写在不同位置，代码执行的时机也不同：

**前中后序是遍历二叉树过程中处理每一个节点的三个特殊时间点**，绝不仅仅是三个顺序不同的List：

- 前序位置的代码在刚刚进入一个二叉树节点的时候执行；
- 后序位置的代码在将要离开一个二叉树节点的时候执行；
- 中序位置的代码在一个二叉树节点左子树都遍历完，即将开始遍历右子树的时候执行。

![img](https://labuladong.online/algo/images/%E4%BA%8C%E5%8F%89%E6%A0%91%E6%94%B6%E5%AE%98/2.jpeg)

**你可以发现每个节点都有「唯一」属于自己的前中后序位置**，所以我说前中后序遍历是遍历二叉树过程中处理每一个节点的三个特殊时间点。

这里你也可以理解为什么多叉树没有中序位置，因为二叉树的每个节点只会进行唯一一次左子树切换右子树，而多叉树节点可能有很多子节点，会多次切换子树去遍历，所以多叉树节点没有「唯一」的中序遍历位置。

说了这么多基础的，就是要帮你对二叉树建立正确的认识，然后你会发现：

**二叉树的所有问题（包括能够建模成二叉树的问题），就是让你在前中后序位置注入巧妙的代码逻辑，去达到自己的目的，你只需要单独思考每一个节点应该做什么，其他的不用你管，抛给二叉树遍历框架，递归会在所有节点上做相同的操作**。

你也可以看到，图论算法 把二叉树的遍历框架扩展到了图，并以遍历为基础实现了图论的各种经典算法。

### 两种解题思路

> 两种解题思路是什么？

- 遍历二叉树得出答案（回溯算法核心逻辑）
- 分解问题计算出答案（动态规划核心逻辑）

#### 二叉树的最大深度

> 题目

最大深度就是根节点到「最远」叶子节点的最长路径上的节点数，比如输入这棵二叉树，算法应该返回 3：

![img](https://labuladong.online/algo/images/%E4%BA%8C%E5%8F%89%E6%A0%91%E6%94%B6%E5%AE%98/tree.jpg)

> 思路1：遍历思路

遍历一遍二叉树，用一个外部变量记录每个节点所在的深度，取最大值就可以得到最大深度，**这就是遍历二叉树计算答案的思路**。

```cpp
// 记录最大深度
int res = 0;
// 记录遍历到的节点的深度
int depth = 0;

// 主函数
int maxDepth(TreeNode* root) {
    traverse(root);
    return res;
}

// 二叉树遍历框架
void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    depth++;
    if (root->left == nullptr && root->right == nullptr) {
        // 到达叶子节点，更新最大深度
        res = std::max(res, depth);
    }
    traverse(root->left);
    traverse(root->right);
    // 后序位置
    depth--;
}
```

这个解法应该很好理解，但为什么需要在前序位置增加 `depth`，在后序位置减小 `depth`？

因为前面说了，前序位置是进入一个节点的时候，后序位置是离开一个节点的时候，`depth` 记录当前递归到的节点深度，你把 `traverse` 理解成在二叉树上游走的一个指针，所以当然要这样维护。

至于对 `res` 的更新，你放到前中后序位置都可以，只要保证在进入节点之后，离开节点之前（即 `depth` 自增之后，自减之前）就行了。

> 思路2：分解问题思路

```cpp
// 定义：输入根节点，返回这棵二叉树的最大深度
int maxDepth(TreeNode* root) {
    if (root == nullptr) {
        return 0;
    }
    // 利用定义，计算左右子树的最大深度
    int leftMax = maxDepth(root->left);
    int rightMax = maxDepth(root->right);
    // 整棵树的最大深度等于左右子树的最大深度取最大值，
    // 然后再加上根节点自己
    int res = std::max(leftMax, rightMax) + 1;
    
    return res;
}
```

因为这个思路正确的核心在于，你确实可以通过**子树的最大深度推导出原树的深度**，所以当然要首先利用递归函数的定义**算出左右子树的最大深度，然后推出原树的最大深度**，主要逻辑自然放在后序位置。

#### 前序遍历

> 思路1：遍历思路

```cpp
#include <list>

std::list<int> res;

// 返回前序遍历结果
std::list<int> preorderTraverse(TreeNode* root) {
    traverse(root);
    return res;
}

// 二叉树遍历函数
void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    
    // 前序位置
    res.push_back(root->val);
    traverse(root->left);
    traverse(root->right);
}
```

> 思路2：分解问题思路

前序遍历的特点是，根节点的值排在首位，接着是左子树的前序遍历结果，最后是右子树的前序遍历结果

那这不就可以分解问题了么，**一棵二叉树的前序遍历结果 = 根节点 + 左子树的前序遍历结果 + 右子树的前序遍历结果**。

```cpp
// 定义：输入一棵二叉树的根节点，返回这棵树的前序遍历结果
vector<int> preorderTraverse(TreeNode* root) {
    vector<int> res;
    if (root == nullptr) {
        return res;
    }
    // 前序遍历的结果，root->val 在第一个
    res.push_back(root->val);
    // 利用函数定义，后面接着左子树的前序遍历结果
    vector<int> left = preorderTraverse(root->left);
    res.insert(res.end(), left.begin(), left.end());
    // 利用函数定义，最后接着右子树的前序遍历结果
    vector<int> right = preorderTraverse(root->right);
    res.insert(res.end(), right.begin(), right.end());


    return res;
}
```

**这个算法的复杂度不好把控**，比较依赖语言特性。

Java 的话无论 ArrayList 还是 LinkedList，`addAll` 方法的复杂度都是 O(N)，所以总体的最坏时间复杂度会达到 O(N^2)，除非你自己实现一个复杂度为 O(1) 的 `addAll` 方法，底层用链表的话是可以做到的，因为多条链表只要简单的指针操作就能连接起来。

### 总结

综上，遇到一道二叉树的题目时的通用思考过程是：

1. **是否可以通过遍历一遍二叉树得到答案**？如果可以，用一个 `traverse` 函数配合外部变量来实现。
2. **是否可以定义一个递归函数，通过子问题（子树）的答案推导出原问题的答案**？如果可以，写出这个递归函数的定义，并充分利用这个函数的返回值。
3. **无论使用哪一种思维模式，你都要明白二叉树的每一个节点需要做什么，需要在什么时候（前中后序）做**。

### 后序位置的特殊之处

前序位置本身其实没有什么特别的性质，之所以发现好像很多题都是在前序位置写代码，实际上是因为我们**习惯把那些对前中后序位置不敏感的代码写在前序位置**罢了。

中序位置主要用在 BST 场景中，你完全可以把 **BST 的中序遍历认为是遍历有序数组**。

>**仔细观察，前中后序位置的代码，能力依次增强**。
>
>前序位置的代码只能从函数参数中获取父节点传递来的数据。
>
>中序位置的代码不仅可以获取参数数据，还可以获取到左子树通过函数返回值传递回来的数据。
>
>后序位置的代码最强，不仅可以获取参数数据，还可以同时获取到左右子树通过函数返回值传递回来的数据。
>
>所以，某些情况下把代码移到后序位置效率最高；有些事情，只有后序位置的代码能做。

举些具体的例子来感受下它们的能力区别。现在给你一棵二叉树，我问你两个简单的问题：

1. 如果把根节点看做第 1 层，如何打印出每一个节点所在的层数？
2. 如何打印出每个节点的左右子树各有多少节点？

第一个问题可以这样写代码：

```cpp
// 二叉树遍历函数
void traverse(TreeNode* root, int level) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    printf("节点 %s 在第 %d 层", root->val, level);
    traverse(root->left, level + 1);
    traverse(root->right, level + 1);
}

// 这样调用
traverse(root, 1);
```

第二个问题可以这样写代码：

```cpp
// Definition for a binary tree node.
struct TreeNode {
  int val;
  TreeNode *left;
  TreeNode *right;
  TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

// 定义：输入一棵二叉树，返回这棵二叉树的节点总数
int count(TreeNode* root) {
    if (root == nullptr) {
        return 0;
    }
    int leftCount = count(root->left);
    int rightCount = count(root->right);
    // 后序位置
    printf("节点 %p 的左子树有 %d 个节点，右子树有 %d 个节点",
            root, leftCount, rightCount);

    return leftCount + rightCount + 1;
}
```

**这两个问题的根本区别在于：一个节点在第几层，你从根节点遍历过来的过程就能顺带记录，用递归函数的参数就能传递下去；而以一个节点为根的整棵子树有多少个节点，你必须遍历完子树之后才能数清楚，然后通过递归函数的返回值拿到答案**。

结合这两个简单的问题，你品味一下后序位置的特点，只有后序位置才能通过返回值获取子树的信息。

**那么换句话说，一旦你发现题目和子树有关，那大概率要给函数设置合理的定义和返回值，在后序位置写代码了**。

#### 二叉树的直径

> 题目

所谓二叉树的「直径」长度，就是任意两个结点之间的路径长度。最长「直径」并不一定要穿过根结点，比如下面这棵二叉树：

![img](https://labuladong.online/algo/images/%E4%BA%8C%E5%8F%89%E6%A0%91%E6%94%B6%E5%AE%98/tree1.png)

它的最长直径是 3，即 `[4,2,1,3]`，`[4,2,1,9]` 或者 `[5,2,1,3]` 这几条「直径」的长度

> 思路

解决这题的关键在于，**每一条二叉树的「直径」长度，就是一个节点的左右子树的最大深度之和**。

现在让我求整棵树中的最长「直径」，那直截了当的思路就是遍历整棵树中的每个节点，然后通过每个节点的左右子树的最大深度算出每个节点的「直径」，最后把所有「直径」求个最大值即可。

> 代码

```cpp
class Solution {
public:
    // 记录最大直径的长度
    int maxDiameter = 0;

    int diameterOfBinaryTree(TreeNode* root) {
        // 对每个节点计算直径，求最大直径
        traverse(root);
        return maxDiameter;
    }

private:
    // 遍历二叉树
    void traverse(TreeNode* root) {
        if (root == nullptr) {
            return;
        }
        // 对每个节点计算直径
        int leftMax = maxDepth(root->left);
        int rightMax = maxDepth(root->right);
        int myDiameter = leftMax + rightMax;
        // 更新全局最大直径
        maxDiameter = max(maxDiameter, myDiameter);

        traverse(root->left);
        traverse(root->right);
    }

    // 计算二叉树的最大深度
    int maxDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }
        int leftMax = maxDepth(root->left);
        int rightMax = maxDepth(root->right);
        return 1 + max(leftMax, rightMax);
    }
};
```

这个解法是正确的，但是运行时间很长，原因也很明显，`traverse` 遍历每个节点的时候还会调用递归函数 `maxDepth`，而 `maxDepth` 是要遍历子树的所有节点的，所以最坏时间复杂度是 O(N^2)。

这就出现了刚才探讨的情况，**前序位置无法获取子树信息，所以只能让每个节点调用 `maxDepth` 函数去算子树的深度**。

那如何优化？我们应该把计算「直径」的逻辑放在后序位置，准确说应该是放在 `maxDepth` 的后序位置，因为 `maxDepth` 的后序位置是知道左右子树的最大深度的。

```cpp
class Solution {
    // 记录最大直径的长度
    int maxDiameter = 0;

public:
    int diameterOfBinaryTree(TreeNode* root) {
        maxDepth(root);
        return maxDiameter;
    }

    int maxDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }
        int leftMax = maxDepth(root->left);
        int rightMax = maxDepth(root->right);
        // 后序位置，顺便计算最大直径
        int myDiameter = leftMax + rightMax;
        maxDiameter = max(maxDiameter, myDiameter);

        return 1 + max(leftMax, rightMax);
    }
};
```

这下时间复杂度只有 `maxDepth` 函数的 O(N) 了。

### 以树视角看动归/回溯/DFS算法的区别和联系

DFS算法和回溯算法非常类似，只在细节上有差别

这个细节上的差别是什么呢？其实就是「做选择」和「撤销选择」到底在 for 循环外面还是里面的区别，DFS 算法在外面，回溯算法在里面。

>动归/DFS/回溯算法都可以看做二叉树问题的扩展，只是它们的关注点不同：
>
>- 动态规划算法属于分解问题的思路，它的关注点在整棵「子树」。
>- 回溯算法属于遍历的思路，它的关注点在节点间的「树枝」。
>- DFS 算法属于遍历的思路，它的关注点在单个「节点」。

理解上面的核心思想，需要三个栗子：

**第一个例子**，给你一棵二叉树，请你用分解问题的思路写一个 `count` 函数，计算这棵二叉树共有多少个节点。

```cpp
// 定义：输入一棵二叉树，返回这棵二叉树的节点总数
int count(TreeNode* root) {
    if (root == nullptr) {
        return 0;
    }
    // 当前节点关心的是两个子树的节点总数分别是多少
    // 因为用子问题的结果可以推导出原问题的结果
    int leftCount = count(root->left);
    int rightCount = count(root->right);
    // 后序位置，左右子树节点数加上自己就是整棵树的节点数
    return leftCount + rightCount + 1;
}

// 这就是动态规划分解问题的思路，它的着眼点永远是结构相同的整个子问题，类比到二叉树上就是「子树」。
```

**第二个例子**，给你一棵二叉树，请你用遍历的思路写一个 `traverse` 函数，打印出遍历这棵二叉树的过程。

```cpp
void traverse(TreeNode* root) {
    // 如果节点为空（说明已经越过叶节点），就返回
    if (root == nullptr) return;
    // 从节点 root 出发，沿着 left 边走
    printf("从节点 %s 进入节点 %s", root, root->left);
    traverse(root->left);
    // 从节点 root 的左边回来
    printf("从节点 %s 回到节点 %s", root->left, root);

    // 从节点 root 出发，沿着 right 边走
    printf("从节点 %s 进入节点 %s", root, root->right);
    traverse(root->right);
    // 从节点 root 的右边回来
    printf("从节点 %s 回到节点 %s", root->right, root);
}

// 这就是回溯算法遍历的思路，它的着眼点永远是在节点之间移动的过程，类比到二叉树上就是「树枝」。
```

**第三个例子**，我给你一棵二叉树，请你写一个 `traverse` 函数，把这棵二叉树上的每个节点的值都加一。

```cpp
void traverse(TreeNode* root) {
    if (root == nullptr) return;
    // 遍历过的每个节点的值加一
    root->val++;
    traverse(root->left);
    traverse(root->right);
}
// 这就是 DFS 算法遍历的思路，它的着眼点永远是在单一的节点上，类比到二叉树上就是处理每个「节点」。
```

有了这些铺垫，你就很容易理解为什么回溯算法和 DFS 算法代码中「做选择」和「撤销选择」的位置不同了，看下面两段代码：

```cpp
// DFS 算法把「做选择」「撤销选择」的逻辑放在 for 循环外面
void dfs(Node* root) {
    if (!root) return;
    // 做选择
    printf("我已经进入节点 %s 啦\n", root->val.c_str());
    for (Node* child : root->children) {
        dfs(child);
    }
    // 撤销选择
    printf("我将要离开节点 %s 啦\n", root->val.c_str());
}

// 回溯算法把「做选择」「撤销选择」的逻辑放在 for 循环里面
void backtrack(Node* root) {
    if (!root) return;
    for (Node* child : root->children) {
        // 做选择
        printf("我站在节点 %s 到节点 %s 的树枝上\n", root->val.c_str(), child->val.c_str());
        backtrack(child);
        // 撤销选择
        printf("我将要离开节点 %s 到节点 %s 的树枝上\n", child->val.c_str(), root->val.c_str());
    }
}
```

### 层序遍历

```cpp
// 层序遍历属于迭代思维
// 输入一棵二叉树的根节点，层序遍历这棵二叉树
void levelTraverse(TreeNode* root) {
    if (root == nullptr) return;
    queue<TreeNode*> q;
    q.push(root);

    // 从上到下遍历二叉树的每一层
    while (!q.empty()) {
        int sz = q.size();
        // 从左到右遍历每一层的每个节点
        for (int i = 0; i < sz; i++) {
            TreeNode* cur = q.front();
            q.pop();
            // 将下一层节点放入队列
            if (cur->left != nullptr) {
                q.push(cur->left);
            }
            if (cur->right != nullptr) {
                q.push(cur->right);
            }
        }
    }
}
```

### 额外

如果你对二叉树足够熟悉，可以想到很多方式通过递归函数得到层序遍历结果，比如下面这种写法：

```cpp
class Solution {
public:
    vector<vector<int>> res;

    vector<vector<int>> levelTraverse(TreeNode* root) {
        if (root == nullptr) {
            return res;
        }
        // root 视为第 0 层
        traverse(root, 0);
        return res;
    }

    void traverse(TreeNode* root, int depth) {
        if (root == nullptr) {
            return;
        }
        // 前序位置，看看是否已经存储 depth 层的节点了
        if (res.size() <= depth) {
            // 第一次进入 depth 层
            res.push_back(vector<int> {});
        }
        // 前序位置，在 depth 层添加 root 节点的值
        res[depth].push_back(root->val);
        traverse(root->left, depth + 1);
        traverse(root->right, depth + 1);
    }
};
```

这种思路从结果上说确实可以得到层序遍历结果，但其本质还是二叉树的前序遍历，或者说 DFS 的思路，而不是层序遍历，或者说 BFS 的思路。因为这个解法是依赖前序遍历自顶向下、自左向右的顺序特点得到了正确的结果。

**抽象点说，这个解法更像是从左到右的「列序遍历」，而不是自顶向下的「层序遍历」**。所以对于计算最小距离的场景，这个解法完全等同于 DFS 算法，没有 BFS 算法的性能的优势。

下面是用递归进行层序遍历：

```cpp
class Solution {
private:
    vector<vector<int>> res;
    
public:
    vector<vector<int>> levelTraverse(TreeNode* root) {
        if (root == NULL) {
            return res;
        }
        vector<TreeNode*> nodes;
        nodes.push_back(root);
        traverse(nodes);
        return res;
    }

    void traverse(vector<TreeNode*>& curLevelNodes) {
        // base case
        if (curLevelNodes.empty()) {
            return;
        }

        // 前序位置，计算当前层的值和下一层的节点列表
        vector<int> nodeValues;
        vector<TreeNode*> nextLevelNodes;
        for (TreeNode* node : curLevelNodes) {
            nodeValues.push_back(node->val);
            if (node->left != NULL) {
                nextLevelNodes.push_back(node->left);
            }
            if (node->right != NULL) {
                nextLevelNodes.push_back(node->right);
            }
        }

        // 前序位置添加结果，可以得到自顶向下的层序遍历
        res.push_back(nodeValues);

        traverse(nextLevelNodes);

        // 后序位置添加结果，可以得到自底向上的层序遍历结果
        // res.push_back(nodeValues);
    }
};
```

## 动态规划框架

首先，**动态规划问题的一般形式就是求最值**。动态规划其实是运筹学的一种最优化方法，只不过在计算机问题上应用比较多，比如说让你求最长递增子序列呀，最小编辑距离呀等等。

既然是要求最值，核心问题是什么呢？**求解动态规划的核心问题是穷举**。因为要求最值，肯定要把所有可行的答案穷举出来，然后在其中找最值呗。

动态规划这么简单，就是穷举就完事了？我看到的动态规划问题都很难啊！

首先，虽然动态规划的核心思想就是穷举求最值，但是问题可以千变万化，穷举所有可行解其实并不是一件容易的事，需要你熟练掌握递归思维，只有列出**正确的「状态转移方程」**，才能正确地穷举。而且，你需要判断算法问题是否**具备「最优子结构」**，是否能够通过子问题的最值得到原问题的最值。另外，动态规划问题**存在「重叠子问题」**，如果暴力穷举的话效率会很低，所以需要你使用「备忘录」或者「DP table」来优化穷举过程，避免不必要的计算。

**明确「状态」-> 明确「选择」 -> 定义 `dp` 数组/函数的含义**。 这就是辅助思考状态转移方程的过程

```python
# 自顶向下的动态规划
def dp(状态1, 状态2, ...):
	for 选择 in 所有可能的选择:
        # 此时的状态已经因为做了选择而改变
        result = 求最值(result, dp(状态1, 状态2, ...))
     return result;


# 自底向上迭代的动态规划
# 初始化 base case
dp[0][0][...] = base case
# 进行状态转移
for 状态1 in 状态1的所有取值：
	for 状态2 in 状态2的所有取值：
    	for ...
        	dp[状态1][状态2][...] = 求最值(选择1， 选择2...)
```

### 斐波那契数列（理解重叠子问题）

> **暴力递归**

```cpp
int fib(int N) {
    if (N == 1 || N == 2) return 1;
    return fib(N - 1) + fib(N - 2);
}
```

![img](https://labuladong.online/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/1.jpg)

这个递归树怎么理解？就是说想要计算原问题 `f(20)`，我就得先计算出子问题 `f(19)` 和 `f(18)`，然后要计算 `f(19)`，我就要先算出子问题 `f(18)` 和 `f(17)`，以此类推。最后遇到 `f(1)` 或者 `f(2)` 的时候，结果已知，就能直接返回结果，递归树不再向下生长了。

**递归算法的时间复杂度怎么计算？就是用子问题个数乘以解决一个子问题需要的时间**。

首先计算子问题个数，即递归树中节点的总数。显然二叉树节点总数为指数级别，所以子问题个数为 O(2^n)。

然后计算解决一个子问题的时间，在本算法中，没有循环，只有 `f(n - 1) + f(n - 2)` 一个加法操作，时间为 O(1)。

所以，这个算法的时间复杂度为二者相乘，即 O(2^n)，指数级别，爆炸。

观察递归树，很明显发现了算法低效的原因：**存在大量重复计算**，比如 `f(18)` 被计算了两次，而且你可以看到，以 `f(18)` 为根的这个递归树体量巨大，多算一遍，会耗费巨大的时间。更何况，还不止 `f(18)` 这一个节点被重复计算，所以这个算法及其低效。

这就是动态规划问题的第一个性质：**重叠子问题**。下面，我们想办法解决这个问题。

> **带备忘录的递归解法**

明确了问题，其实就已经把问题解决了一半。即然耗时的原因是重复计算，那么我们可以造一个「备忘录」，每次算出某个子问题的答案后别急着返回，先记到「备忘录」里再返回；每次遇到一个子问题先去「备忘录」里查一查，如果发现之前已经解决过这个问题了，直接把答案拿出来用，不要再耗时去计算了。

一般使用一个数组充当这个「备忘录」，当然你也可以使用哈希表（字典），思想都是一样的。

```cpp
int fib(int N) {
    // 备忘录全初始化为 0
    int memo[N + 1];
    memset(memo, 0, sizeof(memo));
    // 进行带备忘录的递归
    return dp(memo, N);
}

// 带着备忘录进行递归
int dp(int memo[], int n) {
    // base case
    if (n == 0 || n == 1) return n;
    // 已经计算过，不用再计算了
    if (memo[n] != 0) return memo[n];
    memo[n] = dp(memo, n - 1) + dp(memo, n - 2);
    return memo[n];
}
```

![img](https://labuladong.online/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/2.jpg)

实际上，带「备忘录」的递归算法，把一棵存在巨量冗余的递归树通过「剪枝」，改造成了一幅不存在冗余的递归图，极大减少了子问题（即递归图中节点）的个数。

**递归算法的时间复杂度怎么计算？就是用子问题个数乘以解决一个子问题需要的时间**。

子问题个数，即图中节点的总数，由于本算法不存在冗余计算，子问题就是 `f(1)`, `f(2)`, `f(3)` ... `f(20)`，数量和输入规模 n = 20 成正比，所以子问题个数为 O(n)。

解决一个子问题的时间，同上，没有什么循环，时间为 O(1)。

所以，本算法的时间复杂度是 O(n)，比起暴力算法，是降维打击。

至此，**带备忘录的递归解法的效率已经和迭代的动态规划解法一样了。**实际上，这种解法和常见的动态规划解法已经差不多了，只不过这种解法是「自顶向下」进行「递归」求解，我们更常见的动态规划代码是「自底向上」进行「递推」求解。

>啥叫「自顶向下」？注意我们刚才画的递归树（或者说图），是从上向下延伸，都是从一个规模较大的原问题比如说 `f(20)`，向下逐渐分解规模，直到 `f(1)` 和 `f(2)` 这两个 base case，然后逐层返回答案，这就叫「自顶向下」。

> 啥叫「自底向上」？反过来，我们直接从最底下、最简单、问题规模最小、已知结果的 `f(1)` 和 `f(2)`（base case）开始往上推，直到推到我们想要的答案 `f(20)`。这就是「递推」的思路，这也是动态规划一般都脱离了递归，而是由循环迭代完成计算的原因。

> **`dp` 数组的迭代（递推）解法**

有了上一步「备忘录」的启发，我们可以把这个「备忘录」独立出来成为一张表，通常叫做 DP table，在这张表上完成「自底向上」的推算岂不美哉！

```cpp
int fib(int N) {
    if (N == 0) return 0;
    int* dp = new int[N + 1];
    // base case
    dp[0] = 0; dp[1] = 1;
    // 状态转移
    for (int i = 2; i <= N; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }

    return dp[N];
}
```

![img](https://labuladong.online/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/4.jpg)

### 凑零钱问题（最优子结构）

> 题目

给你 `k` 种面值的硬币，面值分别为 `c1, c2 ... ck`，每种硬币的数量无限，再给一个总金额 `amount`，问你**最少**需要几枚硬币凑出这个金额，如果不可能凑出，算法返回 -1 。

> **暴力递归**

首先，这个问题是动态规划问题，因为它具有「最优子结构」的。**要符合「最优子结构」，子问题间必须互相独立**。啥叫相互独立？你肯定不想看数学证明，我用一个直观的例子来讲解。

比如说，假设你考试，每门科目的成绩都是互相独立的。你的原问题是考出最高的总成绩，那么你的子问题就是要把语文考到最高，数学考到最高…… 为了每门课考到最高，你要把每门课相应的选择题分数拿到最高，填空题分数拿到最高…… 当然，最终就是你每门课都是满分，这就是最高的总成绩。

得到了正确的结果：最高的总成绩就是总分。因为这个过程符合最优子结构，「每门科目考到最高」这些子问题是互相独立，互不干扰的。

但是，如果加一个条件：你的语文成绩和数学成绩会互相制约，不能同时达到满分，数学分数高，语文分数就会降低，反之亦然。

这样的话，显然你能考到的最高总成绩就达不到总分了，按刚才那个思路就会得到错误的结果。因为「每门科目考到最高」的子问题并不独立，语文数学成绩户互相影响，无法同时最优，所以最优子结构被破坏。

回到凑零钱问题，为什么说它符合最优子结构呢？假设你有面值为 `1, 2, 5` 的硬币，你想求 `amount = 11` 时的最少硬币数（原问题），如果你知道凑出 `amount = 10, 9, 6` 的最少硬币数（子问题），你只需要把子问题的答案加一（再选一枚面值为 `1, 2, 5` 的硬币），求个最小值，就是原问题的答案。因为硬币的数量是没有限制的，所以子问题之间没有相互制，是互相独立的。

> **如何思考动态转移方程**

1. **确定「状态」，也就是原问题和子问题中会变化的变量**。由于硬币数量无限，硬币的面额也是题目给定的，只有目标金额会不断地向 base case 靠近，所以唯一的「状态」就是目标金额 `amount`。
2. **确定「选择」，也就是导致「状态」产生变化的行为**。目标金额为什么变化呢，因为你在选择硬币，你每选择一枚硬币，就相当于减少了目标金额。所以说所有硬币的面值，就是你的「选择」。
3. **明确 `dp` 函数/数组的定义**。我们这里讲的是自顶向下的解法，所以会有一个递归的 `dp` 函数，一般来说函数的参数就是状态转移中会变化的量，也就是上面说到的「状态」；函数的返回值就是题目要求我们计算的量。就本题来说，状态只有一个，即「目标金额」，题目要求我们计算凑出目标金额所需的最少硬币数量。

**所以我们可以这样定义 `dp` 函数：`dp(n)` 表示，输入一个目标金额 `n`，返回凑出目标金额 `n` 所需的最少硬币数量**。

```cpp
int dp(vector<int>& coins, int amount);

int coinChange(vector<int>& coins, int amount) {
    // 题目要求的最终结果是 dp(amount)
    return dp(coins, amount);
}

// 定义：要凑出金额 n，至少要 dp(coins, n) 个硬币
int dp(vector<int>& coins, int amount) {
    // base case
    if (amount == 0) return 0;
    if (amount < 0) return -1;

    int res = INT_MAX;
    for (int coin : coins) {
        // 计算子问题的结果
        int subProblem = dp(coins, amount - coin);
        // 子问题无解则跳过
        if (subProblem == -1) continue;
        // 在子问题中选择最优解，然后加一
        res = min(res, subProblem + 1);
    }

    return res == INT_MAX ? -1 : res;
}
```

![img](https://labuladong.online/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/5.jpg)

**递归算法的时间复杂度分析：子问题总数 x 解决每个子问题所需的时间**。

子问题总数为递归树的节点个数，但算法会进行剪枝，剪枝的时机和题目给定的具体硬币面额有关，所以可以想象，这棵树生长的并不规则，确切算出树上有多少节点是比较困难的。对于这种情况，我们一般的做法是按照最坏的情况估算一个时间复杂度的上界。

假设目标金额为 `n`，给定的硬币个数为 `k`，那么递归树最坏情况下高度为 `n`（全用面额为 1 的硬币），然后再假设这是一棵满 `k` 叉树，则节点的总数在 `k^n` 这个数量级。

接下来看每个子问题的复杂度，由于每次递归包含一个 for 循环，复杂度为 `O(k)`，相乘得到总时间复杂度为 `O(k^n)`，指数级别。

> **带备忘录的递归**

```cpp
class Solution {
    private:
        vector<int> memo;
        
        /**
         * memo 数组记录了 amount 对应的最优解，-1 代表还未被计算。
         * 因此，在计算 amount 对应的最优解时需要先查找 memo。
         */
        int dp(vector<int>& coins, int amount){
            if(amount == 0) return 0; // 如果 amount = 0，那么硬币数量为 0
            if(amount < 0) return -1; // 如果 amount < 0，那么无解
            if(memo[amount] != -1) return memo[amount]; // 查备忘录，如果有最优解则直接返回
            
            int res = INT_MAX;
            /**
             * 在硬币数组 coins 中依次选取每个硬币。
             * 对于当前选择的硬币，计算是包含该硬币所需的子问题的最优解 subProblem。
             * 如果子问题无解，则直接跳过该硬币。
             * 在所有子问题中，选取最优解，并加上该硬币的数量。
             * 最终的结果 res 即为 amount 对应的最优解，该结果存入 memo 中。
             */
            for(int coin : coins){
                int subProblem = dp(coins, amount - coin);
                if(subProblem == -1) continue;
                res = min(res, subProblem + 1);
            }
            
            memo[amount] = res == INT_MAX ? -1 : res;
            return memo[amount];
        }
        
    public:
        int coinChange(vector<int>& coins, int amount) {
            /**
             * 初始化备忘录 memo，memo 数组的长度为 amount+1。
             * memo 数组记录了 amount 对应的最优解，-1 代表还未被计算。
             * 因此，在计算 amount 对应的最优解时需要先查找 memo，再根据结果进行计算。
             */
            memo = vector<int>(amount + 1, -666);
            return dp(coins, amount);
        }
};
```

不画图了，很显然「备忘录」大大减小了子问题数目，完全消除了子问题的冗余，所以子问题总数不会超过金额数 `n`，即子问题数目为 `O(n)`。处理一个子问题的时间不变，仍是 `O(k)`，所以总的时间复杂度是 `O(kn)`。

> **dp 数组的迭代解法**

我们也可以自底向上使用 dp table 来消除重叠子问题，关于「状态」「选择」和 base case 与之前没有区别，`dp` 数组的定义和刚才 `dp` 函数类似，也是把「状态」，也就是目标金额作为变量。不过 `dp` 函数体现在函数参数，而 `dp` 数组体现在数组索引：

**`dp` 数组的定义：当目标金额为 `i` 时，至少需要 `dp[i]` 枚硬币凑出**。

```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);
    // 数组大小为 amount + 1，初始值也为 amount + 1
    dp[0] = 0;

    // 外层 for 循环在遍历所有状态的所有取值
    for (int i = 0; i < dp.size(); i++) {
        // 内层 for 循环在求所有选择的最小值
        for (int coin : coins) {
            // 子问题无解，跳过
            if (i - coin < 0) {
                continue;
            }
            dp[i] = min(dp[i], 1 + dp[i - coin]);
        }
    }
    return (dp[amount] == amount + 1) ? -1 : dp[amount];
}

为啥 dp 数组中的值都初始化为 amount + 1 呢，因为凑成 amount 金额的硬币数最多只可能等于 amount（全用 1 元面值的硬币），所以初始化为 amount + 1 就相当于初始化为正无穷，便于后续取最小值。为啥不直接初始化为 int 型的最大值 Integer.MAX_VALUE 呢？因为后面有 dp[i - coin] + 1，这就会导致整型溢出。
```

![img](https://labuladong.online/algo/images/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92%E8%AF%A6%E8%A7%A3%E8%BF%9B%E9%98%B6/6.jpg)

### 总结

第一个斐波那契数列的问题，解释了如何通过「备忘录」或者「dp table」的方法来优化递归树，并且明确了这两种方法本质上是一样的，只是自顶向下和自底向上的不同而已。

第二个凑零钱的问题，展示了如何流程化确定「状态转移方程」，只要通过状态转移方程写出暴力递归解，剩下的也就是优化递归树，消除重叠子问题而已。

**计算机解决问题其实没有任何特殊的技巧，它唯一的解决办法就是穷举**，穷举所有可能性。算法设计无非就是先思考“如何穷举”，然后再追求“如何聪明地穷举”。

列出状态转移方程，就是在解决“如何穷举”的问题。之所以说它难，一是因为很多穷举需要递归实现，二是因为有的问题本身的解空间复杂，不那么容易穷举完整。

备忘录、DP table 就是在追求“如何聪明地穷举”。用空间换时间的思路，是降低时间复杂度的不二法门。

## 回溯算法框架

**抽象地说，解决一个回溯问题，实际上就是遍历一棵决策树的过程，树的每个叶子节点存放着一个合法答案。你把整棵树遍历一遍，把叶子节点上的答案都收集起来，就能得到所有的合法答案**。

站在回溯树的一个节点上，你只需要思考 3 个问题：

1. 路径：也就是已经做出的选择。
2. 选择列表：也就是你当前可以做的选择。
3. 结束条件：也就是到达决策树底层，无法再做选择的条件。

```python
result = []
def backTrack(路径, 选择列表):
    if 满足结束条件:
        result.addd(路径)
        return
    
    for 选择 in 选择列表:
        # 做选择
        将该选择从选择列表移除
        路径.add(选择)
        backtrack(路径, 选择列表)
        # 撤销选择
        路径.remove(选择)
        将该选择再加入选择列表
```

### 全排列问题

给你输入一个数组 `nums`，让你返回这些数字的全排列。（数组`nums`中不包含重复数字）

比方说给三个数 `[1,2,3]`，你肯定不会无规律地乱穷举，一般是这样：

先固定第一位为 1，然后第二位可以是 2，那么第三位只能是 3；然后可以把第二位变成 3，第三位就只能是 2 了；然后就只能变化第一位，变成 2，然后再穷举后两位……

![img](https://labuladong.online/algo/images/backtracking/1.jpg)

只要从根遍历这棵树，记录路径上的数字，其实就是所有的全排列。**我们不妨把这棵树称为回溯算法的「决策树」**。

**为啥说这是决策树呢，因为你在每个节点上其实都在做决策**。

**我们定义的 `backtrack` 函数其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层叶子节点，其「路径」就是一个全排列**。

```cpp
class Solution {
    private:
        vector<vector<int>> res;

        /* 主函数，输入一组不重复的数字，返回它们的全排列 */
        vector<vector<int>> permute(vector<int>& nums) {
            // 记录「路径」
            vector<int> track;
            //「路径」中的元素会被标记为true，避免重复使用
            vector<bool> used(nums.size(), false);

            backtrack(nums, track, used);
            return res;
        }

        // 路径：记录在 track 中
        // 选择列表：nums 中不存在于 track 的那些元素
        // 结束条件：nums 中的元素全都在 track 中出现
        void backtrack(vector<int>& nums, vector<int>& track, vector<bool>& used) {
            // 触发结束条件
            if (track.size() == nums.size()) {
                res.push_back(track);
                return;
            }

            for (int i = 0; i < nums.size(); i++) {
                // 排除不合法的选择
                if (used[i]) {
                    //nums[i] 已经在 track中，跳过
                    continue;
                }
                // 做选择
                track.push_back(nums[i]);
                used[i] = true;
                // 进入下一层决策树
                backtrack(nums, track, used);
                // 取消选择
                track.pop_back();
                used[i] = false;
            }
        }
    public:
        vector<vector<int>> permute(vector<int>& nums) {
            return permute(nums);
        }
};
```

我们这里稍微做了些变通，没有显式记录「选择列表」，而是通过 `used` 数组排除已经存在 `track` 中的元素，从而推导出当前的选择列表：

![img](https://labuladong.online/algo/images/backtracking/6.jpg)

至此，我们就通过全排列问题详解了回溯算法的底层原理。当然，这个算法解决全排列不是最高效的，你可能看到有的解法连 `used` 数组都不使用，通过交换元素达到目的。但是那种解法稍微难理解一些，这里就不写了，有兴趣可以自行搜索一下。

但是必须说明的是，不管怎么优化，都符合回溯框架，而且时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的，你最后肯定要穷举出 N! 种全排列结果。**这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高**。

### N皇后问题

给你一个 `N×N` 的棋盘，让你放置 `N` 个皇后，使得它们不能互相攻击，请你计算出所有可能的放法。皇后可以攻击同一行、同一列、左上左下右上右下四个方向的任意单位。

```cpp
class Solution {
private:
    vector<vector<string>> res;

public:
    // 输入棋盘边长 n，返回所有合法的放置
    vector<vector<string>> solveNQueens(int n) {
        // 每个字符串代表一行，字符串列表代表一个棋盘
        // '.' 表示空，'Q' 表示皇后，初始化空棋盘
        vector<string> board(n, string(n, '.'));
        backtrack(board, 0);
        return res;
    }

    // 路径：board 中小于 row 的那些行都已经成功放置了皇后
    // 选择列表：第 row 行的所有列都是放置皇后的选择
    // 结束条件：row 超过 board 的最后一行
    void backtrack(vector<string>& board, int row) {
        // 触发结束条件
        if (row == board.size()) {
            res.push_back(board);
            return;
        }
        
        int n = board[row].size();
        for (int col = 0; col < n; col++) {
            // 排除不合法选择
            if (!isValid(board, row, col)) {
                continue;
            }
            // 做选择
            board[row][col] = 'Q';
            // 进入下一行决策
            backtrack(board, row + 1);
            // 撤销选择
            board[row][col] = '.';
        }
    }

    // 是否可以在 board[row][col] 放置皇后？
    bool isValid(vector<string>& board, int row, int col) {
        int n = board.size();
        // 检查列是否有皇后互相冲突
        for (int i = 0; i <= row; i++) {
            if (board[i][col] == 'Q') {
                return false;
            }
        }
        // 检查右上方是否有皇后互相冲突
        for (int i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
            if (board[i][j] == 'Q') {
                return false;
            }
        }
        // 检查左上方是否有皇后互相冲突
        for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
            if (board[i][j] == 'Q') {
                return false;
            }
        }
        return true;
    }
};
```

`N` 行棋盘中，第一行有 `N` 个位置可能可以放皇后，第二行有 `N - 1` 个位置，第三行有 `N - 2` 个位置，以此类推，再叠加每次放皇后之前 `isValid` 函数所需的 O(N) 复杂度，所以总的时间复杂度上界是 O(N! * N)，而且没有什么明显的冗余计算可以优化效率。你可以试试 `N = 10` 的时候，计算就已经很耗时了。

## 回溯算法解决排列-组合-子集问题

无论是排列、组合还是子集问题，简单说无非就是让你从序列 `nums` 中以给定规则取若干元素，主要有以下几种变体：

**形式一、元素无重不可复选，即 `nums` 中的元素都是唯一的，每个元素最多只能被使用一次，这也是最基本的形式**。

以组合为例，如果输入 `nums = [2,3,6,7]`，和为 7 的组合应该只有 `[7]`。

**形式二、元素可重不可复选，即 `nums` 中的元素可以存在重复，每个元素最多只能被使用一次**。

以组合为例，如果输入 `nums = [2,5,2,1,2]`，和为 7 的组合应该有两种 `[2,2,2,1]` 和 `[5,2]`。

**形式三、元素无重可复选，即 `nums` 中的元素都是唯一的，每个元素可以被使用若干次**。

以组合为例，如果输入 `nums = [2,3,6,7]`，和为 7 的组合应该有两种 `[2,2,3]` 和 `[7]`。

上面用组合问题举的例子，但排列、组合、子集问题都可以有这三种基本形式，所以共有 9 种变化。

除此之外，题目也可以再添加各种限制条件，比如让你求和为 `target` 且元素个数为 `k` 的组合，那这么一来又可以衍生出一堆变体，怪不得面试笔试中经常考到排列组合这种基本题型。

**但无论形式怎么变化，其本质就是穷举所有解，而这些解呈现树形结构，所以合理使用回溯算法框架，稍改代码框架即可把这些问题一网打尽**。

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/1.jpeg)

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/2.jpeg)

### 子集（元素无重不可复选）

> 题目

题目给你输入一个无重复元素的数组 `nums`，其中每个元素最多使用一次，请你返回 `nums` 的所有子集。

> 思路

比如输入 `nums = [1,2,3]`，算法应该返回如下子集：

```cpp
[ [],[1],[2],[3],[1,2],[1,3],[2,3],[1,2,3] ]
```

好，我们暂时不考虑如何用代码实现，先回忆一下我们的高中知识，如何手推所有子集？

首先，生成元素个数为 0 的子集，即空集 `[]`，为了方便表示，我称之为 `S_0`。

然后，在 `S_0` 的基础上生成元素个数为 1 的所有子集，我称为 `S_1`：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/3.jpeg)

接下来，我们可以在 `S_1` 的基础上推导出 `S_2`，即元素个数为 2 的所有子集：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/4.jpeg)

因为集合中的元素不用考虑顺序，`[1,2,3]` 中 `2` 后面只有 `3`，如果你添加了前面的 `1`，那么 `[2,1]` 会和之前已经生成的子集 `[1,2]` 重复。

**换句话说，我们通过保证元素之间的相对顺序不变来防止出现重复的子集**。

接着，我们可以通过 `S_2` 推出 `S_3`，实际上 `S_3` 中只有一个集合 `[1,2,3]`，它是通过 `[1,2]` 推出的。

整个推导过程就是这样一棵树：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/5.jpeg)

**如果把根节点作为第 0 层，将每个节点和根节点之间树枝上的元素作为该节点的值，那么第 `n` 层的所有节点就是大小为 `n` 的所有子集**。

你比如大小为 2 的子集就是这一层节点的值：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/6.jpeg)

那么再进一步，如果想计算所有子集，那只要遍历这棵多叉树，把所有节点的值收集起来不就行了？

> 代码

```cpp
class Solution {
private:
    vector<vector<int>> res;
    // 记录回溯算法的递归路径
    vector<int> track;

public:
    // 主函数
    vector<vector<int>> subsets(vector<int>& nums) {
        backtrack(nums, 0);
        return res;
    }

    // 回溯算法核心函数，遍历子集问题的回溯树
    void backtrack(vector<int>& nums, int start) {

        // 前序位置，每个节点的值都是一个子集
        res.push_back(track);

        // 回溯算法标准框架
        for (int i = start; i < nums.size(); i++) {
            // 做选择
            track.push_back(nums[i]);
            // 通过 start 参数控制树枝的遍历，避免产生重复的子集
            backtrack(nums, i + 1);
            // 撤销选择
            track.pop_back();
        }
    }
};
```

### 组合（元素无重不可复选）

如果你能够成功的生成所有无重子集，那么你稍微改改代码就能生成所有无重组合了。

你比如说，让你在 `nums = [1,2,3]` 中拿 2 个元素形成所有的组合，你怎么做？

稍微想想就会发现，大小为 2 的所有组合，不就是所有大小为 2 的子集嘛。

**所以我说组合和子集是一样的：大小为 `k` 的组合就是大小为 `k` 的子集**。

> 题目

给定两个整数 `n` 和 `k`，返回范围 `[1, n]` 中所有可能的 `k` 个数的组合。

> 思路

比如 `combine(3, 2)` 的返回值应该是：

```cpp
[ [1,2],[1,3],[2,3] ]
```

这是标准的组合问题，但我给你翻译一下就变成子集问题了：

**给你输入一个数组 `nums = [1,2..,n]` 和一个正整数 `k`，请你生成所有大小为 `k` 的子集**。

还是以 `nums = [1,2,3]` 为例，刚才让你求所有子集，就是把所有节点的值都收集起来；**现在你只需要把第 2 层（根节点视为第 0 层）的节点收集起来，就是大小为 2 的所有组合**：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/6.jpeg)

反映到代码上，只需要稍改 base case，控制算法仅仅收集第 `k` 层节点的值即可：

```cpp
class Solution {
public:
    vector<vector<int>> res;
    // 记录回溯算法的递归路径
    deque<int> track;

    // 主函数
    vector<vector<int>> combine(int n, int k) {
        backtrack(1, n, k);
        return res;
    }

    void backtrack(int start, int n, int k) {
        // base case
        if (k == track.size()) {
            // 遍历到了第 k 层，收集当前节点的值
            res.push_back(vector<int>(track.begin(), track.end()));
            return;
        }

        // 回溯算法标准框架
        for (int i = start; i <= n; i++) {
            // 选择
            track.push_back(i);
            // 通过 start 参数控制树枝的遍历，避免产生重复的子集
            backtrack(i + 1, n, k);
            // 撤销选择
            track.pop_back();
        }
    }
}
```

### 排列（无重不可复选）

> 题目

给定一个**不含重复数字**的数组 `nums`，返回其所有可能的**全排列**。

> 思路

比如输入 `nums = [1,2,3]`，函数的返回值应该是：

```cpp
[
    [1,2,3],[1,3,2],
    [2,1,3],[2,3,1],
    [3,1,2],[3,2,1]
]
```

刚才讲的组合/子集问题使用 `start` 变量保证元素 `nums[start]` 之后只会出现 `nums[start+1..]` 中的元素，通过固定元素的相对位置保证不出现重复的子集。

**但排列问题本身就是让你穷举元素的位置，`nums[i]` 之后也可以出现 `nums[i]` 左边的元素，所以之前的那一套玩不转了，需要额外使用 `used` 数组来标记哪些元素还可以被选择**。

标准全排列可以抽象成如下这棵多叉树：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/7.jpeg)

我们用 `used` 数组标记已经在路径上的元素避免重复选择，然后收集所有叶子节点上的值，就是所有全排列的结果：

> 代码

```cpp
class Solution {
public:
    // 存储所有排列结果的列表
    vector<vector<int>> res;
    // 记录回溯算法的递归路径
    list<int> track;
    // 标记数字使用状态的数组，0 表示未被使用，1 表示已被使用
    bool* used;

    /* 主函数，输入一组不重复的数字，返回它们的全排列 */
    vector<vector<int>> permute(vector<int>& nums) {
        used = new bool[nums.size()]();
        // 满足回溯框架时需要添加 bool 类型默认初始化为 false
        backtrack(nums);
        return res;
    }

    // 回溯算法核心函数
    void backtrack(vector<int>& nums) {
        // base case，到达叶子节点
        if (track.size() == nums.size()) {
            // 收集叶子节点上的值
            res.push_back(vector<int>(track.begin(), track.end()));
            return;
        }
        // 回溯算法标准框架
        for (int i = 0; i < nums.size(); i++) {
            // 已经存在 track 中的元素，不能重复选择
            if (used[i]) {
                continue;
            }
            // 做选择
            used[i] = true;
            track.push_back(nums[i]);
            // 进入下一层回溯树
            backtrack(nums);
            // 取消选择
            track.pop_back();
            used[i] = false;
        }
    }
};

// 但如果题目不让你算全排列，而是让你算元素个数为 k 的排列，怎么算？
// 也很简单，改下 backtrack 函数的 base case，仅收集第 k 层的节点值即可：
```

### 子集/组合（元素可重不可复选）

> 题目

给你一个整数数组 `nums`，其中可能包含重复元素，请你返回该数组所有可能的子集。

> 思路

比如输入 `nums = [1,2,2]`，你应该输出：

```cpp
[ [],[1],[2],[1,2],[2,2],[1,2,2] ]
```

当然，按道理说「集合」不应该包含重复元素的，但既然题目这样问了，我们就忽略这个细节吧，仔细思考一下这道题怎么做才是正事。

就以 `nums = [1,2,2]` 为例，为了区别两个 `2` 是不同元素，后面我们写作 `nums = [1,2,2']`。

按照之前的思路画出子集的树形结构，显然，两条值相同的相邻树枝会产生重复：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/8.jpeg)

```cpp
[ 
    [],
    [1],[2],[2'],
    [1,2],[1,2'],[2,2'],
    [1,2,2']
]
```

你可以看到，`[2]` 和 `[1,2]` 这两个结果出现了重复，所以我们需要进行剪枝，如果一个节点有多条值相同的树枝相邻，**则只遍历第一条，剩下的都剪掉**，不要去遍历：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/9.jpeg)

**体现在代码上，需要先进行排序，让相同的元素靠在一起，如果发现 `nums[i] == nums[i-1]`，则跳过**：

```cpp
class Solution {
    vector<vector<int>> res; // 输出结果
    vector<int> track; // 搜索路径
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end()); // 排序，让相同的元素靠在一起
        backtrack(nums, 0);
        return res; // 返回结果
    }

    void backtrack(vector<int>& nums, int start) { // start 为当前的枚举位置
        res.emplace_back(track); // 前序位置，每个节点的值都是一个子集
        for(int i = start; i < nums.size(); i++) {
            if (i > start && nums[i] == nums[i - 1]) { // 剪枝逻辑，值相同的相邻树枝，只遍历第一条
                continue;
            }
            track.emplace_back(nums[i]); // 添加至路径
            backtrack(nums, i + 1); // 进入下一层决策树
            track.pop_back(); // 回溯
        }
    }
};
```

> 题目

给你输入 `candidates` 和一个目标和 `target`，从 `candidates` 中找出中所有和为 `target` 的组合。

> 思路

`candidates` 可能存在重复元素，且其中的每个数字最多只能使用一次。

说这是一个组合问题，其实换个问法就变成子集问题了：请你计算 `candidates` 中所有和为 `target` 的子集。

所以这题怎么做呢？对比子集问题的解法，只要额外用一个 `trackSum` 变量记录回溯路径上的元素和，然后将 base case 改一改即可解决这道题：

```cpp
class Solution {
public:
    vector<vector<int>> res;
    //记录回溯的路径
    vector<int> track;
    //记录 track 中的元素之和
    int trackSum = 0;

    vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
        // 先排序，让相同的元素靠在一起
        sort(candidates.begin(), candidates.end());
        backtrack(candidates, 0, target);
        return res;
    }

    //回溯算法主函数
    void backtrack(vector<int>& nums, int start, int target) {
        //base case，达到目标和，找到符合条件的组合
        if(trackSum == target) {
            res.push_back(track);
            return;
        }
        //base case，超过目标和，直接结束
        if(trackSum > target) {
            return;
        }

        //回溯算法标准框架
        for(int i = start; i < nums.size(); i++) {
            //剪枝逻辑，值相同的树枝，只遍历第一条
            if(i > start && nums[i] == nums[i-1]) {
                continue;
            }
            //做选择
            track.push_back(nums[i]);
            trackSum += nums[i];
            //递归遍历下一层回溯树
            backtrack(nums, i+1, target);
            //撤销选择
            track.pop_back();
            trackSum -= nums[i];
        }
    }
};
```

### 排列（元素可重不可复选）

> 题目

给你输入一个可包含重复数字的序列 `nums`，请你写一个算法，返回所有可能的全排列

> 思路

比如输入 `nums = [1,2,2]`，函数返回：

```cpp
[ [1,2,2],[2,1,2],[2,2,1] ]
```

> 代码

```cpp
class Solution {
public:
    // 保存结果
    vector<vector<int>> res;
    // 记录当前位置的元素
    vector<int> track;
    // 记录元素是否被使用
    vector<bool> used;

    // 主函数
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        // 排序，让相同的元素靠在一起
        sort(nums.begin(), nums.end());
        // 初始化used数组
        used = vector<bool>(nums.size(), false);
        // 回溯
        backtrack(nums);
        // 返回结果
        return res;
    }

    // 回溯函数
    void backtrack(vector<int>& nums) {
        // 当长度相等时，将结果记录
        if (track.size() == nums.size()) {
            res.push_back(track);
            return;
        }

        // 遍历没有被使用过的元素
        for (int i = 0; i < nums.size(); i++) {
            if (used[i]) {
                continue;
            }
            // 新添加的剪枝逻辑，固定相同的元素在排列中的相对位置
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            // 添加元素，标记为使用过
            track.push_back(nums[i]);
            used[i] = true;
            // 继续回溯
            backtrack(nums);
            // 回溯
            track.pop_back();
            used[i] = false;
        }
    }
};
```

对比一下之前的标准全排列解法代码，这段解法代码只有两处不同：

1、对 `nums` 进行了排序。

2、添加了一句额外的剪枝逻辑。

类比输入包含重复元素的子集/组合问题，你大概应该理解这么做是为了防止出现重复结果。

但是注意排列问题的剪枝逻辑，和子集/组合问题的剪枝逻辑略有不同：新增了 `!used[i - 1]` 的逻辑判断。

这个地方理解起来就需要一些技巧性了，且听我慢慢到来。为了方便研究，依然把相同的元素用上标 `'` 以示区别。

假设输入为 `nums = [1,2,2']`，标准的全排列算法会得出如下答案：

```cpp
[
    [1,2,2'],[1,2',2],
    [2,1,2'],[2,2',1],
    [2',1,2],[2',2,1]
]
```

显然，这个结果存在重复，比如 `[1,2,2']` 和 `[1,2',2]` 应该只被算作同一个排列，但被算作了两个不同的排列。

所以现在的关键在于，如何设计剪枝逻辑，把这种重复去除掉？

**答案是，保证相同元素在排列中的相对位置保持不变**。

比如说 `nums = [1,2,2']` 这个例子，我保持排列中 `2` 一直在 `2'` 前面。

这样的话，你从上面 6 个排列中只能挑出 3 个排列符合这个条件：



```
[ [1,2,2'],[2,1,2'],[2,2',1] ]
```

这也就是正确答案。

进一步，如果 `nums = [1,2,2',2'']`，我只要保证重复元素 `2` 的相对位置固定，比如说 `2 -> 2' -> 2''`，也可以得到无重复的全排列结果。

仔细思考，应该很容易明白其中的原理：

**标准全排列算法之所以出现重复，是因为把相同元素形成的排列序列视为不同的序列，但实际上它们应该是相同的；而如果固定相同元素形成的序列顺序，当然就避免了重复**。

那么反映到代码上，你注意看这个剪枝逻辑：

```cpp
// 新添加的剪枝逻辑，固定相同的元素在排列中的相对位置
if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
    // 如果前面的相邻相等元素没有用过，则跳过
    continue;
}
// 选择 nums[i]
```

**当出现重复元素时，比如输入 `nums = [1,2,2',2'']`，`2'` 只有在 `2` 已经被使用的情况下才会被选择，同理，`2''` 只有在 `2'` 已经被使用的情况下才会被选择，这就保证了相同元素在排列中的相对位置保证固定**。

这里拓展一下，如果你把上述剪枝逻辑中的 `!used[i - 1]` 改成 `used[i - 1]`，其实也可以通过所有测试用例，但效率会有所下降，这是为什么呢？

之所以这样修改不会产生错误，是因为这种写法相当于维护了 `2'' -> 2' -> 2` 的相对顺序，最终也可以实现去重的效果。

但为什么这样写效率会下降呢？因为这个写法剪掉的树枝不够多。

比如输入 `nums = [2,2',2'']`，产生的回溯树如下：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/12.jpeg)

如果用绿色树枝代表 `backtrack` 函数遍历过的路径，红色树枝代表剪枝逻辑的触发，那么 `!used[i - 1]` 这种剪枝逻辑得到的回溯树长这样：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/13.jpeg)

而 `used[i - 1]` 这种剪枝逻辑得到的回溯树如下：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/14.jpeg)

可以看到，`!used[i - 1]` 这种剪枝逻辑剪得干净利落，而 `used[i - 1]` 这种剪枝逻辑虽然也能得到无重结果，但它剪掉的树枝较少，存在的无效计算较多，所以效率会差一些。

当然，关于排列去重，也有别的剪枝思路：

```cpp
// 回溯函数
// nums: 待选择的数组
// track: 记录当前选择的路径
void backtrack(vector<int>& nums, list<int>& track, vector<vector<int>>& res, vector<bool>& used) {
    // 判断是否达到决策树底部，记录路径
    if (track.size() == nums.size()) {
        res.emplace_back(vector<int>(track.begin(), track.end()));
        return;
    }

    // 记录之前树枝上元素的值
    // 题目说 -10 <= nums[i] <= 10，所以初始化为特殊值
    int prevNum = -666;
    for (int i = 0; i < nums.size(); i++) {
        // 排除不合法的选择
        if (used[i]) {
            continue;
        }
        if (nums[i] == prevNum) {
            continue;
        }

        // 做选择
        track.push_back(nums[i]);
        used[i] = true;
        // 记录这条树枝上的值
        prevNum = nums[i];

        // 进入下一层决策树
        backtrack(nums, track, res, used);

        // 撤销选择
        track.pop_back();
        used[i] = false;
    }
}
```

这个思路也是对的，设想一个节点出现了相同的树枝：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/11.jpeg)

如果不作处理，这些相同树枝下面的子树也会长得一模一样，所以会出现重复的排列。

因为排序之后所有相等的元素都挨在一起，所以只要用 `prevNum` 记录前一条树枝的值，就可以避免遍历值相同的树枝，从而避免产生相同的子树，最终避免出现重复的排列。

好了，这样包含重复输入的排列问题也解决了。

### 子集/组合（元素无重可复选）

> 题目

给你一个无重复元素的整数数组 `candidates` 和一个目标和 `target`，找出 `candidates` 中可以使数字和为目标数 `target` 的所有组合。`candidates` 中的每个数字可以无限制重复被选取。

> 思路

比如输入 `candidates = [1,2,3], target = 3`，算法应该返回：

```cpp
[ [1,1,1],[1,2],[3] ]
```

这道题说是组合问题，实际上也是子集问题：`candidates` 的哪些子集的和为 `target`？

想解决这种类型的问题，也得回到回溯树上，**我们不妨先思考思考，标准的子集/组合问题是如何保证不重复使用元素的**？

答案在于 `backtrack` 递归时输入的参数 `start`：

这个 `i` 从 `start` 开始，那么下一层回溯树就是从 `start + 1` 开始，从而保证 `nums[start]` 这个元素不会被重复使用：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/1.jpeg)

那么反过来，如果我想让每个元素被重复使用，我只要把 `i + 1` 改成 `i` 即可：

```cpp
// 可重组合的回溯算法框架
void backtrack(vector<int>& nums, int start) {
    for (int i = start; i < nums.size(); i++) {
        // ...
        // 递归遍历下一层回溯树，注意参数
        backtrack(nums, i);
        // ...
    }
}
```

这相当于给之前的回溯树添加了一条树枝，在遍历这棵树的过程中，一个元素可以被无限次使用：

![img](https://labuladong.online/algo/images/%E6%8E%92%E5%88%97%E7%BB%84%E5%90%88/10.jpeg)

当然，这样这棵回溯树会永远生长下去，所以我们的递归函数需要设置合适的 base case 以结束算法，即路径和大于 `target` 时就没必要再遍历下去了。

> 代码

```cpp
class Solution {
public:
    vector<vector<int>> res;
    // 记录回溯的路径
    deque<int> track;
    // 记录 track 中的路径和
    int trackSum = 0;

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        backtrack(candidates, 0, target);
        return res;
    }

    // 回溯算法主函数
    void backtrack(vector<int>& nums, int start, int target) {
        // base case，找到目标和，记录结果
        if (trackSum == target) {
            res.push_back(vector<int>(track.begin(), track.end()));
            return;
        }
        // base case，超过目标和，停止向下遍历
        if (trackSum > target) {
            return;
        }
        // 回溯算法标准框架
        for (int i = start; i < nums.size(); i++) {
            // 选择 nums[i]
            trackSum += nums[i];
            track.push_back(nums[i]);
            // 递归遍历下一层回溯树
            // 同一元素可重复使用，注意参数
            backtrack(nums, i, target);
            // 撤销选择 nums[i]
            trackSum -= nums[i];
            track.pop_back();
        }
    }
};
```

### 排列（元素无重可复选）

> 题目

`nums` 数组中的元素无重复且可复选的情况下，会有哪些排列？

比如输入 `nums = [1,2,3]`，那么这种条件下的全排列共有 3^3 = 27 种：

```cpp
[
  [1,1,1],[1,1,2],[1,1,3],[1,2,1],[1,2,2],[1,2,3],[1,3,1],[1,3,2],[1,3,3],
  [2,1,1],[2,1,2],[2,1,3],[2,2,1],[2,2,2],[2,2,3],[2,3,1],[2,3,2],[2,3,3],
  [3,1,1],[3,1,2],[3,1,3],[3,2,1],[3,2,2],[3,2,3],[3,3,1],[3,3,2],[3,3,3]
]
```

**标准的全排列算法利用 `used` 数组进行剪枝，避免重复使用同一个元素。如果允许重复使用元素的话，直接放飞自我，去除所有 `used` 数组的剪枝逻辑就行了**。

```cpp
class Solution {
public:
    vector<vector<int>> res;
    deque<int> track;

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        backtrack(nums);
        return res;
    }

    void backtrack(vector<int>& nums) {
        if (track.size() == nums.size()) {
            res.push_back(vector(track.begin(), track.end()));
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            //剪枝操作，判断当前节点是否已经在track中
            if (i > 0 && nums[i] == nums[i - 1] && find(track.begin(), track.end(), nums[i - 1]) != track.end()) {
                continue;
            }
            track.push_back(nums[i]);
            backtrack(nums);
            track.pop_back();
        }
    }
};
```

## 球盒模型：回溯算法穷举的两种视角

1. 回溯算法穷举的本质思维模式是「球盒模型」，一切回溯算法，皆从此出，别无二法。
2. 球盒模型，必然有两种穷举视角，分别为「球」的视角穷举和「盒」的视角穷举，对应的，就是两种不同的代码写法。
3. 从理论上分析，两种穷举视角本质上是一样的。但是涉及到具体的代码实现，两种写法的复杂度可能有优劣之分。你需要选择效率更高的写法。

球盒模型这个词是我随口编的，因为下面我会用「球」和「盒」两种视角来解释，你理解就好。

### 暴力穷举思维方法：球盒模型

**一切暴力穷举算法，都从球盒模型开始，没有例外**。

你懂了这个，就可以随心所欲运用暴力穷举算法，下面的内容，请你仔细看，认真想。

首先，我们回顾一下以前学过的排列组合知识：

1、`P(n, k)`（也有很多书写成 `A(n, k)`）表示从 `n` 个不同元素中拿出 `k` 个元素的排列（Permutation/Arrangement）总数；`C(n, k)` 表示从 `n` 个不同元素中拿出 `k` 个元素的组合（Combination）总数。

2、「排列」和「组合」的主要区别在于是否考虑顺序的差异。

3、排列、组合总数的计算公式：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/math.png)

### 排列P(n, k)

好，现在我问一个问题，这个排列公式 `P(n, k)` 是如何推导出来的？为了搞清楚这个问题，我需要讲一点组合数学的知识。

排列组合问题的各种变体都可以抽象成「球盒模型」，`P(n, k)` 就可以抽象成下面这个场景：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/7.jpeg)

即，将 `n` 个标记了不同序号的球（标号为了体现顺序的差异），放入 `k` 个标记了不同序号的盒子中（其中 `n >= k`，每个盒子最终都装有恰好一个球），共有 `P(n, k)` 种不同的方法。

现在你来，往盒子里放球，你怎么放？其实有两种视角。

**首先，你可以站在盒子的视角**，每个盒子必然要选择一个球。

这样，第一个盒子可以选择 `n` 个球中的任意一个，然后你需要让剩下 `k - 1` 个盒子在 `n - 1` 个球中选择（这就是子问题 `P(n - 1, k - 1)`）：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/8.jpeg)

**另外，你也可以站在球的视角**，因为并不是每个球都会被装进盒子，所以球的视角分两种情况：

1、第一个球可以不装进任何一个盒子，这样的话你就需要将剩下 `n - 1` 个球放入 `k` 个盒子。

2、第一个球可以装进 `k` 个盒子中的任意一个，这样的话你就需要将剩下 `n - 1` 个球放入 `k - 1` 个盒子。

结合上述两种情况，可以得到：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/9.jpeg)

你看，两种视角得到两个不同的递归式，但这两个递归式解开的结果都是我们熟知的阶乘形式：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/math1.png)

至于如何解递归式，涉及数学的内容比较多，这里就不做深入探讨了。

### 组合C(n, k)

组合 `C(n, k)` 可以抽象成下面这个场景：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/c_n_k.jpeg)

因为在组合问题中，我们不在乎元素的顺序，即 `{1, 2, 3}` 和 `{1, 3, 2}` 视为同一种组合。**所以我们不需要对盒进行编号，可以认为只有一个盒，其容量是 `k`**。

**首先，你可以站在盒子的视角**，盒子必然要装 `k` 个球。

那么第一个球可以选择 `n` 个球中的任意一个，然后盒子剩余 `k - 1` 个位置，你需要在剩下的 `n - 1` 个球中选择（这就是子问题 `C(n - 1, k - 1)`）。

想到这里，是不是就想列出关系式 `C(n, k) = nC(n - 1, k - 1)` 了？

不对，这里有个坑，你直接 `nC(n - 1, k - 1)` 是有重复的，正确的答案应该是 `nC(n - 1, k - 1) / k`：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/c_box_view.jpeg)

举例说明

>举例来说就容易看出来了，比方说让你在 `[1,2,3,4,5]` 中选择 3 个元素，有多少种不同的组合？
>
>你站在盒的视角，盒子容量为 3，开始穷举，第一个位置，你可以选择球 `1,2,3,4,5` 中的任意一个，比如说选了球 `1`。
>
>接下来你需要在剩下的元素 `2,3,4,5` 中选择两个元素，比方说最终你穷举出的组合是 `{1,3,4}`。
>
>**要想知道是否有重复，重复了几次，你就盯紧这个 `{1, 3, 4}`，它如果有重复，其他组合结果都会重复**。
>
>容易发现，如果我第一次选择的是球 `3`，然后我可能得到 `{3, 1, 4}`；如果我第一次选择的是球 `4`，那么我可能得到 `{4, 1, 3}`；这两种情况都是和 `{1, 3, 4}` 重复的，**因此 `{1, 3, 4}` 共出现了三次**。
>
>有的读者会问，只有这两种情况是重复的吗？类似 `{3, 4, 1}` 这种情况不也是重复的吗？
>
>不是，因为我们组合不考虑顺序，`{3, 4, 1}` 和 `{3, 1, 4}` 都是球 `3` 开头，所以是等价的。
>
>综上，你不能直接用 `nC(n - 1, k - 1)`，因为其中每种组合都出现了 `k` 次，所以要再除以 `k` 消除重复。

**另外，你也可以站在球的视角**，因为并不是每个球都会被装进盒子，所以球的视角分两种情况：

1、第一个球可以不装进盒子，这样的话你就需要将剩下 `n - 1` 个球放入 `k` 个盒子。

2、第一个球可以装进盒子，这样的话你就需要将剩下 `n - 1` 个球放入 `k - 1` 个盒子。

结合上述两种情况，可以得到：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/c_ball_view.jpeg)

注意组合和前面排列的区别，因为组合不考虑顺序，所以球选择装进盒子时，只有一种选择，而不是 `k` 种不同选择，所以 `C(n-1, k-1)` 不用乘 `k`。

你看，两种视角得到两个不同的递归式，但这两个递归式解开的结果都是我们熟知的组合数公式：

![img](https://labuladong.online/algo/images/%E9%9B%86%E5%90%88%E5%88%92%E5%88%86/math2.jpg)

### 用球盒模型重新理解全排列问题

就以最基本的元素无重不可复选的全排列为例：

```cpp
class Solution {
public:
    vector<vector<int>> res;
    // 记录回溯算法的递归路径
    vector<int> track;
    // track 中的元素会被标记为 true
    vector<bool> used;

    /* 主函数，输入一组不重复的数字，返回它们的全排列 */
    vector<vector<int>> permute(vector<int>& nums) {
        used = vector<bool>(nums.size(), false);
        backtrack(nums);
        return res;
    }

    // 回溯算法核心函数
    void backtrack(vector<int>& nums) {
        // base case，到达叶子节点
        if (track.size() == nums.size()) {
            // 收集叶子节点上的值
            res.push_back(track);
            return;
        }

        // 回溯算法标准框架
        for (int i = 0; i < nums.size(); i++) {
            // 已经存在 track 中的元素，不能重复选择
            if (used[i]) {
                continue;
            }
            // 做选择
            used[i] = true;
            track.push_back(nums[i]);
            // 进入下一层回溯树
            backtrack(nums);
            // 取消选择
            track.pop_back();
            used[i] = false;
        }
    }
};
// 这里回溯树中，横向为选择n个不同的球，纵向为不同的盒子选择球
```

这个代码是以盒的视角进行穷举的，即**站在每个位置的角度来选择球**，站在 `nums` 中的每个索引，来选择不同的元素放入这个索引位置。

为什么是这个答案呢？假设 `nums` 里面有 `n` 个数字，那么全排列问题相当于把 `n` 个球放到包含 `n` 个位置的盒子里，要求盒子必须装满，问你有几种不同的装法。

以盒的视角理解，盒子的第一个位置可以接收 `n` 个球的任意一个，有 `n` 种选择，第二个位置可以接收 `n - 1` 个球的任意一个，有 `n - 1` 种选择，第三个位置有 `n - 2` 种选择，以此类推。

> 还是盒的视角，用swap实现

```cpp
class Solution {
private:
    vector<vector<int>> result;

public:
    vector<vector<int>> permute(vector<int>& nums) {
        backtrack(nums, 0);
        return result;
    }

    // 回溯算法核心框架
    void backtrack(vector<int>& nums, int start) {
        if (start == nums.size()) {
            // 找到一个全排列，Java 需要转化成 List 类型
            result.push_back(nums);
            return;
        }

        for (int i = start; i < nums.size(); i++) {
            // 做选择
            swap(nums, start, i);
            // 递归调用，传入 start + 1
            backtrack(nums, start + 1);
            // 撤销选择
            swap(nums, start, i);
        }
    }

    void swap(vector<int>& nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
};

// 回溯树中：横向为该盒子选不同的球（交换到该位置上），纵向为不同的盒子
```

这个解法是以盒的视角进行穷举的。即 `nums` 数组中的每个索引位置，来选择不同的元素放入这个索引位置。

你看解法代码也可以看出来，那个 `start` 参数就是当前在选择元素的索引位置，在 `start` 之前的元素已经心有所属，被其他位置挑走了，所以 `start` 位置只能从 `nums[start..]` 中选择元素。

> 怎么以球的视角实现

以球的视角来写全排列的解法代码，就是说 `nums` 中的每个元素来选择自己想去的索引。

```cpp
class Solution {
public:
    // 结果列表
    vector<vector<int>> res;
    // 标记元素是否已被使用
    vector<bool> used;
    // 记录有多少个元素已经选择过位置
    int count;

    vector<vector<int>> permute(vector<int>& nums) {
        count = 0;
        used = vector<bool>(nums.size(), false);

        backtrack(nums);
        return res;
    }

    // 回溯算法框架
    void backtrack(vector<int>& nums) {
        if (count == nums.size()) {
            res.push_back(nums);
            return;
        }

        // 找两个未被选择的位置
        int originalIndex = -1, swapIndex = -1;

        for (int i = 0; i < nums.size(); i++) {
            if (used[i]) {
                continue;
            }
            if (originalIndex == -1) {
                originalIndex = i;
            }
            swapIndex = i;

            // 做选择，元素 nums[originalIndex] 选择 swapIndex 位置
            swap(nums, originalIndex, swapIndex);
            used[swapIndex] = true;
            count++;
            // 进入下一层决策树
            backtrack(nums);
            // 撤销选择，刚才怎么做的选择，就原样恢复
            count--;
            used[swapIndex] = false;
            swap(nums, originalIndex, swapIndex);
        }
    }

    void swap(vector<int>& nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
};
```

### 用球盒模型重新理解子集问题

以最基本的元素无重不可复选的子集为例

```cpp
class Solution {
private:

vector<vector<int>> res; 
vector<int> track; 

public:

vector<vector<int>> subsets(vector<int>& nums) {
    backtrack(nums, 0);
    return res;
}

// 回溯算法核心函数，遍历子集问题的回溯树
void backtrack(vector<int>& nums, int start) {

    // 前序位置，每个节点的值都是一个子集
    res.push_back(track);
    
    // 回溯算法标准框架
    for (int i = start; i < nums.size(); i++) {
        // 做选择
        track.push_back(nums[i]);
        // 通过 start 参数控制树枝的遍历，避免产生重复的子集
        backtrack(nums, i + 1);
        // 撤销选择
        track.pop_back();
    }
}
};

// 回溯树中，横向为选 n 个不同的球，纵向为不同的盒子在选球
```

这个解法是以盒的视角穷举的，即**站在 `nums` 中的每个索引的视角，来选择不同的元素放入这个索引位置。**

**因为刚才讲的全排列问题会考虑顺序的差异，**而子集问题不考虑顺序的差异。为了方便理解，我们这里干脆不说「球盒模型」了，说「球桶模型」吧，因为放进盒子的求给人感觉是有顺序的，而丢进桶里的东西给人感觉是无所谓顺序的。

那么，以桶的视角理解，子集问题相当于把 `n` 个球丢到容量为 `n` 的桶里，桶可以不装满。

这样，桶的第一个位置可以选择 `n` 个球中的任意一个，比如选择了球 `i`，然后桶的第二个位置可以选择球 `i` 后面的球中的任意一个（通过固定相对顺序保证不会出现重复的子集），以此类推。

> 以球的视角来解决子集问题

从球的视角理解，每个球都有两种选择，要么在桶中，要么不在桶中。这样，我们可以写出下面的代码：

```cpp
class Solution {
public:
    // 用于存储所有子集的结果
    vector<vector<int>> res;
    // 用于存储当前递归路径的子集
    vector<int> track;

    vector<vector<int>> subsets(vector<int>& nums) {
        backtrack(nums, 0);
        return res;
    }

    void backtrack(vector<int>& nums, int i) {
        if (i == nums.size()) {
            res.push_back(track);
            return;
        }

        // 做第一种选择，元素在子集中
        track.push_back(nums[i]);
        backtrack(nums, i + 1);
        // 撤销选择
        track.pop_back();

        // 做第二种选择，元素不在子集中
        backtrack(nums, i + 1);
    }
};
```

这也解释了，为什么所有子集（幂集）的数量是 `2^n`，因为每个元素都有两种选择，要么在子集中，要么不在子集中，所以其递归树就是一棵满二叉树，一共有 `2^n` 个叶子节点。

### 结论

1. **回溯算法穷举的本质思维模式是「球盒模型」，一切回溯算法，皆从此出，别无二法**。
2. **球盒模型，必然有两种穷举视角，分别为「球」的视角穷举和「盒」的视角穷举，对应的，就是两种不同的代码写法**。
3. **从理论上分析，两种穷举视角本质上是一样的。但是涉及到具体的代码实现，两种写法的复杂度可能有优劣之分**。

> 进一步想想，为啥用「盒」的视角，即让索引取选元素的视角，可以用 `swap` 的方法把 `used` 数组给优化掉呢？

因为索引容易处理，如果你按顺序从小到大让每个索引去选元素，那么一个 `start` 变量作为分割线就能把已选元素和未选元素分开。

反过来，如果你让元素去选索引，那就只能依赖额外的数据结构来记录那些索引已经被选过了，这样就会增加额外的空间复杂度。

## BFS算法框架

BFS 的核心思想应该不难理解的，就是把一些问题抽象成图，从一个点开始，向四周开始扩散。一般来说，我们写 BFS 算法都是用「队列」这种数据结构，每次将一个节点周围的所有节点加入队列。

BFS 相对 DFS 的最主要的区别是：**BFS 找到的路径一定是最短的，但代价就是空间复杂度可能比 DFS 大很多**。

### 算法框架

```cpp
int BFS(Node start, Node target) {
    queue<Node> q;
    set<Node> visited;
    
    q.push(start);
    visited.insert(start);
    
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            Node cur = q.front();
            q.pop();
            if (cur == target) {
                return step;
            }
            for (Node x : cur.adj()) {
                if (visited.count(x) == 0) {
                    q.push(x);
                    visited.insert(x);
                }
            }
        }
    }
    // 走到这里 说明图中没找到目标节点
}
```

`cur.adj()` 泛指 `cur` 相邻的节点，比如说二维数组中，`cur` 上下左右四面的位置就是相邻节点；`visited` 的主要作用是防止走回头路，大部分时候都是必须的，但是像一般的二叉树结构，没有子节点到父节点的指针，不会走回头路就不需要 `visited`。

### 二叉树的最小高度

> 题目

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明：**叶子节点是指没有子节点的节点。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2020/10/12/ex_depth.jpg)

```
输入：root = [3,9,20,null,null,15,7]
输出：2
```

**示例 2：**

```
输入：root = [2,null,3,null,4,null,5,null,6]
输出：5
```

**提示：**

- 树中节点数的范围在 `[0, 105]` 内
- `-1000 <= Node.val <= 1000`

> 思路

怎么套到 BFS 的框架里呢？首先明确一下起点 `start` 和终点 `target` 是什么，怎么判断到达了终点？

**显然起点就是 `root` 根节点，终点就是最靠近根节点的那个「叶子节点」嘛**，叶子节点就是两个子节点都是 `null` 的节点：

> 代码

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        queue<TreeNode*> q;
        q.push(root);
        // root 本身就是一层，depth 初始化为 1
        int depth = 1;
        
        while (!q.empty()) {


            int sz = q.size();
            /* 将当前队列中的所有节点向四周扩散 */
            for (int i = 0; i < sz; i++) {
                TreeNode* cur = q.front();
                q.pop();
                /* 判断是否到达终点 */
                if (cur->left == nullptr && cur->right == nullptr) 
                    return depth;
                /* 将 cur 的相邻节点加入队列 */
                if (cur->left != nullptr)
                    q.push(cur->left);
                if (cur->right != nullptr) 
                    q.push(cur->right);
            }
            /* 这里增加步数 */
            depth++;
        }
        return depth;
    }
};
```

这里注意这个 `while` 循环和 `for` 循环的配合，**`while` 循环控制一层一层往下走，`for` 循环利用 `sz` 变量控制从左到右遍历每一层二叉树节点**：

![img](https://labuladong.online/algo/images/dijkstra/1.jpeg)

**1、为什么 BFS 可以找到最短距离，DFS 不行吗**？

首先，你看 BFS 的逻辑，`depth` 每增加一次，队列中的所有节点都向前迈一步，这保证了第一次到达终点的时候，走的步数是最少的。

DFS 不能找最短路径吗？其实也是可以的，但是时间复杂度相对高很多。你想啊，DFS 实际上是靠递归的堆栈记录走过的路径，你要找到最短路径，肯定得把二叉树中所有树杈都探索完才能对比出最短的路径有多长对不对？而 BFS 借助队列做到一次一步「齐头并进」，是可以在不遍历完整棵树的条件下找到最短距离的。

形象点说，DFS 是线，BFS 是面；DFS 是单打独斗，BFS 是集体行动。这个应该比较容易理解吧。

**2、既然 BFS 那么好，为啥 DFS 还要存在**？

BFS 可以找到最短距离，但是空间复杂度高，而 DFS 的空间复杂度较低。

还是拿刚才我们处理二叉树问题的例子，假设给你的这个二叉树是满二叉树，节点数为 `N`，对于 DFS 算法来说，空间复杂度无非就是递归堆栈，最坏情况下顶多就是树的高度，也就是 `O(logN)`。

但是你想想 BFS 算法，队列中每次都会储存着二叉树一层的节点，这样的话最坏情况下空间复杂度应该是树的最底层节点的数量，也就是 `N/2`，用 Big O 表示的话也就是 `O(N)`。

由此观之，BFS 还是有代价的，一般来说在找最短路径的时候使用 BFS，其他时候还是 DFS 使用得多一些（主要是递归代码好写）。

### 解开密码锁的最少次数

> 题目

你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： `'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'` 。每个拨轮可以自由旋转：例如把 `'9'` 变为 `'0'`，`'0'` 变为 `'9'` 。每次旋转都只能旋转一个拨轮的一位数字。

锁的初始数字为 `'0000'` ，一个代表四个拨轮的数字的字符串。

列表 `deadends` 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。

字符串 `target` 代表可以解锁的数字，你需要给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 `-1` 。

**示例 1:**

```
输入：deadends = ["0201","0101","0102","1212","2002"], target = "0202"
输出：6
解释：
可能的移动序列为 "0000" -> "1000" -> "1100" -> "1200" -> "1201" -> "1202" -> "0202"。
注意 "0000" -> "0001" -> "0002" -> "0102" -> "0202" 这样的序列是不能解锁的，
因为当拨动到 "0102" 时这个锁就会被锁定。
```

**示例 2:**

```
输入: deadends = ["8888"], target = "0009"
输出：1
解释：把最后一位反向旋转一次即可 "0000" -> "0009"。
```

**示例 3:**

```
输入: deadends = ["8887","8889","8878","8898","8788","8988","7888","9888"], target = "8888"
输出：-1
解释：无法旋转到目标数字且不被锁定。
```

**提示：**

- `1 <= deadends.length <= 500`
- `deadends[i].length == 4`
- `target.length == 4`
- `target` **不在** `deadends` 之中
- `target` 和 `deadends[i]` 仅由若干位数字组成

> 思路

题目中描述的就是我们生活中常见的那种密码锁，如果没有任何约束，最少的拨动次数很好算，就像我们平时开密码锁那样直奔密码拨就行了。

但现在的难点就在于，不能出现 `deadends`，应该如何计算出最少的转动次数呢？

**第一步，我们不管所有的限制条件，不管 `deadends` 和 `target` 的限制，就思考一个问题：如果让你设计一个算法，穷举所有可能的密码组合，你怎么做**？

穷举呗，再简单一点，如果你只转一下锁，有几种可能？总共有 4 个位置，每个位置可以向上转，也可以向下转，也就是有 8 种可能对吧。

比如说从 `"0000"` 开始，转一次，可以穷举出 `"1000", "9000", "0100", "0900"...` 共 8 种密码。然后，再以这 8 种密码作为基础，对每个密码再转一下，穷举出所有可能...

**仔细想想，这就可以抽象成一幅图，每个节点有 8 个相邻的节点**，又让你求最短距离，这不就是典型的 BFS 嘛。

> 代码

```cpp
class Solution {
public:
    // 将 s[j] 向上拨动一次
    string plusOne(string s, int j) {
        char ch[s.length()];
        strcpy(ch, s.c_str());
        if (ch[j] == '9')
            ch[j] = '0';
        else
            ch[j] += 1;
        string res(ch);
        return res;
    }

    // 将 s[i] 向下拨动一次
    string minusOne(string s, int j) {
        char ch[s.length()];
        strcpy(ch, s.c_str());
        if (ch[j] == '0')
            ch[j] = '9';
        else
            ch[j] -= 1;
        string res(ch);
        return res;
    }
    int openLock(vector<string>& deadends, string target) {
        // 记录需要跳过的死亡密码
        unordered_set<string> deads(deadends.begin(), deadends.end());
        // 记录已经穷举过的密码，防止走回头路
        unordered_set<string> visited;
        queue<string> q;
        // 从起点开始启动广度优先搜索
        int step = 0;
        q.push("0000");
        visited.insert("0000");
        
        while (!q.empty()) {
            int sz = q.size();
            /* 将当前队列中的所有节点向周围扩散 */
            for (int i = 0; i < sz; i++) {
                string cur = q.front(); q.pop();
                
                /* 判断是否到达终点 */
                if (deads.count(cur))
                    continue;
                if (cur == target)
                    return step;
                
                /* 将一个节点的未遍历相邻节点加入队列 */
                for (int j = 0; j < 4; j++) {
                    string up = plusOne(cur, j);
                    if (!visited.count(up)) {
                        q.push(up);
                        visited.insert(up);
                    }
                    string down = minusOne(cur, j);
                    if (!visited.count(down)) {
                        q.push(down);
                        visited.insert(down);
                    }
                }
            }
            /* 在这里增加步数 */
            step++;
        }
         // 如果穷举完都没找到目标密码，那就是找不到了
        return -1;
    }
};
```

### 双向BFS优化

**传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止**。

为什么这样能够能够提升效率呢？其实从 Big O 表示法分析算法复杂度的话，它俩的最坏复杂度都是 `O(N)`，但是实际上双向 BFS 确实会快一些，我给你画两张图看一眼就明白了：

![img](https://labuladong.online/algo/images/BFS/1.jpeg)

![img](https://labuladong.online/algo/images/BFS/2.jpeg)

图示中的树形结构，如果终点在最底部，按照传统 BFS 算法的策略，会把整棵树的节点都搜索一遍，最后找到 `target`；而双向 BFS 其实只遍历了半棵树就出现了交集，也就是找到了最短距离。从这个例子可以直观地感受到，双向 BFS 是要比传统 BFS 高效的。

**不过，双向 BFS 也有局限，因为你必须知道终点在哪里**。比如我们刚才讨论的二叉树最小高度的问题，你一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是第二个密码锁的问题，是可以使用双向 BFS 算法来提高效率的，代码稍加修改即可：

```cpp
class Solution {
public:
    // 此处unordered_set对应Java中的HashSet
    int openLock(vector<string>& deadends, string target) {
        unordered_set<string> deads(deadends.begin(), deadends.end());
        // 用集合不用队列，可以快速判断元素是否存在
        unordered_set<string> q1, q2, visited;
        
        int step = 0;
        q1.insert("0000");
        q2.insert(target);
        
        while (!q1.empty() && !q2.empty()) {


            // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
            unordered_set<string> temp;

            /* 将 q1 中的所有节点向周围扩散 */
            for (const string& cur : q1) {
                /* 判断是否到达终点 */
                if (deads.count(cur))
                    continue;
                if (q2.count(cur))
                    return step;
                
                visited.insert(cur);

                /* 将一个节点的未遍历相邻节点加入集合 */
                for (int j = 0; j < 4; j++) {
                    string up = plusOne(cur, j);
                    if (!visited.count(up))
                        temp.insert(up);
                    string down = minusOne(cur, j);
                    if (!visited.count(down))
                        temp.insert(down);
                }
            }
            /* 在这里增加步数 */
            step++;
            // temp 相当于 q1
            // 这里交换 q1 q2，下一轮 while 就是扩散 q2
            q1 = q2;
            q2 = temp;
        }
        return -1;
    }

private:
    string plusOne(string s, int j) {
        if (s[j] == '9')
            s[j] = '0';
        else
            s[j] += 1;
        return s;
    }

    string minusOne(string s, int j) {
        if (s[j] == '0')
            s[j] = '9';
        else
            s[j] -= 1;
        return s;
    }
};
```

双向 BFS 还是遵循 BFS 算法框架的，只是**不再使用队列，而是使用 HashSet 方便快速判断两个集合是否有交集**。

另外的一个技巧点就是 **while 循环的最后交换 `q1` 和 `q2` 的内容**，所以只要默认扩散 `q1` 就相当于轮流扩散 `q1` 和 `q2`。

其实双向 BFS 还有一个优化，就是在 while 循环开始时做一个判断：

```cpp
// ...
while (!q1.empty() && !q2.empty()) {
    if (q1.size() > q2.size()) {
        // 交换 q1 和 q2
        queue<int> temp = q1;
        q1 = q2;
        q2 = temp;
    }
    // ...
```

为什么这是一个优化呢？

因为按照 BFS 的逻辑，队列（集合）中的元素越多，扩散之后新的队列（集合）中的元素就越多；在双向 BFS 算法中，如果我们每次都选择一个较小的集合进行扩散，那么占用的空间增长速度就会慢一些，效率就会高一些。

不过话说回来，**无论传统 BFS 还是双向 BFS，无论做不做优化，从 Big O 衡量标准来看，时间复杂度都是一样的**，只能说双向 BFS 是一种 trick，算法运行的速度会相对快一点，掌握不掌握其实都无所谓。最关键的是把 BFS 通用框架记下来，反正所有 BFS 算法都可以用它套出解法。
