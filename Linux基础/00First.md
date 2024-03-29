# 常用指令

## 1.cat指令

`cat 文件名`

查看文件内容

## 2.cd指令

可以用来切换路径

- `cd 不加参数`，回到~(也就是Home)

- `cd ~`，回到~

- `cd -`，回到上一次的路径，并显示上一次的路径

- `cd/`，回到/（根目录也就是此电脑）

## 3.ls指令

查看目录下文件：

  文件路径写完后也可以加这些参数

  如：`ls /etc -l`

  1. 不加参数：当前目录

  2. 加某个路径，该路径下的文件

  3. -l：查看详细信息

  4. -a：查看隐藏文件（以`.`开头的文件）

  5. -al或-la：包含l和a的特点

  6. -R：以递归的形式查看当前路径下的所有文件，完全打开

  	**注意：这个并没有tree那么方便观察，可以用tree替代**

## 4.echo指令

显示字符串

 最简单的使用：`echo "hello world"`

 同样，也可以查看变量内容：

 `echo $SHELL`：查看环境变量SHELL的值，Linux中环境变量要用$，环境变量其实就是全局变量

 **echo还可以输出字符串到文件中**

 `echo "haha" > a`，这种写入是覆盖的，会把整个文件清空然后从头写入

 重定向符号（>）后跟着文件名

## 5.touch指令

创建文件

 `touch a`：创建文件a

 若文件不存在则创建文件，存在则更新创建时间，**不清空内容**

## 6.pwd指令

查看当前路径

## 7.which指令

查看命令路径

 如：`which ls`

## 8.mkdir指令

创建文件夹

 如：`mkdir test`：创建文件夹test在当前路径中

- 也可以创建带层级关系的

  	如:`mkdir x/y/z -p`

  - 也可以创建多个文件夹：

  	`mkdir aa bb cc`

## 9.rmdir指令

删除非空文件夹

## 10.rm指令

因为`rmdir`只能删除非空文件夹，所以不常用

一般都用`rm`

-r:递归删除

-f:不询问















