# 注意点：

## 1.前导空格会被忽略，放在后面的空格会被记录

## 2.缩进只能用tab而不能用空格代替

## 3.顺序执行问题

```makefile
target = tstapp
src = $(wildcard *.c)
ooo = $(patsubst %.c, %.o, $(src))

$(target) : a.o
        gcc $^ -o $@

*.o : *.c
        gcc $^ -c



#output:
#       echo  $(ooo)

clean:
        rm -rf $(target) $(ooo)

```

> 像上面的代码，找到的第一个目标是target那个，于是会将其当为目标

但是，换一下顺序就会不同

```makefile
target = tstapp
src = $(wildcard *.c)
ooo = $(patsubst %.c, %.o, $(src))


*.o : *.c
        gcc $^ -c


$(target) : a.o
        gcc $^ -o $@

#output:
#       echo  $(ooo)

clean:
        rm -rf $(target) $(ooo)

```

> 在这里就只会编译所有的.c生成.o而不会继续执行下去生成target

**同理，如果把clean或其他的放在前面则使用make命令默认就调用了他们**

---

## 4. 一个目标多命令

> 用tab控制缩进，就行

```makefile
install:
    make -C ./source
    make -C ./mod
    mv ./source/Process_Copy_App ./
```

---

# 可选参数

## -C

格式为：`make -C dir`

例如本项目中的：

```bash
make -C ./source

make clean -C ./source
```

> 第一个指的是：调用当前目录下的source文件夹内的makefile
>
> 第二个指的是：调用当前目录下的source文件夹内的makefile的clean指令

**注意：-C后一定要紧跟目录，不能穿插其他的**

## -f

选择某一文件作为makefile文件来执行

例如：当前目录下有makefile、m1、m2等三个makefile文件，若使用make则会调用makefile，但若想用m1作为makefile文件，则需要：
```makefile
make -f m1
```

# Linux文件名通配符

## 1.单字符匹配元字符`?`

可以用来匹配单个字符，例如：`ab?`

可以用来匹配：`abc`、`abd`等三个字符以ab开头的字符串

## 2.多字符匹配元字符`*`

可以匹配0到多个字符，例如：`a*c`

可以匹配：`ac`、`abc`、`asdfsdfdfc`等

## 3.通配符`%`

与`*`不同的是：`%`用来带入需要匹配的字符串

```makefile
target = tstapp
src = $(wildcard *.c)
ooo = $(patsubst %.c, %.o, $(src))

%.o : %.c
        gcc $^ -c


$(target) : a.o
        gcc $^ -o $@

#output:
#       echo  $(ooo)

clean:
        rm -rf $(target) $(ooo)
```

> 当前文件夹下有a.c与b.c，但是经由上面的makefile并不会都编译，而只会编译a.c
>
> 解释：
>
> 首先找到第一个目标中没有通配符的进行构造，然后发现依赖只有a.o，于是去寻找a.o如何获得
>
> 找到通配符往里面带入，发现可以，于是就编译a.c生成了a.o

### 与`*`的区别

`*`会全部进行匹配，而不是带入

```makefile
target = tstapp
src = $(wildcard *.c)
ooo = $(patsubst %.c, %.o, $(src))

%.o : *.c
        gcc $^ -c


$(target) : a.o
        gcc $^ -o $@

#output:
#       echo  $(ooo)

clean:
        rm -rf $(target) $(ooo)
```

> 比如上面第一步还是找到没有通配符的目标，然后寻找如何获得a.o
>
> 带入得a.o : a.c b.c
>
> 于是执行下面的语句，同时编译了a.c与b.c