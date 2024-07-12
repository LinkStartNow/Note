## LCR 189. 设计机械累加器

> Problem: [LCR 189. 设计机械累加器](https://leetcode.cn/problems/qiu-12n-lcof/description/)

### 大体思路

> 这题不让用循环，那么咱可以递归来代替，但是无法用if做判断，那么可以思考一下如何代替if

### 思路1——三目运算

> 这个有点牵强，但总归是没用if吧

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int target) {
        return target ? target + mechanicalAccumulator(target - 1) : 0;
    }
};
```

---

### 思路2——逻辑短路

> 许多语言的逻辑表达式都是支持短路功能的，我们可以通过短路来模拟if判断

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int target) {
        target > 1 && (target += mechanicalAccumulator(target - 1));
        return target; 
    }
};
```

---

### 思路3——快速乘

> 这个思路类似于快速幂的思路，将乘数看成二进制位来进行拆分，比如：
> $$
> \begin{aligned}
> 5 *11 &= 5 * (1 + 2 + 8) \\
> &= 5 * 1 + 5 * 2 + 5 * 8
> \end{aligned}
> $$
> 执行的时候按位运算操作模拟即可，层数很明显是可以看到尽头的，因为根据高斯求和，最终的乘数也就是$n$或者$n + 1$，而n不超过1e5，于是大约$2^{14}$~~手动模拟14次for递推不就彳亍了嘛~~
>
> 但是我是手残，我依旧选择递归哈哈哈

#### Code

```c++
class Solution {
    int ans = 0;
    int dfs(int a, int b)
    {
        (b & 1) && (ans += a);
        b && dfs(a << 1, b >> 1);
        return 0;
    }
public:
    int mechanicalAccumulator(int target) {
        dfs(1 + target, target);
        return ans >> 1;
    }
};
```

---

### 思路4——大小取巧

> 接下来的思路就比较抽象了，比如可以利用二维数组本身的大小来代替乘法的运算

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int n) {
        char ssr[n][n + 1];
        return sizeof(ssr) >> 1;
    }
};
```

---

### 思路5——文字游戏

> 说了不让用乘法就不用了么，好像没说不能直接越级用幂吧哈哈哈
>
> 根据高斯求和，可知：
> $$
> \begin{aligned}
> ans &= \frac {(1 + n) * n} 2 
> \end{aligned}
> $$
> 我们最后再去求这个除2哈~~不能用除就位运算嘿嘿~~，则：
> $$
> \begin{aligned}
> ssr &= 2 * ans \\
> &= (1 + n) * n \\
> &= n^2 + n
> \end{aligned}
> $$
> 那还说啥了，直接上幂吧就

#### Code

```c++
class Solution {
public:
    int mechanicalAccumulator(int n) {
        return (int)pow(n, 2) + n >> 1;
    }
};
```

---

