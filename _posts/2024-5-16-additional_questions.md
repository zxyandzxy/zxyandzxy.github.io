---
layout: post
title: "额外题目"
date: 2024-5-16
tags: [algorithm]
comments: true
author: zxy
---

### 1365.有多少小于当前数字的数字

```
给你一个数组 nums，对于其中每个元素 nums[i]，请你统计数组中比它小的所有数字的数目。
换而言之，对于每个 nums[i] 你必须计算出有效的 j 的数量，其中 j 满足 j != i 且 nums[j] < nums[i] 。
以数组形式返回答案。

示例 1：
输入：nums = [8,1,2,2,3]
输出：[4,0,1,1,3]
解释： 对于 nums[0]=8 存在四个比它小的数字：（1，2，2 和 3）。
对于 nums[1]=1 不存在比它小的数字。
对于 nums[2]=2 存在一个比它小的数字：（1）。
对于 nums[3]=2 存在一个比它小的数字：（1）。
对于 nums[4]=3 存在三个比它小的数字：（1，2 和 2）。

示例 2：
输入：nums = [6,5,4,8]
输出：[2,1,0,3]

示例 3：
输入：nums = [7,7,7,7]
输出：[0,0,0,0]

提示：
2 <= nums.length <= 500
0 <= nums[i] <= 100
```

```
暴力解法：
直接双重for循环，对于每个元素，用内层for循环统计有多少个小于它的数字
时间复杂度O(n^2)  空间复杂度O(1)

哈希空间换时间：
对于一个排好序的数组,小于它的元素个数就是它的索引
所以用一个排好序的数组，用哈希表记录每个元素的索引
那么对于原数组，就可以直接通过值去获取小于它的元素个数
注意：对于元素值相等的情况，我们要取最小的索引（题目要求小于，不是小于等于）
```

```cpp
class Solution {
public:
    vector<int> smallerNumbersThanCurrent(vector<int>& nums) {
        vector<int> vec = nums;
        unordered_map<int, int> hash;
        sort(vec.begin(), vec.end());
        for (int i = nums.size() - 1; i >= 0; i--) {
            hash[vec[i]] = i;
        }
        vector<int> ans(nums.size());
        for (int i = 0; i < nums.size(); i++) {
            ans[i] = hash[nums[i]];
        }
        return ans;
    }
};
```

### 941.有效的山脉数组

```
给定一个整数数组 arr，如果它是有效的山脉数组就返回 true，否则返回 false。
让我们回顾一下，如果 A 满足下述条件，那么它是一个山脉数组：
	arr.length >= 3
	在 0 < i < arr.length - 1 条件下，存在 i 使得：
		arr[0] < arr[1] < ... arr[i-1] < arr[i]
		arr[i] > arr[i+1] > ... > arr[arr.length - 1]

示例 1：
输入：arr = [2,1]
输出：false

示例 2：
输入：arr = [3,5,5]
输出：false

示例 3：
输入：arr = [0,3,2,1]
输出：true
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210729103604.png)

```
思路：
由于山脉数组一定是先严格递增，然后严格递减
所以我们从索引0出发，找到递增的最后一个元素；从数组末尾出发，找到递增的最后一个元素
如果这两个元素索引相同，那么就是山脉数组。
注意特殊情况：如果正向直接递增到数组末尾，反向直接递增到索引0，那么也不是山脉数组
```

```cpp
class Solution {
public:
    bool validMountainArray(vector<int>& arr) {
        if (arr.size() < 3) {
            return false;
        }
        int rise = 1, down = arr.size() - 2;
        while (rise < arr.size() && arr[rise] > arr[rise - 1]) {
            rise++;
        }
        if (rise == arr.size()) {return false;}
        rise--;
        while (down >= 0 && arr[down] > arr[down + 1]) {
            down--;
        }
        if (down == -1) {return false;}
        down++;
        return rise == down;
    }
};
```

### 1207.独一无二的出现次数

```
给你一个整数数组 arr，请你帮忙统计数组中每个数的出现次数。
如果每个数的出现次数都是独一无二的，就返回 true；否则返回 false。

示例 1：
输入：arr = [1,2,2,1,1,3]
输出：true
解释：在该数组中，1 出现了 3 次，2 出现了 2 次，3 只出现了 1 次。没有两个数的出现次数相同。

示例 2：
输入：arr = [1,2]
输出：false

示例 3：
输入：arr = [-3,0,1,-3,1,1,1,-3,10,0]
输出：true

提示：
1 <= arr.length <= 1000
-1000 <= arr[i] <= 1000
```

```
用unordered_map记录每一个元素出现的次数
用unordered_set记录次数，如果次数出现重复，返回FALSE
```

```cpp
class Solution {
public:
    bool uniqueOccurrences(vector<int>& arr) {
        unordered_map<int, int> hash;
        for (auto x : arr) {
            hash[x]++;
        }
        unordered_set<int> cnt;
        for (auto x : hash) {
            if (cnt.count(x.second)) {
                return false;
            }
            cnt.insert(x.second);
        }
        return true;
    }
};
```

### 283. 移动零

```
给定一个数组 nums，编写一个函数将所有 0 移动到数组的末尾，同时保持非零元素的相对顺序。

