> 线程能实现并发，即同步执行，非常的好用

# 创建

线程一般由一个HANDLE来接收，实际上就是一个指针用来指向这个线程

```c++
m_pHandle = CreateThread(nullptr,			// 某个结构体的指针，用来判断是否可继承
                         0,					// 设置初始大小，0的话默认为1mb
                         fun,				// 线程函数，有指定的格式建议去复制
                         lp,					// 线程函数的参数，void*类型
                         CREATE_SUSPENDED,	// 挂起状态，0则为执行状态
                         nullptr);			// 指向线程标识符变量的指针，空则不返回
```

---

# 运行

只要线程处于执行状态，则线程函数会执行下去直到执行完

每一次挂起后再恢复也会继承上次的进度继续执行

要恢复线程只需要用这个函数：

```c++
ResumeThread(m_pHandle);
```

---

# 挂起

一个函数就可以实现线程的挂起：

```c++
SuspendThread(m_pHandle);
```

---

# 回收

所有的`HANDLE`用完之后都要记得回收：

```c++
CloseHandle(m_pHandle);
```

---

# 其他

剩下具体的去看其他笔记中关于线程的知识，这里不逼逼了，详情见`QT`与`NetWork`
