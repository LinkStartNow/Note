# STL

容器分为**序列性容器**和**关联性容器**：

- 序列性容器：容器保持了元素在容器中的原始位置，允许指定插入和删除的位置，每个元素都有固定的位置，取决于插入的时间和地点，和元素的值无关

	如：`list`、`vector`、`deque`等

- 关联性容器：元素的位置取决于容器特定的排序规则，元素的位置和元素的值有关

	如：`map`、`set`、`hash map`等

---

## 链表（List）

C++自带的链表容器

### 初始化

```c++
// 初始化
list<int> lst(2); // 指定链表的大小，且初始化默认的值
list<int>::iterator it = lst.begin();
while (it != lst.end()) { // 0 0
    cout << *it << ' ';
    ++it;
} cout << endl;

list<int> lst2(2, 666); // 指定链表的大小，且指定初始化
for (int v : lst2) { // 666 666
    cout << v << ' ';
} cout << endl; 

list<int> lst3{ 1, 2, 3, 4 }; // 长度为4，指定不同的初始化值
for (int v : lst3) { // 1 2 3 4
    cout << v << ' ';
} cout << endl; 
```

---

### 增

- 两端都可以进行添加

	```c++
	lst.push_back(1); // 尾添加
	lst.push_back(2);
	lst.push_back(3);
	lst.push_back(666);
	lst.push_front(0); // 头添加
	lst.push_front(-666);
	```

	---

- 通过`insert`在任意位置插入

	主要还是看迭代器能定位到什么位置，只要迭代器能定位到，那就可以插入

	```c++
	// insert ：在迭代器指向的位置之前插入节点，返回的是新输入的节点
	puts("--------insert-----------");
	ite = ++lst.begin();
	list<int>::iterator it = lst.insert(ite, 100);
	```

---

### 删

- 两端都可以删除元素

	```c++
	lst.pop_back(); // 尾删除
	lst.pop_front(); // 头删除
	```

	---

- 通过`erase`函数删除元素

	- 删除某个位置的元素

		```c++
		// erase：删除指定的节点，返回的是删除的下一个节点，参数中的迭代器会失效
		puts("--------erase-----------");
		ite = lst.erase(it); // 此时的迭代器it不重新赋值已经没法用了，即使++操作也会报错
		```

	- 删除某两个迭代器之间的元素

		使用方法和`map`的一样

		```c++
		list<int>::iterator itl = ++lst.begin();
		list<int>::iterator itr = --lst.end();
		itl = lst.erase(itl, itr); // 指向了itr指向的元素， 删除同样是左闭右开
		```

	---

- 通过`unique`删除连续相同的值

	```c++
	for (int v : lst3) { // 666 1 1 2 3 1
	    cout << v << ' ';
	} cout << endl;
	
	lst3.unique();
	
	for (int v : lst3) { // 666 1 2 3 1
	    cout << v << ' ';
	} cout << endl;
	```

	---

- 通过`remove`函数删除某种值的全部节点

	```c++
	for (int v : lst3) { // 666 1 2 3 4 4 4 4
	    cout << v << ' ';
	} cout << endl;
	
	lst3.remove(4); // 删除所有值为4的节点
	
	for (int v : lst3) { // 666 1 2 3
	    cout << v << ' ';
	} cout << endl;
	```

	---

- 清空链表

	```c++
	// 清空链表
	lst.clear();
	```

---

### 改

- 遍历到某个节点单独修改

	```c++
	while (it != lst.end()) { // 0 0
	    cout << *it << ' ';
	    ++it;
	} cout << endl;
	
	it = ++lst.begin();
	(*it) = 233;
	
	for (int v : lst) { // 0 233
	    cout << v << ' ';
	} cout << endl;
	```

	---

- 通过`reverse`实现逆序

	```c++
	lst3.reverse();
	
	for (int v : lst3) { // 1 1 2 3 666
	    cout << v << ' ';
	} cout << endl;
	```

	---