示例:
输入: [0,1,0,3,12] 输出: [1,3,12,0,0] 说明:
必须在原数组上操作，不能拷贝额外的数组。 尽量减少操作次数。
```

```
双指针：left指向数组左侧的0，right指针数组右侧的第一个非0元素
每次遇到上面的情况时，交换left和right指向的元素
```

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int left = 0, right = 0;
        while (right < nums.size()) {
            while (left < nums.size() && nums[left] != 0) {
                left++;
            }
            right = left + 1;
            while (right < nums.size() && nums[right] == 0) {
                right++;
            }
            if (right < nums.size()) {
                swap(nums[left], nums[right]);
            }
        }
        return;
    }
};
```

### 189. 旋转数组

```
给定一个数组，将数组中的元素向右移动 k 个位置，其中 k 是非负数。

进阶：
尽可能想出更多的解决方案，至少有三种不同的方法可以解决这个问题。 你可以使用空间复杂度为 O(1) 的 原地 算法解决这个问题吗？

示例 1:
输入: nums = [1,2,3,4,5,6,7], k = 3
输出: [5,6,7,1,2,3,4]
解释: 向右旋转 1 步: [7,1,2,3,4,5,6]。 向右旋转 2 步: [6,7,1,2,3,4,5]。 向右旋转 3 步: [5,6,7,1,2,3,4]。

示例 2:
输入：nums = [-1,-100,3,99], k = 2
输出：[3,99,-1,-100]
解释: 向右旋转 1 步: [99,-1,-100,3]。 向右旋转 2 步: [3,99,-1,-100]。
```

```
方法1：时间复杂度O(n)，空间复杂度O(n)
我们开一个新数组temp，这个数组记录旋转后的结果。
从索引k开始，将temp[k++] = nums[i++] 直到temp到达数组末尾
在从0开始，将temp[0++] = nums[i++]
最后把nums改成temp就行了

方法2：时间复杂度O(2*n)，空间复杂度O(1)
旋转整个数组   +  旋转数组[0 - K - 1]  + 旋转数组[K - n - 1]
```

```cpp
class Solution {
public:
    void reverse(vector<int>& nums,int i,int j){
        while(i<j){
        int temp=nums[i];
        nums[i]=nums[j];
        nums[j]=temp;
        i++;j--;
        }
    }
    void rotate(vector<int>& nums, int k) {
        k=k%nums.size();
        reverse(nums,0,nums.size()-1);
        reverse(nums,0,k-1);
        reverse(nums,k,nums.size()-1);
    }
};
```

### 724.寻找数组的中心下标

```
给你一个整数数组 nums ，请计算数组的 中心下标 。
数组 中心下标 是数组的一个下标，其左侧所有元素相加的和等于右侧所有元素相加的和。
如果中心下标位于数组最左端，那么左侧数之和视为 0 ，因为在下标的左侧不存在元素。这一点对于中心下标位于数组最右端同样适用。
如果数组有多个中心下标，应该返回 最靠近左边 的那一个。如果数组不存在中心下标，返回 -1 。

示例 1：
输入：nums = [1, 7, 3, 6, 5, 6]
输出：3
解释：中心下标是 3。左侧数之和 sum = nums[0] + nums[1] + nums[2] = 1 + 7 + 3 = 11 ，右侧数之和 sum = nums[4] + nums[5] = 5 + 6 = 11 ，二者相等。

示例 2：
输入：nums = [1, 2, 3]
输出：-1
解释：数组中不存在满足此条件的中心下标。

示例 3：
输入：nums = [2, 1, -1]
输出：0
解释：中心下标是 0。左侧数之和 sum = 0 ，（下标 0 左侧不存在元素），右侧数之和 sum = nums[1] + nums[2] = 1 + -1 = 0 。
```

```
第一遍遍历数组，计算数组总和
第二遍遍历数组，从索引0开始，一直记录左边元素和，右边元素和 = 总和 - 左边元素和 - nums[i]
一旦找到左右总和一样就返回即可
```

```cpp
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int sum = 0;
        for (int num : nums) sum += num; // 求和
        int leftSum = 0;    // 中心索引左半和
        int rightSum = 0;   // 中心索引右半和
        for (int i = 0; i < nums.size(); i++) {
            leftSum += nums[i];
            rightSum = sum - leftSum + nums[i];
            if (leftSum == rightSum) return i;
        }
        return -1;
    }
};
时间复杂度：O(n)
空间复杂度：O(1)
```

### 34. 在排序数组中查找元素的第一个和最后一个位置

```
给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
如果数组中不存在目标值 target，返回 [-1, -1]。
进阶：你可以设计并实现时间复杂度为 O(log n) 的算法解决此问题吗？

示例 1：
输入：nums = [5,7,7,8,8,10], target = 8
输出：[3,4]

示例 2：
输入：nums = [5,7,7,8,8,10], target = 6
输出：[-1,-1]

示例 3：
输入：nums = [], target = 0
输出：[-1,-1]
```

