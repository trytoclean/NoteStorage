好的，直接上干货。下面分别给出三题的：**题目要点 → 实现步骤 → C++参考解法 → 复杂度与细节分析**。代码均可在 LeetCode 环境直接提交（已包含必要的结构体与函数签名）。

------

# 1) 链表排序 — LeetCode 148: Sort List

## 题目要点

给定单链表头节点 `head`，在 `O(n log n)` 时间内、且尽可能少的额外空间对链表排序。

## 实现步骤（自顶向下归并）

1. **边界**：空或单节点直接返回。
2. **快慢指针找中点**：`slow/fast` 找到中点，断开成左右两段。
3. **递归排序左右段**：`left = sortList(head)`，`right = sortList(midNext)`。
4. **合并两个有序链表**：用哑结点 `dummy` 线性合并。

> 为避免栈外空间，链表归并排序“合并”可 O(1) 额外空间（只改指针），整体额外空间为递归栈 `O(log n)`。

## C++ 代码

```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(): val(0), next(nullptr) {}
    ListNode(int x): val(x), next(nullptr) {}
    ListNode(int x, ListNode *n): val(x), next(n) {}
};

class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head || !head->next) return head;

        // 1) find middle (end of left)
        ListNode* slow = head;
        ListNode* fast = head->next; // fast 先走，保证 slow 指向左半段末尾
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        ListNode* midNext = slow->next;
        slow->next = nullptr;

        // 2) sort halves
        ListNode* left = sortList(head);
        ListNode* right = sortList(midNext);

        // 3) merge two sorted lists
        return mergeTwoLists(left, right);
    }

private:
    static ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
        ListNode dummy;
        ListNode* tail = &dummy;
        while (a && b) {
            if (a->val <= b->val) {
                tail->next = a; a = a->next;
            } else {
                tail->next = b; b = b->next;
            }
            tail = tail->next;
        }
        tail->next = a ? a : b;
        return dummy.next;
    }
};
```

## 分析

- **时间复杂度**：`O(n log n)`（分治深度 `log n`，每层线性合并）
- **空间复杂度**：`O(log n)`（递归栈）
- **细节/坑**：
  - 快慢指针初始化 `fast = head->next` 可避免无限循环，并令左半段与右半段规模更均衡。
  - 记得**断链**：`slow->next = nullptr`。
  - 归并时使用“`<=`”保证稳定性（虽然题目不要求稳定，但好习惯）。

------

# 2) 逆序对统计 — LeetCode 493: Reverse Pairs

## 题目要点

统计数组中满足 `i < j` 且 `nums[i] > 2 * nums[j]` 的对数。

## 实现步骤（归并 + 计数）

1. **分**：递归将数组二分。
2. **治**：分别统计左/右子数组的逆序对数。
3. **合并前计数**：在合并两个有序子数组前，用双指针 `i` 扫左段、`j` 扫右段：
   - 对每个 `i`，向右移动 `j`，直到 `nums[i] <= 2*nums[j]` 不再成立，此时右段中 `[orig_j, j)` 的元素都满足条件，累加到答案。
4. **合并**：按普通归并排序合并两段为有序。

> 关键：**计数发生在“跨段对”且在合并前**，利用两边已排序性质用线性指针统计。

## C++ 代码

```cpp
class Solution {
public:
    int reversePairs(vector<int>& nums) {
        if (nums.size() < 2) return 0;
        vector<int> tmp(nums.size());
        return (int)mergeSortCount(nums, tmp, 0, (int)nums.size() - 1);
    }

private:
    long long mergeSortCount(vector<int>& a, vector<int>& tmp, int l, int r) {
        if (l >= r) return 0;
        int m = l + (r - l) / 2;
        long long ans = 0;
        ans += mergeSortCount(a, tmp, l, m);
        ans += mergeSortCount(a, tmp, m + 1, r);

        // 计数：a[l..m], a[m+1..r] 已各自有序
        int j = m + 1;
        for (int i = l; i <= m; ++i) {
            while (j <= r && (long long)a[i] > 2LL * (long long)a[j]) ++j;
            ans += (j - (m + 1));
        }

        // 合并
        int i = l; j = m + 1; 
        int k = l;
        while (i <= m && j <= r) {
            if (a[i] <= a[j]) tmp[k++] = a[i++];
            else tmp[k++] = a[j++];
        }
        while (i <= m) tmp[k++] = a[i++];
        while (j <= r) tmp[k++] = a[j++];
        for (int t = l; t <= r; ++t) a[t] = tmp[t];

        return ans;
    }
};
```