- 通过`splice`实现剪切操作

	第一个参数是需要复制到的迭代器，第二个参数是数据源的迭代器

	```c++
	list<int> lst4{ 123, 567, 788 };
	
	it = lst3.begin();
	
	::advance(it, 2); // +2 表示往后偏移两个单位，同样-2表示往前偏移两个
	
	lst3.splice(it, lst4);
	
	for (int v : lst3) { // 1 1 123 567 788 2 3 666
	    cout << v << ' ';
	} cout << endl;
	
	cout << lst4.empty() << endl; // 1，链表4已经被剪切到链表3中了，4已经空了
	```

	如果剪切完整的链表，只需要填写前两个参数，若只剪切一个元素，则另外补充上第三个参数，来表示剪切的位置

	```c++
	list<int> lst4{ 123, 567, 788 };
	
	it = lst3.begin();
	
	::advance(it, 2); // +2 表示往后偏移两个单位，同样-2表示往前偏移两个
	
	lst3.splice(it, lst4, ++lst4.begin());
	
	for (int v : lst3) { // 1 1 567 2 3 666 ，只插入了一个567
	    cout << v << ' ';
	} cout << endl;
	
	for (int v : lst4) { // 123 788 ， 567 被剪切了，所以少了一个567
	    cout << v << ' ';
	} cout << endl;
	```

	如果要剪切一段的元素，则还需要填入第四个参数，来表示剪切的结束位置

	**注意：参数区间左闭右开**

	```c++
	list<int> lst4{ 123, 567, 788 };
	
	it = lst3.begin();
	
	::advance(it, 2); // +2 表示往后偏移两个单位，同样-2表示往前偏移两个
	
	lst3.splice(it, lst4, ++lst4.begin(), lst4.end());
	
	for (int v : lst3) { // 1 1 567 788 2 3 666，成功剪切过来一段
	    cout << v << ' ';
	} cout << endl;
	
	for (int v : lst4) { // 123， 后面都被剪切掉了，所以没了
	    cout << v << ' ';
	} cout << endl;
	```

	---

- 排序通过`sort`函数来实现

	- 默认情况下是升序

		```c++
		lst3.sort(); // 默认升序
		
		for (int v : lst3) { // 1 1 2 3 666
		    cout << v << ' ';
		} cout << endl;
		```

	- 还可以手动书写判断函数，实现降序或者更加复杂的操作

		***Version1版本（不用模板函数）：***

		下面是判断函数代码：

		```c++
		//template <typename T>
		//bool cmps(T a, T b)
		bool cmpj(int a, int b)
		{
			return a > b;
		}
		```

		下面是调用代码：

		```c++
		lst3.sort(&cmpj); // 手动传入判断函数，实现降序
		
		for (int v : lst3) { // 1 1 2 3 666
		    cout << v << ' ';
		} cout << endl;
		```

		***Version2版本（使用模板函数）：***

		判断函数代码：

		```c++
		template <typename T = int>
		bool cmpj(T a, T b)
		//bool cmpj(int a, int b)
		{
			return a > b;
		}
		```

		调用代码：

		```c++
		lst3.sort(cmpj<int>); // 手动传入判断函数，实现降序
		
		for (int v : lst3) { // 666 3 2 1 1
		    cout << v << ' ';
		} cout << endl;
		```

	- 还能调用本身存在的判断函数

		**注意：这里的判断是要加括号的**

		```c++
		lst3.sort(less<int>()); // 升序
		
		for (int v : lst3) { // 1 1 2 3 666
		    cout << v << ' ';
		} cout << endl;
		
		lst3.sort(greater<int>()); // 降序
		
		for (int v : lst3) { // 666 3 2 1 1
		    cout << v << ' ';
		} cout << endl;
		```

---

### 查

