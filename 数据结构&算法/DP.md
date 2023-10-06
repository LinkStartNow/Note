众所周知DP是比较抽象的，比较不好总结，所以我们先以例题为例

# 例题——打家劫舍

[题目链接]([198. 打家劫舍 - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber/))

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

[链接]([213. 打家劫舍 II - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber-ii/description/?envType=daily-question&envId=2023-09-17))

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

[链接]([337. 打家劫舍 III - 力扣（LeetCode）](https://leetcode.cn/problems/house-robber-iii/description/?envType=daily-question&envId=2023-09-18))

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

\> Problem: [188. 买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/description/)

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

