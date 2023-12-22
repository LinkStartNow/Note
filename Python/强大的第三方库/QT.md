# 介绍

> 底层的QT还是用C++实现的，但是有了第三方库的封装之后，使用Python也能很轻易的调用，实现同样的功能

---

# 学习链接

> 链接：[白月黑羽]([Python Qt 简介 | 白月黑羽 (byhy.net)](https://www.byhy.net/tut/py/gui/qt_01/))

---

# 使用前提

> 我们需要先安装第三方库，以下内容全部以PySide2为例，所以我们安装PySide2

```shell
pip install pyside2
```

> 如果我们安装了许多版本的Python解释器，那么我们就先进入到我们想要安装的版本的Python目录下调用pip即可
>
> 本机的Python10的路径为：
>
> `C:\Users\MAX\AppData\Local\Programs\Python\Python310`
>
> 在该路径下打开命令行输入上面的指令即可

---

 # 库的位置

> 我们已经安装了许多第三方库了，但是那些库我们应该怎么从文件夹中找到他们呢？其实很简单
>
> 我们先来到Python的路径下，以当前的Python10为例：
>
> `C:\Users\MAX\AppData\Local\Programs\Python\Python310`
>
> 然后进入`lib->site-packages`，里面全部都是我们安装的库文件
>
> 我们进入刚刚下载的PySide2文件夹内，能找到`designer.exe`这是我们后面需要用到的非常方便且重要的可视化设计工具

---

# 基础介绍

## 基础结构

> 我们现在来介绍一些常用到的类
>
> 首先先导入他们

```python
from PySide2.QtWidgets import QApplication, QMainWindow, QPushButton,  QPlainTextEdit
```

> 这个`QtWidgets`实际上就是QT的组件类，从中我们导入其他的小类

---

### QApplication

> 这个就是之前C++写QT的时候也用到的一个比较核心的类，就是解决一些启动啊，检测之类的，总之就是照着写就行了，具体可以另外去了解

```python
app = QApplication([])

‘’‘
	具体内部实现功能，展示之类的
’‘’

app.exec_()
```

---

### QMainWindow

> 显示一个主窗口

```python
window = QMainWindow()
window.resize(500, 400)
window.move(300, 310)
window.setWindowTitle('薪资统计')
window.show()
```

> 设置大小啊，位置之类的，这些不用咋管，因为之后用可视化设计的时候直接拉动即可

**注意：如果还有其他的小组件是放在这个主窗口上的，那么需要在他们都放置完毕后，再调用`window.show()`，这样可以一起显示，那些小组件就不用另外使用`show`了**

---

### Code

> 随后我们将他们封装成类，方便调用

```python
from PySide2.QtWidgets import QApplication, QMainWindow, QPushButton,  QPlainTextEdit

class stats:
    def __init__(self) -> None:
        self.window = QMainWindow()
        self.window.resize(500, 400)
        self.window.move(300, 310)
        self.window.setWindowTitle('薪资统计')

        self.textEdit = QPlainTextEdit(self.window)
        self.textEdit.setPlaceholderText("请输入薪资表")
        self.textEdit.move(10,25)
        self.textEdit.resize(300,350)

        self.button = QPushButton('统计', self.window)
        self.button.move(380,80)

app = QApplication([])

ui = stats()
ui.window.show()

app.exec_()
```

---

### 信号与槽

> 其实就是将事件与处理函数进行绑定，具体可以去看之前的qt知识

---

```python
def yyds():
    info = textEdit.toPlainText()
    print(info)

button = QPushButton('统计', window)
button.move(380,80)
button.clicked.connect(yyds)
```

> 将按钮的点击事件与yyds函数绑定

---

## Qt Designer

> 这可是一个可视化设计界面的神奇，我们可以通过这个软件然后拖动就能调整大小啊，布局什么的

---

### 入口

> 我们先进入PySide2的文件夹下找到该程序打开就行啦
>
> `C:\Users\MAX\AppData\Local\Programs\Python\Python310\Lib\site-packages\PySide2`
>
> 找到`designer.exe`打开即可

---

### 使用

#### 开始

> 我们就直接从0开始，选择创建一个widget，这是个完全空的窗口，然后点击创建即可

---

#### 布置

> 具体内部的布置就随需求放置即可
>
> 左边栏有我们将用到的那些常用组件，我们在其中找到然后拖入即可

---

#### 保存

> 在布局完后，我们可以进行保存，获得ui文件，然后通过Python相关的调用即可

---

### Python调用

> 我们先导入，加载该文件，然后直接就可以用了，里面的窗口等都通过加载对象来调用即可

```python
from PySide2.QtWidgets import QApplication
from PySide2.QtUiTools import QUiLoader

class stats:
    def __init__(self) -> None:
        # 导入ui文件
        self.ui = QUiLoader().load('tst.ui')

app = QApplication([])

ssr = stats()
ssr.ui.show()
app.exec_()
```

---

### 关于show

> 我们都知道，设置好父子关系，最终只需要父窗口调用show，这样所有的那些组件也都会显示了，但是当我们用动态ui文件生成加载对象后不知道谁是父窗口了咋办
>
> 不用慌，其实只需要让这个加载对象调用`show`方法即可

```python
from PySide2.QtWidgets import QApplication, QWidget
from PySide2.QtUiTools import QUiLoader

class item(QWidget):
    def __init__(self) -> None:
        super().__init__()
        self.ui = QUiLoader().load('item.ui')
        self.ui.show()
```

---

# 窗口嵌套

> 我们在设计类似QQ的好友列表的时候发现在好友多的时候会自动出现滚动条，让我们可以向下滑动，这就要用到我们的窗口嵌套功能了

---

## 创建新的item

> 我们将这没一条好友信息另外用一个item窗口装着
>
> 我们先创建一个widget窗口
>
> 这个时候我们发现这个窗口自己就有上面的一条栏，但我们的好友信息显示那肯定时没有的呀
>
> 这个无需担心，在单个作为窗口显示的时候确实会这样
>
> 但是如果被当做一个组件放入其他窗口中时，这个栏自动就会消失的，我们照常布局即可

**注意：为了最终显示不被压缩，我们对这个item窗口固定死大小，即设置最大大小和最小大小**

---

## 创建item类

> 我们最重要的还是要在Python中调用它，实现相应的功能
>
> 我们来回顾一下前面调用动态ui文件生成界面的步骤：
>
> 我们需要用一个变量来接收动态ui文件加载完成后的对象
>
> 然后通过这个对象来调用该窗口内部的组件
>
> 因为我们后面将把这个item当做组件放入其他的组件中，所以这个item必须具有QWidget的属性，这里我们只需要通过继承即可实现

```python
from PySide2.QtWidgets import QApplication, QWidget
from PySide2.QtUiTools import QUiLoader

class item(QWidget):
    def __init__(self) -> None:
        super().__init__()
        self.ui = QUiLoader().load('item.ui')
        # self.ui.show()
```

**注意：这里千万不要单独show了，会单独出现一个串口来显示这个item，正确的做法应当是像前面设计的一样，设置parent属性值，然后只需要父窗口show即可**

---

## main.ui设计

> 在百般尝试后，我终于悟了，特意来写写心得
>
> 我们首先在需要滑动的区域放置一个`Scroll Area`
>
> 如果我们需要之后竖直放置item，那么我们放一个`Vertical Spacer`（也就是那个竖着的弹簧）
>
> 然后我们看到右边的对象`scrollArea`，我们将这个对象整个采取垂直布局即可
>
> 垂直布局完，他们的位置会比较乱，我们手动拖拉到满意的位置即可

---

## 主窗口的调用

> 我们前面说了，这个item就由主窗口来放入组件了
>
> 我们前面用了垂直布局，于是会出现一个垂直层，我们只需要将这些item放入这些垂直层中即可
>
> 这个垂直层从设计软件上是不可见的，那个层也没显示出来，也看不到名字，但是这个名字是按标准的排序编号的，我们可以通过这个层是第几个垂直层来推测它的名字
>
> 比如当前只放了这一个垂直层，那么这个垂直层的名字就为`verticalLayout`
>
> 我们调用这个对象的`addWidget`方法即可

**注意：我们要放入的是这个窗口，并不是这个类，于是添加的是ui而不是直接把类放进去**

```python
from item import item
from PySide2.QtUiTools import QUiLoader

class kernel:
    def __init__(self) -> None:
        self.ui = QUiLoader().load('main.ui')

        self.ItemList = [item() for _ in range(10)]
        for it in self.ItemList:
            self.ui.verticalLayout.addWidget(it.ui)

        self.ui.show()
```

---

# 自定义信号

## 背景

> 我们在构造GUI项目的时候，难免也会用到多线程的技术来优化用户的体验，但是在QT中，不允许多个线程操作GUI界面，很容易出bug，于是我们需要借助信号与槽的方式解决这个问题
>
> 这里的信号与槽简直是神器，好多操作的bug都很好的被这个机制解决了

---

## 声明&定义

> 我们需要导入signal和qobject

```python
from PySide2.QtCore import QObject, Signal
```

> 我们需要创建一个自己的信号类来继承qobject类
>
> 然后在内部创建各种静态变量信号供后面使用
>
> 然后再用该类创建一个对象，之后就用这个对象来发出信号和绑定信号与槽了

```python
class MySignals(QObject):
    GetSource = Signal()
    PutItem = Signal(int)
    AddLoadItem = Signal(int, str, str, tuple, str, str, str)
    CacheLoadSuccess = Signal()
    DownLoadComplete = Signal()
    AddItemToSql = Signal(int, str)
    Filtrate = Signal()
    GetFiltrate = Signal(str, str, str, str)
    SpiderWorking = Signal()
    SpiderEnd = Signal()
    CheckSqlEmpty = Signal()
    VisitComplete = Signal()
    NewItemFind = Signal(str, str, tuple, str, str, str)
    ItemUpdate = Signal(Item)

sig = MySignals()
```

> 这里创建signal对象的时候所传入的各个参数是类型对象，像上面的str，int，tuple，Item都是类名
>
> 所绑定的槽函数也要一模一样的传参

---

## 抛出信号

> 我们在各个类对象之间通讯，或者线程之间通讯一般都通过抛出信号，然后处理对应的方法来解决
>
> 我们调用信号对象的emit然后传入对应的参数即可

```python
for it in res:
    # 如果是新图片，则添加到新id
    if it[0] not in t:
        sig.NewItemFind.emit(it[1], it[5], it[3], it[2], it[4], it[0])
```

---

## 绑定信号

> 我们在需要捕获信号的类中要绑定对应的信号和槽
>
> 这样在信号发出的时候就会自动捕获然后调用对应的方法
>
> 一般我们习惯将槽函数和信号名取的一样，这样非常好分辨

### 绑定的声明

```python
sig.NewItemFind.connect(self.NewItemFind)
```

### 槽函数的定义

```python
def NewItemFind(self, url, price, size, info, location, picurl):
        self.AddItem(url, price, size, info, location, picurl)
```

---

