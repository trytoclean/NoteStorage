[TOC]



### [1768. 交替合并字符串](https://leetcode.com/problems/merge-strings-alternately/)

You are given two strings `word1` and `word2`. Merge the strings by adding letters in alternating order, starting with `word1`. If a string is longer than the other, append the additional letters onto the end of the merged string.

Return *the merged string.*

**Example 1:**

```
Input: word1 = "abc", word2 = "pqr"
Output: "apbqcr"
Explanation: The merged string will be merged as so:
word1:  a   b   c
word2:    p   q   r
merged: a p b q c r
```

**Example 2:**

```
Input: word1 = "ab", word2 = "pqrs"
Output: "apbqrs"
Explanation: Notice that as word2 is longer, "rs" is appended to the end.
word1:  a   b 
word2:    p   q   r   s
merged: a p b q   r   s
```

**Example 3:**

```
Input: word1 = "abcd", word2 = "pq"
Output: "apbqcd"
Explanation: Notice that as word1 is longer, "cd" is appended to the end.
word1:  a   b   c   d
word2:    p   q 
merged: a p b q c   d
```

 

**Constraints:**

- `1 <= word1.length, word2.length <= 100`
- `word1` and `word2` consist of lowercase English letters.



如果相等：

​	两个迭代器指向两字符串的[0]，有先后。构建一个新的字符串，移动到end()

```cpp
#include <iostream>
#include <string>
using namespace std;

string mergeAlternately(string word1, string word2) {
    string result;
    // 预估合并后的字符串大小，避免多次分配内存
    result.reserve(word1.size() + word2.size());
    
    int i = 0, j = 0;
    while (i < word1.size() && j < word2.size()) {
        result += word1[i++];
        result += word2[j++];
    }
    // 拼接剩余的部分
    result += word1.substr(i);
    result += word2.substr(j);
    
    return result;
}

int main() {
    string word1 = "abc";
    string word2 = "defgh";
    cout << mergeAlternately(word1, word2) << endl;
    return 0;
}

```



​	warning: 题意理解有问题，本意就是想大小不一的情况，把剩下的字符填到后面，本意不是比大小

​	error: string api的熟练使用	



###  [1071. 字符串的最大公约数](https://leetcode.com/problems/greatest-common-divisor-of-strings/)

For two strings `s` and `t`, we say "`t` divides `s`" if and only if `s = t + t + t + ... + t + t` (i.e., `t` is concatenated with itself one or more times).

Given two strings `str1` and `str2`, return *the largest string* `x` *such that* `x` *divides both* `str1` *and* `str2`.

 

**Example 1:**

```
Input: str1 = "ABCABC", str2 = "ABC"
Output: "ABC"
```

**Example 2:**

```
Input: str1 = "ABABAB", str2 = "ABAB"
Output: "AB"
```

**Example 3:**

```
Input: str1 = "LEET", str2 = "CODE"
Output: ""
```

 

**Constraints:**

- `1 <= str1.length, str2.length <= 1000`
- `str1` and `str2` consist of English uppercase letters.

有点像字符串匹配，但依旧使用string api 就可以解决

​	匹配到分子字符串第一次出现的位置，记录匹配到的头和尾，将分子字符串成两个子字符串，然后拼接

（这样的原因就是，如果简单粗暴地把尾之后的保留，就会出现前面不匹配的字符也会删掉）

（但是这个记录头尾位置，怎么做？没有更好的方法）



```cpp
#include <algorithm>
#include <iostream>
#include <string>

int gcd(int a, int b) {
  while (b != 0) {
    int temp = b;
    b = a % b;
    a = temp;
  }
  return a;
}
// 在做算法题，这个最大公约数函数最好自己实现，万一不支持呢
std::string gcdOfStrings(std::string str1, std::string str2) {
  int len1 = str1.size(), len2 = str2.size();
  int gcd_len = gcd(len1, len2); // Calculate GCD of the lengths

  std::string candidate = str1.substr(0, gcd_len); // Get candidate substring

  // Check if the candidate can form both str1 and str2 by repeating
  std::string repeated_str1 = "";
  std::string repeated_str2 = "";
	repeated_str1.reserve(len1); // 提前分配容量，避免反复扩容
  repeated_str2.reserve(len2);
  // Repeat 'candidate' for the length of str1 and str2 respectively
  for (int i = 0; i < len1 / gcd_len; ++i) {
    repeated_str1 += candidate;
    //repeated_str1.append(candidate);  等同，但是append 有很多重载
  }
  for (int i = 0; i < len2 / gcd_len; ++i) {
    repeated_str2 += candidate;
  }
  /*
   if (str1 == std::string(len1 / gcd_len, candidate[0]) &&
          str2 == std::string(len2 / gcd_len, candidate[0])) {
          return candidate;
      }
  */

  // 没有string(num,string) 这种构造函数？？？

  // If both repeated strings match the original strings, return the candidate
  if (str1 == repeated_str1 && str2 == repeated_str2) {
    return candidate;
  }

  return ""; // No common divisor string found
}

int main() {
  std::string str1 = "ABCABC";
  std::string str2 = "ABC";
  std::cout << "Result: " << gcdOfStrings(str1, str2)
            << std::endl; // Output "ABC"

  str1 = "ABABAB";
  str2 = "ABAB";
  std::cout << "Result: " << gcdOfStrings(str1, str2)
            << std::endl; // Output "AB"

  str1 = "LEET";
  str2 = "CODE";
  std::cout << "Result: " << gcdOfStrings(str1, str2) << std::endl; // Output ""

  return 0;
}
```

