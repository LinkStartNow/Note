### 到达目的地的方案数

> Problem: [1976. 到达目的地的方案数](https://leetcode.cn/problems/number-of-ways-to-arrive-at-destination/description/)

#### 思路——双Dijkstra

> 这题需要求最短路的数目，我们通过Dijkstra就可以轻易得到单源最短路，在得到终点（或起点）到任意点的最短路径长度后，我们可以借助这个距离来判断从点a到点b是否是从起点（或终点）到点b的最短路径，如果是，那么到a的所有最短路径就加到b中，然后顺着推到端点，输出端点的计数即可，这个过程就再用一遍Dijkstra就行

##### Code

```c++
using pii = pair<long long, int>;
constexpr int mod = 1e9 + 7;

class Solution {
    vector<long long> dis, dis2;
    vector<int> cnt;

    void dijkstra(vector<pii>* e, int root)
    {
        dis[root] = 0;
        priority_queue<pii> q;
        q.push({0, root});
        while (q.size()) {
            auto [d, u] = q.top(); q.pop();
            if (-d > dis[u]) continue;
            for (auto [v, cost]: e[u]) {
                if (dis[v] == -1 || dis[v] > cost - d) {
                    dis[v] = cost - d;
                    q.push({-dis[v], v});
                }
            }
        }
    }

    void d2(vector<pii>* e, int root)
    {
        dis2[root] = 0;
        priority_queue<pii> q;
        q.push({0, root});
        while (q.size()) {
            auto [d, u] = q.top(); q.pop();
            if (-d > dis2[u]) continue;
            for (auto [v, cost]: e[u]) {
                if (cost - d == dis[v]) cnt[v] = (cnt[v] + cnt[u]) % mod;
                if (dis2[v] == -1 || dis2[v] > cost - d) {
                    dis2[v] = cost - d;
                    q.push({-dis2[v], v});
                }
            }
        }
    }


public:
    int countPaths(int n, vector<vector<int>>& roads) {
        dis.resize(n, -1);
        cnt.resize(n, 0); dis2.resize(n, -1);
        cnt[n - 1] = 1;
        vector<pii> e[n];
        for (auto& v: roads) {
            e[v[0]].emplace_back(v[1], v[2]);
            e[v[1]].emplace_back(v[0], v[2]);
        }
        dijkstra(e, n - 1);
        d2(e, n - 1);
        return cnt[0];
    }
};
```

---

#### 优化

> 我们上面进行两遍Dijkstra的原因是难以判断某两点之间是否是最短路径，于是要先找出最短长度做标准来判定
>
> 但其实这里也是可以优化掉的，假如某个点成功更新了另一个点的最短距离，那么之前其他点到该点的最短路径也就全部作废，我们重新用当前点赋值就行。如果当前点到某点的距离与之前的最短长度相等，那么说明当前的路径也是满足目前最短距离的路径，加上去即可
>
> 这样我们一轮Dijkstra即可得出答案

##### Code

```c++
using pii = pair<long long, int>;
constexpr int mod = 1e9 + 7;

class Solution {
    vector<long long> dis;
    vector<long long> cnt;

    void dijkstra(vector<pii>* e, int root)
    {
        dis[root] = 0;
        priority_queue<pii> q;
        q.push({0, root});
        while (q.size()) {
            auto [d, u] = q.top(); q.pop();
            if (-d > dis[u]) continue;
            for (auto [v, cost]: e[u]) {
                if (dis[v] == -1 || dis[v] > cost - d) {
                    dis[v] = cost - d;
                    q.push({-dis[v], v});
                    cnt[v] = cnt[u];
                }
                else if (dis[v] == cost - d) cnt[v] = (cnt[u] + cnt[v]) % mod;
            }
        }
    }

public:
    int countPaths(int n, vector<vector<int>>& roads) {
        dis.resize(n, -1), cnt.resize(n, 0);
        cnt[n - 1] = 1;
        vector<pii> e[n];
        for (auto& v: roads) {
            e[v[0]].emplace_back(v[1], v[2]);
            e[v[1]].emplace_back(v[0], v[2]);
        }
        
        dis[n - 1] = 0;
        priority_queue<pii> q;
        q.push({0, n - 1});
        while (q.size()) {
            auto [d, u] = q.top(); q.pop();
            if (-d > dis[u]) continue;
            for (auto [v, cost]: e[u]) {
                if (dis[v] == -1 || dis[v] > cost - d) {
                    dis[v] = cost - d;
                    q.push({-dis[v], v});
                    cnt[v] = cnt[u];
                }
                else if (dis[v] == cost - d) cnt[v] = (cnt[u] + cnt[v]) % mod;
            }
        }
        return cnt[0];
    }
};
```

