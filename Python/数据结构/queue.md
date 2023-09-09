# 队列

## 创建

创建队列对象需要导入库`queue`，获取该模块下的`Queue`

```python
import queue

q = queue.Queue()
```

## 入队

调用`put`方法

```q.put(1)
q.put(1)
q.put(2)
```

## 出队

调用`get`方法

```python
while not q.empty():
    print(q.get())
```

## 判空

判空通过`empty`方法即可

```python
print(q.empty())
```

# 优先队列

在队列中的所有对象都会通过优先级进行排序，由于底层是通过堆实现的，所以取出和插入对象的时间复杂度都为`logn`

## 创建

导入`queue`模块下的`PriorityQueue`

```python
from queue import PriorityQueue

pq = PriorityQueue()
```

## 入队

入队方式和普通的队列一样，使用`put`即可

**注意：这里传入的第一个是优先级**

```python
pq.put((1, 'a'))           
pq.put((-2, 'b'))
pq.put((3, 'c'))
```

## 出队

出队也是直接调用`get`即可

```python
print(pq.get())       # (-2, 'b')     
```

## 查询

我也不知道为啥，这个优先队列居然可以使用下标，也可以直接查看整个队列

```python
print(pq.queue)       # [(-2, 'b'), (1, 'a'), (3, 'c')]       
print(pq.queue[1])    # (1, 'a')       
```

## 自定义对象优先级

很多时候我们会将自己调用的对象放入优先队列

此时，我们就需要自定义该对象的`__lt__`方法，类似于C++中的重载

**注意：在自定义的时候只需要返回`self < other`的正确与否，不要返回什么-1啥的，因为非零都算是True**

```python
from queue import PriorityQueue

class yyds:
    def __init__(self, pri, val):
        self.pri = pri
        self.val = val
    
    def __lt__(self, other):
        return self.pri < other.pri
    
pq = PriorityQueue()
pq.put(yyds(5, 3))
pq.put(yyds(2, 7))
pq.put(yyds(3, 6))
while pq.empty() != True:
    t = pq.get()
    print(t.val)

  # 7   
  # 6   
  # 3   
```

当然如果自己声明的类有了独特的`__lt__`方法后，也可以进行sort等排序操作

