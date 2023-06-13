# 进程间通讯

进程间通讯技术简称IPC，可以利用此技术让多个进程间相互传递信息数据，有大量的方案，例如：

1. 匿名管道pipe
2. 命名管道fifo
3. 消息队列Posix
4. 消息队列System
5. 信号Signal
6. 套接字Socket
7. 文件映射表mmap

**注意：绝大多数的进程间通讯是在内核层完成的，因为内核层内存共享**

# 匿名管道pipe

## 缺点

只能实现有亲缘关系的通讯，比如子进程、孙子进程之类的

## 使用

```c
int fds[2];
pipe(fds);
```

> fds用来装两个文件描述符
>
> 通过pipe方法将文件描述符写入数组fds中
>
> 其中fds[0]用来读，fds[1]用来写

每多一个文件描述符就会多一个引用

当close一个文件描述符后就会减少一个引用，当引用为0时就会销毁该管道

也可以直接调用`shutdown`方法将引用计数清0

一般情况下，管道是单工，于是使用时要确定通信方向，父写子读或者子写父读，父子进程分别将不用的描述符关闭

## 例子

```c
#include <stdio.h>
#include <sys/wait.h>

#define msg "hello world!"

int main() {
	int fds[2];
	pid_t pid;
	
	// 0:read,   1:write
	pipe(fds);

	pid = fork();

	if (pid == 0) {
		close(fds[1]);
		
		char buf[1024];
		bzero(buf, sizeof(buf));

		read(fds[0], buf, sizeof(buf));

		printf("child received:%s\n", buf);

		close(fds[0]);

	}
	else if (pid > 0) {
		close(fds[0]);
		write(fds[1], msg, strlen(msg));
		wait(NULL);
		close(fds[1]);
	}
	else {
		perror("fork called fail");
		exit(0);
	}

	return 0;
}
```

## 特殊情况

**注意：以下的四种特殊情况具有普遍意义，在其他技术上也有通用之处（如socket等）**

1. 当读端和写端都存在时，读端读完管道内的数据后，若想再次读则会阻塞
2. 当读端和写端都存在时，若管道已经满了，写端想写入，则会阻塞
3. 若写端不存在，读端读完数据后，再读会读到0
4. 若读端不存在，写端想写入数据，则信号会杀死写端进程（**无论管道是否满了**）

# 命名管道fifo

也是一个内核缓冲区，为环形队列结构，大小为4k，这是管道的特点也就是说匿名管道也是如此

命名管道可以解决无关进程间的通信

## 与文件的不同

管道通信与传统的文件传输不同，传统文件内容被读取后不会删除文件内容，但是管道是队列结构，出队的数据会从管道消失

而且文件是会被分配磁盘空间的，最少有4k，可以做持久化存储

但是管道文件并不会被分配磁盘空间，所以不能做持久化存储

## 使用

首先可以用`mkfifo`指令创建一个fifo“文件”

通过ls可以看出该文件是pipe类型的文件，而且颜色是黄色的，所以实际上并不是个文件，而是个设备

然后用正常读写文件的方式使用该文件就行了

## 简单理解

管道文件是一个指向内核管道缓冲区的指针，所有向管道文件的读写操作，都会重定向到内核管道中

## 例子

### 写

```c
#include <stdio.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>

#define msg "hello"

int main() {
	int fd = open("tstfifo", O_WRONLY);
	
	write(fd, msg, strlen(msg));

	printf("write success!\n");

	return 0;
}
```

### 读

```c
#include <stdio.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>

#define msg "hello"

int main() {
	int fd = open("tstfifo", O_RDONLY);
	
	char buf[1024];
	bzero(buf, sizeof(buf));

	read(fd, buf);

	printf("read success: %s\n", buf);

	return 0;
}
```

> 实际上在执行的时候，只执行其中任何一个都是会被阻塞在open那里的，原因是：要打开fifo文件必须要读和写都有，所以打开任意一个只能有读或写其中一个钥匙，于是只运行一个是运行不了的

**实际上只要凑齐两种权限就可以访问了，即使直接用RDWR也可以直接打开**

如下一个进程就可以直接访问了

```c
#include <stdio.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>

#define msg "hello"

int main() {
        int fd = open("tstfifo", O_RDWR);
        
        write(fd, msg, strlen(msg));

        printf("write success!\n");

        char buf[1024];

        read(fd, buf, sizeof(buf));

        printf("read:%s\n", buf);

        return 0;
}
```

## 特殊情况

### 阻塞非阻塞

一个读端中存在多个读序列的话，阻塞读只对第一个读序列有效，其他序列被设置为非阻塞

常见于多线程中，一个线程的多个连起来的read只是一个读序列，所以多个线程读的时候只有一个线程阻塞着，其他的线程都去干别的事了

### 原子访问与非原子访问

原子访问的时候，每次访问缓存区都会进行原子判定，添加的量要小于等于容器的余量，否则会被阻塞，知道满足要求再次被唤醒

优点：可以保证完整性，避免被拆分

缺点：传输效率较低，不能最大化利用容器空间

---

非原子访问可以最大化利用容器空间，只有容器满时才会限制写端，读写效率传输效率高

缺点：数据会被多次拆分，会对读端造成一些麻烦，读端要验证数据包完整性后才可以使用

**注意：决定是原子还是非原子取决于用户发的数据包大小，如果小于等于管道大小则是原子传输，否则是非原子**

