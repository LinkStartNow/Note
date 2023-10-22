# 库

> 使用该函数需要导入functools模块

```python
from functools import reduce
```

---

# 作用

> 可以将一个接受两个参数的函数以迭代的方式从左到右作用到一个序列上，并返回最终的值

# Code

```python
from functools import reduce

l = [1, 3, 5, 2, 4]
print(reduce(lambda x, y: x + y, l))  # 15
```

## 分析

> 上述的reduce实际完成的是：
>
> 从头到尾的连加，l[0] + l[1] + l[2] + ……

