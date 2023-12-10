# STL——算法部分

有些像`sort`、`find`等方法在`vector`等容器中是没有的，于是在使用的时候就不是很方便

这里的算法就解决了这个问题，它提供了一些通用的方法

当然，需要包含头文件`algorithm`

## 遍历

可以通过使用`for_each`方法来实现，具体传入三个参数，前两个都是迭代器来划分范围，最后一个是遍历执行的命令

```c++
// 遍历---------------------------
list<char> lst{ 'a', 'b', 'e', 'q', 'f' };

::for_each(lst.begin(), lst.end(), &fun); // a b e q f
cout << endl;

::for_each(lst.begin(), --lst.end(), &fun); // a b e q 
cout << endl;

unordered_map<int, string> m{ {2, "我知道"}, {3, "你不懂"} };

::for_each(m.begin(), m.end(), &fun2); // 2 我知道        3 你不懂
cout << endl;
```

```c++
void fun(char c)
{
	cout << c << ' ';
}

void fun2(pair<int, string> p)
{
	cout << p.first << ' ' << p.second << '\t';
}
```

---

## 计数

可以使用`count`方法

也是三个参数，前两个传入迭代器划分范围，最后一个是要查找的值

```c++
// 计数---------------------------
cout << ::count(lst.begin(), lst.end(), 'a') << endl; // 3
```

---

## 判断相等

可以使用`equal`方法

传入三个参数，前两个是容器1的开始与结束，第三个是容器2的开始，长度是由前两个参数决定的，由于设计的某些不合理，当容器2传入的开始达不到这个长度时程序会崩溃

```c++
// 比较------------------------
vector<int> vec1{ 1, 2, 3 };
vector<int> vec2{ 1, 2, 3, 4 };
vector<int> vec3{ 1, 2, 4 };

cout << ::equal(vec1.begin(), vec1.end(), vec2.begin()) << endl; // 1
//cout << ::equal(vec2.begin(), vec2.end(), vec1.begin()) << endl; // 因为2比较长，1的长度不够
// 以2为基准，所以崩了
cout << ::equal(vec2.begin(), --vec2.end(), vec3.begin()) << endl; // 0
```

---

## 查找

### find

可以使用`find`方法

三个参数，前两个依旧是范围，最好一个是值，查找失败则会返回范围结束也就是参数2（不一定是end哦！）

```c++
// 查找----------------------------
//vector<int>::iterator ite = ::find(vec1.begin(), vec1.end(), 2);
auto ite2 = vec1.end() - 2;
auto ite = ::find(vec1.begin(), ite2, 2); // 没找到则返回查找范围的最后（不一定是end）

if (ite != ite2) {
    cout << *ite << endl;
}
else {
    cout << " 找不到" << endl;
}
```

---

### lower_bound

> 该函数有三个参数，前两个是搜索范围，左闭右开
>
> 最后一个可以有两种选择：
>
> 1. 基础版：传递值，最终查询出第一个大于等于该值的迭代器
> 2. 进阶版：再多传递一个cmp规则，返回第一个不符合该规则的迭代器

```c++
//查找[first, last)区域中第一个大于 val 的元素。
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,
                             const T& val);
//查找[first, last)区域中第一个不符合 comp 规则的元素
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,
                             const T& val, Compare comp);
```

> 上面用的都是迭代器，我们在用vector时一般更喜欢下标，于是我们还需要减去begin

---

#### 直接传值

```c++
#include <iostream>
#include <vector>
#include <functional>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {2, 3, 9, 0, 666, 13};
    sort(v.begin(), v.end());
    cout << "排序后..." << endl;
    for (int x: v) cout << x << ' ';
    cout << endl;
    int t = upper_bound(v.begin(), v.end(), 3) - v.begin();
    cout << v[t] << endl;
}
/*
排序后...
0 2 3 9 13 666
9
*/
```

---

#### 自定义比较方法

> 要求有两个参数，**第二个参数**就是传入的比较值
>
> **注意：一定要传入小于或小于等于比较值的比较方法**

##### Code

```c++
#include <iostream>
#include <vector>
#include <functional>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {2, 3, 9, 0, 666, 13};
    sort(v.begin(), v.end());
    cout << "排序后..." << endl;
    for (int x: v) cout << x << ' ';
    cout << endl;
    int t = lower_bound(v.begin(), v.end(), 3,
                        [&] (int x, int y) {
                            return x <= y;
                        }) - v.begin();
    cout << v[t] << endl;
}
/*
排序后...
0 2 3 9 13 666
9
*/
```

