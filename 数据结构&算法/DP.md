众所周知DP是比较抽象的，比较不好总结，所以我们先以例题为例

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

