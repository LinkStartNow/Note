# 神奇的map函数
之前在lambda中已经初步了解过map了，这里浅谈一下常见的用法吧。
首先回顾一下map函数
`map(function, iterable, ...)`
map函数的第一个参数是进行运算处理的函数，对后续的参数序列进行映射变换
```python
a = [1, 2, 3]
a = list(map(lambda x : x * x, a))
print(a)
```
很多情况下读入多个数据的时候会很难受（因为要一个一个从string变成int等类型）
如下：
```python
a, b, c = input().split()
a = int(a); b = int(b); c = int(c)
print(type(a), b, c)
```
这里聪明的同学就知道可以利用`map`优化一下(别忘记int强制类型转换也是个映射函数)
```python
a, b, c = map(int, input().split())
print(type(a), b, c)
```
