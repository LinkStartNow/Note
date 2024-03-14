# 基础知识

进程是最小的分配资源单位，线程是最小的调度单位

每个进程都会自带一个线程，这个线程称为主控线程，其他线程均为普通线程

在初期，所有普通线程都是由主控线程创建出来的

线程就是寄存器和栈，可以访问寄存器即是可以使用CPU，栈表示PCB内核站，线程可以保存和恢复处理器现场，既可以保存恢复现场又可以使用CPU

开发者可以摘系统下安装线程库，可以在不支持线程的系统中使用线程，这类线程无法被系统识别，无法被CPU分配时间片，一般时间片以进程为单位分配

**注意：如果进程退出，则所有相关的线程也会立即退出，多线程模型中一班主线程最后退出**

## 某些查看线程的指令

查看系统下所有线程：

```bash
ps -eLf
```

> LWP表示轻量级进程编号，**注意与tid区分，他俩是无关的**

查看指定进程中的线程：

```bash
ps -Lf pid
```

> 其中的pid就是对应进程的pid

# 线程分类

每个系统支持的调度单位都会有一个内核对象，便于系统访问

线程技术中，某些线程系统是不支持的，所以这些线程无法得到更多系统资源

## 用户级线程

线程技术中，某些线程系统是不支持的，所以这些线程无法得到更多系统资源，这类线程就称为用户级线程

用户级线程虽然不能被直接分配CPU，但是可以共享进程得到的CPU资源，提高CPU利用率，提高效率

用户级线程的切换无需内核参与，用户级线程是安装在用户层的，线程直接访问即可，无需频繁的层级转换

## 内核级线程

系统会为每个线程创建一个内核对象，给这些线程分发CPU时间片

内核级线程可以获取更多时间片，但是层级转换需要系统参与，于是频繁调度更耗费资源

## 混合型线程

吸收了以上两种的部分优点，可以被CPU识别创建内核对象，分配时间片，安装在用户层，可以直接访问，无需层级转换

但是在部分系统下才能使用，兼容性较差

# 资源的共享

## 共享资源

1. 全局变量
2. 进程信息（pid、uid、gid、进程工作路径）
3. 堆空间共享
4. 文件描述符共享
5. 信号行为共享（信号的处理方式）
6. library库空间共享

## 非共享资源

1. 线程tid
2. 处理器现场和内核栈指针
3. 用户空间栈（线程栈）8M
4. 信号屏蔽字
5. 调度优先级
6. errno变量（错误号）非共享

# 线程原语（NPTL）

## 使用前提

- 包含头文件：

	```c
	#include <pthread.h>
	```

- 编译线程代码时要链接库：

	```bash
	gcc a.c -lpthread
	```

## 线程函数

主函数接口其实就是主控线程代码：

```c
int main(void){}
```

普通线程代码框架如下：

```c
void* thread_job(void* arg)
```

## 创建线程

可以调用`pthread_create`方法

```c
int err = pthread_create(pthread_t* tid, NULL, void* (*thread_job)(void*), void* arg);
```

### 参数

1. 传出成功的线程tic
2. 线程属性
3. 线程工作地址（也就是线程函数指针）
4. 线程工作参数

### 返回值

返回整形的错误号，调用成功则为0，失败则返回错误号（`>0`），后续利用err进程错误处理

## 查看线程tid

可以调用`thread_self()`方法

返回调用该函数的线程的tid

## 回收线程

线程和进程一样，退出后需要回收才行，不回收会产生僵尸线程（内存泄漏），TCB残留

一般我们调用`pthread_join()`方法

该方法为阻塞函数，如果线程未退出则一直等待，线程退出后立即回收，并得到线程的返回值（也就是最后return的值，存在reval中）

```c
pthread_join(pthread_t tid, void** reval)
```

## 杀死线程

我们可以调用`pthread_cancel()`方法杀死线程

```c
pthread_cancel(pthread_t tid)
```

**注意：要通过cancel结束目标线程，该线程必须存在系统调用**

比如，像这样的线程函数则不能用cancel杀死，**因为检测不到系统调用**

```c
void* yyds(void* arg) {
	while (1) {
		sleep(1);
	}
	return (void*)666;
}
```

为了解决这个问题，我们一般配合`pthread_testcancel()`方法使用

```c
pthread_testcancel();
```

该方法会产生一次系统调用，并不会做任何事，便于检查cancel事件

**如果线程被cancel取消，即没有执行到return，则返回值为-1，会被join得到**

***因此，开发者不要使用`-1`作为返回码，保留给cancel***

### 实践

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

void* yyds(void* arg) {
	while (1) {
		//sleep(1);
		pthread_testcancel();
	}
	return (void*)666;
}

int main() {
	pthread_t tid;
	void* reval;
	pthread_create(&tid, NULL, yyds, NULL);
	sleep(10);
	pthread_cancel(tid);
	while (1) {
		sleep(1);
	}
	return 0;
}
```

## 分离态

线程分为*回收态*和*分离态*，**默认情况下是回收态**

- 回收态：线程退出后需要其他线程通过join回收，否则会产生内存泄漏
- 分离态：线程退出后，系统自动回收线程资源，用户无法获得线程的返回值或退出码

**注意：线程状态的改变不可逆，回收态可以变成分离态，而分离态不能变为回收态**

### 分离态与回收态互斥

1. 不允许对分离线程进行回收操作，这样做join函数会失败返回
2. 不能对处于回收阶段（join已经在等待回收了）的线程设置分离，设置也不会成功

### 回收态转分离态

我们可以调用`pthread_detach()`方法

```c
pthread_detach(pthread_t tid)
```

### 实践

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

void* yyds(void* args) {
	printf("thread 0x%x set detach...\n", (unsigned int)pthread_self());
	pthread_detach(pthread_self());
	return (void*)666;
}

int main() {
	pthread_t tid;
	pthread_create(&tid, NULL, yyds, NULL);
	printf("master waiting for joining...\n");
	sleep(0);
	int err;
	void * reval;
	if ((err = pthread_join(tid, &reval)) > 0) {
		printf("pthread join failed:%s\n", strerror(err));
		exit(0);
	}
	printf("master thread join success, reval = %ld\n", (long int)reval);

	while (1) sleep(1);
	return 0;
}
```

