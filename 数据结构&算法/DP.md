众所周知DP是比较抽象的，比较不好总结，所以我们先以例题为例

# LCR砍竹子

> Problem: [LCR 131. 砍竹子 I](https://leetcode.cn/problems/jian-sheng-zi-lcof/description/)

## 思路1——动态规划

> 我们要获取数字n拆分成多个数字的和的最大乘积，那么我们可以去考虑如何分
>
> 对于数字`n`，我们可以分成不考虑再分的`i`和剩余部分`n-i`，剩余部分`n-i`也是去求取能算出的最大积，同样的子问题，范围减小了，于是可以通过动态规划来解决

### Code

```c++
class Solution {
    vector<int> dp;
    int n;

    int find(int x)
    {
        int& ssr = dp[x];
        if (ssr) return ssr;
        for (int i = 2; i <= (x >> 1); ++i) ssr = max(ssr, i * find(x - i));
        return ssr;
    }

public:
    int cuttingBamboo(int bamboo_len) {
        n = bamboo_len;
        dp.resize(bamboo_len + 1, 0);
        dp[1] = 1, dp[2] = 2, dp[3] = 3;
        if (bamboo_len == 2) return 1;
        else if (bamboo_len == 3) return 2;
        return find(n);
    }
};
```

---

### 递归转递推

> 因为大的数依赖于小的数，所以顺着推即可

```c++
class Solution {
public:
    int cuttingBamboo(int bamboo_len) {
        if (bamboo_len == 2) return 1;
        else if (bamboo_len == 3) return 2;
        vector<int> dp(bamboo_len + 1, 0);
        dp[1] = 1, dp[2] = 2, dp[3] = 3;
        for (int x = 4; x <= bamboo_len; ++x) {
            for (int i = 2; i <= (x >> 1); ++i) 
                dp[x] = max(dp[x], i * dp[x - i]);
        }
        return dp[bamboo_len];
    }
};
```

## 思路2——数学贪心

> 数字能拆尽量拆，因为乘积一般都比和大，最终拆的边界也就是2和3，拆到1就白拆了
>
> 根据数学证明，3比2更优，于是优先拆3，最后看余数，如果是0则直接全拆成3；如果为1，那么去掉一个3凑4；如果是2，那么也直接乘2

### Code

```c++
class Solution {
    using ll = long long;
    int mypow(ll x, int y)
    {
        long long res = 1;
        while (y) {
            if (y & 1) res *= x;
            x *= x;
            y >>= 1;
        }
        return res;
    }
public:
    int cuttingBamboo(int bamboo_len) {
        if (bamboo_len <= 3) return bamboo_len - 1;
        int x = bamboo_len % 3, y = bamboo_len / 3;
        if (x == 2) return mypow(3, y) * 2;
        else if (x == 1) return mypow(3, y - 1) * 4;
        return mypow(3, y);
    }
};
```

---

## 难度升级——LCR砍竹子2

> Problem: [LCR 132. 砍竹子 II](https://leetcode.cn/problems/jian-sheng-zi-ii-lcof/description/)

### 思路

> 对于新修改数据范围后的砍竹子，动态规划已经不适用了，因为在求取的过程中状态答案非常的大，`long long`都装不下，如果我们强行对状态进行取模运算，则会导致最终结果出错，因为取模和取最大值无法一起使用，例如：
> $$
> max(6 \% 8, 9 \% 8) = max(6, 1) = 1 \\
> 然而实际结果确为：
> max(6, 9) \% 8 = 9 \% 8 = 1
> $$
> 所以我们只能采用贪心的解法，最后求幂可以用快速幂来优化加速

### Code

```c++
class Solution {
    using ll = long long;
    const int mod = 1000000007;
    ll mypow(ll x, int y)
    {
        ll res = 1;
        while (y) {
            if (y & 1) res = x * res % mod;
            x = x * x % mod;
            y >>= 1;
        }
        return res;
    }
public:
    int cuttingBamboo(int bamboo_len) {
        if (bamboo_len <= 3) return bamboo_len - 1;
        int x = bamboo_len % 3, y = bamboo_len / 3;
        if (x == 2) return mypow(3, y) * 2 % mod;
        else if (x == 1) return mypow(3, y - 1) * 4 % mod;
        return mypow(3, y) % mod;
    }
};
```

---

# 删除一次得到子数组最大和

> Problem: [1186. 删除一次得到子数组最大和](https://leetcode.cn/problems/maximum-subarray-sum-with-one-deletion/description/)

## 思路——动态规划

> 原以为这题跟之前求子数组最大和是一样的，but这里可以多一次删除操作，so之前贪心的策略并不好取，因为删当前区间还是删未来区间哪个最优是无法确定的
>
> 既然又是最值问题，不如考虑一下动态规划：
>
> 我们考虑以每个数字为右边界可以得到的最大值，同时还需要保留删除情况，所以定义$dp[i][kill]$表示以第i个元素为右边界并且一定要删除kill个数字（$kill = 0, 1$）
>
> 那么考虑如何状态转移，假如$kill = 0$：
>
> 那么我们考虑选不选择i左边的元素，不选则直接是$arr[i]$，否则为$dp[i - 1][0] + arr[i]$
>
> 假如$kill = 1$：
>
> 那么不可能不选择i左边的元素，因为一定要删除一个元素，但是只剩下唯一一个元素无法删除，但是还是要考虑删除前面部分还是第i个元素，删除前面部分则为$dp[i - 1][1] + arr[i]$，否则为$dp[i - 1][0]$
>
> 于是最终的状态转移方程变成了：
> $$
> dp[i][kill] =
> \begin{cases}
> max(dp[i - 1][0], 0) + arr[i] & kill = 0 \and i > 0 \\
> max(dp[i - 1][1] + arr[i], dp[i - 1][0]) & kill = 1 \and i > 0 \\
> -\infty & i <= 0
> \end{cases}
> $$

## Code

```c++
class Solution {
    vector<vector<int>> dp;
    int find(vector<int>& arr, int right, bool kill)
    {
        if (right < 0) return -1e9 / 2;
        int& res = dp[right][kill];
        if (res != -1e9) return res;
        if (!kill) {
            return res = max(find(arr, right - 1, kill), 0) + arr[right];
        }
        else return res = max(find(arr, right - 1, 0), find(arr, right - 1, kill) + arr[right]);
    }

public:
    int maximumSum(vector<int>& arr) {
        int ans = -1e9, n = arr.size();
        dp.resize(n, vector<int>(2, -1e9));
        for (int i = 0; i < n; ++i) ans = max({ans, find(arr, i, 0), find(arr, i, 1)});
        return ans;
    }
};
```

---

## 递归转递推

> 将dp递推数组照抄完事

### Code

```c++
class Solution {
    vector<vector<int>> dp;
public:
    int maximumSum(vector<int>& arr) {
        int ans = -1e9, n = arr.size();
        dp.resize(n + 1, vector<int>(2, -1e9 / 2));
        for (int i = 0; i < n; ++i) {
            dp[i + 1][0] = max(dp[i][0], 0) + arr[i];
            dp[i + 1][1] = max(dp[i][0], dp[i][1] + arr[i]);
            ans = max({ans, dp[i + 1][0], dp[i + 1][1]});
        }
        return ans;
    }
};
```

---

### 空间优化——翻转数组

> 总共只用到两层的数组，于是直接翻转数组进行空间优化

#### Code

```c++
class Solution {
    vector<vector<int>> dp;
public:
    int maximumSum(vector<int>& arr) {
        int ans = -1e9, n = arr.size();
        dp.resize(2, vector<int>(2, -1e9 / 2));
        for (int i = 0; i < n; ++i) {
            int k = i % 2, z = k ^ 1;
            dp[z][0] = max(dp[k][0], 0) + arr[i];
            dp[z][1] = max(dp[k][0], dp[k][1] + arr[i]);
            ans = max({ans, dp[z][0], dp[z][1]});
        }
        return ans;
    }
};
```

---

### 空间优化——一维

> 由于只依赖于上一层，而且当前层的1依赖于上一层的1和0，而当前层的0只依赖于上一层的0，所以直接先改1后改0即可

#### Code

```c++
class Solution {
    vector<int> dp;
public:
    int maximumSum(vector<int>& arr) {
        int ans = -1e9, n = arr.size();
        dp.resize(2, -1e9 / 2);
        for (int i = 0; i < n; ++i) {
            dp[1] = max(dp[0], dp[1] + arr[i]);
            dp[0] = max(dp[0], 0) + arr[i];
            ans = max({ans, dp[0], dp[1]});
        }
        return ans;
    }
};
```

---

# 例题——卖木头块

> Problem: [2312. 卖木头块](https://leetcode.cn/problems/selling-pieces-of-wood/description/)

## 思路

> 这题很明显是个dp的题目，我们可以考虑这样的状态`dp[i][j]`，表示的是宽为i，高为j的木块所能卖的最大价格，我们切的方式只有横着切和竖着切，每次去枚举切的位置就能把问题分成更小的子问题
>
> 当前木块也可能整个售卖，于是如果当前尺寸可以直接卖，这个价格也需要考虑。我们想要快速确定当前块的尺寸是否可行需要用hash表记录一下，否则去遍历整个prices数组非常耗时肯定不行
>
> 同时递归的时候我们要记忆化搜索，不然非常浪费时间，会做很多重复的计算

### Code

```c++
class Solution {
    unordered_map<int, int> mp;
    int m, n;

    int turn(int x, int y)
    {
        return x * 200 + y;
    }

    long long dp(int x, int y, vector<vector<long long>>& d)
    {
        if (d[x][y] != -1) return d[x][y];
        if (x <= 0 || y <= 0) return d[x][y] = 0;
        d[x][y] = 0;
        if (mp.count(turn(x, y))) d[x][y] = mp[turn(x, y)];
        for (int i = 1; i <= (x >> 1); ++i) d[x][y] = max(d[x][y], dp(i, y, d) + dp(x - i, y, d));
        for (int i = 1; i <= (y >> 1); ++i) d[x][y] = max(d[x][y], dp(x, i, d) + dp(x, y - i, d));
        return d[x][y];
    }

public:
    long long sellingWood(int m, int n, vector<vector<int>>& prices) {
        this->m = m, this->n = n;
        for (auto& v: prices) mp[turn(v[0], v[1])] = v[2];
        vector<vector<long long>> d(m + 1, vector<long long>(n + 1, -1));
        return dp(m, n, d);
    }
};
```

---

## 优化1——递归转递推

> 有了递归的思路，我们发现大的状态总是依赖于前面小的状态，于是我们可以从小到大递推来优化时间

### Code

```c++
class Solution {
    unordered_map<int, int> mp;

    int turn(int x, int y) { return x * 200 + y; }

public:
    long long sellingWood(int m, int n, vector<vector<int>>& prices) {
        for (auto& v : prices) mp[turn(v[0], v[1])] = v[2];
        vector<vector<long long>> d(m + 1, vector<long long>(n + 1, 0));
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                d[i][j] = mp[turn(i, j)];
                for (int k = 1; k < i; ++k) d[i][j] = max(d[i][j], d[k][j] + d[i - k][j]);
                for (int k = 1; k < j; ++k) d[i][j] = max(d[i][j], d[i][k] + d[i][j - k]);
            }
        }
        return d[m][n];
    }
};
```

---

## 优化2——map优化

> 看了数据范围，m和n都比较小，那么用来映射的hash就不需要用map了，直接用数组模拟即可，我们可以维护一个二维数组能表示宽高来进行状态存储
>
> 其实这里就能发现dp数组和这个映射数组两维都是一样的，而且每个dp还得由映射数组转移过来，那么我们其实可以只用一个来完成这个功能

### Code

```c++
class Solution {
public:
    long long sellingWood(int m, int n, vector<vector<int>>& prices) {
        vector<vector<long long>> d(m + 1, vector<long long>(n + 1));
        for (auto& vs : prices) d[vs[0]][vs[1]] = vs[2];
        for (int i = 1; i <= m; ++i) {
            for (int j = 1; j <= n; ++j) {
                for (int k = 1; k <= (i >> 1); ++k) d[i][j] = max(d[i][j], d[k][j] + d[i - k][j]);
                for (int k = 1; k <= (j >> 1); ++k) d[i][j] = max(d[i][j], d[i][k] + d[i][j - k]);
            }
        }
        return d[m][n];
    }
};
```

---



# 例题——数字拆分

> Problem:[数字拆分]([Ignatius and the Princess III - HDU 1028 - Virtual Judge (vjudge.net)](https://vjudge.net/problem/HDU-1028))

## 思路

> 我们需要维护一个二维的dp数组，`dp[i][j]`表示拆分数组i的分组中最大数字不超过j的方案个数
>
> 接下来对于i和j的取值来分类讨论：
>
> - 如果i和j其中一个为1那么很显然方案数只能是1种
> - 如果i与j相等，分组中包含j的情况只能是一种，不包含j的情况为`dp[i][j - 1]`，把这两个加起来就行
> - 如果i小于j，直接由`dp[i][i]`转移过来就行，因为大于i的方案肯定一种都没有，不存在负数嘛
> - 如果i大于j，那么我们接下来的分组可以分为包括j和不包括j的组合：
> 	- 包括j的就用`dp[i - j][j]`表示，然后再补上一个j刚好就是i了
> 	
> 	- ```c++
> 		class Solution
> 		{
> 		public:
> 		    int minDistance(string word1, string word2)
> 		    {
> 		        int n = word1.size(), m = word2.size();
> 		        vector<int> dp(m + 1);
> 		        for (int i = 0; i <= m; ++i)
> 		            dp[i] = i;
> 		        for (int i = 1; i <= n; ++i)
> 		        {
> 		            int ssr = i;
> 		            for (int j = 1; j <= m; ++j)
> 		            {
> 		                int yyds = 0;
> 		                if (word1[i - 1] == word2[j - 1])
> 		                    yyds = 1 + min({dp[j], ssr, dp[j - 1] - 1});
> 		                else
> 		                    yyds = min({ssr, dp[j], dp[j - 1]}) + 1;
> 		                dp[j - 1] = ssr, ssr = yyds;
> 		            }
> 		            dp[m] = ssr;
> 		        }
> 		        return dp[m];
> 		    }
> 		};
> 		```
> 	
> 	- 

## Code

```c++
#include <iostream>
#include <vector>
#include <math.h>
#include <algorithm>
#include <memory>

using namespace std;

int d[121][121];

int Find(int n, int ma)
{
    if (d[n][ma] != -1) return d[n][ma];
    if (n == 1 || ma == 1) return d[n][ma] = 1;
    if (n < ma) return d[n][ma] = Find(n, n);
    if (n > ma) return d[n][ma] = Find(n - ma, ma) + Find(n, ma - 1);
    return d[n][ma] = Find(n, ma - 1) + 1;
}

int main()
{
    for (auto& v: d) for (int& x: v) x = -1; 
    int n;
    while (scanf("%d", &n) != EOF) {
        printf("%d\n", Find(n, n));
    }
}
```

---

# 例题——打家劫舍

> Problem: [198. 打家劫舍](https://leetcode.cn/problems/house-robber/description/)

## 首次自己尝试

这是我首次想到的解决方案，比较的详细

第一维用来记录扫描到了第几号房子，第二维度记录该房子有没有被偷

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)
        ans = [[0] * 2 for _ in range(n)]
        ans[0][0], ans[0][1] = 0, nums[0]
        for i in range(1, n):
            ans[i][0] = max(ans[i - 1][0], ans[i - 1][1])
            ans[i][1] = ans[i - 1][0] + nums[i]
        return max(ans[n - 1][0], ans[n - 1][1])
```

## 一轮优化

后来发现，根本用不着2维，用贪心的思想想想，如果当前的要选那么前一个肯定不选，隔一个的肯定要选，于是`dp[i] = max(dp[i - 1], dp[i - 2] + nums[i])`肯定是成立的，没有人傻到隔着一个也不取吧

这一维表达的意义就是，当扫描到第`i`个时选取的最佳方案，所以不能通过`dp[i]`判断第i个到底取了没有

**注意：因为这里需要直接先默认前两个房子是存在的，所以需要判断一下**

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)
        ans = [0] * n
        if n == 0:
            return 0
        if n == 1:
            return nums[0]
        ans[0], ans[1] = nums[0], max(nums[0], nums[1])
        for i in range(2, n):
            ans[i] = max(ans[i - 1], ans[i - 2] + nums[i])
        return ans[n - 1]
```

## 二轮优化

我们可以发现，要确定当前位的值，能造成影响的只有前两位，于是我们可以通过滚动数组来进一步压缩空间

```python
class Solution:
    def rob(self, nums: List[int]) -> int:
        n = len(nums)
        if n == 0:
            return 0
        if n == 1:
            return nums[0]
        first, second = nums[0], max(nums[0], nums[1])
        for i in range(2, n):
            first, second = second, max(first + nums[i], second)
        return second
```

## 打家劫舍2

> Problem: [213. 打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/description/)

这是例题的强化版，改变就是，所有房子都连在了一起，所以头尾不能同时选择

### 个人思路

我们可以继续借鉴原版题目的经验，先把第一间考虑上，最后一间无论如何都不取；然后再考虑把第一间不选，最后一间考虑要不要选

```python
class Solution:
    def rob(self, nums: list[int]) -> int:
        n = len(nums)
        if n == 0:
            return 0
        if n == 1:
            return nums[0]
        first, second = nums[0], max(nums[0], nums[1])
        for i in range(2, n - 1):
            first, second = second, max(first + nums[i], second)
        ans = max(first, second)
        first, second = 0, nums[1]
        for i in range(2, n):
            first, second = second, max(first + nums[i], second)
        return max(ans, second, first)
```

### 优化1

实际上思路已经没啥好优化的了，但是我们可以通过函数来优化一下代码结构

很显然这两轮动态规划很多地方都是一样的，唯一不一样的地方就是判断范围不同了，于是我们拿范围做参数就可以直接调用同一个函数即可

```python
class Solution:
    def rob(self, nums: list[int]) -> int:
        
        def yyds(l, r):
            first, second = nums[l], max(nums[l], nums[l + 1])
            for i in range(l + 2, r):
                first, second = second, max(first + nums[i], second)
            return second

        n = len(nums)
        if n == 0:
            return 0
        if n == 1:
            return nums[0]
        if n == 2:
            return max(nums[0], nums[1])
        return max(yyds(0, n - 1), yyds(1, n))
```

## 打家劫舍3

> Problem: [337. 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/description/)

这里把所有的房子放在了一颗二叉树上，约束是父节点和子节点不能同时选

于是我们只需要在树上DP就行了

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def rob(self, root: Optional[TreeNode]) -> int:
        def yyds(node):
            if node == None:
                return 0, 0
            if node.left == None and node.right == None:
                return node.val, 0
            tl = yyds(node.left)
            tr = yyds(node.right)
            return max(tl[0] + tr[0], tl[1] + tr[1] + node.val), tl[0] + tr[0]
        return yyds(root)[0]

```

# 例题——买股票的最佳时机

> Problem: [188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/description/)

## 思路

> 我的思路就是用三维的DP来维护状态，第一维用来保存考虑到第几天，第二维用来保存手头是否有股票，第三维用来保存至今已经完成多少次交易了

## 优化

> 又考虑到当前的状态只会受前一天的状态影响，于是第一维可以被优化掉

## Code

```python
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        n = len(prices)
        if n == 1:
            return 0
        buy = [-inf] * (k + 1)
        sell = [-inf] * (k + 1)
        buy[0] = -prices[0]
        sell[0] = 0
        for i in range(1, n):
            buy[0] = max(buy[0], -prices[i])
            for j in range(1, k + 1):
                buy[j], sell[j] = max(buy[j], sell[j] - prices[i]), max(sell[j], buy[j - 1] + prices[i])
        return  max(sell)
```

## 变量说明

> sell[i]的意思是，当进行了当前的卖出操作后，已经完成了多少次交易
>
> buy[i]也是如此

## 细节

> 我们在循环开始前要先进行初始化，最好先把第一天的状态处理掉
>
> 我们都知道在第一天的时候，无论如何操作都不可能有交易（一买一卖）产生，于是那些大于0次交易的变量全部初始化为最小值，表示这些情况毫无意义

## 另一种思路

### 说明

> 其实这一种思路和前面大差不差，只是考虑到没有冷却期，于是可以在当天疯狂的买进卖出同一只股票，在收益不变的情况下改变最终交易次数
>
> 这样可以把最终的答案固定在最多交易次数上，最后就不用再找一遍最大值了

### Code

```python
class Solution:
    def maxProfit(self, k: int, prices: List[int]) -> int:
        n = len(prices)
        if n == 1:
            return 0
        buy = [-prices[0]] * (k + 1)
        sell = [0] * (k + 1)
        for i in range(1, n):
            buy[0] = max(buy[0], -prices[i])
            for j in range(1, k + 1):
                buy[j], sell[j] = max(buy[j], sell[j] - prices[i]), max(sell[j], buy[j - 1] + prices[i])
        return  sell[k]
```

---

# 例题——抛骰子

> Problem：[1155. 掷骰子等于目标和的方法数]([1155. 掷骰子等于目标和的方法数 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-dice-rolls-with-target-sum/description/?envType=daily-question&envId=2023-10-24))

## 思路

> 很明显这就是一道简单的DP问题
>
> 我们只需要建立一个二维数组记录两个状态，一个是目前考虑多少个骰子，第二维是总和加起来为多大即可
>
> 然后每次遍历一下投的大小
>
> `for t in range(1, k + 1): dp[i][sum] = dp[i  - 1][sum - t]`
>
> **注意：dp\[0][0]必须为1，其意义是0个骰子的时候有1种情况相加为0（之后肯定是没有相加为0的机会了）不然所有都是0根本就增加不下去了哈哈哈**

## Code

```python
class Solution:
    def numRollsToTarget(self, n: int, k: int, target: int) -> int:
        dp = [[0] * (target + 1) for _ in range(n + 1)]
        dp[0][0] = 1
        m = 10 ** 9 + 7
        for i in range(1, n + 1):
            for z in range(i, target + 1):
                for j in range(1, min(k + 1, z + 1)):
                    dp[i][z] += dp[i - 1][z - j]
                    dp[i][z] %= m
        return dp[n][target] % m

s = Solution()
n, k, target = 1, 6, 3
print(s.numRollsToTarget(n, k, target))
```

---

## 优化

> 像这种当前状态只会受前一状态影响的，我们不需要记录所有状态，只需要开两个一维然后交替使用即可，这样能省下大量的内存
>
> 当然我们这里直接就追求更极致的版本了，由于很明显每次dp的取值dp\[i][z] += dp\[i - 1][z - j] ，所依赖的第二维下标严格小于当前需要改变的下标位置
>
> 于是，我们如果从后往前处理就可以保证每次在处理的时候依赖的都是上一层的元素，这样就能使用一维数组就解决问题了

### Code

```python
class Solution:
    def numRollsToTarget(self, n: int, k: int, target: int) -> int:
        mod = 10**9 + 7
        f = [1] + [0] * target
        for i in range(1, n + 1):
            for j in range(target, -1, -1):
                f[j] = 0
                for x in range(1, k + 1):
                    if j - x >= 0:
                        f[j] = (f[j] + f[j - x]) % mod
        return f[target]
```

---

### 易错点

> 实际上，我一上来就拿一维写的，但是疯狂出错，找了半天bug才找到问题所在。
>
> 首先，每一轮处理的时候要先将dp[j] = 0，清零当前的元素，因为当前的元素是上一层的，如果直接开始累加的话，那么答案肯定就偏大了
>
> 再者，之前为了省循环次数，我在遍历j的时候直接遍历到了此时理论上的最小结果，因为继续遍历下去数组就会越界，也就没有意义了。这就犯了一个严重的错误，遍历的躺数确实节约了，在二维分离开的情况下很合理肯定没问题。然鹅，这是一维！！！节约了空间，在数据的处理上就需要更加的谨慎。我们每一维的元素都是通过倒序共用的，前面的值直接被跳过了，也许那些值并不会在当轮被更新，但是，会在之后被当做依赖累加起来。所以我们每处理完多一个骰子，就需要保证当前的每个数据都是对应目前数目的骰子的，那些每遍历到的元素是属于上一层的，在当前层理论上肯定是0，于是我们不能偷懒，需要遍历到底全部清0！

---

# 例题——最小化旅行的价格总和

> Problem:[2646. 最小化旅行的价格总和](https://leetcode.cn/problems/minimize-the-total-price-of-the-trips/description/?envType=daily-question&envId=2023-12-06)

## 说明

> 这题跟打家劫舍3十分的像，也算是一道树上dp的题目，相邻的节点不能同时选择，也就和打家劫舍3的条件相同了
>
> 我们可以先对每一段旅行进行dfs遍历找出每一段旅行必须经过的节点记录下来，他们就是最终需要经过的节点
>
> 然后我们一边树上dp一边处理这些节点是否降低值即可

## Code

```c++
class Solution {
public:
    int minimumTotalPrice(int n, vector<vector<int>>& edges, vector<int>& price, vector<vector<int>>& trips) {
        vector<int> e[n];
        for (vector<int>& x: edges) {
            e[x[0]].push_back(x[1]);
            e[x[1]].push_back(x[0]);
        }

        vector<int> cnt(n);

        function<bool(int, int, int)> dfs = [&] (int u, int f, int des) {
            if (des == u) {
                cnt[u]++;
                return true;
            }
            for (int v: e[u]) {
                if (v == f) continue;
                if (dfs(v, u, des)) {
                    cnt[u]++;
                    return true;
                }
            }
            return false;
        };

        for (vector<int>& x: trips) dfs(x[0], x[0], x[1]);

        for (int i = 0; i < n; ++i) {
            cout << i << ' ' << cnt[i] << endl;
        }

        function<pair<int, int>(int, int)> dp = [&] (int u, int f) {
            pair<int, int> res = {
                cnt[u] * price[u],
                cnt[u] * price[u] >> 1
            };
            for (int v: e[u]) {
                if (v == f) continue;
                auto [x, y] = dp(v, u);
                res.first += min(x, y);
                res.second += x;
            }
            return res;
        };
        auto [x, y] = dp(0, 0);
        return min(x, y);
    }
};
```

---

# 例题——得到山形数组的最少删除次数

> Problem: [1671. 得到山形数组的最少删除次数](https://leetcode.cn/problems/minimum-number-of-removals-to-make-mountain-array/description/)

## 思路

> 这题实际上就是用两次dp求最长上升子序列，前后缀分离，前缀做一次，后缀做一次

## Code

```c++
class Solution {
public:
    int minimumMountainRemovals(vector<int>& nums) {
        int n = nums.size();
        vector<int> f(n), t(n), s;
        s.push_back(0);
        int ma = 0;
        auto Find = [&] (int x) {
            int l = 0, r = ma;
            int ans = 0;
            while (l <= r) {
                int m = (l + r) >> 1;
                if (s[m] >= x) r = m - 1;
                else l = m + 1, ans = m;
            }
            return ans;
        };
        for (int i = 0; i < n; ++i) {
            int p = Find(nums[i]);
            f[i] = p + 1;
            if (s.size() - 1 < p + 1) s.push_back(nums[i]);
            else s[p + 1] = min(s[p + 1], nums[i]);
            ma = max(ma, p + 1);
        }
        for (int i = 1; i <= ma; ++i) s[i] = 1e9;
        ma = 0;
        for (int i = n - 1; i >= 0; --i) {
            int p = Find(nums[i]);
            t[i] = p + 1;
            if (s.size() - 1 < p + 1) s.push_back(nums[i]);
            else s[p + 1] = min(s[p + 1], nums[i]);
            ma = max(ma, p + 1);
        }
        int ans = 0;
        for (int i = 1; i < n - 1; ++i) if (f[i] != 1 && t[i] != 1) ans = max(ans, f[i] + t[i] - 1);
        for (int x: f) cout << x << ' ';
        cout << endl;
        for (int x: t) cout << x << ' ';
        return n - ans;
    }
};
```

---

# 例题——收集巧克力

> Problem: [2735. 收集巧克力](https://leetcode.cn/problems/collecting-chocolates/description/)

## 思路

> 买每个类型的巧克力的价格都随着转换的次数而变动，于是我们可以将每个物品转换了j次的价格来存储下来
>
> 那么dp\[i][j] = min(dp\[i][j - 1], nums[(j + i) % n])
>
> 因为总共只有n个物品，所以n - 1次就是最多了，再多就循环回去了，所以第二维开到n即可
>
> 转换k次时的最低消耗就是k * x + sum(dp\[i][j]);
>
> 这样子通过一个dp即可推出结果

---

## Code

```c++
class Solution {
public:
    long long minCost(vector<int>& nums, int x) {
        int n = nums.size();
        long long ans = 0;
        long long dp[n][n];
        for (int i = 0; i < n; ++i) dp[i][0] = nums[i], ans += nums[i];
        for (long long i = 1; i < n; ++i) {
            long long sum = i * x;
            for (int j = 0; j < n; ++j) {
                dp[j][i] = min(dp[j][i - 1], (long long)nums[(j + i) % n]);
                sum += dp[j][i];
            }
            ans = min(ans, sum); 
        }
        return ans;
    }
};
```

---

## 优化

> 每一层的dp\[i][j]只依赖于上一层的dp\[i][j - 1]，于是我们可以只用一维来表示

```c++
class Solution {
public:
    long long minCost(vector<int>& nums, int x) {
        int n = nums.size();
        long long ans = 0;
        long long dp[n];
        for (int i = 0; i < n; ++i) dp[i] = nums[i], ans += nums[i];
        for (long long i = 1; i < n; ++i) {
            long long sum = i * x;
            for (int j = 0; j < n; ++j) {
                dp[j] = min(dp[j], (long long)nums[(j + i) % n]);
                sum += dp[j];
            }
            ans = min(ans, sum); 
        }
        return ans;
    }
};
```

---

# 例题——字符串中的额外字符

> Problem: [2707. 字符串中的额外字符](https://leetcode.cn/problems/extra-characters-in-a-string/description/)

## 思路

> 这题一看到最终求剩余**最少**，那么就要留心眼了，这题也许又是个dp题
>
> 我们最终的那些额外字符可能出现在开头、结尾以及中间的部分，我们可以定义一个dp数组，下标到i前的子串最少的额外字符为几个
>
> 我们啥也不拼接的话，结果就是dp[i] = dp[i - 1] + 1，就直接多了一个字符
>
> 然后我们考虑能不能和前面的某个串来拼接成某个新的字符串，于是我们将dp[j]（j < i）全部比较一遍，能拼接就进行拼接，求最小值来转移

---

## Code

```c++
class Solution {
public:
    int minExtraChar(string s, vector<string>& dictionary) {
        map<string, bool> mp;
        for (string& s: dictionary) mp[s] = true;
        int n = s.size();
        vector<int> dp(n + 7, 2 * n);
        dp[0] = 0;
        for (int i = 1; i <= n; ++i) {
            dp[i] = dp[i - 1] + 1;
            for (int l = 1; l <= i; ++l) {
                string t = s.substr(i - l, l);
                if (mp.count(t)) dp[i] = min(dp[i], dp[i - l]);
            }
        }
        return dp[n];
    }
};
```

## 优化

> 我们在判断当前子串是否在字典中时，每次都需要从头开始判断，这样浪费了大量的时间
>
> 其实我们可以通过字典树来优化这个过程，将每个字典中的字符串倒序插入字典树，最终判断的时候也倒序一个个查询即可省去大量时间

### Code

```c++
class Trie
{
    vector<Trie*> nx;

public:
    bool end;

    Trie(): nx(26, nullptr), end(false) {}

    static void Push(Trie*& t, char c)
    {
        int z = c - 'a';
        if (!t->nx[z]) t->nx[z] = new Trie;
        t = t->nx[z];
    }

    static bool Find(Trie*& t, char c)
    {
        int z = c - 'a';
        t = t->nx[z];
        return t ? t->end : false;
    }
};

class Solution {
public:
    int minExtraChar(string s, vector<string>& dictionary) {
        Trie* t = new Trie;
        for (auto& x: dictionary) {
            Trie* tmp = t;
            for (int i = x.size() - 1; i >= 0; --i) Trie::Push(tmp, x[i]);
            tmp->end = true;
        }
        int n = s.size();
        vector<int> dp(n + 1, 3000);
        dp[0] = 0;
        for (int i = 1; i <= n; ++i) {
            Trie* tmp = t;
            dp[i] = dp[i - 1] + 1;
            for (int j = i - 1; j >= 0 && tmp; --j) {
                if (Trie::Find(tmp, s[j])) 
					dp[i] = min(dp[i], dp[j]);
            }
        }
        return dp[n];
    }
};
```

---

# 例题——石子游戏VII

> Problem: [1690. 石子游戏 VII](https://leetcode.cn/problems/stone-game-vii/description/)

## 思路

> 本来看到熟悉的石子游戏以为还是之前的无脑分奇偶就行了，结果啪啪打脸了，这次的石子堆不一定是偶数个，也就是说爱丽丝无法获得稳定的奇偶下标序列了
>
> 但其实这类题目就是个动态规划的问题，我们这次依旧通过动态规划来解决
>
> 我们可以维护一个二维的dp数组，`dp[l][r]`表示在范围从l到r的区间之中爱丽丝和鲍勃最终的游戏差值，而当前这个状态可以由取掉一个的状态转移过来
>
> 也就是从`sum(l + 1, r) + dp[l + 1][r]`和`sum(l, r - 1) + dp[l][r - 1]`转移过来，这俩表示从左边或右边取一个的结果

### Code

```c++
class Solution {
public:
    int stoneGameVII(vector<int>& stones) {
        int n = stones.size();
        vector<int> pre(n + 1);
        for (int i = 1; i <= n; ++i) pre[i] = pre[i - 1] + stones[i - 1];
        vector<vector<int>> dp(n, vector<int>(n));
        function<int(int, int)> yyds = [&] (int l, int r) {
            if (l >= r) return 0;
            int& ssr = dp[l][r];
            if (ssr) return ssr;
            int t1 = pre[r + 1] - pre[l + 1] - yyds(l + 1, r);
            int t2 = pre[r] - pre[l] - yyds(l, r - 1);
            return ssr = max(t1, t2);
        };
        return yyds(0, n - 1);
    }
};
```

## 优化——递归转递推

> 长度长的区间答案由长度短的得到，最终的状态是长度为1的时候，于是我们可以自然的将递归转化为效率更高的递推

### Code

```c++
class Solution {
public:
    int stoneGameVII(vector<int>& stones) {
        int n = stones.size();
        vector<int> pre(n + 1);
        for (int i = 1; i <= n; ++i) pre[i] = pre[i - 1] + stones[i - 1];
        vector<vector<int>> dp(n, vector<int>(n));
        for (int len = 2; len <= n; ++len) {
            for (int l = 0, r = len - 1; r < n; ++l, ++r) {
                int t1 = pre[r + 1] - pre[l + 1] - dp[l + 1][r];
                int t2 = pre[r] - pre[l] - dp[l][r - 1];
                dp[l][r] = max(t1, t2);
            }
        }
        return dp[0][n - 1];
    }
};
```

---

## 二次优化——空间降维

> 由于我们每次都只依赖于前面一层的空间，于是我们可以将二维空间降为一维（类似背包的优化）

### Code

```c++
class Solution {
public:
    int stoneGameVII(vector<int>& stones) {
        int n = stones.size();
        vector<int> pre(n + 1);
        for (int i = 1; i <= n; ++i) pre[i] = pre[i - 1] + stones[i - 1];
        vector<int> dp(n);
        for (int len = 2; len <= n; ++len) {
            for (int l = 0, r = len - 1; r < n; ++l, ++r) {
                int t1 = pre[r + 1] - pre[l + 1] - dp[l + 1];
                int t2 = pre[r] - pre[l] - dp[l];
                dp[l] = max(t1, t2);
            }
        }
        return dp[0];
    }
};
```

---

# 例题——最长公共子序列

> Problem: [1143. 最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/description/)

## 思路

> 我们可以考虑维护一个二维dp数组，`dp[i][j]`表示第一个第一个串[0, i]和第二个串[0, j]的最长公共子串
>
> 假如s1[i] == s2[j]，那么`dp[i][j] = dp[i - 1][j - 1]`，通过未考虑到第i位和未考虑第j位的状态来转移
>
> 如果不相等，那么用`dp[i][j - 1]和dp[i - 1][j]`来更新（因为第i和第j已经没有匹配上了，所以用尽可能多的来匹配即可）

## Code

```c++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();
        vector<vector<int>> dp(m + 1, vector<int> (n + 1));
        for (int i = 1; i <= m; ++i) for (int j = 1; j <= n; ++j) {
            if (text1[i - 1] == text2[j - 1]) dp[i][j] = dp[i - 1][j - 1] + 1;
            else {
                dp[i][j] = max(dp[i][j - 1], dp[i - 1][j]);
            }
        }
        return dp[m][n];
    }
};
```

## 空间优化

> 转移时不依赖于上一层之前的层的状态，于是空间可以减去很多，但是并不能减到一维，因为既依赖于当前层的新数据又依赖于上一层的旧数据，于是我们可以用两层的滚动数组来交替使用就行

### Code

```c++
class Solution {
public:
    int longestCommonSubsequence(string text1, string text2) {
        int m = text1.size(), n = text2.size();
        vector<vector<int>> dp(2, vector<int> (n + 1));
        for (int i = 1; i <= m; ++i) for (int j = 1; j <= n; ++j) {
            int k = i % 2, ssr = k ^ 1;
            if (text1[i - 1] == text2[j - 1]) dp[k][j] = dp[ssr][j - 1] + 1;
            else {
                dp[k][j] = max(dp[k][j - 1], dp[ssr][j]);
            }
        }
        return max(dp[0][n], dp[1][n]);
    }
};
```

# 例题——编辑距离

> Problem: [72. 编辑距离](https://leetcode.cn/problems/edit-distance/description/)

## 思路

> 假定原串为s1，目标串为s2
>
> 我们首先要对可以进行的操作进行分析，s1删除一个字符相当于s2删除一个字符，s1替换一个字符相当于s2替换一个字符
>
> 我们可以维护一个二维的dp数组，`dp[i][j]`表示s1的前i个字符和s2的前j个字符变成相同的串所需的最少步数
>
> 上面的分析将操作等价转换后我们可以发现：`dp[i][j]`最多可以由`dp[i][j - 1] +1`转化而来（最差情况就在此基础上多添加一个字符）。同理`dp[i][j]`也最多可以由`dp[i - 1][j] + 1`转换而来，当然我们还可以使用替换操作，于是`dp[i - 1][j - 1] + 1`也可以（前面按照`dp[i - 1][j - 1]`来修改，第i和第j位直接替换修改）
>
> 上面是最差的情况，如果当前位置能直接匹配上，那么当前位置将不会消耗步数，可以直接由`dp[i - 1][j - 1]`转换，当然这种情况不一定是最优的，于是还需要与进行操作的步数来做比较取最小值

## Code

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        vector<vector<int>> dp(n + 1, vector<int> (m + 1));
        for (int i = 0; i <= n; ++i) dp[i][0] = i;
        for (int i = 0; i <= m; ++i) dp[0][i] = i;
        for (int i = 1; i <= n; ++i) for (int j = 1; j <= m; ++j) {
            if (word1[i - 1] == word2[j - 1]) dp[i][j] = 1 + min({dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1] - 1});
            else dp[i][j] = min({dp[i][j - 1], dp[i - 1][j], dp[i - 1][j - 1]}) + 1;
        }
        return dp[n][m];
    }
};
```

## 内存优化

> 这里我们可以发现，每次转移只依赖于上一层的旧数据和前一个的新数据，于是我们可以简单的将内存压缩为两层的交替数组

### Code

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        vector<vector<int>> dp(2, vector<int> (m + 1));
        for (int i = 0; i <= m; ++i) dp[0][i] = i;
        dp[1][0] = 1;
        for (int i = 1; i <= n; ++i) for (int j = 1; j <= m; ++j) {
            int k = i % 2, ssr = k ^ 1;
            dp[k][0] = i;
            if (word1[i - 1] == word2[j - 1]) dp[k][j] = 1 + min({dp[ssr][j], dp[k][j - 1], dp[ssr][j - 1] - 1});
            else dp[k][j] = min({dp[k][j - 1], dp[ssr][j], dp[ssr][j - 1]}) + 1;
        }
        return dp[n % 2][m];
    }
};
```

### 进一步内存优化

> 我们发现在当前层值依赖于前一个的新数据，于是我们可以用一个变量暂时记录，最终将数组压缩成一维的

#### Code

```c++
class Solution {
public:
    int minDistance(string word1, string word2) {
        int n = word1.size(), m = word2.size();
        vector<int> dp(m + 1);
        for (int i = 0; i <= m; ++i) dp[i] = i;
        for (int i = 1; i <= n; ++i) {
            int ssr = i;
            for (int j = 1; j <= m; ++j) {
                int yyds = 0;
                if (word1[i - 1] == word2[j - 1]) yyds = 1 + min({ dp[j], ssr, dp[j - 1] - 1 });
                else yyds = min({ ssr, dp[j], dp[j - 1] }) + 1;
                dp[j - 1] = ssr, ssr = yyds;
            }
            dp[m] = ssr;
        }
        return dp[m];
    }
};
```

---

# LCR珠宝的最高价值

> Problem: [LCR 166. 珠宝的最高价值](https://leetcode.cn/problems/li-wu-de-zui-da-jie-zhi-lcof/description/)

## 说明

> 这题非常的easy，就是个经典的dp问题，这里主要探讨的是如何优化空间
>
> 时间复杂度都是`O(mn)`也就是遍历完整个图，优化的都是空间部分

## 初代递推——无优化

> 我们可以直接通过`dp[i][j]`来记录到达从左上角到`[i][j]`所能获得的最大价值，因为只能向右和下移动，所以我们通过上一个和左边一个取最大值来转移即可
>
> 空间复杂度：`O(mn)`

### Code

```c++
class Solution {
public:
    int jewelleryValue(vector<vector<int>>& frame) {
        int n = frame.size(), m = frame[0].size();
        vector<vector<int>> dp(n, vector<int>(m, 0));
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < m; ++j) {
                if (j) dp[i][j] = dp[i][j - 1];
                if (i) dp[i][j] = max(dp[i][j], dp[i - 1][j]);
                dp[i][j] += frame[i][j];
            }
        }
        return dp[n - 1][m - 1];
    }
};
```

---

## 优化1——滚动数组

> 由于，我们只依赖于当前层左边和上一层的数据，所以我们其实只需要两层就可以完成同样的功能
>
> 空间复杂度：`O(2m)`，也就是`O(m)`

### Code

```c++
class Solution {
public:
    int jewelleryValue(vector<vector<int>>& frame) {
        int n = frame.size(), m = frame[0].size();
        vector<vector<int>> dp(2, vector<int>(m, 0));
        for (int i = 0; i < n; ++i) {
            int k = i % 2, ssr = k ^ 1;
            for (int j = 0; j < m; ++j) {
                if (j) dp[k][j] = dp[k][j - 1];
                dp[k][j] = max(dp[k][j], dp[ssr][j]);
                dp[k][j] += frame[i][j];
            }
        }
        return dp[(n - 1) % 2][m - 1];
    }
};
```

----

## 优化2——一维数组

> 其实我们依赖于上一层的元素`dp[i - 1][j]`只是为了更新`dp[i][j]`，于是我们其实可以将新的`dp[i][j]`的值覆盖到原本位置上，这样我们就只需要一维数组即可实现同样功能
>
> 空间复杂度：`O(m)`，单纯就一层

### Code

```c++
class Solution {
public:
    int jewelleryValue(vector<vector<int>>& frame) {
        int n = frame.size(), m = frame[0].size();
        vector<int> dp(m);
        dp[0] = frame[0][0];
        for (int i = 1; i < m; ++i) dp[i] = frame[0][i] + dp[i - 1];
        for (int i = 1; i < n; ++i) {
            dp[0] += frame[i][0];
            for (int j = 1; j < m; ++j) {
                dp[j] = max(dp[j], dp[j - 1]) + frame[i][j];
            }
        }
        return dp[m - 1];
    }
};
```

---

## 优化3——原地修改

> 其实我们可以注意到原本提供的数组也有同样的大小，我们不用开辟新的空间就能实现同样功能
>
> 空间复杂度：`O(1)`

```c++
class Solution {
public:
    int jewelleryValue(vector<vector<int>>& frame) {
        int n = frame.size(), m = frame[0].size();
        // vector<int> dp(m);
        // dp[0] = frame[0][0];
        auto& dp = frame[0];
        for (int i = 1; i < m; ++i) dp[i] += dp[i - 1];
        for (int i = 1; i < n; ++i) {
            dp[0] += frame[i][0];
            for (int j = 1; j < m; ++j) {
                dp[j] = max(dp[j], dp[j - 1]) + frame[i][j];
            }
        }
        return dp[m - 1];
    }
};
```

---

# 区间DP

## 例题——无重叠区间

> Problem: [435. 无重叠区间](https://leetcode.cn/problems/non-overlapping-intervals/description/)

### 思路1——动态规划

> 这题实际上就是求不重叠区间的最大个数
>
> 我们可以将所有区间先按照开始时间（或者结束时间）排序。然后去求以每一个区间为结尾的最大区间个数，这个转移的方程就是官方方案中的往前去找所有结束时间比当前开始时间早的区间的dp值，但很显然这么写复杂度过高了，我们比较了许多没用的值。
> 其实我们可以类比最长上升子序列的问题，那题的优化就是另开一个数组记录某个长度下最小的结尾值。这里也是如此，我们记录下某个长度下最早的结束时间即可
> 这个长度的数组又是显然单调的，可以二分查找优化一下

#### Code

```c++
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end());
        int n = intervals.size();
        vector<int> dp(n + 1, 1e9);
        int ma = 0;
        for (int i = 0; i < n; ++i) {
            auto& v = intervals[i];
            int l = 1, r = ma;
            while (l <= r) {
                int m = l + r >> 1;
                if (v[0] >= dp[m]) l = m + 1;
                else r = m - 1;
            }
            ma = max(ma, ++r);
            dp[r] = min(dp[r], v[1]);
        }
        return n - ma;
    }
};
```

---

### 思路2——贪心

> 这题官方还有一种非常省空间的贪心
>
> 假如我们当前的最优解的最左区间为k，如果存在区间l的结束时间比k的结束时间还早，那么我们用j区间代替k区间将会更优（因为给后面预留的时间更加多了，且不减少总个数）
>
> 于是我们按结束时间排个序然后遍历一下就行了

#### Code

```c++
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), [&](vector<int>& a, vector<int>& b) {
            return a[1] < b[1];
        });
        int r = intervals[0][1];
        int ans = 1, n = intervals.size();
        for (int i = 1; i < n; ++i) {
            if (intervals[i][0] >= r) {
                ++ans;
                r = intervals[i][1];
            }
        }
        return n - ans;
    }
};
```

## 例题——用最少数量的箭引爆气球

> Problem: [452. 用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/description/)

### 思路

> 实际上这题我们可以这么转化，每个区间独立的气球一定需要一次机会去引爆，而区间不独立的气球则一定与某个独立区间的气球相交，我们引爆独立气球的同时也会一起引爆，不用理会
>
> 于是问题转化成了求独立区间的最大个数，直接沿用上题的做法即可

### Code

```c++
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& points) {
        sort(points.begin(), points.end(), [&](vector<int>& a, vector<int>& b) {
            return a[1] < b[1];
        });
        int r = points[0][1];
        int ans = 1, n = points.size();
        for (int i = 1; i < n; ++i) {
            if (points[i][0] > r) {
                ++ans;
                r = points[i][1];
            }
        }
        return ans;
    }
};
```

---

## 例题——规划兼职工作

> Problem: [1235. 规划兼职工作](https://leetcode.cn/problems/maximum-profit-in-job-scheduling/description/)

### 思路

> 这题其实就是一眼动态规划的题目，然后由于区间有先后，排序后更容易通过二分选取合适的插入位置，同时看了一眼数据量暴力n^2确实过不去，肯定考虑通过二分来优化

### 解决

> 这里我们考虑设计一个dp数组，dp[i]表示考虑到第i个元素所能获得的最大价值，我们先根据结尾时间来排序，然后通过二分找到最后一个可能的时间下标（也就是结束时间早于当前开始时间的最晚时间元素），则我们对当前位置可以有选或不选两种选择，不选则直接通过dp[i - 1]来转移，选则通过dp[k] + profit[i]来转移，二者取最大值即可

### Code

```c++
class Solution {
    using tii = tuple<int, int, int>;
public:
    int jobScheduling(vector<int>& startTime, vector<int>& endTime, vector<int>& profit) {
        int n = startTime.size();
        vector<tii> v;
        for (int i = 0; i < n; ++i) v.emplace_back(endTime[i], startTime[i], profit[i]);
        sort(v.begin(), v.end());
        vector<int> dp(n + 1);
        auto find = [&] (const int& time) {
            int l = 0, r = n - 1;
            int ans = -1;
            while (l <= r) {
                int m = l + r >> 1;
                if (get<0>(v[m]) > time) r = m - 1;
                else l = m + 1, ans = m;
            }
            return ans;
        };
        for (int i = 0; i < n; ++i) {
            int time = find(get<1>(v[i]));
            if (time != -1) dp[i] = max(dp[time] + get<2>(v[i]), dp[i - 1]);
            else dp[i] = max(get<2>(v[i]), i ? dp[i - 1] : 0);
        }
        return dp[n - 1];
    }
};
```

---

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

---

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

---

## 戳气球

> Problem: [312. 戳气球](https://leetcode.cn/problems/burst-balloons/description/)

### 思路

> 这题首先贪心走一遍，发现不能贪心，最值问题往`dp`上去想，然后一看数据范围那么小，多半是dp了
>
> 这里我们定义一个开区间的解决方案，`dp[i][j]`表示在开区间`(i, j)`中所能获得的最大金币数量
>
> 要求解这个答案，我们只需要枚举这个区间内部的每个节点k，尝试让他们成为最后一个消除的点，我们最后一步消除k所能获得的金币为`num[i] * num[j] * num[k]`，而在消除k之前所能获得的总和为`dp[i][k] + dp[k][j]`，因为左右两边的区间互不干涉，于是可以直接相加
>
> 于是最终的状态转移方程也就变成了：
> $$
> dp[i][j] = \max_{k = i + 1}^{j - 1} {dp[i][k] + dp[k][j] + num[i] * num[j] * num[k]}
> $$
> 代码也就呼之欲出了

### Code

```c++
class Solution {
    int n;
    vector<vector<int>> dp;