备注：cpp不支持 std::string(string_size ,string)     第二个参数应该是字符类型，而不是字符串   



### [1431. 拥有最多糖果的孩子](https://leetcode.com/problems/kids-with-the-greatest-number-of-candies/)



There are `n` kids with candies. You are given an integer array `candies`, where each `candies[i]` represents the number of candies the `ith` kid has, and an integer `extraCandies`, denoting the number of extra candies that you have.
有 `n` 孩子，每个孩子都有糖果。给你一个整数数组 `candies` ，其中每个 `candies[i]` 表示 `i th` 孩子拥有的糖果数量；还有一个整数 `extraCandies` ，表示你拥有的额外糖果数量。

Return *a boolean array* `result` *of length* `n`*, where* `result[i]` *is* `true` *if, after giving the* `ith` *kid all the* `extraCandies`*, they will have the **greatest** number of candies among all the kids**, or* `false` *otherwise*.
返回*长度为* `n` *布尔数组* `result` *，其中* *，如果在给* `i th` *孩子所有* `extraCandies` 之后 *，他们将拥有所有孩子中**最多的**糖果* *，* 则 `result[i]` *为* `true` ，否则*为* `false` 。

Note that **multiple** kids can have the **greatest** number of candies.
请注意， **多个**孩子可以拥有**最多**数量的糖果。

 

**Example 1: 示例 1：**

```
Input: candies = [2,3,5,1,3], extraCandies = 3
Output: [true,true,true,false,true] 
Explanation: If you give all extraCandies to:
- Kid 1, they will have 2 + 3 = 5 candies, which is the greatest among the kids.
- Kid 2, they will have 3 + 3 = 6 candies, which is the greatest among the kids.
- Kid 3, they will have 5 + 3 = 8 candies, which is the greatest among the kids.
- Kid 4, they will have 1 + 3 = 4 candies, which is not the greatest among the kids.
- Kid 5, they will have 3 + 3 = 6 candies, which is the greatest among the kids.
```

**Example 2: 示例 2：**

```
Input: candies = [4,2,1,1,2], extraCandies = 1
Output: [true,false,false,false,false] 
Explanation: There is only 1 extra candy.
Kid 1 will always have the greatest number of candies, even if a different kid is given the extra candy.
```

**Example 3: 示例 3：**

```
Input: candies = [12,1,12], extraCandies = 10
Output: [true,false,true]
```

 

**Constraints: 限制：**

- `n == candies.length`
- `2 <= n <= 100`
- `1 <= candies[i] <= 100`
- `1 <= extraCandies <= 50`

```cpp
#include <vector>
#include <algorithm>
using namespace std;

vector<bool> kidsWithCandies(vector<int>& candies, int extraCandies) {
    int maxCandies = *max_element(candies.begin(), candies.end()); // O(n)
    vector<bool> result;
    result.reserve(candies.size()); // 减少内存分配

    for (int c : candies) {
        result.push_back(c + extraCandies >= maxCandies);
    }
    return result;
}
```

​	error: 向false的一面。 true：向true的一面，也就是如果不大于maxmax

critical：通过提前分配vector的大小





