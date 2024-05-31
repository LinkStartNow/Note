# 最小生成树

> Problem:[[模板]最小生成树]([P3366 【模板】最小生成树 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P3366))

### 思路

> 可以用Kruskal算法求解最小生成树，在判断是否连通的时候我们可以通过并查集进行优化，快速判断连通

### Code

```c++
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

const int N = 1e5 + 7;

using ll = long long;

struct edge
{
    int u, v, w;

    edge(int u, int v, int w): u(u), v(v), w(w) {}
};

int n, m;

int GetFa(int x, vector<int>& f)
{
    return x == f[x] ? x : f[x] = GetFa(f[x], f);
}

void solve()
{
    cin >> n >> m;
    vector<edge> ssr;
    vector<int> f(n + 1);
    for (int i = 0; i <= n; ++i) f[i] = i;
    for (int i = 0; i < m; ++i) {
        int u, v, w;
        scanf("%d%d%d", &u, &v, &w);
        ssr.emplace_back(u, v, w);
    } 
    sort(ssr.begin(), ssr.end(), [](edge& a, edge& b){
        return a.w < b.w;
    });
    int cnt = 1, ans = 0;
    for (edge& x: ssr) {
        auto& [u, v, w] = x;
        int U = GetFa(u, f), V = GetFa(v, f);
        if (U == V) continue;
        f[U] = V;
        ans += w, ++cnt;
    }
    if (cnt == n) cout << ans;
    else cout << "orz";
}

int main()
{
    // freopen("in.txt", "r", stdin);
    solve();
    return 0;
}
```

---

#  线段树

> Problem:[[模板]线段树1]([P3372 【模板】线段树 1 - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P3372))

这是最基础版本的线段树应用，区间加和区间求和，甚至都还没涉及区间乘

## Code

```c++
#include <iostream>
#include <string>
#include <vector>

using namespace std;

const int N = 1e5 + 7;

using ll = long long;

vector<ll> s, lazy, a;

void build(int root, int l, int r)
{
    if (l > r) return;
    lazy[root] = 0;
    if (l == r) {
        s[root] = a[l];
        return;
    }
    int m = l + r >> 1;
    int ll = root << 1, rr = ll | 1;
    build(ll, l, m), build(rr, m + 1, r);
    s[root] = s[ll] + s[rr];
}

void put(int root, int l, int r)
{
    if (!lazy[root]) return;
    int ll = root << 1, rr = ll | 1;
    int m = l + r >> 1;
    lazy[ll] += lazy[root], lazy[rr] += lazy[root];
    s[ll] += lazy[root] * (m - l + 1);
    s[rr] += lazy[root] * (r - m);
    lazy[root] = 0;
}

void add(int root, int l, int r, int ln, int rn, ll k)
{
    if (ln > rn || l > rn || r < ln) return;
    if (ln >= l && rn <= r) {
        s[root] += k * (rn - ln + 1);
        lazy[root] += k;
        return;
    }
    put(root, ln, rn);
    int m = ln + rn >> 1;
    int ll = root << 1, rr = ll | 1;
    add(ll, l, r, ln, m, k), add(rr, l, r, m + 1, rn, k);
    s[root] = s[ll] + s[rr];
}

ll Getsum(int root, int l, int r, int ln, int rn)
{
    if (ln > rn || ln > r || rn < l) return 0;
    if (ln >= l && rn <= r) {
        return s[root];
    }
    put(root, ln, rn);
    int m = ln + rn >> 1;
    int ll = root << 1, rr = ll | 1;
    return Getsum(ll, l, r, ln, m) + Getsum(rr, l, r, m + 1, rn);
}

void solve()
{
    int n, m;
    cin >> n >> m;
    s.resize(n << 2), lazy.resize(n << 2), a.resize(n + 1);
    for (int i = 1; i <= n; ++i) scanf("%lld", &a[i]);
    build(1, 1, n);
    while (m--) {
        int k;
        scanf("%d", &k);
        ll a, b, c;
        switch (k) {
            case 1:
                scanf("%d%d%d", &a, &b, &c);
                add(1, a, b, 1, n, c);
                break;
            case 2:
                scanf("%d%d", &a, &b);
                printf("%lld\n", Getsum(1, a, b, 1, n));
        }
    }
}

int main()
{
    // freopen("in.txt", "r", stdin);
    solve();
    return 0;
}
```