```
由于数组已经排序，那么使用二分查找即可
由于target在数组中可能出现多次，那么我们在二分第一次找到它时，要以它为中心向两边查找
```

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int i = 0, j = nums.size() - 1;
        vector<int> ans;
        while (i <= j) {
            int mid = (i + j) / 2;
            if (target == nums[mid]) {
                int left = mid, right = mid;
                while (left >= 0 && nums[left] == target) {
                    left--;
                }
                while (right < nums.size() && nums[right] == target) {
                    right++;
                }
                ans.push_back(left+1);
                ans.push_back(right-1);
                return ans;
            }else if (target > nums[mid]) {
                i = mid + 1;
            }else {
                j = mid -1;
            }
        }
        return {-1,-1};
    }
};
时间复杂度：O(logn)
空间复杂度：O(1)
```

### 922. 按奇偶排序数组 II

```
给定一个非负整数数组 A， A 中一半整数是奇数，一半整数是偶数。
对数组进行排序，以便当 A[i] 为奇数时，i 也是奇数；当 A[i] 为偶数时， i 也是偶数。
你可以返回任何满足上述条件的数组作为答案。

示例：
输入：[4,2,5,7]
输出：[4,5,2,7]
解释：[4,7,2,5]，[2,5,4,7]，[2,7,4,5] 也会被接受。
```

```
我们设置一个辅助数组，用于记录一个可行答案。设置odd_idx,even_idx用于记录奇偶索引
我们遍历原数组，当遇到奇数时，就将答案数组odd_idx赋值，否则将even_idx赋值
然后索引更新
```

```c++
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& nums) {
        vector<int> ans(nums.size());
        int odd_idx = 1, even_idx = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] % 2) {
                //奇数
                ans[odd_idx] = nums[i];
                odd_idx += 2;
            }else {
                ans[even_idx] = nums[i];
                even_idx += 2;
            }
        }
        return ans;
    }
};
```

### 35.搜索插入位置

```
给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。
你可以假设数组中无重复元素。

示例 1:
输入: [1,3,5,6], 5
输出: 2

示例 2:
输入: [1,3,5,6], 2
输出: 1

示例 3:
输入: [1,3,5,6], 0
输出: 0
```

```
如果数组存在目标值，二分查找就能找到结果
如果没有目标值，最后左边界就是要插入的位置
```

```cpp
class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int i = 0, j = nums.size() - 1;
        while (i <= j) {
            int mid = (i + j) / 2;
            if (target == nums[mid]) {
                return mid;
            }else if (target > nums[mid]) {
                i = mid + 1;
            }else {
                j = mid - 1;
            }
        }
        return i;
    }
};
时间复杂度：O(logn)
空间复杂度：O(1)
```

### 24. 两两交换链表中的节点

给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

![24.两两交换链表中的节点-题意](https://code-thinking.cdn.bcebos.com/pics/24.%E4%B8%A4%E4%B8%A4%E4%BA%A4%E6%8D%A2%E9%93%BE%E8%A1%A8%E4%B8%AD%E7%9A%84%E8%8A%82%E7%82%B9-%E9%A2%98%E6%84%8F.jpg)

```
特殊情况1：链表为空，直接返回空
特殊情况2：只有1个节点，直接返回该节点
其他情况：按上图示例1解释
	两两交换模拟即可，需要注意的是，当交换1,2时，1的下一个节点得是4，不能是3。也就是说本次交换节点需要考虑下一次交换节点的情况。
```

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        if (!head) {
            return nullptr;
        }
        if (!head->next) {
            return head;
        }
        ListNode* cur = head;
        ListNode* nxt = head->next;
        ListNode* ans = head->next;
        while (cur && nxt) {
            ListNode* first = nxt->next;
            if (!first){
                //没有下一次交换了
                nxt->next = cur;
                cur->next = nullptr;
                cur = nullptr;
            }else if (!first->next) {
                //还有一个节点，它不用交换
                cur->next = first;
                nxt->next = cur;
                cur = first;
                nxt = first->next;
            }else {
                nxt->next = cur;
                cur->next = first->next;
                cur = first;
                nxt = first->next;
            }
        }
        return ans;
    }
};
时间复杂度：O(n)
空间复杂度：O(1)
```

### 205. 同构字符串

```
给定两个字符串 s 和 t，判断它们是否是同构的。
如果 s 中的字符可以按某种映射关系替换得到 t ，那么这两个字符串是同构的。
每个出现的字符都应当映射到另一个字符，同时不改变字符的顺序。不同字符不能映射到同一个字符上，相同字符只能映射到同一个字符上，字符可以映射到自己本身。

示例 1:
输入：s = "egg", t = "add"
输出：true

示例 2：
输入：s = "foo", t = "bar"
输出：false

示例 3：
输入：s = "paper", t = "title"
输出：true

提示：可以假设 s 和 t 长度相同。
```

```
基本思路：使用哈希表记录映射关系，一旦映射关系需要发生改变，那么就不是同构
注意点：由于相同字符只能映射到一个字符上，那么需要s,t都进行映射，不能单向映射
```

```cpp
class Solution {
public:
    bool isIsomorphic(string s, string t) {
        unordered_map<char, char> map1;
        unordered_map<char, char> map2;
        for (int i = 0, j = 0; i < s.size(); i++, j++) {
            if (map1.find(s[i]) == map1.end()) { // map1保存s[i] 到 t[j]的映射
                map1[s[i]] = t[j];
            }
            if (map2.find(t[j]) == map2.end()) { // map2保存t[j] 到 s[i]的映射
                map2[t[j]] = s[i];
            }
            // 发现映射 对应不上，立刻返回false
            if (map1[s[i]] != t[j] || map2[t[j]] != s[i]) {
                return false;
            }
        }
        return true;
    }
};
```

### 1002. 查找常用字符

