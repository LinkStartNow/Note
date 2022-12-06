# List
非常常用的一个类型，跟其他类型也很相似，可以举一反三

---
## 列表的生成
1. 中括号直接创建
    `a = [1, 2]`
2. list中加可迭代对象(如string、range)
    `b = list('abc')`
    ***注意这里的字符串会被分成一个一个字母***
    `c = list(range(1, 7, 2))`
3. 巨无敌的推导式
    参考推导式详情，这里不细讲

---
## append()
将元素加到最后
```python
a = [1, 2, 2]
a.append(3)
print(a)
```

---
## pop()
弹出末尾的元素，尽量用这个不要用del去删除中间的元素
`a.pop()`

---
## del
空格后跟随需要删除的对象
删除列表或列表中的元素,***尽量不要使用，会产生大量的时间消耗***
```python
a = [1, 2, 2]
del a[2]
print(a)
```

---
## 乘法
与string类似可以使用乘法进行复制
`l = [1, 2, 3] * 3`

---
## extend(x)
x为扩展的对象,可以是列表、元组、字符串等对象
合并列表尽量用这个，不要用“+”号,***因为加号会产生新的对象，浪费资源***
```python
a = [1, 2]
b = ['a', ['b', 'c']]
a.extend(b)
print(a)
```

---
## insert(pos, value)
在pos位置插入value元素，原本pos位置的元素全部后移，于是 ***浪费大量时间***，尽量避免使用
```python
a = [1, 2]
a.insert(1, 3)
print(a)
```

---
## remove(x)
删除第一次出现的x
```python
a = [1, 2, 1]
a.remove(1)
print(a)
```

---
## index(v, [beg, [end]])
返回v第一次出现的位置beg和end都是**选填**
```python
a = [1, 2, 3, 5, 7, 3]
print(a.index(3, 3))
```

---
## 排序
1. `sort()`
    内部还能补充reverse属性，默认为假，为真时倒着排序
    ```python
    a = [1, 5, 3, 4]
    a.sort(reverse=True)
    print(a)
    ```
2. `sorted()`
    外部的函数会产生新的列表对象,括号中第一个填排序的对象，后面可以选补reverse属性
    ```python
    a = [1, 5, 3, 4]
    print(sorted(a, reverse=True))
    ```

---
## 逆置
1. `reverse()`
    跟前面一样内置直接把原对象逆置
    ```python
    a = [1, 5, 3, 4]
    a.reverse()
    print(a)
    ```
2. `reversed`
    逆置同样也存在外部函数，这次是返回一个逆置列表的生成器，需要生成或者用for遍历使用
    ```python
    a = [1, 5, 3, 4]
    s = reversed(a)
    for x in s:
        print(x)
    ```

---
## 成员资格判断
- 直接用`count`数个数，如果为0则不在
    ```python
    a = [1, 2, 3]
    print(a.count(3))
    ```
- 用`in`来判断
    ```python
    a = [1, 2, 3]
    print(3 in a)
    ```

---
## 其他
- 与字符串相似的
    - `len()`: 查询列表的元素个数
    - `min()`/`max()`: 返回列表的最小/大值
    - `count()`: 数列表中指定元素的个数
- `sum()`: 求和函数，***前提是列表的元素必须全是数字***
