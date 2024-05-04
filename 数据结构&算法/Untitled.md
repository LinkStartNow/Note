## 例题——求阶乘

> Problem:[求阶乘]([P2060 - [蓝桥杯2022初赛\] 求阶乘 - New Online Judge (ecustacm.cn)](http://oj.ecustacm.cn/problem.php?id=2060))

### 思路

> 其实求末尾的0只需要求2和5的个数就行，他们相乘才能乘出0，但是2比5多很多，所以只需要考虑5
>
> 对于每个数字我们可以通过整除5求出5的倍数，这些数都能贡献一个0，整除25求出25的倍数，这些数可以共享2个0，但是由于他们同时也是5的倍数，在5那里已经贡献过一次了，这里只需要再贡献一次，以此类推即可，很容易求出某个阶乘末尾0的个数
>
> 但是这里数字范围很大，同时又很显然是个单调的答案假如5成立6肯定成立，但4不成立3一定不成立，所以可以用二分来做

### Code

```c++
#include <bits/stdc++.h>

using namespace std;

using ll = long long;

int main()
{
    ll k;
    cin >> k;
    ll ans = -1;
    ll l = 1, r = 1e19;
    auto check = [&] (ll x) {
        ll cnt = 0;
        ll ssr = 5;
        while (ssr <= x) {
            cnt += x / ssr;
            ssr *= 5;
        }
        return cnt;
    };
    while (l <= r) {
        ll m = l + (r - l + 1 >> 1);
        ll t = check(m);
        if (t >= k) {
            if (t == k) ans = m;
            r = m - 1;
        }
        else l = m + 1;
    }
    cout << ans << endl;
    return 0;
}
```

### 吐槽

> 这题数字给的特别大，最终的右区间要取一个很大的值，我这里自信取了1e18，被卡了woc
>
> 结果取1e19就能过

---

## 例题——分巧克力

> Problem[分巧克力]([P1323 - [蓝桥杯2017初赛\]分巧克力 - New Online Judge (ecustacm.cn)](http://oj.ecustacm.cn/problem.php?id=1323))

### 思路

> 我们想求出当前的巧克力可以分出多少个指定大小的巧克力可以直接用长和宽整除对应的边长，然后相乘来算出数量，这里只需要找到最小满足的即可，很显然答案满足单调性，可以通过二分来加快查找

### Code

```c++
#include <bits/stdc++.h>

using namespace std;

using ll = long long;

const int N = 1e5 + 7;

int n, k;
ll h[N], w[N];

bool check(ll x)
{
    ll num = 0;
    for (int i = 0; i < n; ++i) {
        num += (h[i] / x) * (w[i] / x);
        if (num >= k) return true;
    }
    return num >= k;
}

void solve()
{
    cin >> n >> k;
    for (int i = 0; i < n; ++i) scanf("%lld%lld", &h[i], &w[i]);
    ll l = 1, r = N;
    while (l <= r) {
        ll m = l + (r - l + 1 >> 1);
        if (check(m)) l = m + 1;
        else r = m - 1;
    }
    cout << r << endl;
}

int main()
{
    solve();
    return 0;
}
```

### 细节

> 这里算个数的时候，两个分别整除完再相乘，否则由于是整除答案会出错

---

## 例题——青蛙过河

> Problem:[青蛙过河]([P2026 - [蓝桥杯2022初赛\] 青蛙过河 - New Online Judge (ecustacm.cn)](http://oj.ecustacm.cn/problem.php?id=2026))

### 思路

> 这题的关键在于判断青蛙在某一跳跃能力的情况下能否过河，实际上只需要保证在跳跃能力为k的情况下，每k个相邻的石头高度和不小于2x即可。
>
> 我们可以将每k块区域看成一个整体，总共有2x只青蛙，我们只需要判断每个[l, l + k - 1]，到下一个区间[l + 1, l + k]是否都能成功就行，慢慢到终点，这样在满足要求的情况下一定能过河

### Code

```c++
#include <bits/stdc++.h>

using namespace std;

using ll = long long;

const int N = 1e5 + 7;

ll n, x;
ll a[N];

bool check(ll p)
{
    ll sum = 0;
    int i;
    for (i = 0; i < p; ++i) sum += a[i];
    while (i < n) {
        if (sum < x) return false;
        sum += a[i];
        sum -= a[i - p];
        ++i;
    }
    return sum >= x;
}

void solve()
{
    cin >> n >> x;
    --n;
    x <<= 1;
    for (int i = 0; i < n; ++i) scanf("%lld", &a[i]);
    ll l = 1, r = n + 1;
    while (l <= r) {
        int m = l + (r - l + 1 >> 1);
        if (check(m)) r = m - 1;
        else l = m + 1;
    }
    cout << l << endl;
}

int main()
{
    solve();
    return 0;
}
```

---

## 例题——技能升级

> Problem:[技能升级]([P2047 - [蓝桥杯2022初赛\] 技能升级 - New Online Judge (ecustacm.cn)](http://oj.ecustacm.cn/problem.php?id=2047))

---

### 思路

> 这题一眼优先队列贪心着做，但是一看数据发现最多加m次，而m最大为2e9，这可过不去，所以放弃无脑贪心
>
> 再转念一想，其实每次我们都会挑当前技能中点数最高的去加，这就是优先队列的贪心思路，这也就说明我们最后一次增加的点数一定是所有之前增加的点数中最小的，所以我们枚举最小的点数即可，只要达到一个最小的点数刚好能把m次用完那就肯定是最大的了
>
> 由于值越小，使用的次数肯定越多，这个判断是符合单调的，我们可以用二分来加速枚举过程，这样2e9也不在话下

### Code

```c++
#include <bits/stdc++.h>
 
using namespace std;
 
using ll = long long;
 
const int N = 1e5 + 7;
 
ll n, m;
ll a[N], b[N];
 
ll ans = 0;
 
bool check(ll k)
{
    ll t = 0;
    for (int i = 0; i < n; ++i) {
        if (a[i] >= k) {
            ll tmp = (a[i] - k) / b[i] + 1;
            t += tmp;
            if (t > m) return true;
        }
    }
    return false;
}
 
void solve()
{
    // freopen("in.txt", "r", stdin);
    cin >> n >> m;
    for (int i = 0; i < n; ++i) scanf("%lld%lld", &a[i], &b[i]);
    ll l = 1, r = 1e10;
    while (l <= r) {
        ll m = l + (r - l + 1 >> 1);
        if (check(m)) l = m + 1;
        else r = m - 1;
    }
    
    for (int i = 0; i < n; ++i) {
        if (a[i] >= l) {
            ll tmp = a[i] - l;
            tmp /= b[i];
            ++tmp;
            if (a[i] - b[i] * (tmp - 1) == r) tmp - 1;
            m -= tmp;
            ans += (a[i] + a[i] - b[i] * (tmp - 1)) * tmp >> 1;
        }
    }
    cout << ans + m * r << endl;
}

int main()
{
    solve();
    return 0;
}
```

### 设计细节

> 这里我们要求得的最后答案是尽量用完m次，也就是总次数小于等于m的最小次数，所以我们check可以反着来，找出最小的大于m次的r，此时的l不满足条件，也就是l就是最大的小于等于m
>
> 最后我们计算答案的时候最后再处理边界值，因为这个值是最小的，可以用来做保底替补，而不要先加了