- 遍历

	- 通过迭代器遍历

		```c++
		list<int>::iterator ite = lst.begin(); // begin是获取头结点的迭代器
		while (ite != lst.end()) { // end返回的是尾结点的迭代器
		    cout << *ite << ' ';
		    ite++;
		}cout << endl;
		```

	- 增强的范围`for`

		```c++
		for (int x : lst) {
		    cout << x << ' ';
		}cout << endl;
		```

	---

- 获得头尾元素的值

	```c++
	// 直接获取头尾结点的值，你没有看错，并没有返回迭代器，而是直接的值！
	cout << lst.front() << endl; // 获得头元素的值
	cout << lst.back() << endl; // 获得尾元素的值
	```

	---

- 获得链表的长度

	```c++
	cout << lst.size() << endl;
	```

	---

- 判断链表是否为空

	```c++
	// 判断是否为空，为空则输出1，非空则为0
	cout << lst.empty() << endl;
	```

---

### 完整代码

***Version1（基础版本）:***

```c++
#include <iostream>
#include <list> // 链表的头文件

using namespace std;

int main()
{
	list<int> lst;  // 定义一个链表
	lst.push_back(1); // 尾添加
	lst.push_back(2);
	lst.push_back(3);
	lst.push_back(666);
	lst.push_front(0); // 头添加
	lst.push_front(-666);

	// 用迭代器遍历
	list<int>::iterator ite = lst.begin(); // begin是获取头结点的迭代器
	while (ite != lst.end()) { // end返回的是尾结点的迭代器
		cout << *ite << ' ';
		ite++;
	}
	cout << endl;

	lst.pop_back(); // 尾删除
	lst.pop_front(); // 头删除

	// 用增强的范围for遍历
	for (int x : lst) {
		cout << x << ' ';
	}
	cout << endl;

	// insert ：在迭代器指向的位置之前插入节点，返回的是新输入的节点
	puts("--------insert-----------");
	ite = ++lst.begin();
	list<int>::iterator it = lst.insert(ite, 100);
	for (int v : lst) {
		cout << v << ' '; // 0 100 1 2 3
	}cout << endl;
	cout << *it << endl; // 100

	// erase：删除指定的节点，返回的是删除的下一个节点，参数中的迭代器会失效
	puts("--------erase-----------");
	ite = lst.erase(it); // 此时的迭代器it不重新赋值已经没法用了，即使++操作也会报错
	for (int v : lst) {
		cout << v << ' '; // 0 1 2 3
	}cout << endl;
	cout << *ite << endl; // 1
	//cout << *it << endl; // error程序异常，因为it指向的节点已经被删除了


	puts("--------erase__version2.0-----------");
	lst.push_back(233);
	lst.push_back(567);
	lst.push_back(123);
	list<int>::iterator itnew = lst.begin();
	lst.erase(itnew);
	cout << "现在的列表为：";
	for (auto v : lst) {
		cout << v << ' ';
	}cout << endl;
	//++itnew;
	//cout << *itnew << endl;

	// 测试一下其他的方法
	puts("--------other_function-----------");
	
	// 头结点的值
	it = lst.begin();
	cout << "头结点的值为:" << *it << endl;

	// 尾结点的值
	it = lst.end();
	cout << "尾结点的值为:" << *(--it) << endl;

	// 直接获取头尾结点的值，你没有看错，并没有返回迭代器，而是直接的值！
	cout << lst.front() << endl;
	cout << lst.back() << endl;

	// 获取长度
	cout << lst.size() << endl;

	// 判断是否为空，为空则输出1，非空则为0
	cout << lst.empty() << endl;

	// 清空链表
	lst.clear();

	cout << lst.empty() << endl;
	return 0;
}
```

---

***Version2（进阶方法）***

