# 打开文件

使用`open`方法

`int open(const  char  *pathname, int  flags, ...);`

- 文件路径可以指定，若不指定则默认为当前文件夹下

- flags参数有一系列常数值可供选择，可以同时选择多个常数用按位或运算符连接起来，所以这些常数的宏定义都以O_开头，表示or。 

	- 必选：

		- \* O_RDONLY  只读打开 
		- \* O_WRONLY  只写打开 
		- \* O_RDWR  可读可写打开

	- 可选

		- \* O_APPEND  表示追加。如果文件已有内容，这次打开文件所写的数据附加到文件的末尾而不覆盖原来的内容。 

		- \* O_CREAT  若此文件不存在则创建它。使用此选项时需要提供第三个参数mode，表示该文件的访问权限。 

			**注意：这个不像c语言，没有文件会默认创建，只有加上这个参数才会在没有文件的时候创建，否则打开失败**

		- \* O_TRUNC  如果文件已存在，并且以只写或可读可写方式打开，则将其长度截断（Trun-cate）为0字节。 

			***类似于c语言中的w或者w+模式***

**返回值如果为-1则说明有错误**

## 实践代码

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main() {
        umask(0000);
        //int fd = open("xx",O_RDONLY |  O_CREAT, 0777);
        int fd = open("xx",  O_RDONLY);
        printf("fd = %d\n", fd);
        close(fd);
        return 0;
}
```

> 这些头文件要记得包含，都需要用到
>
> 这里umask是临时设置了掩码，只对该程序生效

---

## 手动实现touch

代码：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
        // 判断参数个数是否正确
        if (argc < 2) {
                printf("please input file name\n");
                return 1;
        }

        // 循环创建文件
        int fd = 0;
        umask(0000);
        for (int i = 1; i < argc; ++i) {
                // 创建文件
                fd = open(argv[i], O_CREAT, 0664);
                if (fd < 0) {
                        printf("create %s file fail.\n", argv[i]);
                        continue;
                }

                // 关闭文件
                close(fd);
        }
        return 0;
}
```

> 这里利用了main函数可以传参数
>
> argc默认保存参数的个数
>
> argv则保存传入的字符串

**注意：argv[0]是第一个字符串，也就是`./a.out`，所以正常的文件名应该是从1开始的**

---

# 文件的读与写

`ssize_t read(int fd, void *buf, size_t count); `方法

返回成功读取的字节数，出错返回-1并设置error，如果调用之前就已经读到了文件末尾则返回0

`ssize_t write(int fd, const  void *buf, size_t count); `

成功写入返回字节数，出错返回-1并设置error

## 手动实现copy

代码：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
        // 判断参数个数是否正确
        if (argc != 3) {
                printf("parameter number error\n");
                return 1;
        }

        // 打开源文件，以只读方式
        int fd_src = open(argv[1], O_RDONLY);
        if (fd_src < 0) {
                printf("open file %s fail\n", argv[1]);
                return 1;
        }

        // 打开目标文件
        umask(0000);
        int fd_dest = open(argv[2], O_WRONLY | O_TRUNC | O_CREAT, 0664);
        if (fd_dest < 0) {
                printf("open file %s fail\n", argv[2]);
                close(fd_src);
                return 1;
        }

        // 循环读取源文件
        int nlen = 0;
        char buf[1024] = "";
        while ((nlen = read(fd_src, buf, sizeof(buf))) > 0) {
                // 每次读入成功，写入目标文件
                write(fd_dest, buf, nlen);
        }

        // 关闭源文件和目标文件
        close(fd_src);
        close(fd_dest);
        return 0;
}
```

---

# 获取错误

有三种获取错误的方式

## 方式1：

输出错误代码，也就是errno，需要包含errno.h头文件

`printf("errno = %d\n", errno);`

## 方式2：

直接输出错误，调用`perror`方法

**注意：不需要包含头文件**

`perror("error");`

> 输出：
>
> error:错误原因

## 方式3：

通过strerror获取错误原因字符串，然后手动拼接输出

`printf("open  fail: %s\n", strerror(errno));`

> 输出：
>
> open fail：错误原因

## 实践代码：

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <fcntl.h>
#include <string.h>
#include <errno.h>

int main() {
        int fd = open("123", O_RDONLY);
        if (fd < 0) {
                printf("errno = %d\n", errno);
                perror("error");
                printf("open  fail: %s\n", strerror(errno));
        }
        return 0;
}
```

# 阻塞&非阻塞