    int Find(int l, int r, const vector<int>& nums)
    {
        int& ssr = dp[l][r];
        if (ssr != -1) return ssr;
        if (r - l == 1) {
            return ssr = 0;
        }
        for (int i = l + 1; i < r; ++i) 
            ssr = max(ssr, (l ? nums[l - 1] : 1) * (r <= n ? nums[r - 1] : 1) * nums[i - 1] + Find(l, i, nums) + Find(i, r, nums));
        return ssr;
    }
public:
    int maxCoins(vector<int>& nums) {
        n = nums.size();
        dp.resize(n + 2, vector<int>(n + 2, -1));
        return Find(0, n + 1, nums);
    }
};
```

---

### 递归转递推

> 我们发现大的区间范围肯定依赖于小的区间范围，所以我们递推的时候先处理小范围，然后逐渐往大范围推，左右区间正向反向就没那么重要了，~~能跑就行~~

#### Code

```c++
class Solution {
    int n;
    vector<vector<int>> dp;
public:
    int maxCoins(vector<int>& nums) {
        n = nums.size();
        dp.resize(n + 2, vector<int>(n + 2, 0));
        for (int len = 3; len <= n + 2; ++len) {
            for (int l = 0; l <= n + 2 - len; ++l) {
                int r = l + len - 1;
                int& ssr = dp[l][r];
                for (int i = l + 1; i < r; ++i) {
                    ssr = max(ssr, (l ? nums[l - 1] : 1) * (r <= n ? nums[r - 1] : 1) * nums[i - 1] + dp[l][i] + dp[i][r]);
                }
            }
        }
        return dp[0][n + 1];
    }
};
```

---

# 背包问题

> 背包问题主要是可以将状态分为考虑当前前多少个物品且空间为某某的情况下的最优价值
>
> 当然背包问题也有很多的分类，多多积累就好了

## 01背包

> 可以说最简单的一种背包，就是空间有限，每个物品只能选一个
>
> 通常01背包的空间都是可以优化的，也就是当前层的状态只与上一层有关，可以通过滚动数组优化大量空间

### 例题——使数组和小于等于 x 的最少时间

> Problem: [2809. 使数组和小于等于 x 的最少时间](https://leetcode.cn/problems/minimum-time-to-make-array-sum-at-most-x/description/)

#### 思路

> 我们经过分析可以得到每个位置最多只会被清空一次，多次清空没有意义
>
> 于是问题就变成了给这n个位子的清空排个合理的顺序实现最优
>
> 计sum1为nums1的和，sum2为num2的和：
>
> 则每轮所有数字的和就会增加sum2，于是问题就变成了我们需要知道在到第t轮时我们总共能清空的最多数为多少
>
> 那么我们就可以用一个二维数组`dp[i][j]`来记录当考虑前`i`个物品且进行`j`次清空操作所能清空的最大值，前面分析过每个位置都只会被清空一次，于是`j > i`是没有意义的
>
> 但是每个位置都有可能排在后面，我们如果每次将所有位置都尝试一遍会浪费大量时间
>
> 其实清空的顺序应该按照增长速度从小到大来排的，因为如果增速快的先被清空了，那么相比同等情况下清空小的会有更多的回合数增加大的值，所以肯定是优先清空增速慢的
>
> 经过以上贪心的分析，这个清空的顺序也就出来了，我们按照nums2排序，从小到大开始清空，这样问题变成了一个类似01背包的问题

#### Code

```c++
class Solution {
public:
    int minimumTime(vector<int>& nums1, vector<int>& nums2, int tar) {
        vector<pair<int, int>> ssr;
        int sum1 = 0, sum2 = 0;
        int n = nums1.size();
        int i, j;
        for (i = 0; i < n; ++i) sum1 += nums1[i], sum2 += nums2[i], ssr.emplace_back(pair(nums2[i], i));
        sort(ssr.begin(), ssr.end());
        vector<vector<int>> dp(n, vector<int>(n + 1, 0));
        for (i = 1; i < n + 1; ++i) dp[0][i] = ssr[0].first * i + nums1[ssr[0].second];
        for (i = 1; i < n; ++i) {
            auto [x, y] = ssr[i];
            for (j = 1; j < n + 1; ++j) {
                dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - 1] + x * j + nums1[y]);
            }
        }
        for (i = 0; i < n + 1; ++i) 
            if (sum1 + sum2 * i - dp[n - 1][i] <= tar) 
                return i;
        return -1;
    }
};
```

---

#### 优化

> 我们发现每次的dp状态只与上一层的有关，我们可以通过滚动数组进行优化

##### Code

```c++
class Solution {
public:
    int minimumTime(vector<int>& nums1, vector<int>& nums2, int tar) {
        int n = nums1.size();
        int dp[n + 1];
        pair<int, int> ssr[n];
        memset(dp, 0, sizeof(dp));
        for (int i = 0; i < n; ++i) ssr[i] = {nums2[i], nums1[i]};
        sort(ssr, ssr + n);
        for (int i = 0; i < n; ++i) {
            int x = ssr[i].first, y = ssr[i].second;
            for (int j = i + 1; j; --j) {
                dp[j] = max(dp[j], dp[j - 1] + x * j + y);
            }
        }
        int s1 = accumulate(nums1.begin(), nums1.end(), 0);
        int s2 = accumulate(nums2.begin(), nums2.end(), 0);
        int s = 0;
        for (int i = 0; i < n + 1; ++i) {
            if (s1 + s - dp[i] <= tar) return i;
            s += s2;
        }
        return -1;
    }
};
```

---

### 给墙壁刷油漆

> Problem: [2742. 给墙壁刷油漆](https://leetcode.cn/problems/painting-the-walls/description/)

#### 思路

> 我们对于最终的方案，肯定满足：
> $$
> \begin{cases}
> 免费个数<=付费时间 \\
> 免费个数=n-付费个数
> \end{cases}
> $$
> 所以$n-付费个数<=付费时间$
>
> 移项可得：$n <= \sum(付费时间+1)$
>
> 所以我们的问题就抽象成了使时间不小于n的最小开销，我们可以构造一个01背包：
>
> `dp[i][j]`表示考虑前i个物品空间为j的情况下最小开销
>
> 状态转移方程为：
> $$
> dp[i][j]=
> \begin{cases}
> 0 & j <= 0 \\
> +\infty & i < 0 \\
> \min(dp[i - 1][j], dp[i - 1][j - time[i]] + cost[i]) & others
> \end{cases}
> $$

#### Code

```c++
class Solution {
    vector<vector<int>> dp;

