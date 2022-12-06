# Lambda函数

## 赋值给一个变量，通过变量调用
```python
add = lambda x, y : x + y
print(add(5, 6))
```

---
## 赋值给其他函数进行替换
```python
add = lambda x, y : x + y
print(add(5, 6))
add = lambda *a : None
print(add(5, 6))
```

---
## 在高阶函数中使用匿名函数
### map()
`map(function, iterable, ...)`
第一个参数为function，对参数序列中每一个元素调用function函数，返回包含每次function函数返回值的新列表的生成器
```python
def ssr(x):
    return x * x

# 方式一，用现成的函数
r = list(map(ssr, [1, 2, 3]))
print(r)

# 方式二，用lambda函数
r = list(map(lambda x : x ** 3, [1, 2, 3]))
print(r)

# 方式三，多个列表，相同索引位置的元素参与运算
# 以最短的列表为准
r = list(map(lambda x, y, z : x + y + z, [1, 2, 4, 5], [3, 2, 7], [5, 6]))
print(r)
```

---
### sorted()
`sorted(iterable, /, *, key=None, reverse=False)`
`iterable`是可迭代对象
`key`是比较规则
`reverse`是排序规则，为True则倒着排，默认为False
排序函数，返回一个列表
```python
a = [(100, 'kenny'), (33, 'boy'), (43, 'girl'), (100, 'zly')]

# 以下标为0的元素排序
a = sorted(a, key= lambda x : x[0])
print(a)
```

---
### filter()
`filter(function, iterable)`
筛选符合条件的元素，返回一个新的列表生成器
```python
def is_odd(x): return x % 2 == 0
a = [1, 2, 3, 4, 5]

# 直接用现成的函数
r = list(filter(is_odd, a))
print(r)

# 用lambda函数
r = list(filter(lambda x : x % 2 == 1, a))
print(r)
```