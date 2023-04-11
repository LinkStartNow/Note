# 创建

创建的时候可以使用`CreateSemaphore`函数，即可以用来创建一个信号，也可以用来获得存在信号的`HANDLE`，若信号存在则直接得到那个信号的handle并忽略其他的传参，以存在的那个为准

```c++
// 创建一个信号
if (CreateSemaphore(nullptr, // 安全级别为默认
                    0, // 初始值设定为0
                    2, // 最大值设定为1
                    L"MySemaphore")) // 名字为 MySemaphore
    cout << "Semaphore create success!" << endl;
else {
    cout << "Semaphore create error:" << WSAGetLastError() << endl;
}
```

# 打开

打开和创建不同，打开只能打开存在的信号，得到它的handle，若是不存在则会打开失败返回空

```c++
// 打开一个信号
sem = OpenSemaphore(SEMAPHORE_ALL_ACCESS, // 访问权限为全部
                    false,	// 不然后代继承
                    L"MySemaphore"); // 信号名为 MySemaphore
```

# 机制

用等待函数来实现阻塞：

```c++
WaitForSingleObject(sem,		// 信号的handle
                    INFINITE);  // 等待时间为无限
```

当信号的计数大于0时可以进入，等于0时就一直处于阻塞等待

当进入后，计数会自动减一，然后通过释放来增加计数，如此往复

---

# 释放

当一个行为做完了后可以释放一下，让信号的计数增加，从而让其他的线程（或进程）能结束等待继续执行

我们可以使用`ReleaseSemaphore`函数：

```c++
ReleaseSemaphore(sem, // 信号的handle
                 2,   // 信号计数的增量，可以由此来一次释放多个计数，让更多进程进入
                 nullptr); // 用来接收计数改变前的指针
```

**注意：若释放的数量加上现有的大于了总数则会释放失败，总数不增加**