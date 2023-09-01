# 创建

创建队列对象需要导入库`queue`，获取该模块下的`Queue`

```python
import queue

q = queue.Queue()
```

# 入队

调用`put`方法

```q.put(1)
q.put(1)
q.put(2)
```

# 出队

调用`get`方法

```python
while not q.empty():
    print(q.get())
```

# 判空

判空通过`empty`方法即可

```python
print(q.empty())
```