    int dfs(int k, int time_need, const vector<int>& cost, const vector<int>& time)
    {
        if (time_need <= 0) return 0;
        if (k < 0) return 1e9 / 2;
        int& ssr = dp[k][time_need];
        if (ssr != -1) return ssr;
        return ssr = min(dfs(k - 1, time_need - time[k] - 1, cost, time) + cost[k], dfs(k - 1, time_need, cost, time));
    }

public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        dp.resize(n, vector<int>(n + 1, -1));
        return dfs(n - 1, n, cost, time);
    }
};
```

----

#### 优化

> 优化部分就是01背包的基操了，01背包一般都可以这么优化

##### 递归转递推

```c++
class Solution {
    vector<vector<int>> dp;
public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        dp.resize(n + 1, vector<int>(n + 1, 1e9));
        
        dp[0][0] = 0;
        for (int k = 0; k < n; ++k) {
            for (int t = 0; t <= n; ++t) {
                dp[k + 1][t] = dp[k][t];
                int ssr = t >= time[k] + 1 ? dp[k][t - time[k] - 1] : 0;
                dp[k + 1][t] = min(dp[k + 1][t], ssr + cost[k]);
            }
        }
        return dp[n][n];
    }
};
```

---

##### 空间优化——翻转数组

```c++
class Solution {
    vector<vector<int>> dp;

public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        dp.resize(2, vector<int>(n + 1, 1e9));
        
