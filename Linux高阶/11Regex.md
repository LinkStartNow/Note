# 正则元字符

正则表达式规则都差不多，这里主要强调一些在C语言中需要转义的：

- +：出现一次或多次，如`abc+`

- {}：控制出现次数，例如`b{3,}`

- ()：创建表达式，小括号内的字符表达式会被当成一个整体，如`(abc\|ssr)+`

	**小括号还能创建子表达式哦**

- |：逻辑或只有在中括号`[]`中使用无需转义，其他位置都需要

## 妙用

我们都知道上尖括号`^`可以用来匹配首部，美元符号`$`可以用来匹配尾部，于是`^$`可以用来匹配空行

# 命令使用正则

命令使用正则需要调用`grep`函数，格式为：

```bash
grep '正则表达式' 文件名
```

> 其中正则表达式外面可以用单引号也可以用双引号，因为是在命令行使用的

**注意：-v可以实现取反，配合重定向可以实现去除空行**

```bash
grep '^$' -v file > newfile
```

## 匹配过程

正则语句描述字符串规则，然后调用grep函数，从数据源拷贝一行到模式空间

如果模式匹配成功则标记，失败则丢弃

正则会自动遍历迭代数据源，查找到文件末尾为止

# 函数使用正则

首先需要包含正则表达式的头文件：

```c
#include <regex.h>
```

然后创建一个正则表达式类型的变量用于匹配

```c
regex_t reg;
```

## 创建正则类型变量

可以调用`regcomp`函数

```c
// 创建正则类型变量
regex_t reg;
char* str = "<name>\\([^<]\\+\\)</name>";
regcomp(&reg, str, 0);
```

## 模式匹配

在有了正则类型变量后，就可以利用它对源字符串进行模式匹配，如果匹配到则会返回0

```c
regmatch_t m[2];
regexec(&reg, p, 2, m, 0)
```

> 其中reg是正则类型对象
>
> p是源字符串
>
> 2为总共的分组：
>
> 正则表达式为："<name>\\([^<]\\+\\)</name>"
>
> 其中整个串是一个组，小括号内的又是另一个组，所以是有两个组，其中整个串为第一组，小括号内的按顺序排第2、3……组
>
> m为匹配类型的数组
>
> 最后一位一般填0

最终的m数组就是处理好了的，输出参数

## 匹配类型

```c
regmatch_t re;
```

匹配类型变量包含两个属性：

1. re.rm_so：起始位置
2. re.rm_eo：末尾位置

利用这两个属性就可以轻松取出匹配的所需要的内容了

## 释放正则类型空间

最后调用`regfree`函数即可

```c
regfree(&reg);
```

## 完整实践

```c
#include <stdio.h>
#include <regex.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/mman.h>

int main() {
	int fd = open("file1", O_RDWR);
	
	// 获取文件的大小
	int size = lseek(fd, 0, SEEK_END);
	printf("size = %d\n", size);

	char* p = NULL;
	if ((p = mmap(NULL, size, PROT_READ, MAP_PRIVATE, fd, 0)) == MAP_FAILED) {
		perror("mmap failed");
		return -1;
	}
	close(fd);
	// 创建正则类型变量
	regex_t reg;
	char* str = "<name>\\([^<]\\+\\)</name>";
	regcomp(&reg, str, 0);

	regmatch_t m[2];
	char result[1024];
	while (regexec(&reg, p, 2, m, 0) == 0) {
		snprintf(result, m[1].rm_eo - m[1].rm_so + 1, "%s", p + m[1].rm_so);
		printf("%s\n", result);
		p += m[0].rm_eo; // 取完第一行，指针往后偏移
	}
	regfree(&reg);
	
	printf("end\n");

	return 0;
}
```

