# 常用指令第三弹

## 1.who指令

一般搭配`-uH`

`who -uH`

可以显示当前在线用户的信息

---

## 2.ps指令

查看后台进程

一般搭配aux使用

`ps aux`

查看所有进程并显示创建用户

> a：显示所有
>
> u：显示创建用户

## 3.jobs指令

查看后台作业（也就是被挂起的）

**注意：一般某个指令运行过程中按下了ctrl+z会挂起，按下ctrl+c会直接结束，ctrl+d结束输入**

使用的时候直接jobs就行了

`jobs`

## 4.fg指令

用来把后台的作业移动到前台，参数可以是一个或多个进程的PID或命令，或作业号（需要加%）

例如：`fg %1`

**注意：必须是已经存在的编号**

> 把作业号为1的后台进程放到前台运行

## 5.kill指令

向指定进程发送信号

具体信号可以通过-l查看，例如`kill -l`

有两个常用的信号参数：

1. 9号SIGKILL
2. 15号SIGTERM

都是用来结束进程的

但是9号可以结束任意经常，而15号不能结束后台进程

传入参数既可以传入序号，也可以传入名称

例如：`kill -9`和`kill -SIGKILL`是等同的

## 6.env指令

查看环境变量

直接`env`即可

### 配置环境变量

- 配置当前用户：`vim ~/.bashrc`
- 配置系统环境变量：`vim /etc/profile`（需要root权限）

## 7.用户相关

### 创建用户

使用`useradd`方法

不过需要root权限，例如：

```bash
sudo useradd -s /bin/bash -d /home/tst -m tst
```

> 添加一个用户，该用户登录的shell是/bin/bash，放在文件夹/home/tst中，用户名字为tst

### 设置用户密码

也需要root权限

举例：`sudo passwd tst`

为用户tst设置密码

然后按照步骤设置密码，重复密码即可

**注意：刚创建的用户没有sudo的权利，要进入对应的文件为其添加权限**

若要添加权利，则需要到根目录下的`etc/sudoders.h`中

创建一个文件，并写入`用户名 ALL=(ALL)ALL`即可

### 删除用户

需要root权限

例如：`sudo userdel -r 用户名`

> `-r`一般都会加，作用是把用户的主目录一起删除