# 步骤

## 1、完成子串的next数组

当求next[i]时，先把str[i]与str[next[i - 1]]作比较

- 相等
	- 直接next[i] = next[i - 1] + 1
- 不相等
	- str[i] 与str[next[next[i - 1] - 1]]作比较，比较结果继续递归
	- 直到到头或者相等为止

这里我们一般用一个变量k来表示当前已经成功匹配的长度，也就是这里的next[i - 1]

> next数组：
>
> 所谓next数组就是用来记录以当前字符结尾，的最长公共前后缀长度
>
> next[i] = k; // 表示子串str[:i]的最长公共前后缀长度为k，str为原串

## 2、利用子串的next数组优化暴力跳转

> 这个判断过程和具体步骤和前面获取next数组时是一样的，只是比较的字符串发生了改变而已

---

# 模板题

>  Problem:[[模板]KMP]([P3375 【模板】KMP - 洛谷 | 计算机科学教育新生态 (luogu.com.cn)](https://www.luogu.com.cn/problem/P3375))

思路跟前面步骤分析的一摸一样，这里主要是展示代码的实现

## Code

```c++
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string s, t;

void GetNext(vector<int>& nx)
{
    int k = 0;
    int n = t.size();
    for (int i = 1; i < n; ++i) {
        while (k && t[i] != t[k]) k = nx[k - 1];
        if (t[i] == t[k]) ++k;
        nx[i] = k;
    }
}

void kmp(vector<int>& nx)
{
    int k = 0;
    int n = s.size(), m = t.size();
    for (int i = 0; i < n; ++i) {
        while (k && s[i] != t[k]) k = nx[k - 1];
        if (s[i] == t[k]) ++k;
        if (k == m) {
            printf("%d\n", i - k + 2);
            k = nx[k - 1];
        }
    }
}

int main()
{
    cin >> s >> t;
    int n = t.size();
    vector<int> nx(n);
    GetNext(nx);
    kmp(nx);
    for (int x: nx) printf("%d ", x);
}
```

---

## 错误代码分析

### Code

```c++
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string s, t;

void GetNext(vector<int>& nx)
{
    int k = 0;
    int n = t.size();
    for (int i = 1; i < n; ++i) {
        while (k && t[i] != t[k]) k = nx[k];
        if (t[i] == t[k]) ++k;
        nx[i] = k;
    }
}

void kmp(vector<int>& nx)
{
    int k = 0;
    int n = s.size(), m = t.size();
    for (int i = 0; i < n; ++i) {
        while (k && s[i] != t[k]) k = nx[k];
        if (s[i] == t[k]) ++k;
        if (k == m) {
            printf("%d\n", i - k + 2);
            k = nx[k - 1];
        }
    }
}

int main()
{
    cin >> s >> t;
    int n = t.size();
    vector<int> nx(n);
    GetNext(nx);
    kmp(nx);
    for (int x: nx) printf("%d ", x);
}
```

---

### 分析

> k通过next数组进行回跳的时候出现问题
>
> next[k]表示以k为结尾的最长前后缀长度
>
> 那么，当当前字符与str[k]不匹配时，应该跳转到next[k - 1]
>
> 既然与str[k]已经比较失败了，那么最终所匹配成功的后缀不应该包含字符str[k]，跳next[k]的话，默认是与前面与k相等的字符的后一个比较了
>
> - 例如：
> 	- 原串：`abcabcabcabcf`
> 	- 当匹配到str[-2]时，也就是倒数第二个c，此时对应的next为两个`abc`后的第一个a
> 	- 此时，f与a比较失败了
> 		- 也就是说`abaabca`与`abaabcf`匹配失败
> 	- 那么在f前的前缀将不可能是`abcabc`，但我们不用从头开始比，因为f的前面还有可能是`abc`
> 	- 于是，我们需要跳next[6 - 1]，也就是比较失败的那个a前一个字符对应的下标
> 		- 就是说拿`abca`与`abcf`比较