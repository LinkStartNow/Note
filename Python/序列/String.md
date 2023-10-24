# 主要内容

由于字符串的某些特性与序列极其相似，故姑且把它当做“序列”，***实则并不是哦***
详细版可以 ***[戳这里](https://www.lintcode.com/course/9/learn/?chapterId=61&sectionId=471&extraParams=%7B%22moduleSource%22%3A%22lc-course%3A9%22%7D)***

在Python中，双引号或者单引号括起来的都算字符串，一般在Python中单引号当成双引号用就可以了

---

## 常用格式字符

| 格式字符 |                       说明                       |
| :------: | :----------------------------------------------: |
|    %s    |                      字符串                      |
|    %c    |                     单个字符                     |
|    %d    |                    十进制整数                    |
|    %o    |                    八进制整数                    |
|    %x    |                   十六进制整数                   |
|    %e    |                指数（基底写为e）                 |
|    %E    |                指数（基底写为E）                 |
|  %f或%F  |                      浮点数                      |
|  %g或%G  | 指数或浮点数（根据显示长度判断），g对应e，G对应E |
|    %%    |                    一个字符%                     |

### Code

```python
x = 1235
print('%o'%x)              # 2323  
print('%x'%x)              # 4d3  
print('%e'%x)              # 1.235000e+03    
print('%s'%'123')          # 123       
print('%s'%[1, 2, 3])      # [1, 2, 3]
```

---

## 格式化字符串

***具体[链接](https://blog.csdn.net/aaaaaaze/article/details/123020600?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166470495516782395315081%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166470495516782395315081&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-123020600-null-null.142^v51^pc_rank_34_2,201^v3^control_2&utm_term=python%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1018.2226.3001.4187)***

> xxxxxxxxxx a = {1, 2, 3}b = {2, 4}print(a ^ b)print(a.symmetric_difference(b))python

1. 使用`%`占位符

	```python
	a = 23
	name = 'tom'
	s = '%s的年龄是%d'%(name, a)
	print(s)
	```

	具体格式为`%[-][+][0][m][.n]格式字符%x`

	我们来具体介绍一下这些代表着什么：

	- `%`：格式标志，表示格式开始
	- `-`：指定左对齐输出
	- `+`：对正数加正好
	- `0`：指定空位填0
	- `m`：最小宽度
	- `.n`：指定精度
	- `格式字符`：指定类型
	- `%`：格式运算符
	- `x`：待转换的表达式

2. `format()`方法
	一般格式为：`{<索引>:<填充字符><对齐方式><宽度.精度><格式>}`

	- 设置参数

		```python
		a = ['hello', 'world', 'say', "'s"]
		print('zly {0[2]} {0[3]} {0[0]} {0[1]}'.format(a))
		```

	- 数字格式化

		- 控制精度

			```python
			x = 3.1415926
			print('{:.2f}'.format(x))  # 3.14
			```

		- 转进制

			```python
			print('{:<20b}'.format(5)) # 左对齐二进制，长度为20
			print('{:>20d}'.format(123)) # 右对齐十进制
			print('{:^20x}'.format(666)) # 居中对齐十六进制
			print('{:^20o}'.format(233)) ## 居中对齐八进制
			```

3. `f'str'`
	可以在字符串内部直接使用变量进行格式化

	```python
	b = '???'
	print(f'{b}你好呀')
	```

---

##  转义字符

> 在Python中也使用转义字符，同样的，也是反斜杠`\`

```python
print('hello')
print("hello") # 效果同上

print('it\'s my world') # 将单引号转义
```

---

## 原始字符串

如果我们要打印路径`f:\nabc`，直接打印的话，反斜杠会被当做转义字符而出错

我们可以再使用一个反斜杠将这个反斜杠也转义来实现输出

```python
print('f:\\nabc')
```

如果我们有很多子目录，就会被迫添加很多多余的反斜杠来转义，显然这样并不方便，于是我们可以使用原始字符创来实现

只要在引号前加上`r`，则该字符串为*原始字符串*

我们打印刚刚的路径就可以用以下代码：

```python
print(r'f:\nabc')
```

> 原始字符串：把所有反斜杠当做字符串的一部分，而不会转义

---

## 多行字符串

> 只要用三重引号包着字符串，该字符串就能成为多行字符串
>
> 使用多行字符串，我们就不需要在字符串中手动添加多个`\n`实现换行，而直接换行即可

```python
print('''Dear Alice,
Eve's cat has been arrested for catnapping, cat burglary, and extortion.
Sincerely,
Bob''')
```

---
## 万能切片:[左边界:右边界:步长]
**这个切片在列表等类型中也可以用**
左边界元素的位置到右边界元素的位置，可以正数负数混用，只看元素的位置.步长可正可负，默认为1,为正则从左到右，为负则从右到左
***注意：步长的正负与起始位置要对应，步长表示从左到右则起位置必须在终位置左边。起点永远是起点确定完方向也是从起点（左边界）开始的***
### 一般切片
```python
a = 'abcdef'
print(a[1:-1:2])  # bd
```
如果像这样不匹配则会输出空
```python
a = 'abcdef'
print(a[1:-1:-2])  # None
```
### 多层切片
切片后返回的结果一般与原类型相同所以可以继续切片，规则和前面一样
```python
a = 1, 2, 3, 4, 5
print(a[2::-1][::-1])
```
同时切片的三个参数都可以用表达式
```python
a = 1, 2, 3, 4, 5
print(a[1 * 2 : : 2])
```
---
## 一些常用的方法

### 字符串转换

#### 大小写

> `upper()`和`lower()`方法都会返回一个新的字符串，其中所有英文字母都替换为大写/小写，非字母不变

**注意：返回的是新字符串，原串不变**

```python
a = 'Hi, my name is'
b = a.upper()
c = a.lower()
print(a)  # Hi, my name is
print(b)  # HI, MY NAME IS
print(c)  # hi, my name is
```

---

#### capitalize

> 首字符大写，其余小写

```python
x = 'hELlo hOw ARe You'
print(x.capitalize())  # Hello how are you
```

---

#### title

> 每个单词首字母大写

```python
x = 'hELlo hOw ARe You'
print(x.title())  # Hello How Are You
```

---

#### swapcase

> 大小写交换

```python
x = 'hELlo hOw ARe You'
print(x.swapcase())  # HelLO HoW arE yOU
```

---

### 判断字符串的组成

用来判断字符串的组成，常用的有这些：

- `isupper`：字符串是否全大写

- `islower`：字符串是否全小写

- `isalpha`：字符串是否只包含字母，且非空

- `isalnum`：字符串是否只包含字母和数字，且非空

- `isdecimal`：字符串是否只有数字，且非空

- `isspace`：字符串是否只包含空格、制表符、换行，且非空

- `istitle`：字符串是否仅包含大写字母开头，后面全是小写字母

	**貌似非英文字符直接忽略了，空格没忽略，从第一个非空格字母开始判断**

	```python
	a = 'hello'
	b = 'hLO'
	c = 'Hello'
	d = 'Hello你好'
	e = '你好 Hello'
	f = '你好hello'
	g = '你好 H ello'
	print(a.istitle())    # False
	print(b.istitle())    # False
	print(c.istitle())    # True
	print(d.istitle())    # True
	print(e.istitle())    # True
	print(f.istitle())    # False
	print(g.istitle())    # False
	```

---

### 判断头尾

可以通过`startswith`和`endswith`方法判断

传参就传入字符串就行

```python
a = 'Hi, my name is'
print(a.startswith('h'))  # False             
print(a.startswith('H'))  # True             
print(a.endswith('s'))    # True     
print(a.endswith('is'))   # True     
print(a.endswith(' is'))  # True         
print(a.endswith('e is')) # True 
print(a.endswith('eis'))  # False 
```

---

### 拼接和分割

#### 拼接

> 拼接的话使用`join`方法就可以完成，由字符串对象调用，传入一个字符串列表，表示用该字符串来当分隔符，拼接这个字符串列表为一个新的字符串

```python
a = ['I', 'love', "you"]
b = ','.join(a)
c = ' '.join(a)
print(b)  # I,love,you
print(c)  # I love you
```

----

#### 分割

##### split

> 分割就是个逆过程，使用`split`方法即可，传入的参数为字符串，该字符串作为分隔符进行分割，如果不传则默认为空白分隔符，比如空格、制表符或换行符

```python
a = 'I love you'
b = 'He, him, and me'
print(a.split())      # ['I', 'love', 'you'] 
print(b.split())      # ['He,', 'him,', 'and', 'me']        
print(b.split(','))   # ['He', ' him', ' and me']     
print(b.split('hi'))  # ['He, ', 'm, and me']
```

**注意：如果指定了分隔符，则返回值内会保留切分得到的空字符串**

```python
a = 'I,,,love,,,you'
print(a.split(','))  # ['I', '', '', 'love', '', '', 'you']
```

> 同时rsplit也是支持的，这个是从后往前分割
>
> 那么聪明的小朋友就要问了，从后往前和从前往后有什么区别捏
>
> 这就要说到传入切割次数了，这俩函数都可以传入最大切割次数
>
> 这样一来两者就有区别了

```python
a = 'I love you'
print(a.split(' '))     # ['I', 'love', 'you']      
print(a.rsplit(' '))    # ['I', 'love', 'you']       
print(a.split(' ', 1))  # ['I', 'love you']     
print(a.rsplit(' ', 1)) # ['I love', 'you']
```

##### partition

> partition也可以实现分割，但是只能分割成三部分，第一是分隔符前的内容，第二是分隔符，第三是剩余内容，也就是说不会分割完

**注意：partition同样支持rpartition从后往前**

```python
a = 'I,love,you'
print(a.split(','))      # ['I', 'love', 'you']      
print(a.partition(','))  # ('I', ',', 'love,you')   
```

> 与split的不同点在于：
>
> 1. split能指定分割次数，而partition只能一次
> 2. split会删除分隔符，而partition会保留
> 3. split返回列表，partition返回元祖

---

### 文本对齐

> `rjust`、`ljust`、`center`方法分别能实现右对齐、左对齐、居中对齐，都是穿插空格实现的，传入的参数为最后字符串的总长度
>
> 也可以传入第二个参数，指定填充字符

```python
a = 'hello world'
print(a.rjust(20))       #          hello world
print(a.ljust(20))       # hello world
print(a.center(20))      #     hello world 
print(a.center(20, '*')) # ****hello world***** 
```

> 所谓右对齐就是字母最右边是贴边的，左边可能有空

---

### 删除字符

> `lstrip`和`rstrip`方法分别能保证左边，右边没有空白字符
>
> `strip`方法会返回一个两边都没有空白字符的字符串
>
> 同样的，我们也可以传入参数，指定删除某个字符串

```python
a = 'hello world'
b = a.center(20)
c = a.rjust(20)
d = a.ljust(20)
e = '*' * 8 + ' ' * 8 + a
print(b.strip())      # hello world
print(c.strip())      # hello world
print(d.strip())      # hello world
print(e.strip())      # ********        hello world
print(e.lstrip('*'))  #         hello world
```

**这里传入的字符串可不是整个匹配，而是内部一个个字符匹配**

```python
x = 'aaaaasdffffrff'
print(x.strip('af'))  # sdffffr
```

---

### 复制&粘贴

> 在使用这个方法之前我们需要先安装`pyperclip`第三方模块
>
> 然后可以调用该模块下的`copy`方法，将传入的字符串复制到系统的剪切板
>
> `paste`则可以返回系统剪切板的内容

```python
import pyperclip

pyperclip.copy('Hello world!') # 复制Hello world到剪切板
print(pyperclip.paste())       # 从剪切板粘贴
```

---

### eval

> 实现字符串和list、dict等序列之间的转换

```python
a = '[1, 2, 3]'
print(type(a))  # <class 'str'>
a = eval(a)
print(type(a))  # <class 'list'> 
print(a)        # [1, 2, 3]  
```

> 还可以用来计算表达式的值

```python
print(eval('2+3'))              # 5
print(eval('print("Hello")'))   # 返回值为None但是会打印Hello
```

---
### 子串判断

#### in

> 可以用来判断子串是否在原字符串中

```python
s = 'abc'
print('bc' in s)     # True           
print('ac' in s)     # False           
print('abc' in s)    # True         
print('c' in s)      # True        
```

---

#### count

`count(str, beg=0, end=len(string))`

> 返回str在字符串string中出现的次数，beg和end有初始值且可以自己改变

```python
a = 'aaaaa'
print(a.count('a'))
print(a.count('a', 0))
print(a.count('a', 1))
print(a.count('a', 1, 5))
print(a.count('a', 1, 4))
```
> 从beg开始查找包括beg，但不包括end，**前闭后开**

```python
a = 'aa aa ab'
print(a.count('aa', 1))
```

---

#### 寻找子串位置

##### find

> 可以用来查找子串在原串首次出现的位置
>
> 同样，存在rfind，从后往前找第一个，也就是最后一个
>
> 找不到都返回-1

```python
a = 'ssr'
s = 'abssr666ssr'
print(s.find(a))     # 2
print(s.rfind(a))    # 8 
print(s.find('t'))   # -1
print(s.rfind('t'))  # -1 
```

---

##### index

> 与find的用处基本一样，我们来说说他们的区别：
>
> 1. find未找到会返回-1，index没找到会报错
> 2. index可以用于其他序列，而find不行

```python
a = 'ssr'
s = 'abssr666ssr'
print(s.index(a))     # 2
print(s.rindex(a))    # 8 
print(s.index('t'))   # 找不到，报错
print(s.rindex('t'))  # 找不到，报错
```

```python
a = [1, 2, 3, '666']
print(a.index(3))   # 2
```

##### 应用

> 查找字符串中每个字符第一次出现的位置

```python
text = '''你猜我喜不喜欢你？'''

for i, char in enumerate(text):
    if i == text.find(char):
        print(f'{char}:{i}')

'''
你:0
猜:1
我:2
喜:3
不:4
欢:6
？:8
'''
```

---
### replace

> 实现查找替换的功能

```py
word = ['测试', '非法', '暴力', '话']
text = '这句话里有非法词汇：测试，非法，暴力，话'
for x in word:
    if x in text:
        text = text.replace(x, '**')
print(text)  # 这句话里有**词汇：**，**，**，**
```

---

### 映射加密

> 我们可以先通过字符串对象的maketrans方法生成映射表，然后再让原串调用translate方法来转换

```python
table = str.maketrans('aeiou', '12345')
print('apple'.translate(table)) # 1ppl2
```

---

### max(str)

`max(str)`

> 返回字符串str中最大的字符

***min同理***

```python
a = 'abcd'
a = a.lstrip('a')
print(max(a))
```

#### 个人理解

> 我们可以将字符串看成一个序列（实际上可能就是这样）
>
> 我们将max方法用于其他序列上时也就是返回该序列的最大值，字符串与字符串比大小也就是比他们的Unicode编码大小，恰好就是比大小咯

---

# 字符串常量

> 在Python的标准库string中定义了许多的常量

```python
import string

print(string.digits)            # 0123456789
print(string.ascii_letters)     # abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ
print(string.punctuation)       # !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
print(string.whitespace)        # \t\n\r\x0b\x0c
print(string.ascii_lowercase)   # abcdefghijklmnopqrstuvwxyz
print(string.ascii_uppercase)   # ABCDEFGHIJKLMNOPQRSTUVWXYZ
print(string.printable)         # 0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~ \t\n\r\x0b\x0c
```

---

## 应用

> 可以用来生成随机密码

```python
import string
import random

def random_string(length):
    letters = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters) for i in range(length))

for i in range(5):
    print(random_string(10))

'''
pw3ccrxsnh
d145rfc5ww
ps1suxjf7s
vnofghsmt4
nbql6ivbfu
'''
```

---

# 项目——口令保管箱

```python
from sys import argv
from pyperclip import *

passwords = {'email' : 123456, 
             'game' : 654321,
             'bank' : 987654
             }

if len(argv) < 2:
    print('Usage: python hello.py [account] - copy account password')
    exit()

account = argv[1]

if account in passwords:
    copy(str(passwords[account]))
    print('Password for ' + account + ' copied to clipboard.')
```

使用时，通过使用命令行参数，传入需要查询的账号对应的密码即可获得

```bash
python .\hello.py email
```

> 查询email对应的密码

# 项目——添加星号

```python
from pyperclip import *

text = paste().split('\n')
for i in range(len(text)):
    text[i] = '* ' + text[i]

text = '\n'.join(text)
copy(text)
```

> 上述代码实现的是：将剪切区的文本按行分割开并在行首都加上了`*`号，做成了一个无序列表
