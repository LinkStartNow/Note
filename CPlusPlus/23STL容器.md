# STL——容器部分

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
	
	---
	
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
	
	---
	
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
	
- 通过`merge`实现两个链表的合并

  **注意：两个链表必须同递增或同递减，且合并后被合并的链表会被清空，相当于剪切**

  ```c++
  list<int> tst5{ 2, 3, 4, 6, 15 };
  list<int> tst6{ -1, 3, 3, 6, 15, 16 };
  
  tst5.merge(tst6);
  
  for (int v : tst5) { // -1 2 3 3 3 4 6 6 15 15 16
      cout << v << ' ';
  } cout << endl;
  
  cout << tst6.empty() << endl; // 1
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

    ---

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

- 排序通过`sort`函数来实现

  - 默认情况下是升序

    ```c++
    lst3.sort(); // 默认升序
    
    for (int v : lst3) { // 1 1 2 3 666
        cout << v << ' ';
    } cout << endl;
    ```

    ---

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

    ---

    下面是调用代码：

    ```c++
    lst3.sort(&cmpj); // 手动传入判断函数，实现降序
    
    for (int v : lst3) { // 1 1 2 3 666
        cout << v << ' ';
    } cout << endl;
    ```

    ---

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

    ---

    调用代码：

    ```c++
    lst3.sort(cmpj<int>); // 手动传入判断函数，实现降序
    
    for (int v : lst3) { // 666 3 2 1 1
        cout << v << ' ';
    } cout << endl;
    ```

    ---

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

- 通过`swap`实现交换两个链表

  ```c++
  list<int> tst7{ 233, 666 };
  list<int> tst8{ 57 };
  
  tst7.swap(tst8);
  
  for (int v : tst7) { // 57
      cout << v << ' ';
  } cout << endl;
  
  for (int v : tst8) { // 233 666
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

	list<int> tst5{ 2, 3, 4, 6, 15 };
	list<int> tst6{ -1, 3, 3, 6, 15, 16 };

	tst5.merge(tst6);

	for (int v : tst5) { // -1 2 3 3 3 4 6 6 15 15 16
		cout << v << ' ';
	} cout << endl;

	cout << tst6.empty() << endl; // 1

	list<int> tst7{ 233, 666 };
	list<int> tst8{ 57 };

	tst7.swap(tst8);

	for (int v : tst7) { // 57
		cout << v << ' ';
	} cout << endl;

	for (int v : tst8) { // 233 666
		cout << v << ' ';
	} cout << endl;

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

---

## 向量（Vecotor）

向量就和之前写的动态数组一样，但容量不够用时会扩充到1.5倍

向量实则是个动态数组，静态数组是`Array`

**注意：向量的迭代器是支持`+`、`+=`、`-`、`-=`等运算的**

**同时，向量也是支持下标访问的**

**在向量中，只有swap能减少容量，其他方法只能增大容量**

### 初始化

```c++
// 初始化
vector<int> vec(4); // 定义向量，指定容量和使用量，默认值（0）初始化

cout << vec.capacity() << ' ' << vec.size() << endl; // 4 4
for (int v : vec) { // 0 0 0 0
    cout << v << ' ';
} cout << endl;

vector<int> vec2(4, 233); // 定义向量，指定容量和使用量，指定初始化值

cout << vec2.capacity() << ' ' << vec2.size() << endl; // 4 4
for (int v : vec2) { // 233 233 233 233
    cout << v << ' ';
} cout << endl;

vector<int> vec3{ 2, 5, 7 };

cout << vec3.capacity() << ' ' << vec3.size() << endl; // 3 3
for (int v : vec3) { // 2 5 7
    cout << v << ' ';
} cout << endl;
```

二维初始化：

```c++
int m = 30, n = 40;
vector<vector<int>> x(20, vector<int>(30));
vector<vector<int>> y(m, vector<int>(n)); // 获得m行n列的二维数组
cout << x[0][0] << endl; // 0
cout << y[0].size() << endl; // 40
cout << y.size() << endl; // 30
```

---

### 增

- 通过`push_back()`在尾部添加

	**注意：vector不支持对头部操作哦**

	```c++
	vec2.push_back(6);
	```

	---

- 通过`insert`插入

	- 插入一个

		```c++
		vec2.insert(vec2.begin(), 666);
		
		cout << vec2.capacity() << ' ' << vec2.size() << endl; // 6 6
		for (int v : vec2) { // 666 233 233 233 233 6
		    cout << v << ' ';
		} cout << endl;
		```

		---

	- 插入多个

		```c++
		vector<int>::iterator it = vec2.begin();
		it += 2; // vector的迭代器支持+=
		vec2.insert(it, 2, 567); // 在迭代器位置插入两个567
		
		cout << vec2.capacity() << ' ' << vec2.size() << endl; // 9 8
		for (int v : vec2) { // 666 233 567 567 233 233 233 6
		    cout << v << ' ';
		} cout << endl;
		```

---

### 删

- 通过`erase`删除迭代器指向的节点，同链表

	```c++
	it = vec2.begin() + 2; // vector的迭代器支持加法
	
	vec2.erase(it); // 删除完后迭代器同样会失效
	```

	---

- 通过`clear`实现清空，不多解释

---

### 改

- 直接通过下标更改值

	```c++
	vector<int> vec6{ 233, 555, 666};
	vec6[1] = 2;
	
	for (int i = 0; i < vec6.size(); ++i) { // 233 2 666
	    cout << vec6[i] << ' ';
	} cout << endl;
	```

	---

- `resize`可以重新定义使用量

	- 当重新定义的使用量小于当前使用量，则会按顺序截取前面的数据

		```c++
		vector<int> vec4{ 2, 3, 4, 7 };
		vec4.push_back(8);
		vec4.resize(2);
		
		cout << vec4.capacity() << ' ' << vec4.size() << endl; // 6 2
		for (int v : vec4) { // 2 3
		    cout << v << ' ';
		} cout << endl;
		```

		---

	- 当重新定义的使用量介于当前使用量和容量之间，则会用值补充多出的使用量

		```c++
		vector<int> vec4{ 2, 3, 4, 7, 8, 9 };
		vec4.push_back(8);
		vec4.resize(8);
		
		cout << vec4.capacity() << ' ' << vec4.size() << endl; // 9 8
		for (int v : vec4) { // 2 3 4 7 8 9 8 0
		    cout << v << ' ';
		} cout << endl;
		```

		---

	- 大于容量时，会扩容，并补充多出的使用量

		```c++
		vector<int> vec4{ 2, 3, 4, 7, 8, 9 };
		vec4.push_back(8);
		vec4.resize(12);
		
		cout << vec4.capacity() << ' ' << vec4.size() << endl; // 13 12, 扩容还是按照1.5倍扩容，直到够用
		for (int v : vec4) { // 2 3 4 7 8 9 8 0 0 0 0 0
		    cout << v << ' ';
		} cout << endl;
		```

		---

	- 补充值时可以补充指定的值

		```c++
		vector<int> vec4{ 2, 3, 4, 7, 8, 9 };
		vec4.push_back(8);
		vec4.resize(12, 10); // 补充的值为10
		
		cout << vec4.capacity() << ' ' << vec4.size() << endl; // 13 12, 扩容还是按照1.5倍扩容，直到够用
		for (int v : vec4) { // 2 3 4 7 8 9 8 10 10 10 10 10
		    cout << v << ' ';
		} cout << endl;
		```

		---

- 通过`swap`交换

	**注意：这个交换会把使用量、容量、值全部交换**

	```c++
	cout << vec4.capacity() << ' ' << vec4.size() << endl; // 13 12, 扩容还是按照1.5倍扩容，直到够用
	for (int v : vec4) { // 2 3 4 7 8 9 8 10 10 10 10 10
	    cout << v << ' ';
	} cout << endl;
	
	vector<int> vec5{ 1, 2 };
	
	cout << vec5.capacity() << ' ' << vec5.size() << endl; // 2 2
	for (int v : vec5) { // 1 2
	    cout << v << ' ';
	} cout << endl;
	
	vec4.swap(vec5);
	
	puts("交换后");
	
	cout << vec4.capacity() << ' ' << vec4.size() << endl; // 2 2
	for (int v : vec4) { // 1 2
	    cout << v << ' ';
	} cout << endl;
	
	cout << vec5.capacity() << ' ' << vec5.size() << endl; // 13 12
	for (int v : vec5) { // 2 3 4 7 8 9 8 10 10 10 10 10
	    cout << v << ' ';
	} cout << endl;
	```

---

### 查

- `capacity`方法可以获得**容量**

- `size`可以获得**使用量**

	```c++
	cout << vec.capacity() << ' ' << vec.size() << endl; // 4 4
	```

	---

- 通过下标获取相应位置的值

	```c++
	for (int i = 0; i < vec6.size(); ++i) { // 233 2 666
	    cout << vec6[i] << ' ';
	} cout << endl;
	```

	---

- 通过`front`和`back`直接获得头尾点的值，同链表

	```c++
	vector<int> vec6{ 233, 555, 666};
	vec6[1] = 2;
	
	cout << vec6.front() << ' ' << vec6.back() << endl; // 233 666
	```

---

## 双向队列（Deque）

**注意：复杂度比Vector复杂的多，除非有需要，否则尽量选择Vector**

Deque和Vector的用法几乎一模一样，Deque也可以用下标访问，迭代器也支持加减

与Vector不同的是Deque支持两头都进行增加或者删除操作，其余Vector有的特性Deque几乎都有

```c++
#include <iostream>
#include <deque>

using namespace std;

int main()
{
	// 初始化-----------------------------
	deque<int> de(3);
	cout << de.size() << endl; // 3

	deque<int>::iterator it = de.begin();
	while (it != de.end()) { // 0 0 0
		cout << *it << ' ';
		++it;
	} cout << endl;

	deque<int> de2(3, 2);

	cout << de2.size() << endl; // 3

	it = de2.begin();
	while (it != de2.end()) { // 2 2 2
		cout << *it << ' ';
		++it;
	} cout << endl;

	deque<int> de3{ 233, 567, 666, 123 };

	cout << de3.front() << ' ' << de3.back() << endl; // 233 123

	// 前后都可以插入
	de2.push_back(233);
	de2.push_front(666);

	for (int i = 0; i < de2.size(); ++i) { // 666 2 2 2 233
		cout << de2[i] << ' '; // 可以直接用下标访问
	} cout << endl;

	// 前后也都可以删除
	de2.pop_back();
	de2.pop_front();
	de2.pop_front();

	for (int i = 0; i < de2.size(); ++i) { // 2 2
		cout << de2[i] << ' ';
	} cout << endl;

	

	return 0;
}
```

---

## 映射表（Map）

map：映射表，键值（key），实值（value），可以**根据键值进行自动排序**，**键值唯一**

***其内部每个元素其实就是个`pair`的键值对，`first`就是键值，`second`是实值***

### 增

1. 直接通过`mp[键值] = 实值`的方式实现增加项

	```c++
	// mp[键值] = 实值
	mp['a'] = 1; // 添加元素
	mp['f'] = 12;
	mp['c'] = 13;
	mp['d'] = 134;
	```

	---

2. 通过`insert`函数

	因为之前说过，`map`相当于是多个`pair`组成的，于是可以把`pair`当做元素进行插入

	```c++
	// 法2，通过insert函数插入
	m.insert({ 667, "嗨嗨柚" });
	m.insert(pair<char, int>('e', 5));
	```

	---

	**注意：若是插入的键和之前的键冲突，则还是以之前的键为主，算作插入失败**

	如：

	```c++
	map<int, string> m;
	
	// 法1，直接通过键值赋值
	m[666] = "你干~嘛~";
	
	// 法2，通过insert函数插入
	m.insert({ 666, "嗨嗨柚" });
	
	cout << m[666]; // 输出 你干~嘛~
	```

---

### 删

删除也是使用`erase`函数，具体可以通过以下实现

1. 删除**迭代器**所指向的内容

	```c++
	it = ++mp.begin(); // 指向第二个元素
	it = mp.erase(it); // 返回删除元素的下一个元素
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;
	```

	---

2. 删除给定的**具体键值**的内容

	```c++
	mp.erase('f'); // 删除可以不用迭代器，而直接通过键值来删除
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	```

	---

3. 删除两个迭代器之间的元素

	```c++
	it = ++mp.begin();
	map<char, int>::iterator its = --mp.end();
	it = mp.erase(it, its); // 删除这俩迭代器之间的元素，左闭右开，同样返回指向最后一个删除元素的下一个元素的迭代器
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;
	```

	---

还可以通过`clear`函数实现清空

```c++
mp.clear(); // 清空map
```

---

### 改

与增的操作相同

```c++
// 如果键值存在，则是通过键值修改对应的实值
mp['a'] = 666;
```

---

### 查

- 查找某个键对应的值

	```c++
	cout << mp['z'] << endl; // 不存在，默认输出空值，int对应的是0
	cout << mp['a'] << endl; // 存在则直接访问
	```

	---

- 通过`find`函数返回对应键值元素的迭代器

	```c++
	it = mp.find('c'); // 要传入键值进行查找
	// 未找到的话it会指向mp.end()
	if (it != mp.end()) {
	    cout << (*it).first << ":" << (*it).second << endl;
	}
	else {
	    cout << "未找到" << endl;
	}
	```

	---

- 通过`upper_bound`、`lower_bound`获得大于、大于等于某键值的迭代器

	没找到会返回`end`

	```c++
	map<char, int>::iterator it = m.upper_bound('c'); // 大于c
	map<char, int>::iterator its = m.lower_bound('c'); // 大于等于c
	
	if (it != m.end()) { // d 4
	    cout << (*it).first << ' ' << (*it).second << endl;
	}
	else {
	    cout << "没找到大于的值" << endl;
	}
	
	if (its != m.end()) { // c 3
	    cout << (*its).first << ' ' << (*its).second << endl;
	}
	else {
	    cout << "没找到大于等于的值" << endl;
	}
	```

	---

	**这个也可以用来判断某个元素是否存在**

	```c++
	if (m.upper_bound('f') == m.lower_bound('f')) {
	    cout << "该元素不存在" << endl;
	}
	else {
	    cout << "该元素存在" << endl;
	}
	```

	---

- 通过`count`计数的方法查看是否存在某键值的元素，因为键值存在的话不会重复，于是最多是一个，所以只会输出0或1

	```c++
	cout << mp.count('a') << endl; // 键值为a的存在，输出1
	cout << mp.count('b') << endl; // 键值为b的不存在，输出0
	```

	---

- 通过`size`函数查看元素个数

	```c++
	cout << mp.size() << endl; // 输出map的大小，也就是元素的个数4
	```

	---

- 通过`empty`函数查看是否为空

	```c++
	cout << mp.empty() << endl; // 非空输出0
	mp.clear(); // 清空map
	cout << mp.empty() << endl; // 为空输出1
	```

	---

- 遍历

	- 通过迭代器

		```c++
		map<char, int>::iterator it = mp.begin();
		while (it != mp.end()) { // 已经按默认顺序排序了 a:1    c:13    d:134    f:12
		    // first 键值， second 实值
		    cout << (*it).first << ":" << (*it).second << "    ";
		    ++it;
		}cout << endl;
		```

		---

	- 增强的范围`for`

		```c++
		for (pair<char, int> x : mp) {
				cout << x.first << ':' << x.second << "    ";
			}cout << endl;
		```

---

### 完整代码

实际只是草稿，上面的才是精华

```c++
#include <iostream>
#include <map> // 1. 包含头文件

using namespace std; // 2. 打开标准命名空间

int main()
{
	map<char, int> mp; // 空映射表
	
	// mp[键值] = 实值
	mp['a'] = 1; // 添加元素
	mp['f'] = 12;
	mp['c'] = 13;
	mp['d'] = 134;

	map<char, int>::iterator it = mp.begin();
	while (it != mp.end()) { // 已经按默认顺序排序了 a:1    c:13    d:134    f:12
		// first 键值， second 实值
		cout << (*it).first << ":" << (*it).second << "    ";
		++it;
	}cout << endl;

	// 如果键值存在，则是通过键值修改对应的实值
	mp['a'] = 666;

	it = mp.begin();
	while (it != mp.end()) {
		cout << (*it).first << ":" << (*it).second << "    ";
		++it;
	}cout << endl;

	puts("--------------earse-----------");
	mp.erase('f'); // 删除可以不用迭代器，而直接通过键值来删除
	for (pair<char, int> x : mp) {
		cout << x.first << ':' << x.second << "    ";
	}cout << endl;

	it = ++mp.begin(); // 指向第二个元素
	it = mp.erase(it); // 返回删除元素的下一个元素
	for (pair<char, int> x : mp) {
		cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;

	mp['a'] = 1; // 添加元素
	mp['f'] = 12;
	mp['c'] = 13;
	mp['d'] = 134;

	it = ++mp.begin();
	map<char, int>::iterator its = --mp.end();
	it = mp.erase(it, its); // 删除这俩迭代器之间的元素，左闭右开，同样返回指向最后一个删除元素的下一个元素的迭代器
	for (pair<char, int> x : mp) {
		cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;

	cout << mp['z'] << endl; // 不存在，默认输出空值，int对应的是0
	cout << mp['a'] << endl; // 存在则直接访问

	it = mp.find('c'); // 要传入键值进行查找
	// 未找到的话it会指向mp.end()
	if (it != mp.end()) {
		cout << (*it).first << ":" << (*it).second << endl;
	}
	else {
		cout << "未找到" << endl;
	}

	cout << mp.count('a') << endl; // 键值为a的存在，输出1
	cout << mp.count('b') << endl; // 键值为b的不存在，输出0

	cout << mp.size() << endl; // 输出map的大小，也就是元素的个数4

	cout << mp.empty() << endl; // 非空输出0
	mp.clear(); // 清空map
	cout << mp.empty() << endl; // 为空输出1

	return 0;
}
```

---

## 集合（Set）

Set的用法和Map几乎是一样的，可以看做只有键值的Map

```c++
#include <iostream>
#include <set>

using namespace std;

int main()
{
	set<int> st{ 33, 9, 3, 4 };
	set<int>::iterator it = st.begin();

	while (it != st.end()) {
		cout << (*it++) << ' ';
	} cout << endl;

	st.insert(0);
	st.insert(10);

	for (int x : st) {
		cout << x << ' ';
	} cout << endl;

	it = st.begin();
	::advance(it, 3);

	it = st.erase(it);

	for (int v : st) { // 0 3 4 10 33
		cout << v << ' ';
	} cout << endl;

	cout << *it << endl; // 10

	it = st.begin();

	//(*it) = 666; // 不能通过迭代器改变元素值，其实别的方式也不能修改，只能增加和删除

	pair<set<int>::iterator, bool> p = st.insert(10);

	if (!p.second) { // 插入失败，已经有10了，插入失败
		cout << "插入失败" << endl;
	} cout << *p.first << endl;

	cout << *st.lower_bound(3) << endl; // 3
	cout << *st.upper_bound(3) << endl; // 4

	it = st.find(233);

	if (it != st.end()) {
		cout << "找到了" << endl;
	}
	else {
		cout << "没找到" << endl;
	}

	return 0;
}
```

----

## 无序映射表（Unordered_Map）

无序映射表也称Hash_Map，是通过哈希法吸映射来实现的，原先的映射表是会根据键值自动排序的，而这个无序映射表则不会自动排序，查找和插入效率提升到了O(1)，但是同时空间的浪费也增加了

用法仍然同Map，详情参考Map

```c++
#include <iostream>
#include <unordered_map>

using namespace std;

int main()
{
	unordered_map<char, int> m;
	m['a'] = 1;
	m['c'] = 33;
	m['f'] = 666;
	m['e'] = 233;

	unordered_map<char, int>::iterator it = m.begin();

	while (it != m.end()) {
		cout << (*it++).first << ' ' << (*it).second << endl;
	}

	m.insert(pair<char, int>('b', 5));
	m.insert({ 'd', 3 });

	for (pair<char, int> v : m) {
		cout << v.first << ' ' << v.second << '\t';
	} cout << endl;

	return 0;
}
```

---

## tuple

> 类属于Python中的元组，但是内部的值是可以修改的，实际上就是pair的扩充
>
> 看起来就类似于一个结构体，毕竟功能也差不多，还是很好用哒！

```c++
#include <iostream>
#include <tuple>

using namespace std;

int main()
{
    tuple<int, double, char> t {1, 3.14, 'a'};

    get<1> (t) = 1.23;

    cout << get<0> (t) << endl;    // 1      
    cout << get<1> (t) << endl;    // 1.23
    cout << get<2> (t) << endl;    // a
}
```

---

### 修改

> 当使用变量来创建或赋值的时候，实际上只是简单的值传递，他俩还是独立的，并不会绑定

```c++
#include <iostream>
#include <tuple>

using namespace std;

int main()
{
    tuple<int, double, char> t {1, 3.14, 'a'};

    int a = 3;

    get<0> (t) = a;
    cout << get<0> (t) << endl;  // 3

    a = 666;
    cout << get<0> (t) << endl;  // 3
}
```

