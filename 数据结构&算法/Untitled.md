## 例题——柱状图中的矩形

> Problem: [84. 柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/description/)

### 思路——前后缀分离+单调栈

> 我们想同时考虑两边，要枚举的情况实在太多，但是如果我们将前缀和后缀分开考虑，那么就会简单很多，我们可以通过单调栈在一遍遍历下求出以某个柱子为最低而达到的最左边界，同样倒着遍历一遍就能处理完后缀，将两者结合分开考虑遍历一遍数组答案就出来了

#### Code

```c++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        vector<int> l(n, -1), r(n, n);
        stack<int> s;
        for (int i = 0; i < n; ++i) {
            while (s.size() && heights[i] <= heights[s.top()]) s.pop();
            if (s.size()) l[i] = s.top();
            s.push(i);
        }
        while (s.size()) s.pop();
        for (int i = n - 1; i >= 0; --i) {
            while (s.size() && heights[i] <= heights[s.top()]) s.pop();
            if (s.size()) r[i] = s.top();
            s.push(i);
        }
        int ma = 0;
        for (int i = 0; i < n; ++i) {
            int h = heights[i], le = l[i] + 1, re = r[i] - 1;
            ma = max(ma, h * (re - le + 1));
        }
        return ma;
    }
};
```

---

### 优化——单次遍历

> 其实，我们在用单调栈入栈寻求左边界的过程中，那些弹出节点的右边界也就确定了
>
> 其中有一点要想明白，我们以寻找左边界为例，在弹出的过程中要寻找到最精准的左边界，那么就需要将相等的值也弹出找到最左的边界，但是我们要一边处理弹出节点的右边界，那么如果弹出的节点和当前节点相等是不是就找错又边界了呢？其实不然，此刻的牺牲是为了全局的胜利，虽然这个节点得到了一个并不完全的右边界，但是当前节点依旧可以代替那个节点完成剩下的任务（因为他们的值是一样的），左边界明显也是一样的，所以只要有一个右边界是精准的，那么最后求的时候就是准确的

#### Code

```c++
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        vector<int> l(n, -1), r(n, n);
        stack<int> s;
        for (int i = 0; i < n; ++i) {
            while (s.size() && heights[i] <= heights[s.top()]) {
                r[s.top()] = i;
                s.pop();
            }
            if (s.size()) l[i] = s.top();
            s.push(i);
        }
        int ma = 0;
        for (int i = 0; i < n; ++i) {
            int h = heights[i], le = l[i] + 1, re = r[i] - 1;
            ma = max(ma, h * (re - le + 1));
        }
        return ma;
    }
};
```

---

## 例题——好子数组的最大分数

> Problem: [1793. 好子数组的最大分数](https://leetcode.cn/problems/maximum-score-of-a-good-subarray/description/)

### 思路1——前后缀拆分+二分

> 这是我最原始的思路，这题是以k为中心往两边扩展的，所以我们从k开始分别向前后扩展寻找某一长度下子数组的最小值，很显然这俩数组一定是单调的，于是我们可以通过二分在短时间内获得以某个值为最小值的左边界和右边界，总共的取值只有1e5个，而每次判断只有Logn，我们可以通过这种方式来枚举出答案

#### Code

```c++
class Solution {
public:
    int maximumScore(vector<int>& nums, int k) {
        vector<int> pre, suf;
        unordered_set<int> ssr; 
        int n = nums.size();
        pre.emplace_back(nums[k]);
        suf.emplace_back(nums[k]);
        ssr.insert(nums[k]);
        for (int i = k - 1; i >= 0; --i) pre.emplace_back(min(pre.back(), nums[i])), ssr.insert(nums[i]);
        for (int i = k + 1; i < n; ++i) suf.emplace_back(min(suf.back(), nums[i])), ssr.insert(nums[i]);
        int ans = 0;
        auto Find = [&] (int x, vector<int>& a) {
            int l = 0, r = a.size() - 1;
            while (l <= r) {
                int m = l + r >> 1;
                if (a[m] >= x) l = m + 1;
                else r = m - 1;
            }
            return r;
        };
        for (int x: ssr) {
            int l = Find(x, pre), r = Find(x, suf);
            int len = 1 + l + r;
            ans = max(ans, x * len);
        }
        return ans;
    }
};
```

---

### 思路2——双指针

> 其实这题的最优解应该是双指针，思路依旧是从k开始向两边扩展，用l和r来做左右边界的指针进行扩展，每次枚举当前区间的最小值，将这个最小值慢慢往下降来扩张区间，虽然看起来好像要遍历很多数字，实际上这俩指针跑到头也就听了所以时间复杂度也就O(n + m)，也就是取长度和数值范围的最大值

#### Code

```c++
class Solution {
public:
    int maximumScore(vector<int>& nums, int k) {
        int ma = nums[k];
        int ans = ma;
        int l = k - 1, r = k + 1;
        int n = nums.size();
        while (l >= 0 || r < n) {
            while (l >= 0 && nums[l] >= ma) --l;
            while (r < n && nums[r] >= ma) ++r;
            int len = r - l - 1;
            ans = max(ans, len * ma);
            --ma;
        }
        return ans;
    }
};
```

---

#### 优化——跳转优化

> 在上述双指针枚举数值的过程中，跳转上其实挺花时间的，有些大于左右边界的值被枚举从而浪费了时间，其实可以直接往左右边界的最大值进行跳转，省去那些没用的时间，保证每一步都有指针移动，时间复杂度优化为O(n)

##### Code

```c++
class Solution {
public:
    int maximumScore(vector<int>& nums, int k) {
        int ma = nums[k];
        int ans = ma;
        int l = k - 1, r = k + 1;
        int n = nums.size();
        while (l >= 0 || r < n) {
            while (l >= 0 && nums[l] >= ma) --l;
            while (r < n && nums[r] >= ma) ++r;
            int len = r - l - 1;
            ans = max(ans, len * ma);
            int ssr = l >= 0 ? nums[l] : 0;
            ssr = r < n ? max(ssr, nums[r]) : ssr;
            ma = ssr ? ssr : ma - 1;
        }
        return ans;
    }
};
```

---

### 思路3——前后缀分离+单调栈

> 这题依旧可以延续柱状图矩形中前后缀分离+单调栈的思路，只需要多加一步判断来确定边界是否合理即可

#### Code

```c++
class Solution {
public:
    int maximumScore(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> l(n, -1), r(n, n);
        stack<int> ssr;
        for (int i = 0; i < n; ++i) {
            while (ssr.size() && nums[i] <= nums[ssr.top()]) {
                r[ssr.top()] = i;
                ssr.pop();
            }
            if (ssr.size()) l[i] = ssr.top();
            ssr.emplace(i);
        }
        int ma = 0;
        for (int i = 0; i < n; ++i) {
            int v = nums[i], le = l[i] + 1, re = r[i] - 1;
            if (le <= k && re >= k) ma = max(ma, v * (re - le + 1));
        }
        return ma;
    }
};
```

