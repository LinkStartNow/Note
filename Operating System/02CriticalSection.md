# 创建

首先要有一个CRITICAL_SECTION类型的对象

然后使用`InitializeCriticalSection`函数给它初始化

```c++
InitializeCriticalSection(&cri);
```

然后就能使用了

# 机制

当有线程进入某个临界区时，其他线程不能进入这个临界区，会处于阻塞状态一直等待

可以通过`EnterCriticalSection`进入临界区

```c++
EnterCriticalSection(&cri);
```

当某个线程用完这个临界区后要记得离开临界区，让其他线程进入

```c++
LeaveCriticalSection(&cri);
```

最后不用的时候删除临界区即可

```c++
LeaveCriticalSection(&cri);
```

# 具体类的定义

```c++
#pragma once
#include <Windows.h>

class MyCri {
	CRITICAL_SECTION cri;

public:
	MyCri();

	~MyCri();

	void Enter();

	void Leave();
};
```

```c++
#include "MyCri.h"

MyCri::MyCri() {
	InitializeCriticalSection(&cri);
}

MyCri::~MyCri() {
	DeleteCriticalSection(&cri);
}

void MyCri::Enter() {
	EnterCriticalSection(&cri);
}

void MyCri::Leave() {
	LeaveCriticalSection(&cri);
}
```

