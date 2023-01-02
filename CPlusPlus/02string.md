# String

## bool

- C语言中没有逻辑类型，于是给int用`typedef`取了个别名叫`BOOL`,把`TRUE`和`FALSE`分别`define`为1和0来模拟逻辑类型，但实际上还是`int`类型于是造成了内存浪费
- 而`bool`、`true`、`false`是C++中的关键字，`bool`本身就是C++的一种内置类型，大小为一个字节

---

## string

**需要包含头文件**`string`

- 初始化

	- 当然也可以不初始化，执行过程中赋值也行

	- 传统等号赋值

		```c++
		string sr = "acc";
		```

	- 括号直接赋值

		```c++
		string s("abc");
		```

- 支持单个字符修改

```c++
string  str="1234";
str[1] = 'A';
cout << str << endl;   //1A34
```

- 支持整个字符串的修改

```c++
str = "abcd";
cout << str << endl;  //abcd
```

- 支持直接比较大小

```c++
string s = "abc", sr = "abc";
if (s < sr) cout << "yes" << endl;
else cout << "no" << endl;
```

- 支持拼接

	拼接过程中后续遇到的字符串(如字符数组、字符指针)都会被转化为`string`当然由于后两种字符串不支持拼接，于是在拼接时要么将`string`放在前面，要么手动强转

	```c++
	char s1[] = "abcd";
	const char* s2 = "ssr";
	string s3 = "666";
	//string s4 = s3 + s2 + s1;
	//string s4 = (string)s1 + s2 + s3;
	string s4 = (string)s2 + s1 + s3;
	s4 += "ab";
	cout << s4 << endl;
	```

- 字符串的截取

	用`substr(begin, lenth)`方法，其中`begin`不能越界，但长度可以越界(最终也就会输出到字符串结束)

	```c++
	string s = "abcccde";
	cout << s.substr(2, 59) << endl;
	```

- 求字符串长度

	这里使用`length()`和`size()`方法是一样的

	```c++
	string s = "abcccde";
	cout << s.length() << ' ' << s.size() << endl;
	```

- 返回`const char*`

	```c++
	string s = "hello world";
	fun(s.c_str());
	```

	