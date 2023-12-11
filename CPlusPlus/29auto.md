# 说明

> auto可以自动判断数据类型，非常的好用

```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    auto x = 666;
    auto y = 3.14;
    cout << x << ' ' << sizeof(x) << endl;   // 666 4      
    cout << y << ' ' << sizeof(y) << endl;   // 3.14 8
}
```

---

# 妙用

## auto多个变量

> auto可以用于几乎所有变量类型，所以也包括pair

```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    pair<int, int> p[] {{233, 777}, {666, 4}};
    for (auto& x: p) {
        cout << x.first << ' ' << x.second << endl;
    }
    /*
        233 777
        666 4
    */
}
```

> 但是，我们为什么明明知道只有两个类型还得先获得pair再first，second呢？
>
> 于是我们可以进行如下优化

```c++
#include <bits/stdc++.h>

using namespace std;

int main()
{
    pair<int, int> p[] {{233, 777}, {666, 4}};
    for (auto [x, y]: p) {
        cout << x << ' ' << y << endl;
    }
    /*
        233 777
        666 4
    */
}
```

---

> 同样的我们也可以用于tuple

```c++
#include <iostream>
#include <tuple>

using namespace std;

int main()
{
    tuple<int, double, char> t {1, 3.14, 'a'};

    auto [x, y, z] = t;
    cout << x << endl;       // 1
    cout << y << endl;       // 3.14
    cout << z << endl;       // a
}
```

