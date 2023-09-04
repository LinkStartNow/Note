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

> 输出：
>
> ('a', '1')
> ('b', '2')
> ('c', '3')

---
## `*`运算符

`*`运算符可以用来对列表等序列进行解包：

```python
p = [[1, 2, 3], [4, 5, 6]]
c = zip(p)
d = zip(*p)
print(list(c))     # [([1, 2, 3],), ([4, 5, 6],)]          
print(list(d))     # [(1, 4), (2, 5), (3, 6)]           
```

> 实际上c = zip([[1, 2, 3], [4, 5, 6]])
>
> d = zip([1, 2, 3], [4, 5, 6])

```python
def add(x, y):
    return x + y

t = [2, 5]
print(add(*t))
```

> 实际上执行的就是add(2, 5)

### 就地解包

其实不止在函数传参中，在其他地方也可以直接解包

```python
t = [2, 4, 5]
a = [6, 8, t]
b = [6, 8, *t]

print(a)      # [6, 8, [2, 4, 5]]      
print(b)      # [6, 8, 2, 4, 5]        
```

## zip的还原

当然打包的时候就提前结束的是不能完全还原的
```python
a = ['a', 'b', 'c', 'd']
b = ['1', '2', '3']
s = zip(a, b)
s = list(zip(*s))
a, b = s
print(a)    # ('a', 'b', 'c')      
print(b)    # ('1', '2', '3')        
```

**注意：如下代码会出错**

```python
a = ['a', 'b', 'c', 'd']
b = ['1', '2', '3']
s = zip(a, b)
print(list(s))
s = list(zip(*s))
a, b = s
print(a)    # ('a', 'b', 'c')      
print(b)    # ('1', '2', '3')        
```

> 解释：
>
> zip返回的是一个迭代器，而迭代器只能被遍历一次，于是在list(s)使用完迭代器后，s已经失效了
>
> 后面又想解包s于是失败了

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

> 解释：
>
> 原先的s实际上就是`[('f', 'l', 'o', 'w', 'e', 'r'), ('f', 'l', 'o', 'w'), ('f', 'l', 'i', 'g', 'h', 't')]`
>
> 然后拆解后x将依次获得：`('f', 'f', 'f')`、`('l', 'l', 'l')`、`('o', 'o', 'i')`、`('w', 'w', 'g')`
>
> 然后把x转换为set经过去重，如果长度为1，则说明每个字符都是一样的，于是就添加入公共前缀
