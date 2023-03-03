# STL——其他部分

## 迭代器（Iterator）

直接上代码

```c++
#include <iostream>
#include <list>

using namespace std;

int main()
{
	list<int> lst{ 1, 2, 3, 4, 5 };

	list<int>::iterator it = lst.begin();
	while (it != lst.end()) { // 1 2 3 4 5
		cout << (*it) << ' ';
		++it;
	} cout << endl;

	// 反向迭代器
	list<int>::reverse_iterator its = lst.rbegin();
	while (its != lst.rend()) { // 5 4 3 2 1
		cout << *its << ' ';
		++its;
	} cout << endl;

	// 删除某个值
	its = lst.rbegin();
	while (its != lst.rend()) {
		if (*its == 3) {
			it = --its.base(); // 删除只能用普通迭代器，所以要把反向迭代器转换一下
			// 同时有偏移，从正向的角度看，要向后偏移一位
			cout << *it << endl;
			lst.erase(it);
			break;
		}
		++its;
	}

	it = lst.begin();
	while (it != lst.end()) { // 1 2 3 4 5
		cout << (*it) << ' ';
		++it;
	} cout << endl;



	return 0;
}
```

---

## 容器适配器

**注意：容量适配器不支持前面提到的算法，同样也不支持迭代器**

他们能进行的操作比较受约束

主要包括栈（Stack）和队列（Queue）

之前也经常在接触，所以这里就不细说用法了，知道有哪些方法一般也知道怎么用了

栈支持的操作：

- push
- pop
- size
- empty
- top

队列支持的操作：

- empty
- size
- back
- front
- pop
- push

这里也就front和back不咋用，其实front和back就是和之前一样的，用来直接获取头和尾的值的

```c++
queue<int> q;
q.push(1);
q.push(2);
q.push(2);
q.push(4);

cout << q.front() << ' ' << q.back() << endl; // 1 4
```

---

## 仿函数

话不多说，直接上代码

```c++
#include <iostream>

using namespace std;

class CAdd
{
public:
	int m_sum = 0;

	int operator()(int a, int b)
	{
		m_sum += a + b;
		return a + b;
	}
};

int main()
{
	CAdd add;

	cout << add(2, 3) << endl; // 5
	cout << add(2, 6) << endl; // 8
	cout << add.m_sum << endl; // 13

	return 0;
}
```

