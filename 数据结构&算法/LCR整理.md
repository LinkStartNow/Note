---

---

# 仓库管理问题

## LCR库存管理I（2）

> Problem: [LCR 128. 库存管理 I](https://leetcode.cn/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/description/)

### 思路——二分

> 这题有个有序的条件，所以优先选择二分，那么核心问题就是如何判断每次取到的中间节点是在最小值左边还是右边
>
> 我们都知道左侧和右侧都是递增的，而且左侧的值一定都大于等于右侧的值，于是最终答案一定不会大于当前区间的最右侧值，所以我们可以以右边界为基准，
>
> 假如中间值大于了右边界，那么肯定是在答案的右侧，于是mid以及mid左侧都可以放弃了
>
> 假如中间值小于了右边界，说明当前已经在右侧区间中了，而且当前值也可能是答案，于是mid右侧可以放弃，mid仍然保留
>
> 如果相等，那么即使右端是答案，那么mid也可以给出同样的替代方案，于是右端点可以舍弃

### 复杂度分析

- 时间

	总共只进行了一轮二分查找，于是时间复杂度为$O(logn)$

- 空间

	二分查找只依赖于几个临时变量，于是空间复杂度为$O(1)$

### Code

```c++
class Solution {
public:
    int stockManagement(vector<int>& stock) {
        int l = 0, r = stock.size() - 1;
        int m;
        while (l < r) {
            m = l + r >> 1;
            if (stock[m] < stock[r]) r = m;
            else if (stock[m] > stock[r]) l = m + 1;
            else --r;
        }
        return stock[l];
    }
};
```

---

### 第二次出错

> 选取基准值错误，应该选择右端点进行比较，导致编码判断过于复杂出现漏洞

## LCR 库存管理 II

> Problem: [LCR 158. 库存管理 II](https://leetcode.cn/problems/shu-zu-zhong-chu-xian-ci-shu-chao-guo-yi-ban-de-shu-zi-lcof/description/)

### 思路1——hash

> 这题一上来第一个最好想到的肯定是hash计数，直接记录下每个数出现的次数，大于一半就输出就完事了

#### 复杂度分析

- 时间

	时间上只需要遍历一遍数组，为$O(n)$

- 空间

	空间上主要用于hash存数组，为$O(n)$

#### Code

```c++
class Solution {
public:
    int inventoryManagement(vector<int>& stock) {
        int ssr = stock.size() / 2;
        unordered_map<int, int> cnt;
        for (int x: stock) if (++cnt[x] > ssr) return x;
        return 0;
    }
};
```

---

### 思路2——摩尔投票

> 核心思想为正负抵消，我们可以记录一下当前数量最多的数为多少，以及出现了多少次，如果遇到不是该数的数字，则$计数-1$

#### 合理性

> 因为答案肯定存在，则数量最大的数一定数量大于一半，无论怎么抵消，最终一定能存活下来，其他的数就算怎么相互抵消也不会影响最终答案的正确性

#### 复杂度分析

- 时间

	时间上只需要遍历一遍数组，为$O(n)$

- 空间

	空间上只需要记录最大数以及相应的计数，为$O(1)$

#### Code

```c++
class Solution {
public:
    int inventoryManagement(vector<int>& stock) {
        int cnt = 0, ans = 0;
        for (int x: stock) 
            if (x == ans) ++cnt;
            else if (--cnt < 0) cnt = 1, ans = x;
        return ans;
    }
};
```

---

## LCR仓库管理III

> Problem: [LCR 159. 库存管理 III](https://leetcode.cn/problems/zui-xiao-de-kge-shu-lcof/description/)

### 总体思路

> 这题可以根据我们取第K大的经验来做，套路都是一样的方法也是类似，核心就在于排序，和排序时的处理

### 思路1——直接排序

> 这个很好理解，直接将整个数组进行排序，然后取前cnt个即可

#### 复杂度分析

- 时间

	时间主要用在排序上，对于整个数组排序时间复杂度为$O(nlogn)$，所以总的时间复杂度也为$O(nlogn)$

- 空间

	空间也就用在排序上，需要$O(logn)$层的栈空间，所以空间复杂度为$O(logn)$

#### Code

```c++
class Solution {
public:
    vector<int> inventoryManagement(vector<int>& stock, int cnt) {
        sort(stock.begin(), stock.end());
        vector<int> ans;
        for (int i = 0; i < cnt; ++i) ans.emplace_back(stock[i]);
        return ans;
    }
};
```

---

### 思路2——堆排序

> 我们可以维护一个大小为cnt的大顶堆，由于优先队列默认就为大顶堆，于是我们直接用优先队列处理即可
>
> 遍历数组中的每个元素，放入堆中，一旦超过大小则弹出堆顶元素

#### 复杂度分析

- 时间

	时间上的限制在每次需要维护堆，而维护一次的成本为$O(log(cnt))$，每进入一个数字都得维护，所以总的时间复杂度为$O(nlog(cnt))$

- 空间

	空间复杂度主要在于维护这个堆，所以需要$O(cnt)$的空间复杂度

#### Code

```c++
class Solution {
public:
    vector<int> inventoryManagement(vector<int>& stock, int cnt) {
        priority_queue<int> q;
        for (int x: stock) {
            q.emplace(x);
            if (q.size() > cnt) q.pop();
        }
        vector<int> ans;
        while (q.size()) ans.emplace_back(q.top()), q.pop();
        return ans;
    }
};
```

---

### 思路3——快排

> 这个应用和快速选择相似，我们在每次选取的时候可以根据最终分界处的位置选择放弃一边区间不去维护，相比于其他的排序，这个方法在最终并没有实现完全的排序，于是最终的时间复杂度也更优秀
>
> 题目上也没有要求按照什么顺序返回，于是这种方案非常的香

#### 复杂度分析

- 时间

	这里时间复杂度的期望为$O(n)$是目前方案中最快的，证明过程繁琐。但是在最差的情况下死犟复杂度为$O(n^2)$，具体原因可以参照快速排序，每次划分的点都比较差劲

- 空间

	空间上属于原地的操作，没有额外的开销，但是在递归调用函数的时候用到了栈，所以空间复杂度应该为递归的层数，也就是$O(logn)$，这是期望的时间复杂度，然而最坏的情况下分了n次，就需要$O(n)$的空间复杂度

#### Code

```c++
class Solution {
    void yyds(int l, int r, vector<int>& stock, const int& cnt)
    {
        if (l >= r) return;
        int ssr = stock[l];
        int i = l - 1, j = r + 1;
        while (i < j) {
            do ++i; while (stock[i] < ssr);
            do --j; while (stock[j] > ssr);
            if (i < j) swap(stock[i], stock[j]);
        }
        if (j >= cnt) yyds(l, j, stock, cnt);
        else yyds(j + 1, r, stock, cnt);
    }

public:
    vector<int> inventoryManagement(vector<int>& stock, int cnt) {
        yyds(0, stock.size() - 1, stock, cnt);
        while (stock.size() > cnt) stock.pop_back();
        return stock;
    }
};
```

---

# 训练计划

## LCR 139. 训练计划 I

> Problem: [LCR 139. 训练计划 I](https://leetcode.cn/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/description/)

### 思路

> 这题最优的解法就是直接用双指针来解决，一个指针从前走，一个从后走，类似于快排的思路

### 复杂度

- 时间

	两个指针总共只需要将整个数组遍历一遍，于是是$O(n)$

- 空间

	空间上只依赖于两个新指针，空间复杂度为$O(1)$

### Code

```c++
class Solution {
public:
    vector<int> trainingPlan(vector<int>& actions) {
        int i = -1, j = actions.size();
        while (i < j) {
            do ++i; while (i < j && actions[i] % 2);
            do --j; while (i < j && actions[j] % 2 == 0);
            if (i < j) swap(actions[i], actions[j]);
        }
        return actions;
    }
};
```

---

## LCR 140. 训练计划 II

> Problem: [LCR 140. 训练计划 II](https://leetcode.cn/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/description/)

### 思路1——暴力计数

> 这个思路非常暴力，通过遍历一遍链表求得链表的长度$n$，然后跳跃$n - cnt$次返回答案

#### 复杂度

- 时间

	需要将链表遍历一遍测长度，第二次跳跃，最坏情况需要遍历两遍链表，于是是$O(n)$

- 空间

	空间上只有几个临时变量存储信息，空间复杂度为$O(1)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* trainingPlan(ListNode* head, int cnt) {
        ListNode* ssr = head;
        int n = 0;
        while (ssr) ++n, ssr = ssr->next;
        n -= cnt;
        while (n--) head = head->next;
        return head;
    }
};
```

---

### 思路2——栈

> 栈的结构是后进先出，所以我们可以将所有节点压入栈，然后反向弹出

#### 复杂度

- 时间

	需要将链表遍历一遍压栈，再反向弹出，最坏情况需要遍历两遍链表，于是是$O(n)$

- 空间

	栈的空间需要记录整个链表，空间复杂度为$O(n)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* trainingPlan(ListNode* head, int cnt) {
        stack<ListNode*> s;
        while (head) {
            s.emplace(head);
            head = head->next;
        }
        for (int i = 1; i < cnt; ++i) s.pop();
        return s.top();
    }
};
```

---

### 思路3——快慢指针

> 思路三比较巧妙，先让快指针跳cnt次，接着两个指针一起跳，当快指针出去时，慢指针刚好就是答案

#### 复杂度

- 时间

	最差也只需要快指针遍历一遍链表，于是是$O(n)$

- 空间

	空间上只有几个临时变量存储信息，空间复杂度为$O(1)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* trainingPlan(ListNode* head, int cnt) {
        ListNode* ssr = head;
        while (cnt--) ssr = ssr->next;
        while (ssr) head = head->next, ssr = ssr->next;
        return head;
    }
};
```

---

## LCR 141. 训练计划 III

> Problem: [LCR 141. 训练计划 III](https://leetcode.cn/problems/fan-zhuan-lian-biao-lcof/description/)

### 思路1——递归

> 递归可以实现类似栈的功能，对于这种逆序的题，递归在归的过程中改变方向即可

#### 复杂度

- 时间

	遍历到底再返回，于是是$O(n)$

- 空间

	递归的栈需要存储n层，空间复杂度为$O(n)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
    ListNode* ans;
    void yyds(ListNode* p, ListNode* pre)
    {
        if (p) yyds(p->next, p);
        else {
            ans = pre;
            return;
        }
        p->next = pre;
    }
public:
    ListNode* trainningPlan(ListNode* head) {
        yyds(head, nullptr);
        return ans;
    }
};
```

---

### 思路2——快慢指针

> 我们可以用三个指针一前一后做存储，一遍遍历就完事了

#### 复杂度

- 时间

	遍历一遍链表，于是是$O(n)$

- 空间

	只需要存储几个指针，空间复杂度为$O(n)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* trainningPlan(ListNode* head) {
        if (head) {
            ListNode* ssr = head->next;
            ListNode* p = nullptr;
            while (ssr) {
                head->next = p;
                p = head;
                head = ssr;
                ssr = ssr->next;
            }
            head->next = p;
        }
        return head;
    }
};
```

---

## LCR 142. 训练计划 IV

> Problem: [LCR 142. 训练计划 IV](https://leetcode.cn/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/description/)

### 思路

> 定义一个临时的头结点和尾结点，头结点用来返回，尾结点用于处理连接，将两个链表一个个比较即可

### 复杂度

- 时间

	时间上最差只需要完整遍历完两个链表即可，$O(m + n)$

- 空间

	空间上也只需要引入两个指针，$O(1)$

### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* trainningPlan(ListNode* l1, ListNode* l2) {
        ListNode* end, *head;
        if (!l1) return l2;
        if (!l2) return l1;
        if (l1->val < l2->val) head = end = l1, l1 = l1->next;
        else head = end = l2, l2 = l2->next;
        while (l1 && l2) {
            if (l1->val < l2->val) end->next = l1, end = l1, l1 = l1->next;
            else end->next = l2, end = l2, l2 = l2->next;
        }
        end->next = l1 ? l1 : l2;
        return head;
    }
};
```

---

## LCR 171. 训练计划 V

> Problem: [LCR 171. 训练计划 V](https://leetcode.cn/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/description/)

### 思路1——hash

> 我们可以先遍历一遍其中一个表，然后将所有经过的节点记录下来
>
> 然后再遍历第二个链表，一旦遇到记录过的节点那就是第一个相遇点，直接可以返回

#### 复杂度

- 时间

	时间上只需要最多遍历完两个链表，也就是$O(m + n)$

- 空间

	空间上则需要存储其中一个链表，$O(n)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        unordered_set<ListNode*> s;
        while (headA) s.insert(headA), headA = headA->next;
        while (headB) if (s.count(headB)) return headB;
        else headB = headB->next;
        return nullptr;
    }
};
```

---

### 思路2——栈

> 将两个链表遍历完，分别用一个栈来记录，然后一起出栈，最后一个相同的出栈元素就是第一个相遇的点

#### 复杂度

- 时间

	时间上需要遍历完两个链表，然后再弹出元素，也就是$O(m + n + min(m, n))$

- 空间

	空间上需要存储两个栈，$O(n + m)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        stack<ListNode*> s1, s2;
        ListNode* ans = nullptr;
        while (headA) s1.emplace(headA), headA = headA->next;
        while (headB) s2.emplace(headB), headB = headB->next;
        while (s1.size() && s2.size() && s1.top() == s2.top()) ans = s1.top(), s1.pop(), s2.pop();
        return ans;
    }
};
```

---

### 思路3——差值追逐

> 可以先遍历完两个链表，然后使用双指针，根据差值让其中一个长的先跑，最终就能一起到第一个相遇节点

#### 复杂度

- 时间

	时间上需要遍历完两个链表，然后开始跑，也就是$O(m + n + min(m, n))$

- 空间

	空间上只需要几个局部变量，$O(1)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* ssr = headA;
        int cnta = 0, cntb = 0;
        while (ssr) ++cnta, ssr = ssr->next;
        ssr = headB;
        while (ssr) ++cntb, ssr = ssr->next;
        while (cnta < cntb) headB = headB->next, ++cnta;
        while (cntb < cnta) headA = headA->next, ++cntb;
        while (headA && headB && headA != headB) headA = headA->next, headB = headB->next;
        return headA;
    }
};
```

---

### 思路4——浪漫相遇

> 这算是一个非常浪漫的题解了，我们来分析一下原理。
>
> 两个链表的相遇后的部分是一样长的，差在了前面的那段路，但是如果每个点都将自己的路之后走对方的路，那么每个接节点都走了自己的前半部分和对方的前半部分加上相同的相遇部分，所以一定是相等的能同时到达
>
> 如果双方没有相遇节点，那么最终都会完整走完两段路，所以时间一定是一样的，最终都会在null相遇

#### 复杂度

- 时间

	时间上需要遍历完两个链表，然后开始跑，也就是$O(2(m+n))$

- 空间

	空间上只需要几个局部变量，$O(1)$

#### Code

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        ListNode* a = headA;
        ListNode* b = headB;
        while (a != b) {
            a = a ? a->next : headB;
            b = b ? b->next : headA;
        }
        return a;
    }
};
```