```
给你一个字符串数组 words ，请你找出所有在 words 的每个字符串中都出现的共用字符（ 包括重复字符），并以数组形式返回。你可以按 任意顺序 返回答案。

示例 1：
输入：words = ["bella","label","roller"] 输出：["e","l","l"]

示例 2：
输入：words = ["cool","lock","cook"] 输出：["c","o"]

提示：
1 <= words.length <= 100 1 <= words[i].length <= 100 words[i] 由小写英文字母组成
```

![1002.查找常用字符](https://code-thinking.cdn.bcebos.com/pics/1002.%E6%9F%A5%E6%89%BE%E5%B8%B8%E7%94%A8%E5%AD%97%E7%AC%A6.png)

```cpp
class Solution {
public:
    vector<string> commonChars(vector<string>& A) {
        vector<string> result;
        if (A.size() == 0) return result;
        int hash[26] = {0}; // 用来统计所有字符串里字符出现的最小频率
        for (int i = 0; i < A[0].size(); i++) { // 用第一个字符串给hash初始化
            hash[A[0][i] - 'a']++;
        }

        int hashOtherStr[26] = {0}; // 统计除第一个字符串外字符的出现频率
        for (int i = 1; i < A.size(); i++) {
            memset(hashOtherStr, 0, 26 * sizeof(int));
            for (int j = 0; j < A[i].size(); j++) {
                hashOtherStr[A[i][j] - 'a']++;
            }
            // 更新hash，保证hash里统计26个字符在所有字符串里出现的最小次数
            for (int k = 0; k < 26; k++) {
                hash[k] = min(hash[k], hashOtherStr[k]);
            }
        }
        // 将hash统计的字符次数，转成输出形式
        for (int i = 0; i < 26; i++) {
            while (hash[i] != 0) { // 注意这里是while，多个重复的字符
                string s(1, i + 'a'); // char -> string
                result.push_back(s);
                hash[i]--;
            }
        }

        return result;
    }
};
```

### 925.长按键入

```
你的朋友正在使用键盘输入他的名字 name。偶尔，在键入字符 c 时，按键可能会被长按，而字符可能被输入 1 次或多次。
你将会检查键盘输入的字符 typed。如果它对应的可能是你的朋友的名字（其中一些字符可能被长按），那么就返回 True。

示例 1：
输入：name = "alex", typed = "aaleex"
输出：true
解释：'alex' 中的 'a' 和 'e' 被长按。

示例 2：
输入：name = "saeed", typed = "ssaaedd"
输出：false
解释：'e' 一定需要被键入两次，但在 typed 的输出中不是这样。
```

```
这题不能直接用哈希去记录每个字符，因为要考虑出现顺序
所以模拟同时遍历两个数组，进行对比就可以了。

对比的时候需要一下几点：
	name[i] 和 typed[j]相同，则i++，j++ （继续向后对比）
	name[i] 和 typed[j]不相同
		看是不是第一位就不相同了，也就是j如果等于0，那么直接返回false
		不是第一位不相同，就让j跨越重复项，移动到重复项之后的位置，再次比较name[i] 和typed[j]
			如果 name[i] 和 typed[j]相同，则i++，j++ （继续向后对比）
			不相同，返回false
	对比完之后有两种情况
		name没有匹配完，例如name:"pyplrzzzzdsfa" type:"ppyypllr"
		type没有匹配完，例如name:"alex" type:"alexxrrrrssda"
```

```cpp
class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        int i = 0, j = 0;
        while (i < name.size() && j < typed.size()) {
            if (name[i] == typed[j]) { // 相同则同时向后匹配
                j++; i++;
            } else { // 不相同
                if (j == 0) return false; // 如果是第一位就不相同直接返回false
                // j跨越重复项，向后移动，同时防止j越界
                while(j < typed.size() && typed[j] == typed[j - 1]) j++;
                if (name[i] == typed[j]) { // j跨越重复项之后再次和name[i]匹配
                    j++; i++; // 相同则同时向后匹配
                }
                else return false;
            }
        }
        // 说明name没有匹配完，例如 name:"pyplrzzzzdsfa" type:"ppyypllr"
        if (i < name.size()) return false;

        // 说明type没有匹配完，例如 name:"alex" type:"alexxrrrrssda"
        while (j < typed.size()) {
            if (typed[j] == typed[j - 1]) j++;
            else return false;
        }
        return true;
    }
};
时间复杂度：O(n) 空间复杂度：O(1)
```

### 844.比较含退格的字符串

```
给定 S 和 T 两个字符串，当它们分别被输入到空白的文本编辑器后，判断二者是否相等，并返回结果。 # 代表退格字符。
注意：如果对空文本输入退格字符，文本继续为空。

示例 1：
输入：S = "ab#c", T = "ad#c"
输出：true
解释：S 和 T 都会变成 “ac”。

示例 2：
输入：S = "ab##", T = "c#d#"
输出：true
解释：S 和 T 都会变成 “”。

示例 3：
输入：S = "a##c", T = "#a#c"
输出：true
解释：S 和 T 都会变成 “c”。

示例 4：
输入：S = "a#c", T = "b"
输出：false
解释：S 会变成 “c”，但 T 仍然是 “b”。
```

```
栈模拟：
对两个字符串用栈进行模拟，找到最后输入到文本编辑器的字符串
最后对比两个字符串是否相同即可
```

```cpp
class Solution {
private:
string getString(const string& S) {
    string s;
    for (int i = 0; i < S.size(); i++) {
        if (S[i] != '#') s += S[i];
        else if (!s.empty()) {
            s.pop_back();
        }
    }
    return s;
}
public:
    bool backspaceCompare(string S, string T) {
        return getString(S) == getString(T);
    }
};
时间复杂度：O(n + m)
空间复杂度：O(n + m)
```

```
双指针：
由于#只能回退前面输入过的字符，对于出现在#后的字符是没办法回退的
所有我们可以从后向前遍历，当遇到#时就处理，将所有会被回退的字符删去。
比较每两个出现的字符是否相同即可
```

```cpp
class Solution {
public:
    bool backspaceCompare(string S, string T) {
        int sSkipNum = 0; // 记录S的#数量
        int tSkipNum = 0; // 记录T的#数量
        int i = S.size() - 1;
        int j = T.size() - 1;
        while (1) {
            while (i >= 0) { // 从后向前，消除S的#
                if (S[i] == '#') sSkipNum++;
                else {
                    if (sSkipNum > 0) sSkipNum--;
                    else break;
                }
                i--;
            }
            while (j >= 0) { // 从后向前，消除T的#
                if (T[j] == '#') tSkipNum++;
                else {
                    if (tSkipNum > 0) tSkipNum--;
                    else break;
                }
                j--;
            }
            // 后半部分#消除完了，接下来比较S[i] != T[j]
            if (i < 0 || j < 0) break; // S 或者T 遍历到头了
            if (S[i] != T[j]) return false;
            i--;j--;
        }
        // 说明S和T同时遍历完毕
        if (i == -1 && j == -1) return true;
        return false;
    }
};
时间复杂度：O(n + m)
空间复杂度：O(1)
```

### 129. 求根节点到叶节点数字之和

```
给你一个二叉树的根节点 root ，树中每个节点都存放有一个 0 到 9 之间的数字。
每条从根节点到叶节点的路径都代表一个数字：
例如，从根节点到叶节点的路径 1 -> 2 -> 3 表示数字 123 。
计算从根节点到叶节点生成的 所有数字之和 。
叶节点 是指没有子节点的节点。
```

**示例 1：**

![img](https://assets.leetcode.com/uploads/2021/02/19/num1tree.jpg)

```
输入：root = [1,2,3]
输出：25
解释：
从根到叶子节点路径 1->2 代表数字 12
从根到叶子节点路径 1->3 代表数字 13
因此，数字总和 = 12 + 13 = 25
```

```
从题目上看，我们需要知道所有节点的值才能计算和
所有需要遍历二叉树，同时我们需要记录路径才能获取这条路径表示的数字，所以我们需要回溯找寻所有可能路径
```

```cpp
class Solution {
public:
    int ans;
    void backtracking(TreeNode* root, int cur) {
        if (root->left == nullptr && root->right == nullptr) {
            cur = cur * 10 + root->val;
            ans += cur;
            return;
        }
        if (root->left) {
            backtracking(root->left, cur * 10 + root->val);
        }
        if (root->right) {
            backtracking(root->right, cur * 10 + root->val);
        }
    }
    int sumNumbers(TreeNode* root) {
        ans = 0;
        backtracking(root, 0);
        return ans;
    }
};
```

### 1382.将二叉搜索树变平衡

```
给你一棵二叉搜索树，请你返回一棵 平衡后 的二叉搜索树，新生成的树应该与原来的树有着相同的节点值。
如果一棵二叉搜索树中，每个节点的两棵子树高度差不超过 1 ，我们就称这棵二叉搜索树是 平衡的 。
如果有多种构造方法，请你返回任意一种。

示例：
输入：root = [1,null,2,null,3,null,4,null,null]
输出：[2,1,3,null,null,null,4]
解释：这不是唯一的正确答案，[3,1,4,null,2,null,null] 也是一个可行的构造方案。

提示：
树节点的数目在 1 到 10^4 之间。
树节点的值互不相同，且在 1 到 10^5 之间。
```

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210726154512.png)

```
平衡二叉树最大的特点就是左右子树高度相差小于等于1
那么我们构建子树，就需要让左右子树的节点数目差小于等于1

我们中序遍历题目给定的二叉搜索树，存到数组中
取中点作为根节点，然后左右区间分别作为左右子树的节点区间
这样递归下去就能保证，每个子树的左右子树的节点数差值小于等于1
```

```cpp
class Solution {
private:
    vector<int> vec;
    // 有序树转成有序数组
    void traversal(TreeNode* cur) {
        if (cur == nullptr) {
            return;
        }
        traversal(cur->left);
        vec.push_back(cur->val);
        traversal(cur->right);
    }
    // 有序数组转平衡二叉树
    TreeNode* getTree(vector<int>& nums, int left, int right) {
        if (left > right) return nullptr;
        int mid = left + ((right - left) / 2);
        TreeNode* root = new TreeNode(nums[mid]);
        root->left = getTree(nums, left, mid - 1);
        root->right = getTree(nums, mid + 1, right);
        return root;
    }

public:
    TreeNode* balanceBST(TreeNode* root) {
        traversal(root);
        return getTree(vec, 0, vec.size() - 1);
    }
};
```

### 100. 相同的树

给定两个二叉树，编写一个函数来检验它们是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210726172932.png)

