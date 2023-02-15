---
title: LeetCode题解（一）
mathjax: true
tags:
  - Hot 100
categories:
  - [刷题, LeetCode]
abbrlink: 33fe21e
date: 2021-03-27 11:55:04
---

### [LeetCode 热题 Hot 100](https://leetcode-cn.com/problemset/leetcode-hot-100/)

<!-- toc -->
<!-- more -->

## 1. 两数相加

> 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出和为目标值的那两个整数，并返回它们的数组下标。
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
> 你可以按任意顺序返回答案。

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    map<int, int> map;
    vector<int> v(2,-1);
    for (int i = 0; i < nums.size(); i++) {
        if (map.count(target - nums[i]) > 0) {
            v[1] = map[target - nums[i]];
            v[0] = i;
            break;
        }
        map[nums[i]] = i;
    }
    return v;
}
```

这里要注意，若 map 中没有元素，用 if 条件判断时会自动调用构造函数添加该元素。判断是否有该元素课用`map.count()`或`map.find()`.`map.count()`找到返回`1`，否则返回`0`。`map.find()`找到返回元素位置，找不到返回`map.end()`迭代器指针。

## 2. 两数相加(链表)

> 给你两个非空的链表，表示两个非负的整数。它们每位数字都是按照逆序的方式存储的，并且每个节点只能存储一位数字。
> 请你将两个数相加，并以相同形式返回一个表示和的链表。
> 你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode *p = new ListNode(0);
    ListNode *temp = p;
    int carry = 0;
    while (l1 != nullptr || l2 != nullptr || carry != 0) {
        int val1 = l1 != nullptr ? l1->val : 0;
        int val2 = l2 != nullptr ? l2->val : 0;
        int sum = val1 + val2 + carry;
        carry = sum / 10;
        ListNode *sumNode = new ListNode(sum % 10);
        temp->next = sumNode;
        temp = temp->next;
        if (l1 != nullptr) l1 = l1->next;
        if (l2 != nullptr) l2 = l2->next;
    }
    return p->next;
}
```

## 3. 无重复字符的最长子串(滑动窗口)

> 给定一个字符串，请你找出其中不含有重复字符的最长子串的长度

