# 字符串
由于字符串的某些特性与序列极其相似，故姑且把它当做“序列”，***实则并不是哦***
详细版可以 ***[戳这里](https://www.lintcode.com/course/9/learn/?chapterId=61&sectionId=471&extraParams=%7B%22moduleSource%22%3A%22lc-course%3A9%22%7D)***

---
## 格式化字符串
***具体[链接](https://blog.csdn.net/aaaaaaze/article/details/123020600?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166470495516782395315081%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166470495516782395315081&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-123020600-null-null.142^v51^pc_rank_34_2,201^v3^control_2&utm_term=python%E6%A0%BC%E5%BC%8F%E5%8C%96%E5%AD%97%E7%AC%A6%E4%B8%B2&spm=1018.2226.3001.4187)***
1. 使用`%`占位符
    ```python
    a = 23
    name = 'tom'
    s = '%s的年龄是%d'%(name, a)
    print(s)
    ```
2. `format()`方法
    一般格式为：`{<索引>:<填充字符><对齐方式><宽度.精度><格式>}`
    - 设置参数
        ```python
        a = ['hello', 'world', 'say', "'s"]
        print('zly {0[2]} {0[3]} {0[0]} {0[1]}'.format(a))
        ```
    - 数字格式化
        - 百分比格式
            `print('{:.2%}'.format(0.12345))`
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
## 万能切片:[左边界:右边界:步长]
**这个切片在列表等类型中也可以用**
左边界元素的位置到右边界元素的位置，可以正数负数混用，只看元素的位置.步长可正可负，默认为1,为正则从左到右，为负则从右到左
***注意：步长的正负与起始位置要对应，步长表示从左到右则起位置必须在终位置左边。起点永远是起点确定完方向也是从起点（左边界）开始的***
### 一般切片
```python
a = 'abcdef'
print(a[1:-1:2])
```
如果像这样不匹配则会输出空
```python
a = 'abcdef'
print(a[1:-1:-2])
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
## eval()
实现字符串和list、dict等序列之间的转换
```python
a = '[1, 2, 3]'
print(type(a))
a = eval(a)
print(type(a))
print(a)
```

---
## center(width, fillchar)
返回一个宽度为width居中的字符串，fillchar为填充字符默认为空格
```python
a = 'a'
a = a.center(5, '*')
print(a)
```

---
## count(str, beg=0, end=len(string))
返回str在字符串string中出现的次数，beg和end有初始值且可以自己改变
```python
a = 'aaaaa'
print(a.count('a'))
print(a.count('a', 0))
print(a.count('a', 1))
print(a.count('a', 1, 5))
print(a.count('a', 1, 4))
```
从beg开始查找包括beg，但不包括end，**前闭后开**
```python
a = 'aa aa ab'
print(a.count('aa', 1))
```

---
## join(seq)
以字符串为指定分隔符，把seq(**一般是列表之类的**)合成一个新的字符串
```python
a = ['s', 'd', 'r']
a = '.'.join(a)
print(a)
```

---
## ljust(width, fillchar)
返回一个左对齐的字符串，width是总长度，fillchar是补充的字符默认为空格，显示的效果是fillchar从左往右补充
***rjust与这个同理***
```python
a = 'abcd'
a = a.ljust(5, '*')
print(a)
```

---
## lstrip()
截掉字符串左边的空格或指定字符(括号中内容)
***rstrip同理***
***strip就是同时执行lstrip和rstrip***
```python
a = 'abcd'
a = a.lstrip('a')
print(a)
```

---
## max(str)
返回字符串str中最大的字符
***min同理***
```python
a = 'abcd'
a = a.lstrip('a')
print(max(a))
```

---
## split(str='', num=string.count(str))
以str为分隔符截取字符串，若num有值则最终截取num+1个字符串
```python
a = 'a b c d'
a = a.split(' ', 2)
print(a)
```
---
## islower()
判断字符串是否全是小写，同理`isupper()`判断是否全是大写。
当然也可以用c的方式，毕竟`'a' > 'b'`也是可以用的
