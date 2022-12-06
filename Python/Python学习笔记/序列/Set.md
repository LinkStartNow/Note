# 集合
标志是大括号，本身可变，但是内部元素是不重复且不可变的，也就是不能把列表塞进去~~元组又占便宜啦哈哈哈~~

---
## 创建
由于大括号与字典的特征冲突了，所以还得让给大哥字典
于是要创建空集合的话只能用set()来生成了
```python
a = {}
b = set()
print(type(a))
print(type(b))
```

---
## 添加元素
由于set是无序的所以不能在尾部插入，也就说没有`append()`，取而代之的是`add()`
```python
a = {1, 2}
a.add(3)
print(a)
```
如果要添加多个元素则可采用`update()`
```python
a = {1, 2}
a.update([1, 3, 5, 5])
print(a)
```

---
## 删除
可以使用`remove()`或者`discard()`删除，区别在于，remove在没有元素删除的情况下会报错，但discard不会
- remove
    普普通通的删除
- discard
    与remove几乎一样的功能，只不过discard删除失败时不会报错，而remove会
- pop
    由于集合是无序的，所以随机删除一个元素
- clear
    删除所有元素

---
## 数学中关于集合的操作
至于这些集合到底是啥意思，建议数学重学哈哈哈
1. 并集
    可以用`|`或`union()`来实现
    ```python
    a = {1, 2, 3}
    b = {2, 4}
    print(a | b)
    print(a.union(b))
    ```
2. 交集
   可以用`&`或者`intersection()`实现
   ```python
   a = {1, 2, 3}
    b = {2, 4}
    print(a & b)
    print(a.intersection(b))
    ```
3. 差集
    可以用`-`或者`difference()`实现
    ```python
    a = {1, 2, 3}
    b = {2, 4}
    print(a - b)
    print(a.difference(b))
    ```
4. 对称差集
    可以用`^`或者`symmetric_difference()`实现
    ```python
    a = {1, 2, 3}
    b = {2, 4}
    print(a ^ b)
    print(a.symmetric_difference(b))
    ```