> 上述的比较函数实际上可以这样翻译：
>
> x为遍历到的vector中的元素，y为传入的比较值3（lower_bound的第三个参数）
>
> 当遍历到不满足x < 3时就会返回，也就是遍历到第一个x > 3
>
> 于是最后返回了9

----

##### Code1

```c++
#include <iostream>
#include <vector>
#include <functional>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {2, 3, 9, 0, 666, 13};
    sort(v.begin(), v.end());
    cout << "排序后..." << endl;
    for (int x: v) cout << x << ' ';
    cout << endl;
    int t = lower_bound(v.begin(), v.end(), 9,
                        [&] (int x, int y) {
                            return x < y;
                        }) - v.begin();
    cout << v[t] << endl;
}
/*
排序后...
0 2 3 9 13 666
9
*/
```

> 上述代码也是相同的分析，我们找到第一个不满足比较关系的元素，也就是第一个大于等于9的元素
>
> 最终返回了9

---

### upper_bound

> 与lower_bound相同，这个函数也有两种调用方式

---

#### 直接传值

> 返回第一个大于给定值的迭代器

```c++
#include <iostream>
#include <vector>
#include <functional>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {2, 3, 9, 0, 666, 13};
    sort(v.begin(), v.end());
    cout << "排序后..." << endl;
    for (int x: v) cout << x << ' ';
    cout << endl;
    int t = upper_bound(v.begin(), v.end(), 9) - v.begin();
    cout << v[t] << endl;
}
/*
排序后...
0 2 3 9 13 666
13
*/
```

---

#### 自定义比较方法

> 给定值会被当做**第一个参数**，而且会返回第一个满足比较方法的结果（这两点完全就和lower反着来了）
>
> **注意：一定要传入大于等于或大于比较值的方法**

---

##### Code

```c++
#include <iostream>
#include <vector>
#include <functional>
#include <algorithm>

using namespace std;

int main()
{
    vector<int> v = {2, 3, 9, 0, 666, 13};
    sort(v.begin(), v.end());
    cout << "排序后..." << endl;
    for (int x: v) cout << x << ' ';
    cout << endl;
    int t = upper_bound(v.begin(), v.end(), 9,
                        [&] (int x, int y) {
                            return x < y;
                        }) - v.begin();
    cout << v[t] << endl;
}
/*
排序后...
0 2 3 9 13 666
13
*/
```

---

#### 记忆技巧

> 固定让第一参数（x）放左边，第二参数（y）放在右边
>
> 如果原本是从小到大排序的，则用小于或小于等于
>
> 如果是从大到小的排序，则用大于或大于等于

---

## 排序

排序所使用的规则和其他的`sort`函数一样

```c++
vector<int> vec{ 1, 233, 4, 687, 0, -233 };

::sort(vec.begin(), vec.end()); // 默认升序
::for_each(vec.begin(), vec.end(), &fun<int>);
puts("");

::sort(vec.begin(), vec.end(), greater<int>()); // 手动降序
::for_each(vec.begin(), vec.end(), &fun<int>);
puts("");

::sort(vec.begin(), vec.end() - 2, greater<int>()); // 部分降序
::for_each(vec.begin(), vec.end(), &fun<int>);
puts("");
```

---

## 比较大小

比较大小就是像字符串那样从头开始一个一个元素比较，若直到一个比较完还没比较出结果，则长的大

其中某一位比令一个容器的对应位大则也可以判断出大小

**注意：映射表比的是键值哦！**

```c++
list<int> lst1{ 1, 2, 3 };
list<int> lst2{ 1, 2, 3, 4 };
list<int> lst3{ 1, 2, 2, 4 };

list<int> lst4 = ::max(lst1, lst2);
for (int v : lst4) { // 1 2 3 4
    cout << v << ' ';
} cout << endl;

list<int> lst5 = ::min(lst1, lst3);
for (int v : lst5) { // 1 2 2 4
    cout << v << ' ';
} cout << endl;

map<string, int> s1{ {"12", 3} };
map<string, int> s2{ {"123", 1} };

auto s3 = ::max(s1, s2);
for (auto x : s3) {
    cout << x.first << ' ' << x.second << '\t';
} cout << endl;
```

---