读常规文件是不会阻塞的，不管读多少字节，read一定会在有限的时间内返回。从终端设备或网络读则不一定，如果从终端输入的数据没有换行符，调用read读终端设备就会阻塞，如果网络上没有接收到数据包，调用read从网络读就会阻塞，至于会阻塞多长时间也是不确定的，如果一直没有数据到达就一直阻塞在那里。同样，写常规文件是不会阻塞的，而向终端设备或网络写则不一定。 

## 阻塞

代码：

```c
#include <unistd.h>

int main() {
        char buf[1024] = "";
        int nlen = read(STDIN_FILENO, buf, sizeof(buf));

        write(STDOUT_FILENO, buf, nlen);
        return 0;
}
```

## 非阻塞

对于已经打开的文件，我们依旧可以对它设置属性

比如说标准输入输出文件就是已打开的，我们要实现非阻塞，就需要在后面手动修改

我们可以通过`fcntl`方法获取文件属性，再通过`fcntl`方法追加属性（通过按位或`|`）

## 实践代码

```c
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

int main() {
	// 获取标准输入文件的属性
	int flags = fcntl(STDIN_FILENO, F_GETFL);

	// 在原有属性的基础上增加非阻塞属性
	flags |= O_NONBLOCK;

	// 把属性重新设置到标准输入文件中
	if (-1 == fcntl(STDIN_FILENO, F_SETFL, flags)) {
		perror("fcntl fail");
		return 1;
	}

	// 循环读数
	int nlen = 0;
	char buf[1024] = "";
	while (1) {
		// 读入标准输入文件
		nlen = read(STDIN_FILENO, buf, sizeof(buf));

		// 判断返回值，如果返回值小于0
		if (nlen < 0) {
			// 判断errno是否是EAGAIN
			if (errno == EAGAIN) {
				// 休眠一会再读
				sleep(10); // 休眠10秒
			}
			else {
				// 读取出错，打印错误日志
				perror("read fail");
				return 1;
			}
		}
		else {
			// 如果返回值大于0,则说明读取到了内容，就结束循环
			break;
		}
	}

	// 把读取到的内容写入标准输出文件中
	write(STDOUT_FILENO, buf, nlen);
	return 0;
}
```

# 文件最大打开数

- 查看当前系统允许打开最大文件个数

	`cat  /proc/sys/fs/file-max`

- 当前默认设置最大打开文件个数1024

	`ulimit -a`

- 修改默认设置最大打开文件个数为4096

	`ulimit -n  4096`

## 实践代码

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main() {
        char name[100] = "";
        int fd = 0;
        umask(0000);
        for (int i = 0; ; ++i) {
                sprintf(name, "file%d", i);
                fd = open(name, O_CREAT, 0664);
                if (fd < 0) {
                        perror("open fail");
                        return 1;
                }
        }
        return 0;
}
```

> 观察最终能生成多少个文件，即打开的个数

# lseek方法

每个打开的文件都记录着当前读写位置，打开文件时读写位置是0，表示文件开头，通常读写多少个字节就会将读写位置往后移多少个字节。但是有一个例外，如果以O_APPEND方式打开，每次写操作都会在文件末尾追加数据，然后将读写位置移到新的文件末尾。lseek和标准I/O库的fseek函数类似，可以移动当前读写位置（或者叫偏移量）。

```c
#include <sys/types.h>

#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
```

参数offset和whence的含义和fseek函数完全相同。只不过第一个参数换成了文件描述符。和fseek一样，偏移量允许超过文件末尾，这种情况下对该文件的下一次写操作将延长文件，中间空洞的部分读出来都是0。 

若lseek成功执行，则返回新的偏移量，因此可用以下方法确定一个打开文件的当前偏移量

## 实践代码

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char* argv[]) {
	// 判断参数个数是否合法
	if (argc < 2) {
		printf("please input file name\n");
		return 1;
	}

	// 打开文件
	int fd = open(argv[1], O_RDONLY);
	if (fd < 0) {
		perror("open fail");
		return 1;
	}

	// 移动光标到文件尾
	int size = lseek(fd, 0, SEEK_END);

	// 打印返回值，就是文件大小
	printf("file size = %d\n", size);

	// 移动光标到文件开头后10个字符
	lseek(fd, 10, SEEK_SET);

	// 读取文件内容，输出到标准输出文件中
	int nlen = 0;
	char buf[1024] = "";
	while (nlen = read(fd, buf, sizeof(buf))) {
		write(STDOUT_FILENO, buf, nlen);
	}

	// 关闭文件
	close(fd);

	return 0;
}
```

