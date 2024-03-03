## 统计树中合法路径数目

> Problem: [2867. 统计树中的合法路径数目](https://leetcode.cn/problems/count-valid-paths-in-a-tree/description/)

### 思路

> 读完这题，大致方向就是从质数节点着手向外扩展，对于每个质数节点我们都只需要知道它的每条出边在到达第一个质数节点前的长度，然后答案就出来了

### 解题方法

> 这里其实通过并查集就可以维护相连的关系，同时控制一下质数节点，保证相邻的几个点连起来就是非质数串，再通过每组的祖先节点来统计链长度就能很快找出每个非质数节点所在链的长度了
> 记录的时候也只需记录质数节点，以及质数节点的各个出边
> 我最初的设想对于所有的数我遇到了再去判断这个数是否是质数，判断方法也是草草的用暴力遍历根号来解决，通过共用一个静态数组来将平均的时间复杂度卡到过关

#### Code

```c++
class Solution {
    static bool IsP[100007], Check[100007];
    bool IsPri(int x)
    {
        if (Check[x]) return IsP[x];
        Check[x] = true;
        if (x == 1) return IsP[1] = false;
        int t = sqrt(x);
        t = min(t, x - 1);
        for (int i = 2; i <= t; ++i) if (x % i == 0) return false;
        return IsP[x] = true;
    }

    struct node
    {
        vector<int> e;
    };

    int GetFa(int x, vector<int>& Fa)
    {
        return Fa[x] == x ? x : Fa[x] = GetFa(Fa[x], Fa);
    }

    void Union(int x, int y, vector<int>& Fa, vector<int>& has)
    {
        int X = GetFa(x, Fa), Y = GetFa(y, Fa);
        if (has[X] < has[Y]) {
            has[Y] += has[X];
            has[X] = 0;
            Fa[X] = Y;
        }
        else {
            has[X] += has[Y];
            has[Y] = 0;
            Fa[Y] = X;
        }
    }

public:

    long long countPaths(int n, vector<vector<int>>& edges) {
        unordered_map<int, node*> mp;
        vector<int> Fa;
        vector<int> has(n + 1, 1);
        for (int i = 0; i <= n; ++i) Fa.emplace_back(i);
        for (auto& v: edges) {
            int& a = v[0];
            int& b = v[1];
            if (IsPri(a)) {
                if (!mp.count(a)) mp[a] = new node;
                mp[a]->e.emplace_back(b);
                has[a] = 0;
            }
            if (IsPri(b)) {
                if (!mp.count(b)) mp[b] = new node;
                mp[b]->e.emplace_back(a);
                has[b] = 0;
            }
            if (!IsPri(a) && !IsPri(b)) Union(a, b, Fa, has);
        }
        long long ans = 0;
        for (auto& [_, p]: mp) {
            cout << _ << endl;
            long long c = 1;
            for (int& v: p->e) {
                int sum = has[GetFa(v, Fa)];
                ans += c * sum;
                c += sum;
            }
        }
        return ans;
    }
};

bool Solution::IsP[100007];
bool Solution::Check[100007];
```

---

### 质数筛优化

> 对于求质数，给的数据范围最大为1e5，于是我们直接一次性把所有质数判断完成也行，更加省时间，这里以极致的欧拉筛为例做优化

#### Code

```c++
constexpr int N = 1e5 + 7;

class Solution {
    static bool IsP[N], GetFalg;
    static vector<int> Pri;

    void InitPri()
    {
        GetFalg = true;
        IsP[1] = IsP[0] = true;
        for (int i = 2; i < N; ++i) {
            if (!IsP[i]) Pri.emplace_back(i);
            for (const int& x: Pri) {
                if (x * i >= N) break;
                IsP[x * i] = true;
                if (i % x == 0) break;
            }
        }
    }

    bool IsPri(int x)
    {
        if (!GetFalg) InitPri();
        return !IsP[x];
    }

    int GetFa(int x, vector<int>& Fa)
    {
        return Fa[x] == x ? x : Fa[x] = GetFa(Fa[x], Fa);
    }

    void Union(int x, int y, vector<int>& Fa, vector<int>& has)
    {
        int X = GetFa(x, Fa), Y = GetFa(y, Fa);
        if (has[X] < has[Y]) {
            has[Y] += has[X];
            has[X] = 0;
            Fa[X] = Y;
        }
        else {
            has[X] += has[Y];
            has[Y] = 0;
            Fa[Y] = X;
        }
    }

public:

    long long countPaths(int n, vector<vector<int>>& edges) {
        unordered_map<int, vector<int>> mp;
        vector<int> Fa;
        vector<int> has(n + 1, 1);
        for (int i = 0; i <= n; ++i) Fa.emplace_back(i);
        for (auto& v: edges) {
            int& a = v[0];
            int& b = v[1];
            if (IsPri(a)) {
                mp[a].emplace_back(b);
                has[a] = 0;
            }
            if (IsPri(b)) {
                mp[b].emplace_back(a);
                has[b] = 0;
            }
            if (!IsPri(a) && !IsPri(b)) Union(a, b, Fa, has);
        }
        long long ans = 0;
        for (auto& [_, p]: mp) {
            long long c = 1;
            for (int& v: p) {
                int sum = has[GetFa(v, Fa)];
                ans += c * sum;
                c += sum;
            }
        }
        return ans;
    }
};

bool Solution::IsP[N];
bool Solution::GetFalg = false;
vector<int> Solution::Pri;
```

---
