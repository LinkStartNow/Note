# stat函数

可以获取当前文件的状态

需要包含头文件：

```c
#include <sys/types.h> 

#include <sys/stat.h> 

#include <unistd.h>
```

有三种调用方式：

1. `int stat(const char *path, struct stat *buf); `
2. `int fstat(int fd, struct stat *buf); `
3. `int lstat(const char *path, struct stat *buf);`

最终的状态都存在传入的结构体对象中

**注意：第一个和第三个的区别是：第一个获取的是真实文件的inode，而第三个获取的是软链接指向的块的inode也就是aa块的inode**

代码：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int main() {
        struct stat st;
        if (stat("sylink", &st) == -1) {
                perror("stat fail");
                return 1;
        }
        printf("file size:%d, inode num:%d\n", st.st_size, st.st_ino);

        if (lstat("sylink", &st) == -1) {
                perror("stat fail");
                return 1;
        }
        printf("file size:%d, inode num:%d\n", st.st_size, st.st_ino);
        return 0;
}
```

# access函数

一般用于测试文件权限或者文件是否存在

```c
#include <unistd.h>

int access(const char *pathname, int mode);
```

R_OK 是否有读权限 

W_OK 是否有写权限 

X_OK 是否有执行权限 

F_OK 测试一个文件是否存在

```c
#include <unistd.h>
#include <stdio.h>

int main() {
        if (-1 == access("aaxx", F_OK)) {
                perror("access fail");
                return 1;
        }
        printf("file exist.\n");
        return 0;
}
```

# chmod函数

```c
#include <sys/stat.h> 

int chmod(const char *path, mode_t mode); 

int fchmod(int fd, mode_t mode);
```

**注意：这里的模式一般都是三位8进制数，而传入的是字符串，直接用atoi转换成的是十进制数**

于是我们要手动将传入的8进制数值转换成对应的10进制数值

```c
#include <sys/stat.h>
#include <stdio.h>

int convert(int o) {
        int res = 0;
        int cnt = 1;
        while (o) {
                res += (o % 10) * cnt;
                cnt *= 8;
                o /= 10;
        }
        return res;
}

int main(int argc, char* argv[]) {
        // 判断参数个数是否正确
        if (argc != 3) {
                printf("参数个数错误！\n");
                return 1;
        }

        // 转换权限
        int mode = convert(atoi(argv[2]));

        // 改变文件权限
        if (-1 == chmod(argv[1], mode)) {
                perror("chmod fail");
                return 1;
        }

        return 0;
}
```

# 目录相关操作

## mkdir

创建目录

```c
#include <sys/stat.h> 

#include <sys/types.h>

int mkdir(const char *pathname, mode_t mode);
```

## rmdir

删除目录

```c
#include <unistd.h> 

int rmdir(const char *pathname);
```

## 打开目录

```c
#include <sys/types.h> 
#include <dirent.h>

DIR *opendir(const char *name); 
DIR *fdopendir(int fd);
```

## 读取文件

```c
#include <dirent.h>

struct dirent *readdir(DIR *dirp);

struct dirent { 

ino_t d_ino;                            /* inode number */ 

off_t d_off;                              /* offset to the next dirent */ 

unsigned short d_reclen;     /* length of this record */ 

unsigned char d_type;          /* type of file; not supported by all file system types */ 

char d_name[256];                /* filename */ 

};
```

readdir每次返回一条记录项，DIR*指针指向下一条记录项

## 查询文件实践

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <dirent.h>

void walkDir(char* path) {
	// 打开文件夹
	DIR* dir = opendir(path);
	if (NULL == dir) {
		perror("opendir fail");
		return;
	}

	// 循环读取文件夹记录项
	struct dirent* dp = NULL;
	char pathName[1024] = "";
	while (NULL != (dp = readdir(dir))) {
		// 判断是不是..和.
		if (0 == strcmp("..", dp->d_name) || 0 == strcmp(".", dp->d_name)) {
			continue;
		}

		// 判断拼接完的路径长度是否超过数组长度，没超过就拼接，超过就打印错误
		if (sizeof(pathName) < strlen(path) + strlen(dp->d_name) + 2) {
			printf("%s/%s is too long\n", path, dp->d_name);
		}
		else {
			sprintf(pathName, "%s/%s", path, dp->d_name);
			ll(pathName);
		}

	}

	// 关闭文件夹
	closedir(dir);
}

// 打印文件的绝对路径和大小
void ll(char* path) {
	// 判断文件是否存在并获取文件属性
	struct stat st;
	if (-1 == stat(path, &st)) {
		perror("stat fail");
		return;
	}

	// 判断路径是否是文件夹
	if (S_IFDIR == (st.st_mode & S_IFMT)) {
		// 如果是文件夹，就进入文件夹
		walkDir(path);
	}
	else {
		// 如果不是文件夹就直接打印绝对路径和大小
		printf("%s:%d\n", path, st.st_size);
	}
}

int main(int argc, char* argv[]) {
	if (argc == 1) {
		// 访问当前路径
		ll(".");
	}
	else {
		// 循环遍历所有路径
		for (int i = 1; i < argc; ++i) {
			ll(argv[i]);
		}
	}
	return 0;
}
```

# 虚拟文件系统

文件描述符可以用来指向一个文件

这里有两个函数来复制文件描述符

```c
#include <unistd.h>

int dup(int oldfd); 

int dup2(int oldfd, int newfd);
```

> dup和dup2都可用来复制一个现存的文件描述符，使两个文件描述符指向同一个file结构体。如果两个文件描述符指向同一个file结构体，File Status Flag和读写位置只保存一份在file结构体中，并且file结构体的引用计数是2。如果两次open同一文件得到两个文件描述符，则每个描述符对应一个不同的file结构体，可以有不同的File Status Flag和读写位置。请注意区分这两种情况。

## 实践代码

**注意：在打开a文件时一定要加上写入权限，不然后面的写入就会出问题**

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main() {
	// 创建一个a文件
	umask(0000);
	int fd = open("a", O_CREAT | O_RDWR, 0664);
	if (-1 == fd) {
		perror("open fail");
		return 1;
	}
	
	// 复制标准输出的文件描述符
	int fd_save = dup(STDOUT_FILENO);
	
	// 让标准输出的文件描述符指向a文件
	dup2(fd, STDOUT_FILENO);

	// 往标准输出的文件描述符写文字
	write(STDOUT_FILENO, "hello world\n", sizeof("hello world\n"));

	// 使用printf打印日志
	printf("this is a test\n");

	// 关闭文件
	close(fd);
	close(fd_save);

	return 0;
}
```

> 标准输出的文件描述符指向a文件后，printf也会写进a文件，而不会显示到窗口上