---

## LCR 178. 训练计划 VI

> Problem: [LCR 178. 训练计划 VI](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/description/)

### 思路1——hash计数

> 直接用hash把每个数字出现的次数记录下来，再遍历一遍这个hash数组，出现了一次的数字就是答案

#### 复杂度

- 时间

	时间只需要跑完整个数组，然后再跑一遍hash数组即可，为$O(\frac 4 3 n)$

- 空间

	空间上需要记录一个hash数组，$O(\frac 1 3 n)$

#### Code

```c++
class Solution {
public:
    int trainingPlan(vector<int>& actions) {
        unordered_map<int, int> cnt;
        for (int x: actions) ++cnt[x];
        for (const auto& [x, y]: cnt) if (y == 1) return x;
        return 0;
    }
};
```

---

### 思路2——排序

> 我们可以先对数组进行一遍排序，然后由于相同的数字都凑在一起，所以我们判断次数的时候和旁边的比较即可

#### 复杂度

- 时间

	时间主要在排序上，$O(nlogn)$

- 空间

	空间上也是需要排序空间，$O(logn)$

#### Code

```c++
class Solution {
public:
    int trainingPlan(vector<int>& actions) {
        sort(actions.begin(), actions.end());
        for (int i = 0; i < actions.size() - 1; ++i) if (actions[i] == actions[i + 1]) i += 2;
        else return actions[i];
        return actions[actions.size() - 1];
    }
};
```