![img](https://code-thinking-1253855093.file.myqcloud.com/pics/20210726173011.png)

```
根节点是否相同，左右子树是否相同
直接递归判断即可

1.确定递归函数的参数和返回值
我们要比较的是两个树是否是相互相同的，参数也就是两个树的根节点。

2.确定终止条件
要比较两个节点数值相不相同，首先要把两个节点为空的情况弄清楚！否则后面比较数值的时候就会操作空指针了。
节点为空的情况有：
	tree1为空，tree2不为空，不对称，return false
	tree1不为空，tree2为空，不对称 return false
	tree1，tree2都为空，对称，返回true
此时已经排除掉了节点为空的情况，那么剩下的就是tree1和tree2不为空的时候：
	tree1、tree2都不为空，比较节点数值，不相同就return false

3.确定单层递归的逻辑
比较二叉树是否相同 ：传入的是tree1的左孩子，tree2的右孩子。
如果左右都相同就返回true ，有一侧不相同就返回false 。
```

```cpp
class Solution {
public:
    bool compare(TreeNode* tree1, TreeNode* tree2) {
        if (tree1 == NULL && tree2 != NULL) return false;
        else if (tree1 != NULL && tree2 == NULL) return false;
        else if (tree1 == NULL && tree2 == NULL) return true;
        else if (tree1->val != tree2->val) return false; // 注意这里我没有使用else

        // 此时就是：左右节点都不为空，且数值相同的情况
        // 此时才做递归，做下一层的判断
        bool left = compare(tree1->left, tree2->left);   // 左子树：左、 右子树：左
        bool right = compare(tree1->right, tree2->right);  // 左子树：右、 右子树：右
        bool isSame = left && right;                    // 左子树：中、 右子树：中（逻辑处理）
        return isSame;

    }
    bool isSameTree(TreeNode* p, TreeNode* q) {
        return compare(p, q);
    }
};
```

### 116. 填充每个节点的下一个右侧节点指针

给定一个 **完美二叉树** ，其所有叶子节点都在同一层，每个父节点都有两个子节点。二叉树定义如下：

```
struct Node {
  int val;
  Node *left;
  Node *right;
  Node *next;
}
```

填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 `NULL`。

初始状态下，所有 next 指针都被设置为 `NULL`。

**示例 1：**

![img](https://assets.leetcode.com/uploads/2019/02/14/116_sample.png)

```
输入：root = [1,2,3,4,5,6,7]
输出：[1,#,2,3,#,4,5,6,7,#]
解释：给定二叉树如图 A 所示，你的函数应该填充它的每个 next 指针，以指向其下一个右侧节点，如图 B 所示。序列化的输出按层序遍历排列，同一层节点由 next 指针连接，'#' 标志着每一层的结束。
```

```cpp
层序遍历，每一层所有节点进行赋值right指针即可
class Solution {
public:
    Node* connect(Node* root) {
        queue<Node*> que;
        if (root != nullptr) que.push(root);
        while (!que.empty()) {
            int size = que.size();
            for (int i = 0; i < size; ++i) {
                Node* node = que.front();
                que.pop();
                if (i != size - 1) {
                    node->next = que.front(); //如果不是最后一个Node 则指向下一个Node
                } else node->next = nullptr;  //如果是最后一个Node 则指向nullptr
                if (node->left != nullptr) que.push(node->left);
                if (node->right != nullptr) que.push(node->right);
            }
        }
        return root;
    }
};
```

### 649. Dota2 参议院

```
Dota2 的世界里有两个阵营：Radiant(天辉)和 Dire(夜魇)
Dota2 参议院由来自两派的参议员组成。现在参议院希望对一个 Dota2 游戏里的改变作出决定。他们以一个基于轮为过程的投票进行。在每一轮中，每一位参议员都可以行使两项权利中的一项：
禁止一名参议员的权利：参议员可以让另一位参议员在这一轮和随后的几轮中丧失所有的权利。
宣布胜利：如果参议员发现有权利投票的参议员都是同一个阵营的，他可以宣布胜利并决定在游戏中的有关变化。
给定一个字符串代表每个参议员的阵营。字母 “R” 和 “D” 分别代表了 Radiant（天辉）和 Dire（夜魇）。然后，如果有 n 个参议员，给定字符串的大小将是 n。
以轮为基础的过程从给定顺序的第一个参议员开始到最后一个参议员结束。这一过程将持续到投票结束。所有失去权利的参议员将在过程中被跳过。
假设每一位参议员都足够聪明，会为自己的政党做出最好的策略，你需要预测哪一方最终会宣布胜利并在 Dota2 游戏中决定改变。输出应该是 Radiant 或 Dire。

示例 1：
输入："RD"
输出："Radiant"
解释：第一个参议员来自 Radiant 阵营并且他可以使用第一项权利让第二个参议员失去权力，因此第二个参议员将被跳过因为他没有任何权利。然后在第二轮的时候，第一个参议员可以宣布胜利，因为他是唯一一个有投票权的人

示例 2：
输入："RDD"
输出："Dire"
解释： 第一轮中,第一个来自 Radiant 阵营的参议员可以使用第一项权利禁止第二个参议员的权利， 第二个来自 Dire 阵营的参议员会被跳过因为他的权利被禁止， 第三个来自 Dire 阵营的参议员可以使用他的第一项权利禁止第一个参议员的权利， 因此在第二轮只剩下第三个参议员拥有投票的权利,于是他可以宣布胜利。
```

```
例如输入"RRDDD"，执行过程应该是什么样呢？
第一轮：senate[0]的R消灭senate[2]的D，senate[1]的R消灭senate[3]的D，senate[4]的D消灭senate[0]的R，此时剩下"RD"，第一轮结束！
第二轮：senate[0]的R消灭senate[1]的D，第二轮结束
第三轮：只有R了，R胜利
估计不少同学都困惑，R和D数量相同怎么办，究竟谁赢，其实这是一个持续消灭的过程！ 即：如果同时存在R和D就继续进行下一轮消灭，轮数直到只剩下R或者D为止！

那么每一轮消灭的策略应该是什么呢？
例如：RDDRD
第一轮：senate[0]的R消灭senate[1]的D，那么senate[2]的D，是消灭senate[0]的R还是消灭senate[3]的R呢？
当然是消灭senate[3]的R，因为当轮到这个R的时候，它可以消灭senate[4]的D。
所以消灭的策略是，尽量消灭自己后面的对手，因为前面的对手已经使用过权利了，而后序的对手依然可以使用权利消灭自己的同伴！
那么局部最优：有一次权利机会，就消灭自己后面的对手。全局最优：为自己的阵营赢取最大利益。
```

```cpp
实现代码，在每一轮循环的过程中，去过模拟优先消灭身后的对手，其实是比较麻烦的。

这里有一个技巧，就是用一个变量记录当前参议员之前有几个敌对对手了，进而判断自己是否被消灭了。这个变量我用flag来表示。

由于外层有个while循环，所以每一轮通过flag的计数，就可以实现后面的D消灭前面的R的操作
class Solution {
public:
    string predictPartyVictory(string senate) {
        // R = true表示本轮循环结束后，字符串里依然有R。D同理
        bool R = true, D = true;
        // 当flag大于0时，R在D前出现，R可以消灭D。当flag小于0时，D在R前出现，D可以消灭R
        int flag = 0;
        while (R && D) { // 一旦R或者D为false，就结束循环，说明本轮结束后只剩下R或者D了
            R = false;
            D = false;
            for (int i = 0; i < senate.size(); i++) {
                if (senate[i] == 'R') {
                    if (flag < 0) senate[i] = 0; // 消灭R，R此时为false
                    else R = true; // 如果没被消灭，本轮循环结束有R
                    flag++;
                }
                if (senate[i] == 'D') {
                    if (flag > 0) senate[i] = 0;
                    else D = true;
                    flag--;
                }
            }
        }
        // 循环结束之后，R和D只能有一个为true
        return R == true ? "Radiant" : "Dire";
    }
};
```

### 1221. 分割平衡字符串

```
在一个 平衡字符串 中，'L' 和 'R' 字符的数量是相同的。
给你一个平衡字符串 s，请你将它分割成尽可能多的平衡字符串。
注意：分割得到的每个字符串都必须是平衡字符串。
返回可以通过分割得到的平衡字符串的 最大数量 。

示例 1：
输入：s = "RLRRLLRLRL"
输出：4
解释：s 可以分割为 "RL"、"RRLL"、"RL"、"RL" ，每个子字符串中都包含相同数量的 'L' 和 'R' 。

示例 2：
输入：s = "RLLLLRRRLR"
输出：3
解释：s 可以分割为 "RL"、"LLLRRR"、"LR" ，每个子字符串中都包含相同数量的 'L' 和 'R' 。
```

```
从前向后遍历，只要遇到平衡子串，计数就+1，遍历一遍即可。
局部最优：从前向后遍历，只要遇到平衡子串 就统计
全局最优：统计了最多的平衡子串。
```

```cpp
class Solution {
public:
    int balancedStringSplit(string s) {
        int result = 0;
        int count = 0;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == 'R') count++;
            else count--;
            if (count == 0) result++;
        }
        return result;
    }
};
```

### 5.最长回文子串

```
给你一个字符串 s，找到 s 中最长的回文子串。

示例 1：
输入：s = "babad"
输出："bab"
解释："aba" 同样是符合题意的答案。

示例 2：
输入：s = "cbbd"
输出："bb"
```

```
从答案开始想，最后的最长回文子串一定是某一个区间，起点终点都不确定，那么就要用动态规划进行统计

1.确定dp数组（dp table）以及下标的含义
dp[i][j]表示[i, j]区间的子串是不是回文串

2.确定递推公式
i == j 	dp[i][j] = true;
s[i] != s[j] dp[i][j] = false;
s[i] == s[j]
	j - i <= 1 dp[i][j] = true;
	j - i > 1  dp[i][j] = dp[i + 1][j - 1]

3.确定遍历顺序
从dp[i][j] = dp[i + 1][j - 1]可知，i从n - 1向0去，j从0向n - 1去

4.dp数组如何初始化
dp[i][i] = true; dp[i][j] = false;
```

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int maxLength = 0;
        int n = s.size();
        int left = 0,right = n-1;
        vector<vector<bool>> dp(n,vector<bool>(n,false));
        for(int i=n-1;i>=0;i--){
            for(int j=i;j<n;j++){
                if(s[i] == s[j]){
                    if(j - i <= 1){
                        dp[i][j] = true;
                    }else if(dp[i+1][j-1]){
                        dp[i][j] = true;
                    }
                }
                if(dp[i][j] && j-i+1>maxLength){
                    left = i;
                    right = j;
                    maxLength = j - i + 1;
                }
            }
        }
        return s.substr(left,maxLength);
    }
};
```

### 132. 分割回文串 II

```
给你一个字符串 s，请你将 s 分割成一些子串，使每个子串都是回文。
返回符合要求的 最少分割次数 。