        dp[0][0] = 0;
        for (int k = 0; k < n; ++k) {
            int y = k % 2;
            int z = y ^ 1;
            for (int t = 0; t <= n; ++t) {
                dp[z][t] = dp[y][t];
                int ssr = t >= time[k] + 1 ? dp[y][t - time[k] - 1] : 0;
                dp[z][t] = min(dp[z][t], ssr + cost[k]);
            }
        }
        return dp[n % 2][n];
    }
};
```

---

##### 空间再优化——一维数组

```c++
class Solution {
    vector<int> dp;

public:
    int paintWalls(vector<int>& cost, vector<int>& time) {
        int n = cost.size();
        dp.resize(n + 1, 1e9);
        
        dp[0] = 0;
        for (int k = 0; k < n; ++k) {
            for (int t = n; t >= 0; --t) {
                int ssr = t >= time[k] + 1 ? dp[t - time[k] - 1] : 0;
                dp[t] = min(dp[t], ssr + cost[k]);
            }
        }
        return dp[n];
    }
};
```

---

### 目标和

> Problem: [494. 目标和](https://leetcode.cn/problems/target-sum/description/)

#### 思路

> 对于每一个数我们可以考虑选不选，我们定义`find(target, k)`表示考虑到第k个数时能凑到`target`的方案数，则很好推出状态转移方程
> $$
> find(target, k) =
> \begin{cases}
> target == 0 & k == n \\
> find(target + nums[k], k + 1) + find(target - nums[k], k + 1) & others
> \end{cases}
> $$
> 有了状态转移方程就可以直接莽着出答案了

#### Code

```c++
class Solution {
    int n;
    int find(int target, int k, const vector<int>& nums)
    {
        if (k == n) {
            return target == 0;
        }
        return find(target - nums[k], k + 1, nums) + find(target + nums[k], k + 1, nums);
    }

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        return find(target, 0, nums);
    }
};
```

---

#### 优化——记忆化搜索

> 虽然刚刚几行代码就结束了这个问题，但是能过是因为题目给的数据范围非常的小，其实我们是有很多的重复计算的，这些部分我们可以用记忆化搜索来减去重复计算的部分
>
> 定义的记忆化数组根据递归函数的参数来即可，一一对应
>
> 这里没有多想，直接暴力将数组开到可能的最大，把负数完全加成非负数，以免下标越界

##### Code

```c++
class Solution {
    int n;
    vector<vector<int>> dp;
    int find(int target, int k, const vector<int>& nums)
    {
        int& ssr = dp[k][target + 20000];
        if (ssr != -1) return ssr;
        if (k == n) {
            return ssr = target == 0;
        }
        return ssr = find(target - nums[k], k + 1, nums) + find(target + nums[k], k + 1, nums);
    }

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        dp.resize(n + 1, vector<int>(40003, -1));
        return find(target, 0, nums);
    }
};
```

---

#### 优化2——01背包转思路

> 其实这题的思路已经很接近一个01背包问题了，01背包就是关于取或不取对最后价值的关系影响，这题很容易看出来是要选取其中某几个物品最终达到指定的空间。
>
> 那么难点其实就在怎么优化这个空间，使得空间尽量不要浪费
>
> 我们定义：
> $$
> s=所有数的和 \\
> p=所有取正的数的和 \\
> q=所有取负的数的和 \\
> $$
> 那么就会有：
> $$
> \begin{cases}
> s=p+q \\
> target=p-q
> \end{cases}
> $$
> 联立可得：
> $$
> \begin{cases}
> p=\frac {s+target} {2} \\
> q=\frac {s-target} {2}
> \end{cases}
> $$
> 则很明显，我们既可以凑负数也可以凑正数结果都是一样的，因为除了取正的数剩余就是取负的数，为了最优，我们选取小的来凑，也就是$ssr = \frac {s - |target|} {2}$，不去考虑到底是选正的还是选负的
>
> 最终问题抽象成对于n个数每个数可以选或不选，最终凑得`ssr`即可
>
> 在处理的时候由于最终的ssr肯定是个整数，于是$(s-|target|) \% 2 == 0$，并且数组中的数全是非负数，如果$ssr < 0$那么也直接返回即可，这两种情况都是不可能的
>
> 思路上其实没有多大的变化，只是这么一想空间几乎被压缩到了极致

##### Code

```c++
class Solution {
    int n, m;
    vector<vector<int>> dp;
    int find(int target, int k, const vector<int>& nums)
    {
        if (target < 0) return 0;
        int& ssr = dp[k][target];
        if (ssr != -1) return ssr;
        if (k == n) {
            return ssr = target == 0;
        }
        return ssr = find(target - nums[k], k + 1, nums) + find(target, k + 1, nums);
    }

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        m = reduce(nums.begin(), nums.end(), 0) - abs(target);
        if (m < 0 || m % 2) return 0;
        m >>= 1;
        dp.resize(n + 1, vector<int>(m + 1, -1));
        return find(m, 0, nums);
    }
};
```

----

##### 递归转递推

> 此时递归转递推也非常好转，将记忆化数组当成递推数组，然后按照归的过程正向即可完成递推

###### Code

```c++
class Solution {
    int n, m;
    vector<vector<int>> dp;

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        m = reduce(nums.begin(), nums.end(), 0) - abs(target);
        if (m < 0 || m % 2) return 0;
        m >>= 1;
        dp.resize(n + 1, vector<int>(m + 1, 0));
        dp[n][0] = 1;
        for (int k = n - 1; k >= 0; --k) {
            for (int target = 0; target <= m; ++target) {
                dp[k][target] = dp[k + 1][target];
                if (target >= nums[k]) dp[k][target] += dp[k + 1][target - nums[k]];
            }
        }
        return dp[0][m];
    }
};
```

---

##### 空间优化——翻转数组

> 01背包基操，不谈。很明显只依赖于两层，翻转着能省掉剩余的空间

###### Code

```c++
class Solution {
    int n, m;
    vector<vector<int>> dp;

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        m = reduce(nums.begin(), nums.end(), 0) - abs(target);
        if (m < 0 || m % 2) return 0;
        m >>= 1;
        dp.resize(2, vector<int>(m + 1, 0));
        dp[n % 2][0] = 1;
        for (int k = n - 1; k >= 0; --k) {
            int ssr = k % 2;
            int z = ssr ^ 1;
            for (int target = 0; target <= m; ++target) {
                dp[ssr][target] = dp[z][target];
                if (target >= nums[k]) dp[ssr][target] += dp[z][target - nums[k]];
            }
        }
        return dp[0][m];
    }
};
```

---

##### 空间再优化——一维数组

> 同理基操，由于当前层只依赖于上一层，而且都是左上角的数据，于是用一维数组反向遍历的方式也能处理，这样不仅优化了空间，而且对于许多拷贝和计算的操作也省略了

###### Code

```c++
class Solution {
    int n, m;
    vector<int> dp;

public:
    int findTargetSumWays(vector<int>& nums, int target) {
        n = nums.size();
        m = reduce(nums.begin(), nums.end(), 0) - abs(target);
        if (m < 0 || m % 2) return 0;
        m >>= 1;
        dp.resize(m + 1, 0);
        dp[0] = 1;
        for (int k = n - 1; k >= 0; --k) {
            for (int target = m; target >= nums[k]; --target) {
                dp[target] += dp[target - nums[k]];
            }
        }
        return dp[m];
    }
};
```

---

## 完全背包

> 完全背包就是物品可以无限制的选取，解决这类题目的方法就是以当前层前面刚更新的新数据作为参考来转移

### 例题——组合总和 Ⅳ

> Problem: [377. 组合总和 Ⅳ](https://leetcode.cn/problems/combination-sum-iv/description/)

#### 思路

> 这题不像传统的完全背包，这里的数据选取顺序不同也能算上，所以我们第一层不能枚举每个物品了，那样不会考虑到物品的顺序问题
>
> 我们第一层应该枚举背包大小，第二层枚举每个物品，理由如下：
>
> 对于每个空间大小v，我们的方案可以是任意一种物品放在最后一个，也就是从dp[v - t]由添加大小为t的物品来转移过来的方案，所以第二层才去枚举物品种类可以考虑到顺序问题

#### Code

```c++
class Solution {
public:
    int combinationSum4(vector<int>& nums, int target) {
        int n = nums.size();
        vector<int> dp(target + 1);
        dp[0] = 1;
        for (int i = 1; i <= target; ++i) {
            for (const int& v: nums) {
                if (i >= v && dp[i - v] < INT_MAX - dp[i]) dp[i] += dp[i - v];
            }
        }
        return dp[target];
    }
};
```

---

# 状压dp

## 例题——参加考试的最大学生数

> Problem: [1349. 参加考试的最大学生数](https://leetcode.cn/problems/maximum-students-taking-exam/description/)

### 思路

> 这题显然我们得通过dp来考虑，但我们并不能像往常一样一个坑一个坑来dp考虑，我们将一整行的组合来当做一种状态来考虑
>
> 我们用dp\[i]\[j]表示当第i行的组合为j时，前i行的最大学生数
>
> 我们可以枚举当前行可能的组合，然后加上在这种组合情况下，之前行所能得到的最大结果即可得到当前行的dp\[i]。
>
> 一行最多8个位置，我们一位一位比过去虽然也还好，但是我们有更优的方案做判断：由于每个位置只有坐与不坐两种状态，也就是说通过一个二进制位刚好就能表示一个位置的状态，那么一行的状态也就通过几个二进制数就可以表示了，我们将这个状态映射成这些二进制数组成的值
>
> 这样子我们通过简单的位运算就可以快速判断某些组合是否合理，以及能否与前面的某种状态搭配了

---

### Code

```c++
class Solution {
public:
    int maxStudents(vector<vector<char>>& seats) {
        auto lowbit = [&] (int x) {
            return x & -x;
        };

        auto Check = [&] (int low, int high) {
            if ((low << 1) & high) return false;
            if ((low >> 1) & high) return false;
            return true;
        };

        int n = seats.size(), m = seats[0].size();
        int t = 1 << m;
        vector<vector<int>> dp(n, vector<int>(t));
        vector<int> mp(n), Has[n];
        for (int i = 0; i < n; ++i) {
            int sum = 0;
            for (int j = 0; j < m; ++j) {
                int z = seats[i][j] == '.' ? 1 : 0;
                sum = sum * 2 + z;
            }
            mp[i] = sum;
        }
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < t; ++j) {
                // 判断当前排列是否全是好用的位置,且不会左右相邻
                if ((j & mp[i]) == j && (((j << 1) & j) == 0)) {
                    Has[i].push_back(j);

                    // 先加上当前行排列的人数
                    int sum = 0, tmp = j;
                    while (tmp) {
                        ++sum;
                        tmp -= lowbit(tmp);
                    }
                    dp[i][j] = sum;

                    int ma = 0;
                    if (i != 0)
                        for (int x: Has[i - 1]) {
                            if (Check(j, x)) ma = max(ma, dp[i - 1][x]);
                        }
                    dp[i][j] += ma;
                } 
            }
        }
        int ans = 0;
        for (int x: Has[n - 1]) ans = max(ans, dp[n - 1][x]);
        return ans;
    }
};
```

---

### 细节优化

> 每次都去遍历上一层的每个状态是很浪费时间的，因为有很多位置根本不合理，比如安排人坐不能用的位置，或者左右相邻了。但其实我们在考虑当前行状态时已经判断完这个了，下一层的时候再回过头重新判断一遍实在是浪费，于是我们可以开一个vector来记录每一层合理的组合方案，可以大大节约时间。
>
> 在判断当前行某种搭配所能容纳的人数时，我们可以直接遍历每一位，当然也可以用lowbit直接开减（个人感觉可能会快些），虽然差不了多少，毕竟也就最多8位

---

## 优美的排列

> Problem: [526. 优美的排列](https://leetcode.cn/problems/beautiful-arrangement/description/)

### 思路

> 我们可以一位一位"暴力"的尝试每个位置，比如总共5个数，p1此时填入了2，那么p2只能从除2以外的其他剩余数字中选取了，这样子我们尝试每个集合的剩余部分能有多少种组合即可，由于剩余部分我们对于如何排列的并不关心，只对于这个集合所能组成的排列数关心，所以这里可以记忆化搜索优化，避免多次计算
>
> 因为数据取值只有1-15，所以我们通过32位的int类型就能存储下各种的取值集合，这就可以通过位运算处理，也就是状态压缩，递归时传参也只需要传入集合即可，因为具体考虑到第几位可以通过集合中1的个数来获得
>
> 于是这个状态转移方程也就变成了：
> $$
> dfs(s)=
> \begin{cases}
> 1 & s == U \\
> \sum_{i=1}^n dfs(s \cup i) & i \notin s,i \mod pos=0 \or pos \mod i = 0
> \end{cases}
> $$
> 

### 复杂度分析

- 时间复杂度

	由于记忆化搜索优化，所以每个状态只计算了一次
	$$
	时间复杂度=状态个数*每个状态处理的时间
	$$
	所以这里状态个数为$2^n$，每个状态处理的时间为$n$，所以总的时间复杂度为$O(2^nn)$

- 空间复杂度

	空间复杂度也就是这个记忆化数组的大小，为$O(2^n)$

### Code

```c++
class Solution {
    vector<int> dp;
    int n;

