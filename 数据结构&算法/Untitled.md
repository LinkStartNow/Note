## 相同分数的最大操作数目2

> Problem: [3040. 相同分数的最大操作数目 II](https://leetcode.cn/problems/maximum-number-of-operations-with-the-same-score-ii/description/)

### 思路

> 刚看见题目尝试贪心解题，发现每一步并不能随意的进行三选一，也就是说局部最优解无法促成全局最优解，这题大概率是个动态规划
>
> 原问题的解法和子问题相似，每选取一次就往内收缩左右边界，也就是区间DP
>
> 每次就对三个方式进行判断即可，注意要记忆化搜索，因为有大量的重复计算

### 首次尝试——TLE

> 第一次照着思路写了一堆屑代码结果T了

```c++
class Solution {
    vector<vector<vector<int>>> dp;
    int n;

    int Get(int l, int r, const int& ssr, int k, vector<int>& nums)
    {
        if (l >= r) return 0;
        if (l < 0 || r >= n) return 0;
        if (dp[l][r][k] != -1) return dp[l][r][k];
        // cout << l << ' ' << r << ' ' << k << endl;
        if (k == 1) {
            if (nums[l] + nums[l + 1] != ssr) return dp[l][r][k] = 0;
            int ls = l + 2, rs = r;
            int a = Get(ls, rs, ssr, 1, nums);
            int b = Get(ls, rs, ssr, 2, nums);
            int c = Get(ls, rs, ssr, 3, nums);
            int t = max(a, b);
            t = max(t, c);
            return dp[l][r][k] = t + 1;
        }
        else if (k == 2) {
            if (nums[r] + nums[r - 1] != ssr) return dp[l][r][k] = 0;
            int ls = l, rs = r - 2;
            int a = Get(ls, rs, ssr, 1, nums);
            int b = Get(ls, rs, ssr, 2, nums);
            int c = Get(ls, rs, ssr, 3, nums);
            int t = max(a, b);
            t = max(t, c);
            return dp[l][r][k] = t + 1;
        }
        else {
            if (nums[r] + nums[l] != ssr) return dp[l][r][k] = 0;
            int ls = l + 1, rs = r - 1;
            int a = Get(ls, rs, ssr, 1, nums);
            int b = Get(ls, rs, ssr, 2, nums);
            int c = Get(ls, rs, ssr, 3, nums);
            int t = max(a, b);
            t = max(t, c);
            return dp[l][r][k] = t + 1;
        }
    }

public:
    int maxOperations(vector<int>& nums) {
        n = nums.size();
        int l = 0, r = n - 1;
        dp.resize(n, vector<vector<int>>(n, vector<int>(4, -1)));
        int a = Get(l, n - 1, nums[l] + nums[l + 1], 1, nums);
        int b = Get(l, n - 1, nums[n - 1] + nums[n - 2], 2, nums);
        int c = Get(l, n - 1, nums[l] + nums[n - 1], 3, nums);
        return max({a, b, c});
    }
};
```

---

### 问题分析

> 每次递归的跑的时候都将三种方式尝试了一遍导致分支开的过多
>
> 在往下递归之前其实就可以先进行判断，能减少很多的分支

### Code

```c++
class Solution {
    vector<vector<int>> dp;

    int find(int l, int r, const int& target, const vector<int>& nums)
    {
        if (l >= r) {
            return 0;
        }
        int& res = dp[l][r];
        if (res) return res;
        if (nums[l] + nums[l + 1] == target) {
            res = max(res, 1 + find(l + 2, r, target, nums));
        }
        if (nums[l] + nums[r] == target) {
            res = max(res, 1 + find(l + 1, r - 1, target, nums));
        }
        if (nums[r] + nums[r - 1] == target) {
            res = max(res, 1 + find(l, r - 2, target, nums));
        }
        return res;
    }

public:
    int maxOperations(vector<int>& nums) {
        int n = nums.size();
        dp.resize(n, vector<int>(n));
        int a = find(0, n - 1, nums[0] + nums[1], nums);
        for (auto& v: dp) for (auto& x: v) x = 0;
        int b = find(0, n - 1, nums[0] + nums[n - 1], nums);
        for (auto& v: dp) for (auto& x: v) x = 0;
        int c = find(0, n - 1, nums[n - 2] + nums[n - 1], nums);
        return max({a, b, c});
    }
};
```

### 剪枝再优化

> 这里最多其实就只能是`n >> 1`，所以只要一个方法能到达最大值，其他的都可以不用判断了
>
> 这个优化可以减去大量的分支，节约很多时间

#### Code

```c++
class Solution {
    vector<vector<int>> dp;
    bool over = false;

    int find(int l, int r, const int& target, const vector<int>& nums)
    {
        if (l >= r) {
            over = true;
            return 0;
        }
        if (over) return 0;
        int& res = dp[l][r];
        if (res) return res;
        if (nums[l] + nums[l + 1] == target) {
            res = max(res, 1 + find(l + 2, r, target, nums));
        }
        if (nums[l] + nums[r] == target) {
            res = max(res, 1 + find(l + 1, r - 1, target, nums));
        }
        if (nums[r] + nums[r - 1] == target) {
            res = max(res, 1 + find(l, r - 2, target, nums));
        }
        return res;
    }

public:
    int maxOperations(vector<int>& nums) {
        int n = nums.size();
        dp.resize(n, vector<int>(n));
        int a = find(0, n - 1, nums[0] + nums[1], nums);
        if (over) return n >> 1;
        for (auto& v: dp) for (auto& x: v) x = 0;
        int b = find(0, n - 1, nums[0] + nums[n - 1], nums);
        if (over) return n >> 1;
        for (auto& v: dp) for (auto& x: v) x = 0;
        int c = find(0, n - 1, nums[n - 2] + nums[n - 1], nums);
        return max({a, b, c});
    }
};
```

---

###  递归转递推

> 递归转递推我们需要考虑的问题中比较重要的一个就是正向跑还是反向跑
>
> 这里很显然我们用i表示左区间，j表示右区间。我们要求`dp[i, x]`的状态需要依赖`dp[i + 1, x]`的状态，所以i一定是反向遍历的，同理我们要求`dp[x, j]`的状态需要依赖`dp[x, j - 1]`的状态，于是j是反向遍历的，那么递推也就出来了

#### Code

```c++
class Solution {
    bool over = false;
    int n;

    int find(int l, int r, const int& target, const vector<int>& nums)
    {
        // if (over) return n >> 1;
        vector<vector<int>> dp(n + 1, vector<int>(n + 1));
        for (int i = r - 1; i >= l; --i) {
            // if (over) return n >> 1;
            for (int j = i + 1; j <= r; ++j) {
                // if (over) return n >> 1;
                if (nums[i] + nums[j] == target) {
                    dp[i][j + 1] = max(dp[i][j + 1], dp[i + 1][j] + 1);
                }
                if (nums[i] + nums[i + 1] == target) {
                    dp[i][j + 1] = max(dp[i][j + 1], dp[i + 2][j + 1] + 1);
                }
                if (nums[j] + nums[j - 1] == target) {
                    dp[i][j + 1] = max(dp[i][j + 1], dp[i][j - 1] + 1);
                }
                // if (dp[i][j + 1] == n / 2) over = true;
            }
        }
        return dp[l][r + 1];
    }

public:
    int maxOperations(vector<int>& nums) {
        n = nums.size();
        int a = find(2, n - 1, nums[0] + nums[1], nums);
        int b = find(1, n - 2, nums[0] + nums[n - 1], nums);
        int c = find(0, n - 3, nums[n - 2] + nums[n - 1], nums);
        return max({a, b, c}) + 1;
    }
};
```

