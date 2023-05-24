# 查看版本号

直接输入指令`gcc -v`即可查看版本号，以及其他的具体信息

# 编译生成可执行文件

`gcc 要编译的源文件`：默认生成a.out

如果添加了`-o 可执行文件名`，即可生成指定名称的可执行文件

例如：`gcc add.c Main.c`：编译add.c和Main.c并生成a.out

`gcc add.c Main.c -o myapp`：编译add.c和Main.c并生成myapp`

# 运行可执行文件

最终运行的时候就按照文件路径运行，如果是当前目录下的文件也需要加`./`

例如运行当前目录下的文件myapp，`./myapp

# 指定头文件目录

可以通过`-I路径`即可添加头文件目录

**注意：`-I`和路径之间没有空格**

例如：`gcc add.c Main.c -I../include`

> 指明头文件目录为当前文夹的父文件夹下的文件夹include

# 显示警告信息

`-Wall`就可以显示警告信息

> 警告信息是指不影响编译的信息，不理会也可以运行

# 编译时定义宏

可以通过`-D`在编译时定义宏

例如：`gcc add.c Main.c -I../include -D_DEBUG_`

> 在编译的时候宏定义了`_DEBUG_`

Main.c的内容为：
```c
#include <stdio.h>
#include "add.h"

int main() {
        int d = 0;
        int c = add(3, 4);
#ifdef _DEBUG_
        printf("debug\n");
#else
        printf("no debug\n");
#endif
        printf("3 + 4 = %d\n", c);
        return 0;
}
```

# 包含调试信息

`-g`可以在编译时包含调试信息

**注意：如果没有调试信息，是不能正常进行调试的哦！**

具体调试操作看`gdb`

# 生成.o文件

可以在命令中加入参数`-c`，会只编译生成`.o`文件而不生成可执行文件