    inline int lowbit(int x)
    {
        return x & -x;
    }

    int dfs(int s)
    {
        if (dp[s] != -1) return dp[s];
        int& ssr = dp[s];
        ssr = 0;
        int pos = 0, t = s;
        while (t) ++pos, t -= lowbit(t);
        if (pos == n) return ssr = 1;
        ++pos;
        for (int i = 1; i <= n; ++i) {
            if ((s & (1 << (i - 1))) == 0 && ((pos % i == 0) || (i % pos == 0))) {
                ssr += dfs(s | (1 << (i - 1)));
            }
        }
        return ssr;
    }
public:
    int countArrangement(int n) {
        dp.resize(1 << n, -1);
        this->n = n;
        return dfs(0);
    }
};
```

---

### 递归转递推

> 只要将递归中递的部分给去掉，自底向上来推就能实现递推，也就是何时到递归的边界，那么就将其当做递推的入口来推
>
> 因为我们每次状态转移依赖的数肯定是当前位按位或上一个新的1的数，所以在递推枚举状态的时候从大到小一个个枚举即可

#### Code

```c++
class Solution {
    inline int lowbit(int x)
    {
        return x & -x;
    }
public:
    int countArrangement(int n) {
        vector<int> dp(1 << n, 0);
        dp[(1 << n) - 1] = 1;
        for (int s = (1 << n) - 2; s >= 0; --s) {
            int t = s, pos = 0;
            while (t) ++pos, t -= lowbit(t);
            ++pos;
            for (int i = 1; i <= n; ++i) {
                if ((s & (1 << (i - 1))) == 0 && ((pos % i == 0) || (i % pos == 0))) {
                    dp[s] += dp[s | (1 << (i - 1))];
                }
            }
        }
        return dp[0];
    }
};
```

---

## 特别的排列

> Problem: [2741. 特别的排列](https://leetcode.cn/problems/special-permutations/description/)

### 思路

> 相较于`优美的排列`，这题是相邻相关的取值问题，所以我们需要额外记录一下前一位取的是什么
>
> 这题的取值范围比较大，但是总的个数不多，所以可以按数组下标进行状态压缩，其他思路同理

### Code

```c++
class Solution {
    using ll = long long;
    vector<vector<ll>> dp;
    int n;
    const int mod = 1e9 + 7;

