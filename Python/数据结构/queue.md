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

# 双向队列

`queue.Queue`主要为了线程间通信，而队列只是附带功能，而`collections.deque`才是真正的容器

使用双向队列可以比`Queue`快很多

实际上`Queue`的底层就是用`deque`实现的

```python
# Override these methods to implement other queue organizations
# (e.g. stack or priority queue).
# These will only be called with appropriate locks held

# Initialize the queue representation
def _init(self, maxsize):
    self.queue = deque()
```

接下来，我们来讲讲如何使用这个双向队列

## 创建

需要导入`collections`模块

```python
import collections

q = collections.deque() 
```

当然，他的初始化比较高级，可以直接用一个可迭代对象进行初始化

```python
import collections

q = collections.deque(range(10))
q2 = collections.deque([x for x in range(10) if x % 2])

print(q)             # deque([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])     
print(q2)            # deque([1, 3, 5, 7, 9])  
```

## 入队

双向队列顾名思义，两头都可以用，而且都是既能出又能入的

```python
import collections

q = collections.deque() 

q.append(2)
q.append(4)
q.append(3)
print(q)               # deque([2, 4, 3])

q.appendleft(233)
q.appendleft(666)
print(q)               # deque([666, 233, 2, 4, 3])
```

当然，还能通过`extend`直接添加可迭代对象，同样支持两端添加

```python
import collections

q = collections.deque() 

q.append(2)
q.append(4)
q.append(3)
print(q)               # deque([2, 4, 3])

q.appendleft(233)
q.appendleft(666)
print(q)               # deque([666, 233, 2, 4, 3])

q.extend(range(5))
print(q)               # deque([666, 233, 2, 4, 3, 0, 1, 2, 3, 4])

q.extendleft(range(3))
print(q)               # deque([2, 1, 0, 666, 233, 2, 4, 3, 0, 1, 2, 3, 4])
```

## 插入

这个其实有点离谱了，但是你没看错，它真的能插入

盲猜一手，这个就是套了个壳的list吧

```python
print(q)               # deque([2, 1, 0, 666, 233, 2, 4, 3, 0, 1, 2, 3, 4])

q.insert(2, 123)
print(q)               # deque([2, 1, 123, 0, 666, 233, 2, 4, 3, 0, 1, 2, 3, 4])
```

## 弹出元素

直接用`pop`

```python
import collections

q = collections.deque(range(10))

print(q.pop())         # 9          
print(q.popleft())     # 0          
print(q)               # deque([1, 2, 3, 4, 5, 6, 7, 8]) 
```

## 查看元素

> 双端队列只能在查看两端的元素
>
> 这里比较特殊，可以通过下标的方式在不弹出的情况下查看两端的元素

```python
import collections

q = collections.deque()

q.append(233)
q.append(666)

print(q[-1])    # 666     
print(q[0])     # 233    
```

> 当然在迫不得已的情况下，这个双向队列甚至可以当做list用，可以通过下标直接查找中间的元素，但是能别用还是别用，时间消耗应该挺高的

```python
from collections import deque
q = deque()
q.append(1)
q.append(2)
q.append(3)
print(q[1])  # 2
```