[605. 可以放置鲜花](https://leetcode.com/problems/can-place-flowers/)

You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.

Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` *if* `n` *new flowers can be planted in the* `flowerbed` *without violating the no-adjacent-flowers rule and* `false` *otherwise*.

 

**Example 1:**

```
Input: flowerbed = [1,0,0,0,1], n = 1
Output: true
```

**Example 2:**

```
Input: flowerbed = [1,0,0,0,1], n = 2
Output: false
```

 

**Constraints:**

- `1 <= flowerbed.length <= 2 * 104`
- `flowerbed[i]` is `0` or `1`.
- There are no two adjacent flowers in `flowerbed`.
- `0 <= n <= flowerbed.length`



```cpp
#include <vector>
using namespace std;

class Solution {
public:
    bool canPlaceFlowers(vector<int>& flowerbed, int n) {
        int len = flowerbed.size();
        for (int i = 0; i < len && n > 0; ++i) {
            if (flowerbed[i] == 0) {
                int left = (i == 0) ? 0 : flowerbed[i - 1];
                int right = (i == len - 1) ? 0 : flowerbed[i + 1];
                if (left == 0 && right == 0) {
                    flowerbed[i] = 1; // 种花
                    --n;
                }
            }
        }
        return n <= 0;
    }
};

```

​	warning ： 我先想到了左中右都是零，就可以种，但是我一直纠结把数组置为一，然后转而就去像这个数组没有说明哪个位置有一，于是越想越复杂。







---

### temp: about algorithms philosophy 

about double-pointer



非常好 👍 这是算法训练中一个非常核心、常考且高频使用的技巧。我们系统性地来讲解 **双指针算法（Two Pointers Technique）**。

------

## 🧩 一、双指针算法的核心思想

> **核心思想：**
>  通过两个索引（指针）在同一个序列（数组、字符串、链表等）上**有序移动**，来减少不必要的遍历，从而降低算法复杂度。

比如传统的两层循环 (O(n^2)) 问题，有时通过双指针可以优化为 (O(n))。

------

## 🧠 二、双指针算法的常见“形态”

| 类型                                          | 说明                               | 典型例子                                          |
| --------------------------------------------- | ---------------------------------- | ------------------------------------------------- |
| **对撞指针（Opposite / Collision Pointers）** | 两个指针分别从序列两端向中间移动   | 有序数组两数之和、回文判断、盛水最多的容器        |
| **快慢指针（Fast-Slow Pointers）**            | 一个指针移动快，一个移动慢         | 链表中点查找、环检测（Floyd Cycle）、删除重复元素 |
| **滑动窗口（Sliding Window）**                | 双指针维护一个区间 `[left, right)` | 子数组和问题、最小覆盖子串、最长不重复子串        |
| **多指针变种**                                | 常见于排序归并、K路合并等          | 归并排序、合并区间、K路最小堆合并                 |

------

## 🧰 三、适合的数据结构

| 数据结构           | 适用原因                       | 示例                              |
| ------------------ | ------------------------------ | --------------------------------- |
| **数组 / 向量**    | 随机访问、可索引移动           | Two Sum、移除元素、排序双指针     |
| **字符串**         | 类似数组，可以按字符比较或拼接 | 回文判断、去除多余空格            |
| **链表**           | 快慢指针尤其常用               | 环检测、找中点、删除倒数第N个节点 |
| **区间型数据结构** | 适合滑动窗口                   | 滑动窗口求最大/最小值、区间求和   |

------

## ⚙️ 四、双指针常见应用场景

| 场景          | 典型题目                             | 思考模式                 |
| ------------- | ------------------------------------ | ------------------------ |
| **去重/合并** | 删除排序数组重复项、合并两个有序数组 | 一指读，一指写           |
| **查找目标**  | 有序数组两数之和、三数之和           | 左右夹逼、控制和的大小   |
| **区间移动**  | 最长子数组和、子字符串问题           | 用 right 扩展，left 收缩 |
| **链表操作**  | 环检测、倒数第 N 个节点              | 快慢指针配合             |

------

## ⚠️ 五、实现时需要注意的关键点

| 问题             | 原因                                 | 对策                                      |
| ---------------- | ------------------------------------ | ----------------------------------------- |
| **边界条件**     | 指针容易越界                         | 判断 `left < right` 或 `i < n`            |
| **指针移动策略** | 移动方向不正确可能死循环             | 明确每步如何前进                          |
| **数据修改顺序** | 一边读一边写可能覆盖数据             | 先处理再移动指针                          |
| **快慢步差异**   | 链表操作要特别注意同步移动           | 如 `fast = fast->next->next` 时需判断非空 |
| **窗口更新逻辑** | 滑动窗口中 left/right 更新条件要明确 | 判断条件与维护逻辑分离                    |

------

## 💡 六、典型示例讲解

### 1️⃣ 对撞指针：两数之和（有序数组）

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    int l = 0, r = nums.size() - 1;
    while (l < r) {
        int sum = nums[l] + nums[r];
        if (sum == target) return {l, r};
        else if (sum < target) l++;
        else r--;
    }
    return {};
}
```

- 时间复杂度：O(n)
- 空间复杂度：O(1)

### 2️⃣ 快慢指针：删除重复元素

```cpp
int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    int slow = 0;
    for (int fast = 1; fast < nums.size(); fast++) {
        if (nums[fast] != nums[slow])
            nums[++slow] = nums[fast];
    }
    return slow + 1;
}
```

### 3️⃣ 滑动窗口：最长无重复子串

```cpp
int lengthOfLongestSubstring(string s) {
    vector<int> freq(128, 0);
    int left = 0, maxlen = 0;
    for (int right = 0; right < s.size(); right++) {
        freq[s[right]]++;
        while (freq[s[right]] > 1)
            freq[s[left++]]--;
        maxlen = max(maxlen, right - left + 1);
    }
    return maxlen;
}
```

------

## 📈 七、总结：双指针思维模式

| 思维角度          | 关键要点                            |
| ----------------- | ----------------------------------- |
| **1. 空间换时间** | 用两个指针同时推进，减少重复遍历    |
| **2. 保持单调性** | 指针移动要遵循“单向”原则，不后退    |
| **3. 边界驱动**   | 算法停止条件由边界或窗口约束决定    |
| **4. 分离职责**   | 一个指针读数据，一个写数据 / 控窗口 |

------

是否希望我帮你生成一份「双指针算法总结 + 题目分类表 + 模板代码」的 **Markdown笔记文件（或Word文档）**？
 可以直接导入你的算法学习笔记中使用。
