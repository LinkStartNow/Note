## 除法求值

> Problem: [399. 除法求值](https://leetcode.cn/problems/evaluate-division/description/)

### 思路1——dfs递归

> 我们可以将所有的除法关系转换成一张图点之间的权值用商表示这样子相乘就能推出除法关系。然后通过是否能到达判断询问的两个字符串是否能算出结果，用一个集合存储已经存在的字符串来判断询问字符串是否合法
>
> 不过实现的过程太过于繁杂了，当初设计代码的时候也没怎么想，直接用字符串映射节点了

#### Code

```c++
class Solution {
    struct node
    {
        using pnv = pair<node*, double>;
        unordered_map<string, pnv> To;
    };
    
    unordered_map<node*, bool> vis;

    node* target;
    double ssr;

    void dfs(node* root, double value)
    {
        if (vis[root]) return;
        vis[root] = true;
        if (root == target) {
            ssr = value;
            return;
        }
        for (auto& [x, y]: root->To) {
            dfs(y.first, y.second * value);
        }
    }
public:
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        unordered_map<string, node*> mp;
        int n = equations.size();
        for (int i = 0; i < n; ++i) {
            string& a = equations[i][0];
            string& b = equations[i][1];
            if (!mp.count(a)) mp[a] = new node;
            if (!mp.count(b)) mp[b] = new node;
            mp[a]->To[b] = {mp[b], values[i]};
            mp[b]->To[a] = {mp[a], 1 / values[i]};
        }
        vector<double> ans;
        for (auto& v: queries) {
            string& a = v[0];
            string& b = v[1];
            if (!mp.count(a) || !mp.count(b)) {
                ans.emplace_back(-1.0);
                continue;
            }
            for (auto& [x, y]: mp) vis[y] = false;
            target = mp[b];
            ssr = -1;
            dfs(mp[a], 1);
            ans.emplace_back(ssr);
        }
        return ans;
    }
};
```

---

### 思路2——加权并查集

> 众所周知连通性的问题都可以用并查集来解决，这题也是，我们由以上dfs的推理可知，两个点如果连通，那么他们肯定是能求出解的，这样在维护完并查集的情况下，对于每次查询都可以O(1)判断出是否能求出值
>
> 当然，能求出值还不够，我们对并查集进行扩展，使之成为加权并查集来压缩路径，因为除法具有传递性支持这个压缩条件，所以这里可以通过并查集加权来优化，在维护完成后，对于每次查询也是O(1)就能求出结果

#### 对于映射的优化

> 因为上一份代码直接通过字符串映射的节点，这导致处理邻接关系的时候，也必须要借助字符串实现映射到点，然而这俩map映射不是同一个map，导致又要hash一次，非常的浪费
>
> 所以这里的优化是索性将所有字符串都直接映射成一个int值，然后重新构建我们熟悉的int值的图，其他的逻辑都没变

#### Code

```c++
using pii = pair<int, double>;

class Solution {
    vector<pii> Fa;
    pii GetFa(int x)
    {
        if (x == Fa[x].first) return Fa[x];
        pii f = GetFa(Fa[x].first);
        return Fa[x] = {f.first, Fa[x].second * f.second};
    }

    void Union(int x, int y, double val)
    {
        auto X = GetFa(x);
        auto Y = GetFa(y);
        double ssr = X.second / Y.second * val;
        Fa[Y.first] = {X.first, ssr};
    }

public:
    vector<double> calcEquation(vector<vector<string>>& equations, vector<double>& values, vector<vector<string>>& queries) {
        unordered_map<string, int> mp;
        int n = equations.size();
        for (int i = 0; i < n; ++i) {
            auto& v = equations[i];
            if (!mp.count(v[0])) mp[v[0]] = Fa.size(), Fa.emplace_back(Fa.size(), 1);
            if (!mp.count(v[1])) mp[v[1]] = Fa.size(), Fa.emplace_back(Fa.size(), 1);
            Union(mp[v[0]], mp[v[1]], values[i]);
        }
        vector<double> ans;
        for (auto& v: queries) {
            double res = -1;
            if (mp.count(v[0]) && mp.count(v[1])) {
                auto&& [x, v1] = GetFa(mp[v[0]]);
                auto&& [y, v2] = GetFa(mp[v[1]]);
                if (x == y) res = v2 / v1;
            }
            ans.emplace_back(res);
        }
        return ans;
    }
};
```

