# Print的一丢丢知识

***详情还是可以直接看[大佬的博客](https://blog.csdn.net/sinat_28576553/article/details/81154912?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166454023016782388065745%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166454023016782388065745&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-81154912-null-null.142^v51^pc_rank_34_2,201^v3^control_2&utm_term=python%E7%9A%84print&spm=1018.2226.3001.4187)***
`print(*objects, sep=' ', end='\n', file=sys.stdout)`
这是参数列表，可见print是不限制输出的类型的，不管是啥都能输出，毕竟传入进去的全变成元组中的元素了
```python
a, b, c = 1, 'abc', 'd'
print(a, b, c, 'hello', '!')
```
当然通过参数还能看出，`sep`和`end`都是可以设置的，也就是说由什么补充间隔和由什么结尾是可以自己设置的。这也解释了为什么每次输出默认都由空格隔开，且会自动回车换行
```python
a, b, c = 1, 'abc', 'd'
print(a, b, c, 'hello', '!', sep='.', end='hello world!')
```

---
# 格式化输出

## `%`字符

> 可以类比c语言的%，前面先是一个字符串，后面紧跟%和一个元组
> 这个字符串称为 ***格式控制符***，后面的元组是 ***转换说明符***，最终会把元组中的元素按顺序带入前面的占位符

### 基础使用

```python
a = 'ac'
b = 'yes'
print('我%s了,%s'%(a, b))  # 我ac了,yes
```

---

### 最小子段宽度和精度

> 在 ***格式控制符*** 中的%后用来设置输出的样式
> 如`%x.yf`说明子段宽度为x，小数点后保留y位
> `print('%6.3f'%3.14)`
> 其中的x和y也可以先用`*`占位，最后同样从元组中读出他俩的值，当然x和y也可以不填使用默认值
> `print('%*.*f'%(10, 10, 3.14))`

---

### 转换标志

不知道为啥这些转换标志好像 ***不能叠加使用***

 1. `-`表示左对齐（默认右对齐）
	也就是说有空格什么的填充字符是填充在右侧的
	`print('%-*.*f'%(10, 10, 3.14))`
 2. `+`表示数值前加上正负号（也就正数需要，毕竟负数本来就带着负号）
	`print('%+*.*f'%(10, 10, -3.14))`
 3. `0`表示位数不够用`'0'`填充
	`print('%0*.*f'%(10, 1, 3.14))`

---

## `f`创建格式化字符串

### 基础使用

```python
a = 123
b = '你'

print(f'{b} 是 {a}') # 你 是 123
```

---

## format函数

### 一点介绍

> 基本语法就是通过`{}`和`:`来代替以前的`%`
>
> {}中可以填写后面元组中的下标，也可以填写后面的字段名
>
> format函数与%不一样的地方就是，%需要知道替换字符的类型，而format不需要

### 基础使用

```python
a = 'hi'
b = 'ac了!'

print('{}, 我{}'.format(a, b)) # hi, 我ac了!
```

### 使用字段名

> 实际上就是在原本的大括号内填入某些字段，然后在format函数中再传入即可

**注意：这里的字段不像字典的键一定要用字符串这样的不变量，这里就相当于函数的形参，后面的是实参**

```python
a = 'hi'
b = 'ac了!'

print('{打招呼}, 我{感叹}'.format(打招呼 = a, 感叹 = b)) # hi, 我ac了!
```

### 使用下标

> 其实就是括号内给出后面元组对应的下标填入字符串

**注意：下标要写在冒号前面**

```python
print('{1}， {0}'.format('hello', 'world'))  # world， hello
```

### 逗号

> 逗号可以用来做千位分隔符

**注意：逗号以及后面的格式限定符都放在冒号之后**

```python
print('{:,}'.format(12345678))  # 12,345,678
```

### 控制格式

> 在冒号后面可以输入原本百分号后面的格式限定

```python
# 总长为10，保留两位小数，不足的左填0
print('{:010.2f}'.format(3.1415))  # 0000003.14
```

#### 格式限定符

> 我们可以设置填充字符，填充字符为冒号后面的第一个字符，不指定则默认为空格，这里我们需要配合对齐符号一起使用
>
> ^、<、>：分别代表居中、左对齐、右对齐

```python
print('{:p^10.2f}'.format(3.1415))  # ppp3.14ppp
```

### 高端写法

> 将字符串的format方法整体作为一个函数对象，来调用

```python
weather = [('Monday', 'rainy'), 
            ('Tuesday', 'sunny'),
            ('Wednesday', 'sunny'),
            ('Thursday', 'rainy'),
            ]

formatter = 'Today is {} and it is {}'.format

print(formatter(weather[0][0], weather[0][1]))  # Today is Monday and it is rainy
```

可以循环跑

```python
weather = [('Monday', 'rainy'), 
            ('Tuesday', 'sunny'),
            ('Wednesday', 'sunny'),
            ('Thursday', 'rainy'),
            ]

formatter = 'Today is {} and it is {}'.format

for item in weather:
    print(formatter(*item))

'''
Today is Monday and it is rainy
Today is Tuesday and it is sunny
Today is Wednesday and it is sunny
Today is Thursday and it is rainy
'''
```

与下面代码等价

```python
weather = [('Monday', 'rainy'), 
            ('Tuesday', 'sunny'),
            ('Wednesday', 'sunny'),
            ('Thursday', 'rainy'),
            ]

formatter = 'Today is {} and it is {}'.format

for item in map(formatter, *zip(*weather)):
    print(item)

'''
Today is Monday and it is rainy
Today is Tuesday and it is sunny
Today is Wednesday and it is sunny
Today is Thursday and it is rainy
'''
```

为了看懂上面的解包与zip的套用，我们拆成了下面代码帮助理解

```python
weather = [('Monday', 'rainy'), 
            ('Tuesday', 'sunny'),
            ('Wednesday', 'sunny'),
            ('Thursday', 'rainy'),
            ]

formatter = 'Today is {} and it is {}'.format

print(*weather)              # ('Monday', 'rainy') ('Tuesday', 'sunny') ('Wednesday', 'sunny') ('Thursday', 'rainy')      
print(list(zip(*weather)))   # [('Monday', 'Tuesday', 'Wednesday', 'Thursday'), ('rainy', 'sunny', 'sunny', 'rainy')]        
print(*(zip(*weather)))      # ('Monday', 'Tuesday', 'Wednesday', 'Thursday') ('rainy', 'sunny', 'sunny', 'rainy')    

for item in map(formatter, ('Monday', 'Tuesday', 'Wednesday', 'Thursday'), ('rainy', 'sunny', 'sunny', 'rainy')):
    print(item)

'''
Today is Monday and it is rainy
Today is Tuesday and it is sunny
Today is Wednesday and it is sunny
Today is Thursday and it is rainy
'''
```