[不重复最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

思路：

- 定义一个 map 数据结构存储 (k, v)，其中 key 值为字符，value 值为字符位置 +1，加 1 表示从字符位置后一个才开始不重复
  我们定义不重复子串的开始位置为 start，结束位置为 end
- 随着 end 不断遍历向后，会遇到与 [start, end] 区间内字符相同的情况，此时将字符作为 key 值，获取其 value 值，并更新 start，此时 [start, end] 区间内不存在重复字符
- 无论是否更新 start，都会更新其 map 数据结构和结果 ans。
- 时间复杂度：O(n)

```cpp
int lengthOfLongestSubstring(string s) {
    map<char, int> map;
    int left = 0, right = 0, len = 0;
    while (right < s.length()) {
        if (map.count(s[right]) == 0) {
            map[s[right]] = right;
        } else {
            if (left > map[s[right]]) {     //若出现的字符不在当前窗口内
                map[s[right]] = right;
            } else {
                left = map[s[right]] + 1;
                map[s[right]] = right;
            }
        }
        right++;
        len = ((right - left) > len) ? (right - left) : len;
    }
    return len;
}
```

## 4. 搜索旋转排序数组

> 整数数组 nums 按升序排列，数组中的值互不相同。
> 在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为 [4,5,6,7,0,1,2] 。
> 给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回 -1
> 要求时间复杂度为 O(log(n))

可以发现的是，我们将数组从中间分开成左右两部分的时候，一定有一部分的数组是有序的。拿示例来看，我们从 6 这个位置分开以后数组变成了 [4, 5, 6] 和 [7, 0, 1, 2] 两个部分，其中左边 [4, 5, 6] 这个部分的数组是有序的，其他也是如此。

这启示我们可以在常规二分查找的时候查看当前 mid 为分割位置分割出来的两个部分 [l, mid] 和 [mid + 1, r] 哪个部分是有序的，并根据有序的那个部分确定我们该如何改变二分查找的上下界，因为我们能够根据有序的那部分判断出 target 在不在这个部分：

如果 [l, mid - 1] 是有序数组，且 target 的大小满足 [nums[l],nums[mid])，则我们应该将搜索范围缩小至 [l, mid - 1]，否则在 [mid + 1, r] 中寻找。
如果 [mid, r] 是有序数组，且 target 的大小满足 (nums[mid+1],nums[r]]，则我们应该将搜索范围缩小至 [mid + 1, r]，否则在 [l, mid - 1] 中寻找。

![搜索排序数组](LeetCode题解（一）/搜索旋转排序数组.png)

```cpp
int search(vector<int>& nums, int target) {
    int n = nums.size();
    if (!n) return -1;
    if (n == 1) return target == nums[0] ? 0 : -1;
    int l = 0, r = n - 1;
    int mid = 0;
    while (l <= r) {
        mid = (l + r) / 2;
        if (target == nums[mid]) return mid;
        if (nums[mid] > nums[r]) {                          //前面有序，后面无序
            if (target >= nums[l] && target < nums[mid]) {  //target在前半部分
                r = mid - 1;
            } else {                                        //target在后半部分
                l = mid + 1;
            }
        } else {
            if (target > nums[mid] && target <= nums[r]) {
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
    }
    return -1;
}
```

## 5. 最大数

> 给定一组非负整数 nums，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数.

[题目](https://leetcode-cn.com/problems/largest-number/)

[题解](https://leetcode-cn.com/problems/largest-number/solution/zui-da-shu-by-leetcode-solution-sid5/)

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

using namespace std;

string largestNumber(vector<int>& nums) {
    sort(nums.begin(), nums.end(), [](int x, int y){
        int sx = 10, sy = 10;
        while (sx <= x) {
            sx *= 10;
        }
        while (sy <= y) {
            sy *= 10;
        }
        return x * sy + y > y * sx + x;
    });
    if (nums[0] == 0) return "0";
    string str = "";
    for (int i : nums) {
        str += to_string(i);
    }
    return str;
}

int main() {
    vector<int> v = {78, 7};
    cout << largestNumber(v) << endl;
    return 0;
}
```

## 6. 盛最多水的容器

> 给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0) 。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

思路：双指针，从两端往中间遍历，每次将两端较小的指针向中间移动，直到相遇。

证明如下：
双指针代表的是 可以作为容器边界的所有位置的范围。在一开始，双指针指向数组的左右边界，表示 数组中所有的位置都可以作为容器的边界，因为我们还没有进行过任何尝试。在这之后，我们每次将 对应的数字较小的那个指针 往 另一个指针 的方向移动一个位置，就表示我们认为 这个指针不可能再作为容器的边界了。

为什么对应的数字较小的那个指针不可能再作为容器的边界了？

在上面的分析部分，我们对这个问题有了一点初步的想法。这里我们定量地进行证明。

考虑第一步，假设当前左指针和右指针指向的数分别为 $x$ 和 $y$，不失一般性，我们假设 $x \leq y$。同时，两个指针之间的距离为 $t$。那么，它们组成的容器的容量为：

$$min(x, y)*t =x*t$$

我们可以断定，如果我们保持左指针的位置不变，那么无论右指针在哪里，这个容器的容量都不会超过 $x*t$ 了。注意这里右指针只能向左移动，因为 我们考虑的是第一步，也就是 指针还指向数组的左右边界的时候。

我们任意向左移动右指针，指向的数为 $y_1$，两个指针之间的距离为 $t_1$，那么显然有 $t_1 < t$，并且 $min(x, y_1) \leq \min(x, y)$

- 如果 $y_1 \leq y$，那么 $min(x, y_1) \leq \min(x, y)$

- 如果 $y_1 > y$，那么 $min(x, y_1) = x = min(x, y)$

因此有：

$$min(x, y_t) * t_1 < min(x, y) * t$$

即无论我们怎么移动右指针，得到的容器的容量都小于移动前容器的容量。也就是说，这个左指针对应的数不会作为容器的边界了，那么我们就可以丢弃这个位置，将左指针向右移动一个位置，此时新的左指针于原先的右指针之间的左右位置，才可能会作为容器的边界。

这样一来，我们将问题的规模减小了 1，被我们丢弃的那个位置就相当于消失了。此时的左右指针，就指向了一个新的、规模减少了的问题的数组的左右边界。便按照上述思路继续解决这个问题。

```cpp
//11.盛水最多的容器
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>

using namespace std;

int maxArea(vector<int>& height) {
    int MAX = 0;
    int res = 0;
    int l = 0, r = height.size() - 1;
    while (l < r) {
        res = (height[l] < height[r] ? height[l] : height[r]) * (r - l);
        if (height[l] < height[r]) l++;
        else r--;
        if (res > MAX) MAX = res;
    }
    return MAX;
}

int main() {
    vector<int> v = {1,8,6,2,5,4,8,3,7};
    cout << maxArea(v) << endl;
    return 0;
}
```
