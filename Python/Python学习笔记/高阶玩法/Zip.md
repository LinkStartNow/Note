# Zip函数
把多个序列打包成多个元组，序列中最短的结束打包也结束，返回一个tuple迭代器

---
## 普通打包
```python
a = ['a', 'b', 'c', 'd']
b = ['1', '2', '3']
s = zip(a, b)
for x in s:
    print(x)
```

---
## 逆操作
也就是zip的还原，当然打包的时候就提前结束的是不能完全还原的
```python
a = ['a', 'b', 'c', 'd']
b = ['1', '2', '3']
s = zip(a, b)
s = list(zip(*s))
a, b = s
print(a)
print(b)
```

---
## 同时遍历多个字典
```python
a = {'name': 'hd', 'age': 19}
b = {'name': 'zly', 'age': 19}
z = zip(a.items(), b.items())
for (k1, v1), (k2, v2) in z:
    print(k1, v1)
    print(k2, v2)
```

---
## 对多个元素同时排序
```python
a = ['hd', 'zly', 'hhh']
b = [100, 97, 30]
p = list(zip(a, b))
p.sort()
print(p)
```
如果想以分数排序则
```python
a = ['hd', 'zly', 'hhh']
b = [100, 97, 30]
p = list(zip(b, a))
p.sort()
print(p)
```

---
## 将数据成对进行计算
```python
a = [5000, 6000, 8000]
b = [2000, 2500, 3000]
for x, y in (zip(a, b)):
    print(x - y)
```

---
## 构建字典
由于字典是键和值一一对应，所以只能是两个列表通过这种方式变成字典
```python
a = ['name', 'score', 'age']
b = ['zly', 100, 19]
c = dict(zip(a, b))
print(c)
```

---
## 通过*zip秒杀最长公共前缀
```python
s = ['flower', 'flow', 'flight']
res = ''
for x in zip(*s):
    if len(set(x)) == 1:
        res += x[0]
print(res)
```

