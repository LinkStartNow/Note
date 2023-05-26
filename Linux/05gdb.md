# 进入调试模式

`gdb 可执行文件名字`

例如：`gdb app2`

> debug可执行文件APP2

# start指令

开始调试，会停在main函数的第一行

# next指令

快捷键为`n`

逐过程，遇到函数不进入

**注意，有断点的话还是会进入的**

# step指令

快捷键为`s`

逐语句，遇到函数进入函数

# 回车

执行上一条指令

# p指令

查询变量的值

例如：`p m`

> 查询m的值

# 持续观察

- display 变量名：持续观察某个变量的值
- undisplay 变量的编号：结束持续观察

# 断点相关

## 下断点

快捷键为`b`

break 行号/函数名：下断点

例如：`break 10`

> 在第10行下断点

又如：`b qwpj`

> 在qwpj函数的第一行下断点

---

## 查看断点信息

还可以通过`info breakpoints`指令查看断点信息

## run指令

从开头执行到第一个断点处，没有断点则直接运行到程序结尾

**注意：这个第一个断点指的是按照文件运行的顺序看的，也就是从main函数开始执行**

## continue指令

继续运行到下一个断点处

## 禁用断点

`disable 断点编号`

## 重新使用断点

`enable 断点编号`

# 退出调试模式

`quit`即可退出

或者快捷键q也行
