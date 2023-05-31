# 优点

1. 可以便捷的管理代码，不需要每次都写复杂的编译命令
2. 大大节省编译时间，减少编译成本
3. 重用性强，可以反复使用编译不用的项目

节省编译时间:采用空间换时间的方式，第一次编译的时候不会节省编译时间，但是会保存编译生成的中间文件.o文件，第二次编译的时候，只编译修改的源文件，没有编译的源文件不会重新编译，会使用保存下来的.o文件，这样大大节省了再次编译的时间。

# 使用要求

- 文件名字：只能是makefile或者Makefile

- 执行命令：make

- 注释：#

# 三要素

1. 目标：最终要生成的，或者要生成最终的过程中需要的中间目标
2. 依赖：生成最终目标需要的文件或者资源
3. 命令：使用依赖生成目标要执行的命令

# 格式

```
生成目标名称:依赖文件
	命令
```

**注意：命令前要用tab键做缩进**

举例：

```
myapp:main.o add.o des.o mul.o sub.o
	gcc main.o add.o des.o mul.o sub.
```

**注意：这里的命令和原本的命令一模一样，也就是说目录也需要在这里指定**

使用举例：
```
 #version 1.0 
 myapp:add.o des.o main.o mul.o sub.o
     gcc add.o des.o main.o mul.o sub.o -o myapp
 
 add.o:add.c
     gcc add.c -c -I../include
 
 des.o:des.c
     gcc des.c -c -I../include
  
 main.o:main.c
     gcc main.c -I../include -c
 
 mul.o:mul.c
     gcc mul.c -c -I../include
  
 sub.o:sub.c
 gcc sub.c -c -I../include
```

---

# 进阶语法

## 变量

1. 自定义变量

	没有数据类型，都是字符串

	变量名只能是字母数字下划线，为了和高级语言区分，一般使用全大写的变量

	不允许数字开头

	使用变量：$(变量名)

2. 特殊变量（makefile内置）

	- $@：表示目标名
	- $^：表示所有依赖项
	- $<：表示第一个依赖项

举例：

```
 TST = "this is a test"
 
 output:
     echo $(TST)

```

**注意：这里的output是固定的输出文件，一定要这么写**

## 内置函数

1. `wildcard`：文件名处理函数

	举例：`SOURCEFILE = $(wildcard *.c)`

	> 获取当前路径下所有的.c文件的名字，赋值给SOURCEFIEL
	>
	> 这里的SOURCEFILE 是变量名，可以随便取

2. `patsubst`：字符串处理函数

	举例：`DFILE = $(patsubst %.c, %.o, $(SOURCEFILE))`

	> 把SOURCEFILE变量中保存的所有的.c文件替换成.o文件，赋值给DFILE

## 内建语法

```
%.o:%.c
	生成命令
```

最终利用上面这些特性化简完makefile就变成了，如下的简略形式

```
#version 1.0

SRC = $(wildcard *.c)
OBJ = $(patsubst %.c, %.o, $(SRC))

myapp:$(OBJ)
        gcc $^ -o $@

%.o:%.c
        gcc $< -c -I../include
```

## 常用变量定义

```
#存储目标名字
TARGET = myapp
#存储编译器版本
CC = gcc
#存储头文件路径
INCLUDE_PATH = ../include
#存储库文件路径
LIBRARY_PATH = ../lib
#存储安装路径
INSTALL_PATH = /etc/bin
```

## 编译选项

- `CFLAGS = -I(INCLUDE_PATH) -c -Wall`
- `CPPFLAGS = -D -L -O3`：预处理选项

## 功能目标

```
1、清理：
clean:
	rm -rf $(DFILE) $(TARGET)
2、安装：
install:
	sudo cp $(TARGET) $(INSTALL_PATH)
3、卸载：
disclean:
	sudo rm -rf $(INSTALL_PATH)/$(TARGET)
4、输出：
output:
	echo $(INSTALL_PATH)/$(TARGET)
```

### 使用

如果要使用功能目标，格式为：`make 功能名`

如：`make clean`：表示调用clean下的指令

---

## 最终完善版

```
#version 1.0

myapp:$(OBJ)
#存储目标名字
TARGET = myapp
#存储编译器版本
CC = gcc
#存储头文件路径
INCLUDE_PATH = ../include
#存储库文件路径
LIBRARY_PATH = ../lib
#存储安装路径
INSTALL_PATH = /etc/bin

CFLAGS = -I$(INCLUDE_PATH) -c -Wall
CPPFLAGS = -D -L -O3

clean:
        rm -rf $(DEFILE) $(TARGET)
install:
        sudo cp $(TARGET) $(INSTALL_PATH)
disclean:
        sudo rm -rf $(INSTALL_PATH)/$(TARGET)
output:
        echo $(INSTALL_PATH)/$(TARGET)

SRC = $(wildcard *.c)
DEFILE = $(patsubst %.c, %.o, $(SRC))

$(TARGET):$(DEFILE)
        $(CC) $^ -o $@

%.o:%.c
        $(CC) $< $(CFLAGS)
```