    inline int lowbit(int x) 
    {
        return x & -x;
    }

    ll dfs(int s, int pre, const vector<int>& nums)
    {
        auto& ssr = dp[s][pre];
        if (ssr != -1) return ssr;
        ssr = 0;
        int t = s, pos = 0;
        while (t) ++pos, t -= lowbit(t);
        if (pos == n) return ssr = 1;
        ++pos;
        for (int i = 0; i < n; ++i) {
            if ((s & (1 << i)) == 0 && (pre == n || nums[i] % nums[pre] == 0 || nums[pre] % nums[i] == 0)) {
                ssr = (ssr + dfs(s | (1 << i), i, nums)) % mod;
            }
        }
        return ssr;
    }

public:
    int specialPerm(vector<int>& nums) {
        n = nums.size();
        dp.resize(1 << n, vector<ll>(n + 1, -1));
        return dfs(0, n, nums);
    }
};
```

### 递归转递推

```c++
class Solution {
    using ll = long long;
    vector<vector<ll>> dp;
    int n;
    const int mod = 1e9 + 7;

    inline int lowbit(int x) 
    {
        return x & -x;
    }

public:
    int specialPerm(vector<int>& nums) {
        n = nums.size();
        dp.resize(1 << n, vector<ll>(n, 0));
        for (int i = 0; i < n; ++i) dp[(1 << n) - 1][i] = 1;
        for (int s = (1 << n) - 2; s; --s) {
            for (int pre = 0; pre < n; ++pre) {
                if ((s & (1 << pre)) == 0) continue;
                for (int i = 0; i < n; ++i) {
                    if ((s & (1 << i)) == 0 && (nums[i] % nums[pre] == 0 || nums[pre] % nums[i] == 0)) {
                        dp[s][pre] += dp[s | (1 << i)][i];
                    }
                }
            }
        }

        ll ans = 0;
        for (int i = 0; i < n; ++i) ans += dp[1 << i][i];
        return ans % mod;
    }
};
```

---

## 到达第 K 级台阶的方案数

> Problem: [3154. 到达第 K 级台阶的方案数](https://leetcode.cn/problems/find-number-of-ways-to-reach-the-k-th-stair/description/)

### 思路

> 这题思路主要还是从dp出发去考虑。
>
> 我们在选取未来方案前是根据当前状态选择的，限制未来选择的状态有当前的位置，上一步是否回跳，以及当前jump了多少次，所以这些将作为状态的设定来考虑方案数
>
> 因为每次改变完状态后判断方法和原问题相似的子问题，所以可以通过递归来解决
>
> 这里由于到达终点后依旧可以回跳等操作继续增加方案数，所以到达目标并不是递归终点，只能让返回的记录数+1，真正的递归终点限制应该是在回跳上，因为不能连续回跳所以如果$now > k + 1$那么无论怎么操作都只会越变越大，所以无需继续递归了。而对于下限没有要求（虽然再低也不会太低）

### Code

```c++
class Solution {