示例 1：
输入：s = "aab" 输出：1 解释：只需一次分割就可将 s 分割成 ["aa","b"] 这样两个回文子串。

示例 2： 输入：s = "a" 输出：0

示例 3： 输入：s = "ab" 输出：1

提示：
1 <= s.length <= 2000
s 仅由小写英文字母组成
```

```
1.确定dp数组的含义
dp[i]表示[0, i]的子串分割为回文串最小的分割次数

2.确定递推公式
如果[0, i]子串本身就是回文串，那么dp[i] = 0
否则我们就去分割，设分割点为j，如果[j + 1, i]是回文串，那么dp[i] = dp[j] + 1
我们遍历所有分割点，找到最小的dp[j]即可

3.确定递推顺序
从前向后即可

4.确定初始化
dp[0] = 0，dp[i] = INT_MAX
```

```cpp
class Solution {
public:
    int minCut(string s) {
        vector<vector<bool>> isPalindromic(s.size(), vector<bool>(s.size(), false));
        for (int i = s.size() - 1; i >= 0; i--) {
            for (int j = i; j < s.size(); j++) {
                if (s[i] == s[j] && (j - i <= 1 || isPalindromic[i + 1][j - 1])) {
                    isPalindromic[i][j] = true;
                }
            }
        }
        // 初始化
        vector<int> dp(s.size(), 0);
        for (int i = 0; i < s.size(); i++) dp[i] = i;

        for (int i = 1; i < s.size(); i++) {
            if (isPalindromic[0][i]) {
                dp[i] = 0;
                continue;
            }
            for (int j = 0; j < i; j++) {
                if (isPalindromic[j + 1][i]) {
                    dp[i] = min(dp[i], dp[j] + 1);
                }
            }
        }
        return dp[s.size() - 1];

    }
};
```

### 673.最长递增子序列的个数

```
给定一个未排序的整数数组，找到最长递增子序列的个数。

