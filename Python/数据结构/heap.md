# 创建

要使用堆，首先要导入相关的模块

可以直接调用`heapify`方法将列表转换为堆

这样排序过的列表就满足了堆的规律，默认第一个是最小的，之后就可以正常堆操作了

当然，如果是空列表就不需要变换了

```python
import heapq

l = [2, 3, 5, 8, 1, 7, 9, 0]

heapq.heapify(l)

print(l)              # [0, 1, 5, 3, 2, 7, 9, 8]
while l:
    print(heapq.heappop(l), end='\t')
# 0       1       2       3       5       7       8       9
```

# 插入元素

插入就调用`heappush`方法

第一个参数是堆对象，第二个是插入的元素

```python
heapq.heappush(l, 233)
```

# 弹出堆首元素

就是`heappop`方法

```python
while l:
    print(heapq.heappop(l), end='\t')
```

# 元素替换

可以用一个元素替换掉最小值

`heapreplace`

```python
import heapq

l = [2, 3, 5, 8, 1, 7, 9, 0]

heapq.heapify(l)



print(l)              # [0, 1, 5, 3, 2, 7, 9, 8]

heapq.heapreplace(l, 233)

while l:
    print(heapq.heappop(l), end='\t')
# 1       2       3       5       7       8       9       233
```