---

### 方法3——处理位

> 这是基于hash计数的优化，我们其实不用记录下每个数字出现了多少次，只需要记录下每个二进制位出现了多少次，最后对每个位的次数取模3即可，余数也只可能是1或0，所以不用担心

#### 复杂度

- 时间

	时间只需要跑完整个数组，然后对每个数的位的统计最多只有32位，于是时间复杂度为$O(32n)=O(n)$

- 空间

	空间上只需要记录32位，所以为$O(32)=O(1)$

#### Code

```c++
class Solution {
public:
    int trainingPlan(vector<int>& actions) {
        int cnt[32];
        for (int x: actions) {
            int k = 0;
            while (x) cnt[k] = (cnt[k] + (x & 1)) % 3, x >>= 1, ++k;
        }
        int ans = 0;
        for (int i = 31; i >= 0; --i) {
            ans <<= 1;
            ans += cnt[i];
        }
        return ans;
    }
};
```

---

# 数组

## LCR 160. 数据流中的中位数

> Problem: [LCR 160. 数据流中的中位数](https://leetcode.cn/problems/shu-ju-liu-zhong-de-zhong-wei-shu-lcof/description/)

### 思路——对顶堆

> 对顶堆裸的板子题，用一个小堆维护大的半边，用一个大堆维护小的那边，保持他们的大小差不超过1。那么他们的堆顶就是中位数

### Code

```c++
class MedianFinder {
public:
    /** initialize your data structure here. */
    priority_queue<int> big;
    priority_queue<int, vector<int>, greater<int>> small;
    MedianFinder() {

    }
    
    void addNum(int num) {
        if (small.size() && num > small.top()) small.emplace(num);
        else big.emplace(num);
        if (big.size() > small.size() + 1) {
            int t = big.top(); big.pop();
            small.emplace(t);
        }
        if (small.size() > big.size()) {
            int t = small.top(); small.pop();
            big.emplace(t);
        }
    }
    
    double findMedian() {
        if (big.size() == small.size()) return ((double)big.top() + small.top()) / 2;
        return big.top();
    }
};

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder* obj = new MedianFinder();
 * obj->addNum(num);
 * double param_2 = obj->findMedian();
 */
```

---

## LCR 164. 破解闯关密码

> Problem: [LCR 164. 破解闯关密码](https://leetcode.cn/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/description/)

### 思路

> 这题主要在于排序，考虑两个字符串谁应该在前谁应该在后只需要将他们的这两种排列都尝试一下，然后用小的那个就行了
>
> 对于使用C++的，学会用函数`to_string`这题就可以直接秒了，直接将数字转化成`string`然后拼接，没有任何难度

### Code

```c++
class Solution {
public:
    string crackPassword(vector<int>& password) {
        sort(password.begin(), password.end(), [&](int a, int b) {
            string x = to_string(a), y = to_string(b);
            return x + y < y + x;
        });
        string ans = "";
        for (int x: password) ans += to_string(x);
        return ans;
    }
};
```

---

## LCR 168. 丑数

> Problem: [LCR 168. 丑数](https://leetcode.cn/problems/chou-shu-lcof/description/)

### 思路1-暴力推导

> 最早的思路是遍历每一个数，直到丑数的个数达到1690为止，对于每个数字通过判断其有没有2,3,5以外的质因子
>
> 但后来发现，越到后面跨度越大，两个丑数之间可能差了很多，没满1600呢，已经超时了
>
> 所以这是一个废案

### 思路2-set

> 既然无法遍历数字，那么就反着来思考，假如一个数字是丑数，那么说明它只有2,3,5为质因子，假如让这个数除2或3或5，那么商必定也是一个丑数，所以可得，丑数乘2,3,5会是另一个丑数
>
> 于是现在的方案变成了正向通过丑数推未知的丑数，这样数字跨度跳跃就很大了，因为题目给了最大1690，所以取得前1690个丑数即可。
>
> 为了保证数字尽量是有序的，我们在求的过程中直接从已知的丑数从小到大往后推，但是避免不了肯定还会和后面的有重复的，因为$num[i] < num[j] \and i < j$并不能使得$num[i] * 5 < num[j] * 2$
>
> 于是乎我们如果将数字刚好卡1690肯定是会有比较大的误差的，保险起见我开了1800，多开了一百多个数确保前1690在里面
>
> 然后又找了个同时可以解决排序和去重的数据结构，也就是`set`来完成

#### Code

```c++
using ll = long long;

set<ll> ug;
vector<int> ans;

auto yyds = [] () {
    ug.insert(1);
    auto it = ug.begin();
    while (ug.size() < 1800) {
        ll x = *it;
        ug.insert(x * 2);
        ug.insert(x * 3);
        ug.insert(x * 5);
        ++it;
    }
    it = ug.begin();
    for (int i = 0; i < 1690; ++i, ++it) ans.emplace_back(*it); 
    return 0;
}();

class Solution {
public:
    int nthUglyNumber(int n) {
        return ans[n - 1];
    }
};
```

---

### 思路3-动态规划

> 其实对于这种重复的问题我们还有其他的解决方案，我们可以用三个指针将需要乘的质因数分开来思考，定义$dp[i]$为第i个丑数，那么状态转移方程为
> $$
> dp[i] = min(dp[p_x]*x) \and x = 2,3,5
> $$
> 其中$p_x$代表对应乘x的指针指向的下标，含义是使得$dp[j] * x > dp[i - 1]$的最小下标
>
> 初始化：
> $$
> \begin{cases}
> dp[0] = 1; \\
> p_x = 0 & x = 2, 3, 5
> \end{cases}
> $$
> 每次都通过得到的最小的值进行转移，同时将对应相等值的指针往后移动

#### Code

```c++
using ll = long long;

ll ans[1691];

auto yyds = [] () {
    ans[0] = 1;
    int i, j, k;
    i = j = k = 0;
    for (int z = 1; z <= 1690; ++z) {
        ll a = ans[i] * 2, b = ans[j] * 3, c = ans[k] * 5;
        int mi = min({a, b, c});
        ans[z] = mi;
        if (mi == a) ++i;
        if (mi == b) ++j;
        if (mi == c) ++k;
    }
    return 0;
}();

class Solution {
public:
    int nthUglyNumber(int n) {
        return ans[n - 1];
    }
};
```

---

## LCR 170. 交易逆序对的总数

> Problem: [LCR 170. 交易逆序对的总数](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/description/)

### 思路1-归并排序

> 归并排序与逆序对的契合度是相当的高，因为逆序对其实也可以分区间考虑，每次归并排序对于子区间的处理中就可以顺便将子区间的逆序对处理完毕，随着回溯过程的进行，各个大区间之间的逆序对也就处理完成了
>
> 具体如何处理一个区间的逆序对：
>
> 该区间首先会被分为左区间和右区间，在排序时左区间的数会和右区间的数进行比较，我们定义
> $$
> \begin{cases}
> l=左区间左边界 \\
> m=左区间右边界 \\
> m+1=右区间左边界 \\
> r=右区间右边界 \\
> i为左区间某下标，j为右区间某下标 \\
> a为原数组
> \end{cases}
> $$
> 假如此时$a[i] > a[j]$
>
> 此时我们有两种思路：
>
> 1. 当$a[i] > a[j]$去记录，因为$a[t]>a[i] \and i<t<=m$所以$a[t]>a[j] \and i<t<=m$；也就是说，此时$a[j]$对于左区间$[i, m]$的逆序对都有贡献，且也只对这个区间有贡献。于是可以根据这个为依据来记录逆序对的个数。但是这个在记录的时候要注意，别在左区间空的时候加入的右区间值也算进去了，记得要特判的！！！
> 2. 当$a[i] <= a[j]$时去记录，因为$a[j] < a[s] \and j < s <= r$，所以既然$a[i] < a[j]$，那么$a[i] < a[s]$也是理所应当的，也就是说$a[i]$的使命就到这了，在当前区间内的逆序对不会再多了，也就直接将此时右区间的前半部分（$[m + 1, j)$）贡献给$a[i]$。这是不需要加啥特判的，因为$a[i]$是在右侧大了或者右区间空了才计数的，无论怎样，右区间的前半部分就是$a[i]$”合情合理赢来的“，自然是要加贡献的

#### 复杂度

- 时间

	时间复杂度和归并排序一样，总的操作也就多了一个计数，所以时间没啥压力，$O(nlogn)$

- 空间

	空间主要也是归并排序依赖于外部额外的临时数组，于是是$O(n)$

#### Code

> 这里采用了不用特判的第二种

```c++
class Solution {
    int ans = 0;
    vector<int> t;

    void merge(int l, int r, vector<int>& record)
    {
        if (l >= r) return;
        int m = l + r >> 1;
        merge(l, m, record), merge(m + 1, r, record);
        int p = l, q = m + 1, s = l;
        while (p <= m || q <= r) {
            if (q > r || p <= m && record[p] <= record[q]) {
                ans += q - m - 1;
                t[s++] = record[p++];
            } 
            else {
                t[s++] = record[q++];
            }
        }
        for (int i = l; i <= r; ++i) record[i] = t[i];
    }

public:
    int reversePairs(vector<int>& record) {
        int n = record.size();
        t.resize(n);
        merge(0, n - 1, record);
        return ans;
    }
};
```

---

### 思路2-树状数组+离散化

> 每谈到逆序对总是第一时间想到归并排序和树状数组~~某种奇怪的条件反射~~。树状数组也是解决逆序对的一把利器，特别是其中的动态处理非常精妙的
>
> 不了解树状数组的可以去看看~~或者不去看也没事，这玩意找工作几乎用不到~~，这里不具体讲啥是树状数组，只需要知道它非常基础的一个功能是动态的可以求解区间和问题，这题我们可以将逆序对的贡献抽象成区间和问题，所谓逆序对就是前面的值比后面大就是一个逆序对，我们可以顺着处理或者逆着处理，这里以顺着处理为例
>
> 逆序对的贡献从前面的值来，而且是从前面大的值来，所以我们每遍历到一个值，则对于小于该值的所有区间都有贡献，求和的时候就反向求取所有大于该值的个数和，由于都是前面出现的值才进行记录的，所以不太需要考虑逆序对的顺序约束，只需要管大小，然后这里求的和又全是大于该值的，所以这不纯纯就是完美契合嘛
>
> hold on please！你以为结束了？还没完呢，这题直接这么裸着上怎么死的都不知道。这题有个很坑的点，压根没有给数据范围，只给了个数，我们的树状数组实际上也是基于值的区间控制，下标映射全都是跟值有关的，稍微给几个负数，下标都随便越界re吃到饱
>
> 好在数据范围给的还是比较友善的，才1e5，假如我们通过离散化将每个数字再做映射，那么总共的区间也就限制在1e5之内了，最终维护树状数组也不会太大。因为逆序对与值的大小无关，所以随便离散化。
>
> 哈哈哈这就完事了？能过了？不能！还有些细节要注意，`lowbit`最怕的就是0，因此树状数组最怕的也是0，所以即使离散化了也要稍微留点余量，多开几个数，不然出个0就等着原地死循环一直t吧哈哈哈

#### 复杂度

- 时间

	离散化复杂度主要在排序上，也就是$O(nlogn)$，而树状数组的维护和查询均为$log$级别的总共处理完n个数也就$O(nlogn)$，所以总的时间复杂度为$O(nlogn)$

- 空间

	空间上不仅离散化需要克隆一个原数组，对于后来维护的树状数组同样范围也是那么大，去重操作去掉的几个就忽略不计了，所以空间复杂度为$O(n)$

#### Code

```c++
class Solution {
    vector<int> t;
    int n;

    inline int lowbit(int x)
    {
        return x & -x;
    }

    void add(int x)
    {
        for (; x; x -= lowbit(x)) ++t[x];
    }

    int query(int x)
    {
        int res = 0;
        for (; x <= n; x += lowbit(x)) res += t[x];
        return res;
    }

public:
    int reversePairs(vector<int>& record) {
        int ans = 0;

        // 离散化
        vector<int> ssr(record);
        sort(ssr.begin(), ssr.end());
        n = unique(ssr.begin(), ssr.end()) - ssr.begin();

        auto find = [&] (int x) {
            int l = 0, r = n - 1;
            while (l <= r) {
                int m = l + r >> 1;
                if (ssr[m] == x) return m + 2;
                else if (ssr[m] < x) l = m + 1;
                else r = m - 1;
            }
            return 233;
        };

        t.resize(n + 2);
        for (int i = 0; i < record.size(); ++i) ans += query(find(record[i])), add(find(record[i]) - 1);
        return ans;
    }
};
```

---

## LCR 172. 统计目标成绩的出现次数

> Problem: [LCR 172. 统计目标成绩的出现次数](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/description/)

### 思路1——暴力

> 直接将整个数组遍历一遍，碰到target就计数，无脑暴力完事

#### 复杂度

- 时间

	遍历一遍数组的时间，$O(n)$

- 空间

	就记录一个答案，$O(1)$

#### Code

```c++
class Solution {
public:
    int countTarget(vector<int>& scores, int target) {
        int cnt = 0;
        for (int x: scores) cnt += x == target;
        return cnt;
    }
};
```

---

### 思路2——二分

> 由于题目给的是有序的数组，然而我们并没有用上这个重要的条件，即使是无序数组暴力也能跑，但是有序数组更优秀的查找应该是二分，我们分别通过二分找到target第一次出现的位置和最后一次出现的位置，就能马上算出数量

#### 复杂度

- 时间

	只需要进行两次二分，和最后一次算答案，所以时间复杂度为$O(logn)$

- 空间

	只用到几个临时变量，$O(1)$

#### Code

```c++
class Solution {
public:
    int countTarget(vector<int>& scores, int target) {
        int n = scores.size();
        int l = 0, r = n - 1;
        while (l <= r) {
            int m = l + r >> 1;
            if (scores[m] >= target) r = m - 1;
            else l = m + 1;
        }
        if (l >= n || scores[l] != target) return 0;
        int first = l;
        l = 0, r = n - 1;
        while (l <= r) {
            int m = l + r >> 1;
            if (scores[m] <= target) l = m + 1;
            else r = m - 1;
        }
        return r - first + 1;
    }
};
```

---

#### 范围优化

> 实际上在我们算出左边界或右边界后，第二次求另一个边界我们不需要将整个区间都用进来，
>
> 假如我们已经有了左边界，那么说明该下标就是第一个target，左边不可能有其他target，可以直接舍弃l的左半部分

##### 复杂度

- 时间

	同普通二分，$O(logn)$

- 空间

	同普通二分，$(1)$

##### Code

```c++
class Solution {
public:
    int countTarget(vector<int>& scores, int target) {
        int n = scores.size();
        int l = 0, r = n - 1;
        while (l <= r) {
            int m = l + r >> 1;
            if (scores[m] >= target) r = m - 1;
            else l = m + 1;
        }
        if (l >= n || scores[l] != target) return 0;
        int first = l;
        r = n - 1;
        while (l <= r) {
            int m = l + r >> 1;
            if (scores[m] <= target) l = m + 1;
            else r = m - 1;
        }
        return r - first + 1;
    }
};
```

---

## LCR 173. 点名

> Problem: [LCR 173. 点名](https://leetcode.cn/problems/que-shi-de-shu-zi-lcof/description/)

### 思路1——暴力

> 因为数组是有序的从0开始的，则我们直接暴力跑一遍数组，找到第一个不等于自己下标的数就行

#### 复杂度

- 时间

	遍历一遍数组的时间，$O(n)$

- 空间

	就记录一个答案，$O(1)$

#### Code

```c++
class Solution {
public:
    int takeAttendance(vector<int>& records) {
        for (int i = 0; i < records.size(); ++i) if (records[i] != i) return i;
        return records.size();
    }
};
```

---

### 思路2——二分

> 其实题目给的时候说是有序就得有个二分的考虑了，但是难就难在如何去把这题抽象成一个二分，我们又不知道最终的具体目标是啥，咋去找这个目标呢？这就是对二分理解不到位的地方了
>
> 二分并不单单可以从某个有序数组中寻找到某个值，为啥要求这个数组是有序的？二分的本质又是啥？
>
> 实际上二分依赖于单调性，也就是说在某个范围中有个中间的分界点，在这个分界点左侧都是能满足（或不满足），在右侧都是不满足（或满足），这样每次通过二分查询中间值一下子就能判断出当前处于左半边还是右半边，进而缩小一半的区间，最终缩减到很小的范围答案也就出来了。我们原先查找某个值也就是这个原理，如果数组有序的情况下，那么要查找的值左侧都小于该值，右侧都大于该值，于是也很好判断是在左区间还是右区间
>
> 所以我们只需要去考虑这题到底满足不满足单调就行了，这也是我之前疏忽的一个点，暴力的时候只意识到了碰到的第一个下标不等于值的一定是缺项，但其实在它之后的位置也由于这一个缺失而都与自身值不等。于是这个单调性就出来了，在缺失点左侧下标都等于值，而从缺失位置开始后面的下标都不等于值，然后就开始愉快二分吧！

#### 复杂度

- 时间

	只进行了一遍二分查找，$O(logn)$

- 空间

	只用到几个临时变量，$O(1)$

#### Code

```c++
class Solution {
public:
    int takeAttendance(vector<int>& records) {
        int l = 0, r = records.size() - 1;
        while (l <= r) {
            int m = l + r >> 1;
            if (records[m] == m) l = m + 1;
            else r = m - 1;
        }
        return l;
    }
};
```

---

## LCR 179. 查找总价格为目标值的两个商品

> Problem: [LCR 179. 查找总价格为目标值的两个商品](https://leetcode.cn/problems/he-wei-sde-liang-ge-shu-zi-lcof/description/)

### 思路1——暴力

> 一个数组中找俩数和为target，题意非常简单，我们可以上来就遍历每个数，然后为它去匹配另一个数，为了避免重复计算，我们规定我们遍历的第一个数为两个数中下标小的数
>
> 但是很遗憾，这题数组长度为1e5，这种暴力跑两层的$O(n^2)$时间复杂度肯定是会T的

#### 复杂度

- 时间

	最多要遍历n个数，然后每次遍历判断的时间复杂度为$O(n)$，所以总的时间复杂度为$O(n^2)$

- 空间

	空间上只有常数个变量的额外消耗，所以为$O(1)$

---

### 思路2——hash

> 我们参考一下暴力的思路，发现先找到其中一个数肯定是没问题的，但是根据这个数寻找第二个数的复杂度太高了，我们可以通过hash将所有数字有没有出现过都记录下来，那么我们每次判断的时间就只有$O(1)$了，能快一大截

#### 复杂度

- 时间

	时间上预处理hash数组需要跑一遍数组，为$O(n)$，之后判断n个数，每次只需要$O(1)$，所以总共的时间复杂度为$O(n)$（严格来讲是$O(2n)$）

- 空间

	然鹅，空间上就需要多存一个hash数组了，复杂度为$O(n)$

#### Code

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& price, int target) {
        unordered_map<int, bool> mp;
        for (int x: price) mp[x] = true;
        for (int x: price) if (mp.count(target - x)) return {x, target - x};
        return {};
    }
};
```

---

### 思路3——二分

> 怎么说来着，数组有序，优先考虑二分，这题我们显然没用上这个条件，hash这种流氓做法啥情况都可以做，哪用得着数组有序捏
>
> 我们可以通过二分来优化判断的时间，找到第一个数后，通过二分在剩余区间中寻找第二个数

#### 复杂度

- 时间

	每次判断复杂度为$O(logn)$，所以总的时间复杂度为$O(nlogn)$

- 空间

	空间上也没开啥数组，为$O(1)$

#### Code

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& price, int target) {
        int n = price.size();
        for (int i = 0; i < n; ++i) {
            int k = target - price[i];
            int l = i + 1, r = n - 1;
            while (l <= r) {
                int m = l + r >> 1;
                if (price[m] == k) return {price[i], price[m]};
                else if (price[m] > k) r = m - 1;
                else l = m + 1;
            }
        }
        return {};
    }
};
```

---

### 思路4——双指针

> 双指针一来这题基本就是开始秀操作了，而且复杂度基本都是爆炸的几乎是极致的复杂度
>
> 这题很亲切的给了个有序条件，那么双指针也非常适合处理这题，让两个指针i、j分别指向首尾，如果和小了就i向后移，大了就j向前移，思路也非常清晰，而且不会有任何遗漏

#### 复杂度

- 时间

	时间上最多也就遍历了整个数组，为$O(n)$

- 空间

	空间上没有啥额外数组，所以为$O(1)$

#### Code

```c++
class Solution {
public:
    vector<int> twoSum(vector<int>& price, int target) {
        int i = 0, j = price.size() - 1;
        while (i < j) {
            int k = target - price[j];
            while (i < j && price[i] < k) ++i;
            if (price[i] == k) return {price[i], price[j]};
            k = target - price[i];
            while (i < j && price[j] > k) --j;
        }
        return {};
    }
};
```

---

#### 二分优化

> 还有高手？是的！这位可能是集大成者了，双指针和二分的强强联合
>
> 这题不是给了有序条件嘛，虽然双指针用上了有序条件，但是可以再考虑一下二分，我们每次指针移动其实不需要一格一格移动，可以直接利用二分实现跳转，优化指针移动的效率

##### 复杂度

- 时间

	时间上其实并不一定能看出明显减少，但是跳转过程通过二分优化一定是加快了的，所以复杂度小于$O(n)$

- 空间

	空间上也没啥压力，依旧是$O(1)$

##### Code

```c++
class Solution {
    int find(vector<int>& price, int l, int r, int target, bool left)
    {
        while (l <= r) {
            int m = l + r >> 1;
            if (price[m] == target) return m;
            else if (price[m] > target) r = m - 1;
            else l = m + 1;
        }
        return left ? l : r;
    }
public:
    vector<int> twoSum(vector<int>& price, int target) {
        int i = 0, j = price.size() - 1;
        while (i < j) {
            int k = target - price[j];
            i = find(price, i, j - 1, k, true);
            if (price[i] == k) return {price[i], price[j]};
            k = target - price[i];
            j = find(price, i + 1, j, k, false);
        }
        return {};
    }
};
```

---

## LCR 183. 望远镜中最高的海拔

> Problem: [LCR 183. 望远镜中最高的海拔](https://leetcode.cn/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/description/)

### 大体方向

> 这题就是个滑动窗口求最值的问题，问题就在于我们如何求出指定区间内的最大值

### 思路1——暴力

> 可以通过暴力遍历整个区间的方式来求出某个区间内的最值，由于本体给的数据范围其实没那么大，暴力勉强还是能跑过的

#### 复杂度

- 时间

	时间上有n个区间，每次遍历的长度为limit，所以时间复杂度为$O(num*limit)$，$num = n - limit$，所以最坏的情况下就是$n^2$级别的复杂度

- 空间

	空间上没有依赖于额外空间，为$O(1)$

#### Code

```c++
class Solution {
public:
    vector<int> maxAltitude(vector<int>& heights, int limit) {
        vector<int> ans;
        for (int i = limit - 1; i < heights.size(); ++i) {
            int ma = -10005;
            for (int j = i; j >= i - limit + 1; --j) ma = max(ma, heights[j]);
            ans.emplace_back(ma);
        }
        return ans;
    }
};
```

---

### 思路2——单调栈

> 这题的最优解就是单调栈，一般题目用上单调栈了就非常的巧妙，而且复杂度都巨低
>
> 我们可以通过维护一个区间范围为`limit`的单调栈，每次有新元素进来了就将前面小于它的元素给弹出，将其放在尾部，因为前面的序号比它小，肯定比它被抛弃，而且当前区间考虑的情况下由于值又没有新元素大所以前面的小值也不可能成为答案，所以直接抛弃即可
>
> 对于每轮被滑动窗口移除的元素，判断一下是不是队首元素，如果是，那么弹出队首，因为区间已经无效了，这个值再大也没用了，由于维护的是一个单调栈，所以它之后的一个元素就是当前区间最大的

#### 复杂度

- 时间

	时间上只遍历了一遍数组，以及一些入队出队操作，所以复杂度为$O(n)$

- 空间

	空间上最大的限制就是单调栈的空间，为$O(limit)$

#### Code

```c++
class Solution {
public:
    vector<int> maxAltitude(vector<int>& heights, int limit) {
        deque<int> dq;
        int i = 0, n = heights.size();
        for (; i < limit - 1; ++i) {
            while (dq.size() && dq.back() < heights[i]) dq.pop_back();
            dq.emplace_back(heights[i]);
        }
        vector<int> ans;
        for (; i < n; ++i) {
            while (dq.size() && dq.back() < heights[i]) dq.pop_back();
            dq.emplace_back(heights[i]);
            ans.emplace_back(dq.front());
            if (dq.front() == heights[i - limit + 1]) dq.pop_front();
        }
        return ans;
    }
};
```

---

### 思路3——st表

> 依赖于数据结构st表我们也能实现$O(1)$复杂度的区间最值查询，只不过需要进行预处理

#### 复杂度

- 时间

	时间主要在预处理上，需要$O(nlogn)$的时间复杂度

- 空间

	空间上需要创建st表辅助数组，所以为$O(nlogn)$

#### Code

```c++
class Solution {
    int n;
    void init(vector<vector<int>>& st, const vector<int>& heights, int size)
    {
        for (int i = 0; i < n; ++i) st[i][0] = heights[i];
        int k = 2;
        for (int t = 1; t < size; ++t) {
            for (int i = 0; i < n - k + 1; ++i) {
                st[i][t] = max(st[i][t - 1], st[i + (k >> 1)][t - 1]);
            }
            k <<= 1;
        }
    }

    int find(const vector<vector<int>>& st, int l, int r)
    {
        int size = log(r - l + 1) / log(2);
        return max(st[l][size], st[r - (1 << size) + 1][size]);
    }
public:
    vector<int> maxAltitude(vector<int>& heights, int limit) {
        if (!heights.size()) return {};
        n = heights.size();
        int size = log(limit) / log(2) + 1;
        vector<vector<int>> st(n, vector<int>(size, -10005));
        init(st, heights, size);
        vector<int> ans;
        for (int i = 0; i < n - limit + 1; ++i) ans.emplace_back(find(st, i, i + limit - 1));
        return ans;
    }
};
```

---

### 思路4——树状数组

> 树状数组也可以用来快速获取区间最值，只不过效率相对不太理想，想要稳定一点可以考虑线段树
>
> 使用树状数组的时候一定要注意0！

#### 复杂度

- 时间

	时间上树状数组每次查询一个区间的最值复杂度为$O((logn)^2)$，构造的时候每次添加需要$O(logn)$所以总的时间复杂度为$O(n(logn)^2)$

- 空间

	空间上需要存储对应的树状数组，复杂度为$O(k)$，k为取值范围

#### Code

```c++
class Solution {
    int n;

    inline int lowbit(int x)
    {
        return x & -x;
    }

    void add(int x, int i, vector<int>& t)
    {
        for (; i <= n; i += lowbit(i))  t[i] = max(t[i], x);
    }

    int query(int l, int r, const vector<int>& t, const vector<int>& heights)
    {
        if (l == r) return heights[l - 1];
        if (r - lowbit(r) + 1 > l) return max(t[r], query(l, r - lowbit(r), t, heights));
        else if (r - lowbit(r) + 1 == l) return t[r];
        return max(heights[r - 1], query(l, r - 1, t, heights));
    }

public:
    vector<int> maxAltitude(vector<int>& heights, int limit) {
        if (!heights.size()) return {};
        n = heights.size();
        vector<int> t(n + 1, -10003);
        for (int i = 0; i < n; ++i) add(heights[i], i + 1, t);
        vector<int> ans;
        for (int i = 0; i < n - limit + 1; ++i) 
            ans.emplace_back(query(i + 1, i + limit, t, heights));
        return ans;
    }
};
```

---

## LCR 189. 设计机械累加器

> Problem: [LCR 189. 设计机械累加器](https://leetcode.cn/problems/qiu-12n-lcof/description/)

### 大体思路

> 这题不让用循环，那么咱可以递归来代替，但是无法用if做判断，那么可以思考一下如何代替if

### 思路1——三目运算

> 这个有点牵强，但总归是没用if吧

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int target) {
        return target ? target + mechanicalAccumulator(target - 1) : 0;
    }
};
```

---

### 思路2——逻辑短路

> 许多语言的逻辑表达式都是支持短路功能的，我们可以通过短路来模拟if判断

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int target) {
        target > 1 && (target += mechanicalAccumulator(target - 1));
        return target; 
    }
};
```

---

### 思路3——快速乘

> 这个思路类似于快速幂的思路，将乘数看成二进制位来进行拆分，比如：
> $$
> \begin{aligned}
> 5 *11 &= 5 * (1 + 2 + 8) \\
> &= 5 * 1 + 5 * 2 + 5 * 8
> \end{aligned}
> $$
> 执行的时候按位运算操作模拟即可，层数很明显是可以看到尽头的，因为根据高斯求和，最终的乘数也就是$n$或者$n + 1$，而n不超过1e5，于是大约$2^{14}$~~手动模拟14次for递推不就彳亍了嘛~~
>
> 但是我是手残，我依旧选择递归哈哈哈

#### Code

```c++
class Solution {
    int ans = 0;
    int dfs(int a, int b)
    {
        (b & 1) && (ans += a);
        b && dfs(a << 1, b >> 1);
        return 0;
    }
public:
    int mechanicalAccumulator(int target) {
        dfs(1 + target, target);
        return ans >> 1;
    }
};
```

---

### 思路4——大小取巧

> 接下来的思路就比较抽象了，比如可以利用二维数组本身的大小来代替乘法的运算

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int n) {
        char ssr[n][n + 1];
        return sizeof(ssr) >> 1;
    }
};
```

---

### 思路5——文字游戏

> 说了不让用乘法就不用了么，好像没说不能直接越级用幂吧哈哈哈
>
> 根据高斯求和，可知：
> $$
> \begin{aligned}
> ans &= \frac {(1 + n) * n} 2 
> \end{aligned}
> $$
> 我们最后再去求这个除2哈~~不能用除就位运算嘿嘿~~，则：
> $$
> \begin{aligned}
> ssr &= 2 * ans \\
> &= (1 + n) * n \\
> &= n^2 + n
> \end{aligned}
> $$
> 那还说啥了，直接上幂吧就

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int n) {
        return (int)pow(n, 2) + n >> 1;
    }
};
```

---

## LCR 191. 按规则计算统计结果

> Problem: [LCR 191. 按规则计算统计结果](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/description/)

### 思路1——暴力

> 题意就是很直接，求剩余部分的积，那么其实暴力跑一遍就能求出剩下数的积是多少，but数据量太大了，很明显会超时，所以这个方案显然不行

#### 复杂度

- 时间

	时间上每求一个答案需要跑一遍$O(n)$，总共有n个答案要求，所以是$O(n^2)$

- 空间

	没有额外空间消耗，$O(1)$

---

### 思路2——除法

> 我们可以事先将所有数的积求出来，然后对于每个答案挨个去抛去对应位置上的数就行，用除法除一下就行，但是这题有个巨坑，有数据0！！！所以这种方案也pass了，因为一旦出现0，那么除法将会error，而且做些特判也非常的没必要，所以放弃该方案

#### 复杂度

- 时间

	先跑一遍求出总积，然后遍历一遍出答案，$O(n)$

- 空间

	记录一下总积就行，$O(1)$

---

### 思路3——前后缀分离

> 我们可以这么来思考问题：比如我们要求第i个答案$ans[i]$，就是要求除了$a[i]$以外元素的乘积
>
> 可以将这个乘积转换为前半部分和后半部分，他们分别都是连续的区间，我们可以根据前缀和的思想来预处理出前缀积和后缀积，预处理的过程也只需要遍历两遍数组
>
> 然后求答案的时候只需要一次遍历就行了

#### 复杂度

- 时间

	总共只遍历了三遍，所以时间复杂度为$O(n)$

- 空间

	空间上需要创建两个辅助数组，所以需要$O(n)$

#### Code

```c++
class Solution {
public:
    vector<int> statisticalResult(vector<int>& arrayA) {
        int n = arrayA.size();
        if (!n) return {};
        vector<int> suf(n, 1), pre(n, 1);
        suf[n - 1] = arrayA[n - 1];
        pre[0] = arrayA[0];
        for (int i = n - 2; i >= 0; --i) suf[i] = suf[i + 1] * arrayA[i];
        for (int i = 1; i < n; ++i) pre[i] = pre[i - 1] * arrayA[i];
        vector<int> ans;
        ans.emplace_back(suf[1]);
        for (int i = 1; i < n - 1; ++i) ans.emplace_back(pre[i - 1] * suf[i + 1]);
        ans.emplace_back(pre[n - 2]);
        return ans;
    }
};
```

---

#### 优化——单个数组

> 其实前缀积和后缀积只需要求一个就够了，另一个可以直接维护一个动态的变量，一遍遍历一边添加答案就行

##### 复杂度

- 时间

	时间上可以少遍历一轮，但是也还是$O(n)$

- 空间

	空间上少了一个数组，但是还是$O(n)$

##### Code

```c++
class Solution {
public:
    vector<int> statisticalResult(vector<int>& arrayA) {
        int n = arrayA.size();
        if (!n) return {};
        vector<int> suf(n, 1);
        suf[n - 1] = arrayA[n - 1];
        for (int i = n - 2; i >= 0; --i) suf[i] = suf[i + 1] * arrayA[i];
        vector<int> ans;
        ans.emplace_back(suf[1]);
        int pre = arrayA[0];
        for (int i = 1; i < n - 1; ++i) ans.emplace_back(pre * suf[i + 1]), pre *= arrayA[i];
        ans.emplace_back(pre);
        return ans;
    }
};
```

---

#### 再优化——原地返回

> 其实用来记录的前缀积或者后缀积数组是可以用来作为返回答案的，于是这个空间也就算不需要了

##### 复杂度

- 时间

	时间上没有变化，还是$O(n)$

- 空间

	空间上辅助数组和答案是同一个数组，可以认为没有额外空间消耗，所以是$O(1)$

##### Code

```c++
class Solution {
public:
    vector<int> statisticalResult(vector<int>& arrayA) {
        int n = arrayA.size();
        if (!n) return {};
        vector<int> suf(n, 1);
        suf[n - 1] = arrayA[n - 1];
        for (int i = n - 2; i; --i) suf[i] = suf[i + 1] * arrayA[i];
        int pre = 1;
        for (int i = 0; i < n - 1; ++i) suf[i] = pre * suf[i + 1], pre *= arrayA[i];
        suf[n - 1] = pre;
        return suf;
    }
};
```

---

### 思路4——对角线分割

> 我们要求这个积可以通过画图来分析，也是基于前后缀分离的思想，先处理前缀的积，再处理后缀的积，如下图所示：
>
> ![image-20240712132131760](/0.png)
>
> 我们可以用一个变量动态的求出每个答案的前缀积答案，然后再反向遍历一遍求出后缀积答案并直接写入答案

#### 复杂度

- 时间

	时间上遍历两遍数组，所以为$O(n)$

- 空间

	空间上无需额外空间（除了返回答案需要的空间），所以为$O(1)$

#### Code

```c++
class Solution {
public:
    vector<int> statisticalResult(vector<int>& arrayA) {
        int n = arrayA.size();
        if (!n) return {};
        vector<int> ans(n, 1);
        for (int i = 1; i < n; ++i) ans[i] = ans[i - 1] * arrayA[i - 1];
        int suf = arrayA[n - 1];
        for (int i = n - 2; i >= 0; --i) ans[i] *= suf, suf *= arrayA[i];
        return ans; 
    }
};
```

---

# 回溯

## LCR 085. 括号生成

> Problem: [LCR 085. 括号生成](https://leetcode.cn/problems/IDBivT/description/)

### 思路——剪枝+回溯
