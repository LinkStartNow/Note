# map

map：映射表，键值（key），实值（value），可以**根据键值进行自动排序**，**键值唯一**

***其内部每个元素其实就是个`pair`的键值对，`first`就是键值，`second`是实值***

## 增

1. 直接通过`mp[键值] = 实值`的方式实现增加项

	```c++
	// mp[键值] = 实值
	mp['a'] = 1; // 添加元素
	mp['f'] = 12;
	mp['c'] = 13;
	mp['d'] = 134;
	```

2. 通过`insert`函数

	因为之前说过，`map`相当于是多个`pair`组成的，于是可以把`pair`当做元素进行插入

	```c++
	// 法2，通过insert函数插入
	m.insert({ 667, "嗨嗨柚" });
	```

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

## 删

删除也是使用`erase`函数，具体可以通过以下实现

1. 删除迭代器所指向的内容

	```c++
	it = ++mp.begin(); // 指向第二个元素
	it = mp.erase(it); // 返回删除元素的下一个元素
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;
	```

2. 删除给定的具体键值的内容

	```c++
	mp.erase('f'); // 删除可以不用迭代器，而直接通过键值来删除
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	```

3. 删除一个返回内的元素

	```c++
	it = ++mp.begin();
	map<char, int>::iterator its = --mp.end();
	it = mp.erase(it, its); // 删除这俩迭代器之间的元素，左闭右开，同样返回指向最后一个删除元素的下一个元素的迭代器
	for (pair<char, int> x : mp) {
	    cout << x.first << ':' << x.second << "    ";
	}cout << endl;
	cout << (*it).first << ":" << (*it).second << endl;
	```

还可以通过`clear`函数实现清空

```c++
mp.clear(); // 清空map
```

---

## 改

与增的操作相同

```c++
// 如果键值存在，则是通过键值修改对应的实值
mp['a'] = 666;
```

---

## 查

- 查找某个键对应的值

	```c++
	cout << mp['z'] << endl; // 不存在，默认输出空值，int对应的是0
	cout << mp['a'] << endl; // 存在则直接访问
	```

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

- 通过`count`计数的方法查看是否存在某键值的元素，因为键值存在的话不会重复，于是最多是一个，所以只会输出0或1

	```c++
	cout << mp.count('a') << endl; // 键值为a的存在，输出1
	cout << mp.count('b') << endl; // 键值为b的不存在，输出0
	```

- 通过`size`函数查看元素个数

	```c++
	cout << mp.size() << endl; // 输出map的大小，也就是元素的个数4
	```

- 通过`empty`函数查看是否为空

	```c++
	cout << mp.empty() << endl; // 非空输出0
	mp.clear(); // 清空map
	cout << mp.empty() << endl; // 为空输出1
	```

- 遍历

	- xxxxxxxxxx #include <iostream>​using namespace std;​typedef struct Node{    int val;    Node* pNext;    Node* pPre;​public:    Node(int v)    {        val = v;        pPre = pNext = nullptr;    }} *pn;​// 迭代器类class CIterator{public:    pn m_tp;​    CIterator(pn t):m_tp(t){}​    ~CIterator()    {        m_tp = nullptr;    }​    operator bool()    {        return m_tp;    }​    // 左++    pn operator ++()    {        m_tp = m_tp->pNext;        return m_tp;    }​    // 右++    pn operator ++(int)    {        pn t = m_tp;        m_tp = m_tp->pNext;        return t;    }​    // 取值    int operator *()    {        return m_tp->val;    }};​class CList{    pn m_pHead; // 头指针    pn m_pEnd; // 尾指针    int m_Len; // 链表长度​public:    CList()    {        m_pHead = m_pEnd = nullptr;        m_Len = 0;    }​    ~CList()    {        pn pTemp = nullptr;        while (m_pHead){            pTemp = m_pHead;            m_pHead = m_pHead->pNext;            delete pTemp;        }        m_pHead = m_pEnd = nullptr;        m_Len = 0;    }​public:    // 从尾部加入节点    void PushBack(int v)    {        m_Len++;        if (m_pHead) {            m_pEnd->pNext = new Node(v);            m_pEnd->pNext->pPre = m_pEnd;            m_pEnd = m_pEnd->pNext;        }        else {            m_pHead = m_pEnd = new Node(v);        }    }​    // 普通遍历    void ShowList()    {        pn t = m_pHead;        while (t) {            cout << t->val << ' ';            t = t->pNext;        }        cout << endl;    }​    // 用模拟的迭代器遍历    void Shows()    {        CIterator t(m_pHead);        while (t) {            cout << *t << ' ';            t++;        }        cout << endl;    }};​int main(){    CList  ts;    ts.PushBack(1);    ts.PushBack(23);    ts.PushBack(232);    ts.PushBack(2255);    //ts.ShowList();    ts.Shows();    cout << 666 << endl;}c++

		```c++
		map<char, int>::iterator it = mp.begin();
		while (it != mp.end()) { // 已经按默认顺序排序了 a:1    c:13    d:134    f:12
		    // first 键值， second 实值
		    cout << (*it).first << ":" << (*it).second << "    ";
		    ++it;
		}cout << endl;
		```

	- 增强的范围`for`

		```c++
		for (pair<char, int> x : mp) {
				cout << x.first << ':' << x.second << "    ";
			}cout << endl;
		```

---

## 完整代码

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

