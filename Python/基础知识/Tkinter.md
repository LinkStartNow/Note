# 参考链接

[参考]([【精选】Python tkinter(GUI编程)模块最完整教程（上）-CSDN博客](https://blog.csdn.net/qq_48979387/article/details/125706562))

# 坐标系

> 计算机中的坐标系一般都是左上角为(0, 0)
>
> 然后x轴向右变大，y轴向下变大
>
> 如(50, 30)指的是离左边50像素，离上面30像素

# 颜色

> 在tkinter中的颜色只有两种方式：
>
> 1. 颜色名称，例如"green"， ”brown“
> 2. 颜色的16进制形式，比如"#00ffff"
>
> **注意：虽然不支持rgb模式，但是可以通过16进制表示rgb**

# 导入库

> 一般情况下，我们在安装Python的时候就已经装了tkinter库了，我们直接导入就行了

```python
import tkinter
```

> 一般我们为了方便会用tk表示tkinter

```python
import tkinter as tk
```

> 导入的时候直接用`*`就行，不会有冲突

```python
from tkinter import *
```

---

# 创建根窗口

> 根窗口是最主要的一个窗口，最好只有一个
>
> 我们可以用Tk方法创建，在窗口中，我们可以添加各种各样打的组件（widget），窗口可以有子窗口，父窗口关闭子窗口也会关闭，子窗口关闭父窗口并不会

```python
import tkinter as tk

root = tk.Tk()
tk.mainloop()
```

> 如果程序结束了那么主窗口也就消失了，于是我们需要用一个死循环卡着，可以用while 1等操作，但是一般还是用mainloop

## 相关方法

### 设置标题

`title(string=None)`

> 我们可以用title方法设置标题

```python
root.title('测试')
```

---

### 设置窗口大小

`geometry(newGeometry=None)`

>  设置窗口尺寸大小
>
> 一般都是传入字符串，如str = 'widthxheight+x+y'
>
> '700x500+20+50'，中间是英文字符x，表示宽700像素，高500像素，离左边20像素，离上面50像素

可以只写大小

```python
root.geometry('700x500')
```

也可以只写位置

```python
root.geometry('+400+50')
```

但是要填的话不能只填宽或只填高如`+400`或`500`

---

### 固定窗口大小

`resizable(width=None, height=None)`

> 我们可以设定窗口大小是否能改，第一个限定宽度，第二个限定高度
>
> 这里的能否改变是当程序运行的时候，能不能人为拉动窗口，而不是下面能不能通过程序修改大小

```python
root.resizable(True, False)  # 可以拉宽，不能拉长
```

### 设置背景颜色

> 我们可以通过config方法改变窗口的属性

```python
root.config(bg='blue')
```

---

# 组件

## 共同的参数

> 这里就介绍一些常用的

### master

> 组件的父容器，一般必选

### name

> 组件在Tcl/Tk内部的名称

### 颜色

> bg：改变组件的背景
>
> fg：改变组件的前景色，一般是文本的颜色

### 尺寸

> width：宽度
>
> height：高度
>
> 一般单位为像素，但在文本输入类型的组件中，单位为字符数量

---

## 共同的方法

> 同样介绍一些常用到的

### after

`after(ms, func=None)`

> 在等待ms毫秒后执行一次func方法

**注意：只会执行一次哦**

### 绑定事件相关

`bind(sequence=None, func=None)`

> 绑定事件，检测到事件后调用func，事件用sequence表示

`unbind(sequence)`

> 解除事件绑定

### 获取值

`cget(key)`

> 可以通过关键字获取对应的值

```python
print(root.cget('bg'))  # pink
```

### 设置属性值

`config(**kw)`

> 可以通过传入键值对的方式修改对应的属性值

```python
root.config(bg='pink')  # 将背景改成粉色
```

# 简单组件

## Label

> 可以用来显示文字或图片，用户不可修改

### 属性

#### text

> 显示的文本

#### font

> 显示的字体

#### textvariable

> 绑定的文本变量

#### 间距

> padx：标签内容与左右的间距
>
> pady：标签内容与上下的间距

#### anchor

> 文本靠标签哪个方向显示
>
> 可以是n,s,w,e,ne,nw,sw,se,center，即北、南、西、东、东北、西北、西南、东南、中间，默认靠中间显示