    int dfs(int target, bool down, int jump)
    {
        if (target > 1) return 0;
        int res = 0;
        if (!target) res = 1;
        if (!down) res += dfs(target - 1, true, jump);
        res += dfs(target + (1 << jump), false, jump + 1);
        return res;
    }
public:
    int waysToReachStair(int k) {
        return dfs(1 - k, false, 0);
    }
};
```

---

### 优化——记忆化搜索（状态压缩）

> dp的递归一般都得搭配记忆化搜索来食用，这题也不例外，状态会被重复计算，所以可以采取记忆化搜索的方式保留之前搜索的结果来避免不必要的重复
>
> 但是这题有个问题，k是个巨大的值，也就是说我们在到达k之前的位置也会经历巨大的值，那下标将会巨大，咋记录呢？
>
> 首先想到的可以是用map来做映射，但是map咋能记录三个状态做key值呢，这里就得手动做映射了
>
> 因为当前位置是int，步数是int（不大于32），上一跳状态（1位），所以我们可以用64位（long long）来记录三个值的映射值
> $$
> key = now << 32 | jump << 1 | last
> $$
> 这样也能保证key值对应状态的唯一

#### Code

```c++
class Solution {
    using ll = long long;
    unordered_map<ll, int> dp;
    int dfs(int target, bool down, int jump)
    {
        if (target > 1) return 0;
        ll ssr = (ll)target << 32 | jump << 1 | down;
        if (dp.count(ssr)) return dp[ssr];
        int& res = dp[ssr];
        if (!target) res = 1;
        if (!down) res += dfs(target - 1, true, jump);
        res += dfs(target + (1 << jump), false, jump + 1);
        return res;
    }
public:
    int waysToReachStair(int k) {
        return dfs(1 - k, false, 0);
    }
};
```

---

### 再优化——全局共享记忆化

> 如果记录的是now以及上一条和当前跳那么重复度其实不会太高，我们可以设置成相对距离的跳跃，也就是与目标点相距target，另外两个状态不变，这样所有的样例其实都可以用同一份记忆化数组，平均下来能大大降低开销

#### Code

```c++
using ll = long long;
unordered_map<ll, int> dp;
class Solution {
    int dfs(int target, bool down, int jump)
    {
        if (target > 1) return 0;
        ll ssr = (ll)target << 32 | jump << 1 | down;
        if (dp.count(ssr)) return dp[ssr];
        int& res = dp[ssr];
        if (!target) res = 1;
        if (!down) res += dfs(target - 1, true, jump);
        res += dfs(target + (1 << jump), false, jump + 1);
        return res;
    }
public:
    int waysToReachStair(int k) {
        return dfs(1 - k, false, 0);
    }
};
```

---

# 数位dp

## 解释

> 常见的数位dp我们一般直接上灵神的模板，一位一位模拟过去
>
> 通常定义一个`int fun(int pos, int mask, bool Limit, bool Isnum)`函数
>
> 返回最终符合要求的答案数
>
> 参数表示为：
>
> 1. pos：当前考虑到的位数
> 2. mask：通过一个数的各个数位记录哪些数字受限了
> 3. Limit：记录前面的数字是否已经到达极限，若到达极限则当前的枚举将受限
> 4. Isnum：用来记录前面的数字是否为空
>
> 由于数位dp往往伴随着多次的重复筛选，于是我们可以通过记忆化搜索来优化处理时间
>
> 具体题目具体分析

## 例题——统计特殊整数

> Problem: [2376. 统计特殊整数](https://leetcode.cn/problems/count-special-integers/description/)

### 思路

> 直接上灵神模板就行，这就是个模板题

---

### Code

```c++

