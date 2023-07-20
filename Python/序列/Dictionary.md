# 字典

把字典转换成其他序列时，不另外调用函数的话是把键进行拆分

用`in`判断字典对象时是对其所有的键进行判断，而不会判断值

如下：

- 判断`19 in d`会返回`False`因为键中没有`19`,`19`是值

- 判断`'age' in d`会返回`True`因为键中有`'age'`

```python
d = {'name' : 'hhh', 'age' : 19}
print(19 in d, 'age' in d) # False True
```

---
## 创建
1. 直接用大括号创建
   `a = {'name' : 'zly', 'age' : 19}`
2. 使用`dict`函数
   1. 使用键值对
        `b = dict(name = 'hd', age = 19)`
    2. 列表套元组
        `c = dict([('name', 'dsk'), ('age', 19)])`
3. zip函数创建
    前面的放键后面的放值
    ```python
    k = ['name', 'age', 'job']
    v = ['me', 19, 'student']
    d = dict(zip(k, v))
    ```
4. 通过`fromkeys`创建空值字典，内部用列表
    `f = dict.fromkeys(['name', 'age'])`

---
## 访问
1. 直接用键访问
    `print(a['name'])`
2. 通过get函数用键访问，较推荐，不存在时返回None
   `print(a.get('name'))`
3. items()
    列出所有键值对
4. keys()
   列出所有键
5. values()
   列出所有值

---
## 删除
1. `del`直接删除键值对
   `del a['sex']`
2. `pop`删除指定的键值对并返回值
   `print(a.pop('age'))`
3. `popitem`删除最后一个键值对并返回

---
## 遍历
1. 直接遍历键
   ```python
   for x in a:
     print(x, a[x])
   ```
2. 遍历键值对的元组
    ```python
    for x in a.items():
     print(x)
    ```
3. 把元组也拆开分配出去
    ```python
    for x, y in a.items():
        print(x, y)
    ```

---
## 拼接
字典没有加法，拼接两个字典只能用`update()`
如果存在**相同**的键则会 ***直接覆盖值***
```python
a = {1: 6, 2: 3, 4: 7}
b = {3: 5}
a.update(b)
print(len(a))
```

---

## setdefault方法

可以用来设置某个键对应值的默认值，可以用来计数

如果没有该键值对而直接用的话就会报错

```python
import pprint
count = {}
str = '22332336348724'
for x in str:
    count.setdefault(x, 0)
    count[x] += 1
# for x in count.items():
#     print(x)
print(count)
pprint.pprint(count)
print(pprint.pformat(count))
```