示例 1:
输入: [1,3,5,4,7]
输出: 2
解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。

示例 2:
输入: [2,2,2,2,2]
输出: 5
解释: 最长递增子序列的长度是1，并且存在5个子序列的长度为1，因此输出5。
```

```
本题是最长递增子序列的进阶版，需要统计的是最长子序列的个数
1.确定dp数组的含义
我们需要维护两个数组
dp[i]表示以nums[i]为结尾的最长子序列长度
count[i]表示以nums[i]为结尾的最长子序列的个数

2.确定递推公式
对于dp[i],我们需要寻找分割点j，如果nums[i] > nums[j]，那么dp[i] = dp[j] + 1
对于所有的j，我们取最大的，所以dp[i] = max(dp[i], dp[j] + 1)

同时count[i]也需要分割点j,在nums[i] > nums[j]的情况下：
如果dp[j] + 1 > dp[i]就说明找到了更长的子序列，count[i] = count[j]
如果dp[j] + 1 == dp[i]就说明同样长的子序列又出现了，count[i] += count[j]

3.确定递推方向
从前向后递推即可

4.确定初始化条件
dp[i] = 1 以自己结尾长度为1
count[i] = 1 自己作为最长子序列，个数为1
```

```cpp
class Solution {
public:
    int findNumberOfLIS(vector<int>& nums) {
        if (nums.size() <= 1) return nums.size();
        vector<int> dp(nums.size(), 1);
        vector<int> count(nums.size(), 1);
        int maxCount = 0;
        for (int i = 1; i < nums.size(); i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    if (dp[j] + 1 > dp[i]) {
                        dp[i] = dp[j] + 1;
                        count[i] = count[j];
                    } else if (dp[j] + 1 == dp[i]) {
                        count[i] += count[j];
                    }
                }
                if (dp[i] > maxCount) maxCount = dp[i];
            }
        }
        int result = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (maxCount == dp[i]) result += count[i];
        }
        return result;
    }
};
时间复杂度：O(n^2)
空间复杂度：O(n)
```

### 841.钥匙和房间

```
有 N 个房间，开始时你位于 0 号房间。每个房间有不同的号码：0，1，2，...，N-1，并且房间里可能有一些钥匙能使你进入下一个房间。

在形式上，对于每个房间 i 都有一个钥匙列表 rooms[i]，每个钥匙 rooms[i][j] 由 [0,1，...，N-1] 中的一个整数表示，其中 N = rooms.length。 钥匙 rooms[i][j] = v 可以打开编号为 v 的房间。

最初，除 0 号房间外的其余所有房间都被锁住。
你可以自由地在房间之间来回走动。
如果能进入每个房间返回 true，否则返回 false。

示例 1：
输入: [[1],[2],[3],[]]
输出: true
解释: 我们从 0 号房间开始，拿到钥匙 1。 之后我们去 1 号房间，拿到钥匙 2。 然后我们去 2 号房间，拿到钥匙 3。 最后我们去了 3 号房间。 由于我们能够进入每个房间，我们返回 true。

示例 2：
输入：[[1,3],[3,0,1],[2],[0]]
输出：false
解释：我们不能进入 2 号房间。
```
