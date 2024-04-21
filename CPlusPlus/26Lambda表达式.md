# 说明

> 跟Python的Lambda表达式大差不差，简便之处在于使用了可以使C++在函数内部实现内嵌函数了！

---

# 参考链接

> [大佬滴链接]([C++ Lambda表达式的完整介绍 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/384314474))

# 格式

> 在C++中，Lambda表达式的格式一般为：
>
> [捕获列表] [参数列表] [可变规则] [返回类型] [函数体]
>
> 接下来，我们一一介绍这些部分

## 捕获列表

> 这是整个Lambda表达式中最精华的部分
>
> 通过捕获列表，Lambda函数就能调用上下文所用到的数据
>
> 具体分为以下类别：

### 基础分类

> 1. []：不捕获任何变量，也就是说，不能使用上下文的数据，只能使用传入的参数（这和外部定义个函数没啥区别了呀，恼）
> 2. [变量名]：以值传递的方式捕获该变量，但是只会捕获这个变量，其他的变量仍然是不能用的
> 3. [=]：以值传递的方式捕获所有父作用域的变量（包括this）
> 4. [&变量名]：以引用的方式捕获该变量，也是只会捕获这个变量，不能捕获其他变量
> 5. [&]：以引用传递的方式捕获所有父作用域的变量（包括this）
> 6. [this]：以值传递的方式捕获当前的this指针（个人理解就是把this指针也当做普通的指针变量了，实际上捕获了个变量名，所以也就值传递了）

### 混合搭配

> 在捕获列表中是可以使用逗号分隔的，这也是混合搭配的基础
>
> [=, &a, &b]：表示对于变量a和变量b采用引用传递，而其他剩余的所有采用值传递
>
> [&, a, c]：除了对a和c值传递，其他的都采用引用传递

**注意：混合搭配是不能重复捕获的**

> 例如[=, a]，这里值传递已经捕获了所有变量了，后面又指定a值传递是不可以的，应该修改成[=, &a]，将a用引用捕获，或者直接[=]

## 参数列表

> 这里的参数列表和普通函数的参数列表一样，需要变量名，需要变量类型

```c
int a = 3, b = 5;
auto add = [&] (int a, int b) {
    a = 6;
    return a + b;
};

cout << add(3, 4) << endl;      // 10     
cout << "a = " << a << endl;    // a = 3   
```

> 在没有参数的时候，也可以不写参数列表

```c++
int a = 3, b = 5;
auto add = [&] {
    a = 6;
    return a + b;
};

cout << add() << endl;          // 11      
cout << "a = " << a << endl;    // a = 6
```

## 可变规则

### 使用背景

> 如果不存在任何一个引用传递的话，那么当前函数默认是常函数，常函数中的值都是不能修改的

#### Code

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main()
{
    int a = 6;

    [=] () {
        a = 666;
    }();

    cout << a << endl;
}
```

> 上述代码就是错误的，因为采用了值传递，当前函数为常函数，内部的a是无法修改的

---

> 但是如果不是被捕获的参数，而是被传入的参数是不受限制的

#### Code1

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main()
{
    int a = 6;

    [=] (int a) {
        a = 666;
    }(a);

    cout << a << endl;
}
```

> 上述代码是能实现的，但是修改的是值传递进来的a，也就是个临时变量，不会影响外部的值

---

> 即便如此，也不要高兴的太早，因为只有传入的会不受限制，被捕获的每一个变量都是被盯的牢牢的，虽然整个函数不是常函数了，但是当个值传递的捕获的变量想要修改仍然会报错

#### Code2

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main()
{
    int a = 6;
    int b = 3;

    [=] (int a) {
        a = 666;
        b = 666;
    }(a);

    cout << a << endl;
}
```

---

> 因此，我们需要破局就必须使用可变规则的参数

#### Code3

```c++
#include <iostream>
#include <algorithm>

using namespace std;

int main()
{
    int a = 6;
    int b = 3;

    [=] (int a) mutable {
        a = 666;
        b = 666;
    }(a);

    cout << b << endl;  // 3
}
```

> 上述代码已经不再会报错，但由于b是值传递的捕获，于是修改并不会对外部的b造成影响

---

### 使用说明

> 可变则必须加在参数列表之后，也就是说如果要使用可变规则，那么参数列表是必须写的，即使参数列表为空

```c++
int a = 3;
auto ssr = [=] () mutable {
    a = 6;
    cout << a << endl;      // 6
};
ssr();
cout << a << endl;          // 3
```

> 解释：
>
> 这里采用的是值传递的a，而在不加mutable的情况下，该Lambda函数默认是常函数，也就是说内部的变量是不允许修改的，但在添加了mutable关键字后就可以修改了，但由于是值传递，并不会影响到外部的值

---

## 返回类型

> 返回类型一般会有编译器自动判断，程序员不用手动添加（Python狂喜）

# 直接调用

> 有些匿名函数就是一次性的，也就是说我们一般只会需要调用一次，所以我们不需要给它另外取个名字（没有名字之后也就不能再找到它了，也就不能调用，便成为了一次性的调用）

```c++
#include <iostream>
using namespace std;

int main()
{
    int m = 0;
    int n = 0;
    [&, n] (int a) mutable { m = ++n + a; }(4);
    cout << m << endl << n << endl;
    // 5  0
}
```

---

# 常用场合

> 这种方便使用的短小精悍的函数我们一般会当做参数应用于其他的函数。
>
> 用过Python的Lambda表达式我们都知道，很多的排序规则什么的都可以直接用Lambda表达式代替，C++也是如此
>
> 当然，个人认为C++的Lambda能写出更复杂的排序规则

## Code

```c++
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    int arr[] = {53, 29, 1, 10, 37};
    sort(arr, arr + 5,
         [] (int a, int b) {return a % 10 < b % 10;});
    for (int x: arr) {
        cout << x << ' ';
    } cout << endl;
    // 10 1 53 37 29 
}
```

> 上面的排序规则，传入的就是Lambda表达式，非常方便吧

## Code2

```c++
#include <iostream>
#include <algorithm>
using namespace std;

struct ssr
{
    int a, b;

    ssr(int a, int b): a(a), b(b) {}
};

int main()
{
    ssr arr[] = {ssr(53, 29), ssr(1, 10), ssr(37, 33)};

    sort(arr, arr + 3,
         [] (const ssr& a, const ssr& b) {return a.b < b.b;});

    for_each(arr, arr + 3,
             [] (const ssr& it) {cout << it.a << ' ' << it.b << endl;});

    /*
        1 10
        53 29
        37 33
    */
}
```

---

## 开局初始化

> 我们可以在定义的时候直接调用lambda表达式，所以我们可以在外面创建全局变量初始化时直接创建一个lambda表达式顺便调用了，这样在某些场合（例如力扣）可以刚开始就初始化完毕能共用的那些数据，节约大量时间

### 注意

> 这里的返回值一定要有，比如`return 0`
>
> 接收最好用auto接收
>
> 这个操作稳定性并不高尽量不要使用

### Code

```c++
#include <bits/stdc++.h>

using namespace std;

auto fun = [] () {
    cout << "hello world" << endl;
    return 0;
}();

int main()
{
    // fun();
}
```