## 分析

- **时间复杂度**：`O(n log n)`（每层线性计数 + 线性合并）
- **空间复杂度**：`O(n)`（辅助数组）+ 递归栈 `O(log n)`
- **细节/坑**：
  - **溢出**：比较 `a[i] > 2 * a[j]` 必须使用 64 位：`(long long)a[i] > 2LL * a[j]`。
  - **计数顺序**：先计数再合并，确保两段有序以便双指针线性推进。
  - **严格大于**：注意题意是 `>` 而非 `>=`。

------

# 3) K 路合并有序链表 — LeetCode 23: Merge k Sorted Lists

## 题目要点

给定 `k` 个升序链表，合并为一个升序链表。

## 解法 A（推荐）：最小堆（优先队列）

### 实现步骤

1. 将每个链表的首节点入小根堆（按 `val`）。
2. 循环：每次取出堆顶最小节点接到答案尾部，并将该节点的 `next` 入堆。
3. 直至堆空。

### C++ 代码

```cpp
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(): val(0), next(nullptr) {}
    ListNode(int x): val(x), next(nullptr) {}
    ListNode(int x, ListNode *n): val(x), next(n) {}
};

class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        struct Cmp {
            bool operator()(ListNode* a, ListNode* b) const {
                return a->val > b->val; // 小根堆
            }
        };
        priority_queue<ListNode*, vector<ListNode*>, Cmp> pq;
        for (auto* node : lists) if (node) pq.push(node);

        ListNode dummy, *tail = &dummy;
        while (!pq.empty()) {
            auto* cur = pq.top(); pq.pop();
            tail->next = cur;
            tail = tail->next;
            if (cur->next) pq.push(cur->next);
        }
        return dummy.next;
    }
};
```

### 分析

- **时间复杂度**：`O(N log k)`，`N` 为总节点数；每个节点入/出堆各一次。
- **空间复杂度**：`O(k)`（堆）

## 解法 B：分治两两合并（归并思想）

### 实现步骤

1. 令步长 `step=1`，两两合并 `lists[i]` 与 `lists[i+step]`。
2. `step *= 2`，重复直到 `step >= k`。
3. 合并两个有序链表复用 148 题的 `mergeTwoLists`。

### C++ 代码（分治）

```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if (lists.empty()) return nullptr;
        int n = (int)lists.size();
        int step = 1;
        while (step < n) {
            for (int i = 0; i + step < n; i += 2 * step) {
                lists[i] = mergeTwoLists(lists[i], lists[i + step]);
            }
            step <<= 1;
        }
        return lists[0];
    }
private:
    static ListNode* mergeTwoLists(ListNode* a, ListNode* b) {
        ListNode dummy; ListNode* t = &dummy;
        while (a && b) {
            if (a->val <= b->val) { t->next = a; a = a->next; }
            else { t->next = b; b = b->next; }
            t = t->next;
        }
        t->next = a ? a : b;
        return dummy.next;
    }
};
```

### 分析

- **时间复杂度**：`O(N log k)`（每一层线性合并，总层数 `log k`）
- **空间复杂度**：`O(1)` 额外空间（迭代分治，不用递归栈）

------

## 小结与对比

| 题目         | 核心思想                | 关键技巧                                    | 复杂度                             |
| ------------ | ----------------------- | ------------------------------------------- | ---------------------------------- |
| 148 链表排序 | 归并排序（分治 + 合并） | 快慢指针找中点 + O(1) 指针合并              | 时间 `O(n log n)`，空间 `O(log n)` |
| 493 逆序对   | 归并计数                | 合并前用双指针统计跨段对 + 64 位比较        | 时间 `O(n log n)`，空间 `O(n)`     |
| 23 K 路合并  | 堆/分治                 | 最小堆 `O(N log k)` 或两两归并 `O(N log k)` | 空间：堆 `O(k)` / 分治 `O(1)`      |

如果你愿意，我可以把这三题的**单元测试脚手架**（本地 `main()` + 随机用例 + 断言）也补上，方便你一次性在本地跑通与对拍。你更偏好**最小堆版**还是**分治版**的第 23 题？我就按你的偏好生成可直接编译运行的完整模板。