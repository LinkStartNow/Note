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

# 正文开始

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