```c++
#include <iostream>
#include <list>

using namespace std;

template <typename T = int>
bool cmpj(T a, T b)
//bool cmpj(int a, int b)
{
	return a > b;
}

template <typename T>
bool cmps(T a, T b)
{
	return a < b;
}

int main()
{
	// 初始化
	list<int> lst(2); // 指定链表的大小，且初始化默认的值
	list<int>::iterator it = lst.begin();
	while (it != lst.end()) { // 0 0
		cout << *it << ' ';
		++it;
	} cout << endl;

	it = ++lst.begin();
	(*it) = 233;

	for (int v : lst) { // 0 233
		cout << v << ' ';
	} cout << endl;

	list<int> lst2(2, 666); // 指定链表的大小，且指定初始化
	for (int v : lst2) { // 666 666
		cout << v << ' ';
	} cout << endl; 

	list<int> lst3{ 1, 2, 3, 4 }; // 长度为4，指定不同的初始化值
	for (int v : lst3) { // 1 2 3 4
		cout << v << ' ';
	} cout << endl; 

	// 增加
	lst3.push_back(5);
	lst3.push_front(0);

	lst3.insert(++lst3.begin(), 666); // 在该迭代器的位置插入值，原先的值被挤到后面去

	for (int v : lst3) { // 0 666 1 2 3 4 5
		cout << v << ' ';
	} cout << endl; 

	// 删除
	lst3.pop_back();
	lst3.pop_front();

	for (int v : lst3) { // 666 1 2 3 4
		cout << v << ' ';
	} cout << endl;

	lst3.push_back(4);
	lst3.push_back(4);
	lst3.push_back(4);

	for (int v : lst3) { // 666 1 2 3 4 4 4 4
		cout << v << ' ';
	} cout << endl;

	lst3.remove(4); // 删除所有值为4的节点

	for (int v : lst3) { // 666 1 2 3
		cout << v << ' ';
	} cout << endl;

	it = ++lst3.begin();
	lst3.insert(it, 1);
	lst3.push_back(1);

	for (int v : lst3) { // 666 1 1 2 3 1
		cout << v << ' ';
	} cout << endl;

	lst3.unique();

	for (int v : lst3) { // 666 1 2 3 1
		cout << v << ' ';
	} cout << endl;

	// 排序
	lst3.sort(); // 默认升序

	for (int v : lst3) { // 1 1 2 3 666
		cout << v << ' ';
	} cout << endl;

	lst3.sort(cmpj<int>); // 手动传入判断函数，实现降序
	
	for (int v : lst3) { // 666 3 2 1 1
		cout << v << ' ';
	} cout << endl;

	lst3.sort(less<int>()); // 升序

	for (int v : lst3) { // 1 1 2 3 666
		cout << v << ' ';
	} cout << endl;

	lst3.sort(greater<int>()); // 降序

	for (int v : lst3) { // 666 3 2 1 1
		cout << v << ' ';
	} cout << endl;

	lst3.reverse();

	for (int v : lst3) { // 1 1 2 3 666
		cout << v << ' ';
	} cout << endl;

	list<int> lst4{ 123, 567, 788 };

	it = lst3.begin();

	::advance(it, 2); // +2 表示往后偏移两个单位，同样-2表示往前偏移两个

	lst3.splice(it, lst4, ++lst4.begin(), lst4.end());

	for (int v : lst3) { // 1 1 567 788 2 3 666，成功剪切过来一段
		cout << v << ' ';
	} cout << endl;

	for (int v : lst4) { // 123， 后面都被剪切掉了，所以没了
		cout << v << ' ';
	} cout << endl;

	//for (int v : lst3) { // 1 1 567 2 3 666 ，只插入了一个567
	//	cout << v << ' ';
	//} cout << endl;

	//for (int v : lst4) { // 123 788 ， 567 被剪切了，所以少了一个567
	//	cout << v << ' ';
	//} cout << endl;

	//for (int v : lst3) { // 1 1 123 567 788 2 3 666
	//	cout << v << ' ';
	//} cout << endl;

	//cout << lst4.empty() << endl; // 1，链表4已经被剪切到链表3中了，4已经空了

	return 0;
}
```