class Solution {
public:
    int countSpecialNumbers(int n) {
        string s = to_string(n);
        int len = s.size();
        int dp[len][1024];
        memset(dp, -1, sizeof(dp));
        
        function<int(int, int, bool, bool)> yyds = [&] (int pos, int mask, bool Limit, bool IsNum) {
            if (pos == len) {
                return int(IsNum);
            }

            if (!Limit && IsNum && dp[pos][mask] != -1) return dp[pos][mask];

            int res = 0;

            if (!IsNum) res += yyds(pos + 1, mask, false, false);

            int up = Limit ? s[pos] - '0' : 9;

            for (int i = !IsNum; i <= up; ++i) {
                if ((mask >> i) & 1) continue;
                res += yyds(pos + 1, mask | (1 << i), Limit && i == s[pos] - '0', true);
            }
            if (!Limit && IsNum) dp[pos][mask] = res;
            return res;
        };
        return yyds(0, 0, true, false);
    }
};
```

---

### 细节

> 细节的地方主要在于记忆化存储部分，我们的函数用到了四个变量来存储状态，然而，用来存储的数组居然只使用了两维
>
> 这是因为，记忆化的本质是记录重复计算得出来的结果，从而优化计算时间，所以如果没有重复计算的内容将不需要存储
>
> 我们从前往后调用的时候只有一种情况会使得limit为true，同样也只有一种情况会使Isnum为false，所以这两种情况单拎出来即可

---

### 细节2

> 判断已经完成计算的应该写在前面（被调用的时候直接return），这样在调用判断的时候就不用写又臭又长的判断了哈哈哈

---

## 例题——最大为N的数字组合

> Problem: [902. 最大为 N 的数字组合](https://leetcode.cn/problems/numbers-at-most-n-given-digit-set/description/)

### 思路

> 一样的板子题，这里甚至连数字相同的限制都没了，直接再少一维

---

### Code

```c++
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& digits, int n) {
        string s = to_string(n);
        n = s.size();
        int dp[n];
        memset(dp, -1, sizeof(dp));
        
        function<int(int, bool, bool)> yyds = [&] (int pos, bool Limit, bool Isnum) {
            if (pos == n) return int(Isnum);
            if (!Limit && Isnum && dp[pos] != -1) return dp[pos];
            int res = 0;
            if (!Isnum) res = yyds(pos + 1, false, false);
            char up = Limit ? s[pos] : '9';
            for (string& st: digits) {
                if (st[0] > up) break;
                res += yyds(pos + 1, Limit && st[0] == up, true);
            }
            if (!Limit && Isnum) dp[pos] = res;
            return res;
        };
        return yyds(0, true, false);
    }
};
```

---

## 例题——数字1的个数

> Problem: [233. 数字 1 的个数](https://leetcode.cn/problems/number-of-digit-one/description/)

### 思路

> 直接上模板即可
>
> 要注意的是，这里我们需要记录1出现的个数，这样函数传参中就多传了1的个数，同样的记忆化搜索就需要多记录这一位

---

### Code

```c++
class Solution {
public:
    int countDigitOne(int n) {
        string s = to_string(n);
        n = s.size();
        int dp[n][10];
        memset(dp, -1, sizeof(dp));
        function<int(int, int, bool, bool)> yyds = [&] (int pos, int cnt, bool Limit, bool Isnum) {
            if (pos == n) return Isnum ? cnt : 0;
            if (!Limit && Isnum && dp[pos][cnt] != -1) return dp[pos][cnt];
            int res = 0;
            if (!Isnum) res = yyds(pos + 1, cnt, false, false);
            int up = Limit ? s[pos] - '0' : 9;
            for (int i = !Isnum; i <= up; ++i) {
                res += yyds(pos + 1, cnt + (i == 1), Limit && i == up, true);
            }
            if (!Limit && Isnum) dp[pos][cnt] = res;
            return res;
        };
        return yyds(0, 0, true, false);
    }
};
```

---

## 例题——不含连续1的非负整数

> Problem: [600. 不含连续1的非负整数](https://leetcode.cn/problems/non-negative-integers-without-consecutive-ones/description/)

### 思路

> 这题和前面几乎一样，我采用的方式是将这个整数先化成二进制数的形式存储起来
>
> 然后每次记录一下前一个位是否用了1即可

---

### Code

```c++
class Solution {
public:
    int findIntegers(int n) {
        bool ssr[31];
        for (int i = 0; i < 31; ++i) {
            if (n & 1) ssr[30 - i] = true;
            else ssr[30 - i] = false;
            n >>= 1;
        }
        n = 31;
        int dp[n][2];
        memset(dp, -1, sizeof(dp));
        function<int(int, bool, bool)> yyds = [&] (int pos, bool One, bool Limit) {
            if (pos == n) return 1;
            if (!Limit && dp[pos][One] != -1) return dp[pos][One];
            bool up = Limit ? ssr[pos] : true;
            int res = 0;
            res += yyds(pos + 1, false, Limit && ssr[pos] == false);
            if (up && !One) res += yyds(pos + 1, true, Limit && ssr[pos] == true);
            if (!Limit) dp[pos][One] = res;
            return res;
        };
        return yyds(0, false, true);
    }
};
```

---

## 例题——统计整数数目

> Problem: [2719. 统计整数数目](https://leetcode.cn/problems/count-of-integers/description/)

### 思路

> 初步思路就是将这个分成两次数位dp的模板问题进行求解
>
> 将nums2求一次，然后nums1-1求一次
>
> 由于不太好处理nums1-1后的变化情况，于是我们求出nums1，然后单独判断nums1这一个数字

---

### Code

```c++
const int mod = 1e9 + 7;

class Solution {
public:
    int count(string num1, string num2, int min_sum, int max_sum) {
        int x = Find(num1, min_sum, max_sum);
        int y = Find(num2, min_sum, max_sum);
        int cnt = 0;
        for (char z: num1) cnt += z - '0';
        if (cnt >= min_sum && cnt <= max_sum) --x;
        return ((y - x) % mod + mod) % mod;
    }

    int Find(string& s, int l, int r)
    {
        int n = s.size();
        int dp[n][401];
        memset(dp, -1, sizeof(dp));
        function<int(int, int, bool)> yyds = [&] (int pos, int cnt, bool Limit) {
            if (pos == n) {
                if (cnt >= l && cnt <= r) return 1;
                return 0;
            }
            if (!Limit && dp[pos][cnt] != -1) return dp[pos][cnt];
            int res = 0;
            int up = Limit ? s[pos] - '0' : 9;
            for (int i = 0; i <= up; ++i) {
                res += yyds(pos + 1, cnt + i, Limit && (i == up));
                res %= mod;
            }
            if (!Limit) dp[pos][cnt] = res;
            return res;
        };
        return yyds(0, 0, true) % mod;
    }
};
```

---

### 小坑

> 这里减法取模容易求出负数，于是要加上取的模再取模以防万一

---

### 优化

> 是的，你没看错，本题还有优化
>
> 由于这里遍历的数不止有上界，还存在下界，于是我们也可以不止使用一个Limit标记来表示前面的位到达了上界，我们多用一个标记表示前面的位到达了下界
>
> 这样我们通过一次记忆化搜索即可实现，大大节约时间

---

#### Code

```c++
const int mod = 1e9 + 7;

class Solution {
public:
    int count(string num1, string num2, int min_sum, int max_sum) {
        int n = num2.size();
        num1 = string(n - num1.size(), '0') + num1;
        int dp[n][401];
        memset(dp, -1, sizeof(dp));

        function<int(int, int, bool, bool)> yyds = [&] (int pos, int cnt, bool Hi, bool Lo) {
            if (cnt > max_sum) return 0;
            if (pos == n) return int(cnt >= min_sum);

            if (!Hi && !Lo && dp[pos][cnt] != -1) return dp[pos][cnt];
            int res = 0;
            int hi = Hi ? num2[pos] - '0' : 9;
            int lo = Lo ? num1[pos] - '0' : 0;
            for (int i = lo; i <= hi; ++i) {
                res += yyds(pos + 1, cnt + i, Hi && (i == hi), Lo && (lo == i));
                res %= mod;
            }
            if (!Hi && !Lo) dp[pos][cnt] = res;
            return res;
        };
        return yyds(0, 0, true, true);
    }
};
```

---

# 树形DP

> 树形dp就是在树上做dp，原树的形状一般和子树会相似，所以对于整棵树的问题一般都能转移成子树的子问题

## 例题——二叉树的最大深度

> Problem: [104. 二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)

### 思路

> 求二叉树的最大深度，其实可以分成求左右两棵子树的最大深度，然后取最大+1就是当前树的最大深度了，这也就是通过dp来将大问题分为子问题，边界就是空树深度为0

### Code

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (!root) return 0;
        int l = maxDepth(root->left), r = maxDepth(root->right);
        return max(l, r) + 1;
    }
};
```

---

## 例题——二叉树的直径

> Problem: [543. 二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/description/)

### 思路

> 二叉树的直径实际上就是求二叉树中链的最长长度，而中间的转折点不一定是根节点
>
> 其实要解决这个问题我们只需要枚举转折点即可，然后去判断每个转折点的左右子树的最大深度连起来就是当前转折点的最大链长

### Code

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
    int ans = 0;
    int dfs(TreeNode* root)
    {
        if (!root) return 0;
        int l = dfs(root->left), r = dfs(root->right);
        ans = max(ans, l + r);
        return max(l, r) + 1;
    }
public:
    int diameterOfBinaryTree(TreeNode* root) {
        dfs(root);
        return ans;
    }
};
```

---

## 例题——感染二叉树需要的总时间

> Problem: [2385. 感染二叉树需要的总时间](https://leetcode.cn/problems/amount-of-time-for-binary-tree-to-be-infected/description/)

### 思路——直径&深度

> 这题其实一眼从start节点开始dfs判断最原举例即可，但是这样需要重新建图，因为没有指向父节点的指针，除非是从根节点开始，否则无法完成遍历
>
> 其实这题我们可以拆成两部分来看，假如从start点为根开始往下，我们可以不建图就找到start子树的最大深度，而对于要从start的父亲去遍历的另一边我们就通过求直径的方式来解决
>
> 我们固定这个树直径的一端为start节点，来求剩余的最长直径，这里默认将start子树的节点全部忽略即可，因为求深度之后这些节点已经没有意义了。
>
> 求深度可以沿用之前的思路，但是这里的要求是确定了一端，那么我们在枚举每个转折点的时候就要考虑左右子树中是否出现了start节点，如果出现了，那么此时计算的直径才是有效的，向上传递的链长也是如果出现了start节点那么那条链优先

### Code

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
    using pib = pair<int, bool>;
    int ans = 0;
    pib dfs(TreeNode* root, const int& start)
    {
        if (!root) return {0, false};
        auto [l, lf] = dfs(root->left, start);
        auto [r, rf] = dfs(root->right, start);
        if (root->val == start) {
            ans = max(l, r);
            return {0, true};
        }
        if (lf || rf) {
            ans = max(ans, l + r + 1);
            return {lf ? l + 1 : r + 1, true};
        }
        return {max(l, r) + 1, false};
    }

public:
    int amountOfTime(TreeNode* root, int start) {
        dfs(root, start);
        return ans;
    }
};
```

---

### 优化——1个返回值

> 在前面返回的时候我们返回了一个长度和一个是否找到start的bool值，但是其实后一个是可以优化掉的
>
> 我们都知道长度都是非负数，所以我们在没有找到start节点的时候就传回对应长度的相反数就可以用来判断了

#### Code

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
    int ans = 0;
    int dfs(TreeNode* root, const int& start)
    {
        if (!root) return 0;
        int l = dfs(root->left, start);
        int r = dfs(root->right, start);
        if (root->val == start) {
            ans = -min(l, r);
            return 1;
        }
        if (l > 0 || r > 0) {
            ans = max(ans, abs(l) + abs(r));
            return l > 0 ? l + 1 : r + 1;
        }
        return min(l, r) - 1;
    }

public:
    int amountOfTime(TreeNode* root, int start) {
        dfs(root, start);
        return ans;
    }
};
```