## 线程的四种退出方式

**线程没有严格主从限制，所有的方法，主线程可以使用，普通线程也可以，例如线程的创建和回收等**

1. `return`
	- 普通线程执行return，当前线程退出
	- 主线程执行return，整个进程退出，所有线程退出
2. `exit`
	- 普通线程执行，进程退出，所有线程退出
	- 主线程执行，也同上
3. `cancel`
	- 只要得到目标线程的tid，就可以取消目标
4. `pthread_exit`
	- 普通线程使用，结束当前线程
	- 主线程使用，结束当前线程，不影响进程

# 线程属性

在创建线程的时候可以设置线程属性，如果传NULL，表示使用默认线程属性

线程属性允许开发者自定义线程

## 默认属性

线程属性结构体（`struct pthread_attr_t`），有这些属性：

- ### 线程优先级指针

	改变线程的权限和优先级

- ### 线程警戒缓冲区（4K）

	在线程栈拼接的一块空间，如果线程溢出访问警戒缓冲区，进程马上抛出错误（非法操作内存）

- ### 线程退出状态（PTHREAD_JOINABLE）

	决定线程是分离态还是回收态

- ### 线程栈地址与大小

	- stack_addr = NULL

	- stack_size = 0

		默认就是8M

## 查看线程属性的默认退出状态

```c
int detachstate;
pthread_attr_getdetachstate(&attr, &detachstate);
if (detachstate == PTHREAD_CREATE_JOINABLE) {
	printf("thread DFL exit state, JOINABLE\n");
}
else {
	printf("thread DFL exit state, DETACH\n");
}
```



## 修改属性中的线程退出状态

1. 定义线程属性

	```c
	pthread_attr_t attr;
	```

2. 初始化线程属性，初始化后为默认属性

	```c
	pthread_attr_init(&attr);
	```

3. 修改属性中退出状态，变为分离态

	```c
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
	```

4. 属性设置完毕，创建线程，一定要使用自定义属性

	```c
	pthread_t tid;
	pthread_create(&tid, &attr, yyds, NULL);
	```

5. 使用完毕销毁属性

	```c
	pthread_attr_destroy(&attr);
	```

### 完整代码

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>
#include <signal.h>

void* job(void* arg) {
	while (1) sleep(1);
}

int main() {
	pthread_attr_t attr;
	pthread_attr_init(&attr); // 初始化后为默认属性
	// 查看属性中的推出状态
	//int detachstate;
	//pthread_attr_getdetachstate(&attr, &detachstate);
	//if (detachstate == PTHREAD_CREATE_JOINABLE) {
	//	printf("thread DFL exit state, JOINABLE\n");
	//}
	//else {
	//	printf("thread DFL exit state, DETACH\n");
	//}
	// 修改属性中的状态，改为分离态
	pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
	pthread_t tid;
	pthread_create(&tid, &attr, job, NULL);
	int err;
	if ((err = pthread_join(tid, NULL)) > 0) {
		printf("join failed:%s\n", strerror(err));
		exit(0);
	}
	pthread_attr_destroy(&attr); // 销毁属性
	
	return 0;
}
```

## 修改栈地址与栈大小，提高线程的创建数量

### 获取栈地址与大小

```c
pthread_attr_getstack(&attr, void** stackaddr, size_t* stacksize);
```

### 设置属性中的栈地址与大小

```c
pthread_attr_setstack(&attr, void* stackaddr, size_t stacksize);
```

### 实践

**注意：只修改属性中的栈大小，并不能让系统申请自定义大小栈空间，需要用户自行申请（malloc）**

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

void* yyds(void* arg) {
	while (1) sleep(1);
}

int main() {
	// 定义线程属性
	void* stackaddr;
	size_t stacksize;
	pthread_attr_t attr;
	pthread_attr_init(&attr);
	pthread_attr_getstack(&attr, &stackaddr, &stacksize);
	printf("DFL attr, stackaddr %p, stacksize %d\n", stackaddr, (int)stacksize);
	stacksize = 0x100000;
	pthread_t tid;
	int err;
	int cnt = 0;
	while (1) {
		// 申请栈空间
		if ((stackaddr = (void*)malloc(stacksize)) == NULL) {
			perror("malloc fail");
			exit(0);
		}
		pthread_attr_setstack(&attr, stackaddr, stacksize);

		if ((err = pthread_create(&tid, &attr, yyds, NULL)) > 0) {
			perror("thread create fail");
			exit(0);
		}
		printf("thread %d\n", ++cnt);
	}
	pthread_attr_destory(&attr);
	return 0;
}
```

## 两种线程设置分离的方法

1. 通过`pthread_detach()`改变线程状态
2. 通过修改属性创建分离态线程

### 对比

- 如果需要大批量创建分离线程，需使用修改属性的方法，开销更小
- 只有部分线程是分离的，大多数是回收态，则手动用`detach`函数分离

