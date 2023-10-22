# 基础知识介绍

> 文件分为文本文件和二进制文件:
>
> - 文本文件：存储的是**常规字符串**，由若干文本行组成，通常每行以换行符`\n`结尾
> 	- 常规字符串是指，能被记事本或其他文本编辑器直接正常显示并能被人直接阅读与理解的字符串
> - 二进制文件：对象内容以字节串进行存储，**无法用**记事本或其他普通字处理软件进行编辑，通常也无法被人们直接阅读与理解

---

# 斜杠

在Windows系统上，路径使用反斜杠作为文件夹之间的分隔符

在OS X和Linux系统上，使用正斜杠作为它们的路径分隔符

对于这种情况，我们可以用`os.path.join`方法来解决，只需要传入文件夹名称的字符串，最终就能返回一个正确的路径

```python
import os

a = os.path.join('user', 'bin', 'spam', 'book.txt')
b = os.path.join('user', 'bin', 'spam', 'yyds')

print(a)    # user\bin\spam\book.txt         
print(b)    # user\bin\spam\yyds          
```

---

# os与os.path模块

## 获取信息

### 获取当前工作目录

每个运行在计算机上的程序都有一个当前工作目录（`cwd`），我们可以利用`os.getcwd`函数获取当前路径的字符串，并可以利用`os.chdir`改变它

**注意：这里改变的路径一定是要存在的路径，否则会报错**

```python
import os

print(os.getcwd()) # E:\Desktop\666

os.chdir('C:\\')

print(os.getcwd()) # C:\

os.chdir('C:\\ssr')

print(os.getcwd()) # Error：path not find
```

---

### 查看文件大小和文件夹内容

#### 获取文件大小

可以调用`os.path.getsize(path)`获取路径中的文件字节数

```python
import os

print(os.path.getsize('tst.py')) # ./tst.py文件的大小为1260
```

#### 获取目录下的文件（目录）名

可以调用`os.listdir(path)`

**注意：这是在os下的而不是在os.path下的**

```python
import os

# 获取当前路径下的所有文件、目录名
print(os.listdir('.')) # ['.cph', '.vscode', '1.cpp', '1.exe', 'a', 'build', 'CMakeLists.txt', 'head', 'hello.py', 'Main.java', 'src', 'ssr', 'sss', 't.asm', 't.java', 'test.txt', 'tst.java', 'tst.py']
```

---

## 绝对路径&相对路径

- 绝对路径：总是从根文件夹开始

- 相对路径：相对于程序的当前工作目录

### 相对路径

在普通文件夹之外还有`.`和`..`文件夹，他们并不是真正的文件夹，一般用于相对路径

一个点`.`表示当前文件夹

两个点`..`表示父文件夹

**注意：在相对路径开始处的`.\`是可以选的，例如`.\a.txt`和`a.txt`指的是一个文件**

## 创建新文件夹

程序可以用`os.makedirs`函数创建新文件夹（目录）

**注意：该函数会创建完整的路径**

例如：

```python
import os

os.makedirs('sss/abc')
```

上述代码实际上生成的路径是`./sss/abc`

其中当前目录下并没有sss文件夹，也没有abc文件夹，但是运行时并不会报错，而会生成这俩文件夹

## 处理绝对路径&相对路径

### 相对路径转绝对路径

我们都知道，可以直接通过`.`获取当前的路径，而`os.path.abspath`方法就可以将传入的路径转换（一般都是`.`或`..`开头的相对路径）为绝对路径

```python
import os

print(os.path.abspath('.'))     # E:\Desktop\666                    
print(os.path.abspath('..'))    # E:\Desktop             
```

### 判断绝对路径

调用`os.path.isabs`方法就行

```python
import os

print(os.path.isabs('.'))      # False          
print(os.path.isabs('c:\\'))   # True         
```

### 路径寻找

可以通过方法`os.path.relpath`查找

```python
import os

print(os.path.relpath('e:\\'))              # ..\..         
print(os.path.relpath('e:/', 't.java'))     # ..\..\..   
```

---

### 获取目录、文件名

可以通过`os.path.split`获取一个有目录和文件名组成的元组

也可以通过`dirname`获取目录，通过`basename`获取文件名

```python
import os

path = os.path.abspath('.')
a, b = os.path.dirname(path), os.path.basename(path)
print(os.path.split(path))      # ('E:\\Desktop', '666')         
print(a)                        # E:\Desktop           
print(b)                        # 666            
```

当然如果要获取目录的每个文件夹可以将完整的目录字符串进行分割，传入`os.path.sep`作为分隔符即可

```python
print(a.split(os.path.sep))     # ['E:', 'Desktop']
```

---

## 检查路径的有效性

###   判断是否存在

```python
import os

print(os.path.exists('hello.py'))    # True             
print(os.path.exists('hello.p'))     # False          
```

### 判断是文件还是文件夹

```python
import os

print(os.path.isfile('tst.py'))  # True            
print(os.path.isfile('ssr'))     # False          
print(os.path.isdir('tst.py'))   # False           
print(os.path.isdir('ssr'))      # True           
```

---

# 文件读写

## 过程

1. 调用`open`方法，返回一个`File`对象
2. 调用`File`对象的`read`或`write`方法
3. 调用`File`对象的`close`方法，关闭该文件

**注意：对文件操作完以后，一定要关闭文件对象，这样才能保证修改都被保存到文件中了**

---

## open打开文件

```python
f = open('tst.py')
```

默认情况下，都是以读纯文本方式打开文件的

也就是说，上面的代码与下面的作用一样

```python
f = open('1.cpp', 'r')
```

---

## with关键字

> 有时候，即使写了关闭文件的代码，也无法保证文件一定能被正常关闭，例如文件打开后关闭前，程序崩溃了
>
> 如果我们使用了with关键字，则可以很好的避免整个问题

### 语法

```python
with open(filename, mode, encoding) as fp:
    # 这里写通过文件对象fp读写文件内容的语句
    pass
# 不能再使用文件对象fp了
```

```python
with open('hello.py', 'r') as f:
    print(f.read())

print(f.read())  # 这里直接报错，因为已经没有f了
'''
with open('hello.py', 'r') as f:
    print(f.read())
'''
```

---

## 读取文件

### read获取

`read`方法可以直接获取全部文本保存在字符串中

```python
f = open('1.cpp')

s = f.read()

print(s)
'''
#include <iostream>

using namespace std;

int main()
{
    cout << "hello world" << endl;
}
'''
```

---

### readline读取

> 从文件中读取一行返回

```python
with open('Main.java', 'r') as f:
    print(f.readline())

'''
hi

'''
```

### readlines读取

`readlines`方法可以读取每一行的内容，分开来存进列表

中间用`\n`分隔

```python
f = open('hello.py')

s = f.readlines()

print(s)

# ["f = open('hello.py')\n", '\n', 's = f.readlines()\n', '\n', 'print(s)\n', '\n', '# for x in s:\n', '#     print(x)']
```

## 写入文件

要写入文件的话，文件的打开方式就得是`w`（写入）或者`a`（追加）

如果文件不存在，则会直接创建

### write写入

> 把s的内容写入文件

```python
f = open('tst.txt', 'w')

f.write('abc')
```

```py
with open('Main.java', 'w') as f:
    f.write('''Hi, how are you?
hi, my name is lee
            ''')
```

> 写入结果为：

```
Hi, how are you?
hi, my name is lee
```

---

### writelines写入

> 把字符串列表写入文本文件，不添加换行符
>
> 若要换行，请手动添加`\n`

```python
with open('Main.java', 'w') as f:
    f.writelines(['hi\n', 'no\n'])
```

---

## 光标偏移

`seek(offset[, whence])`

> 光标便宜需要调用seek方法，可以实现光标的移动
>
> 第一个参数是偏移量（可以为负数），第二个参数是基准：
>
> - 0：从头开始
> - 1：从当前位置
> - 2：从结束位置
>
> 默认是0

```py
with open('Main.java', 'r') as f:
    f.seek(1)
    f.seek(1)  # 无效动作，因为刚刚已经偏移到这个位置了
    print(f.read())

'''
i
no

'''
```

```python
with open('Main.java', 'r') as f:
    f.seek(1)
    f.seek(0, 2)
    print(f.read())

# 读了个回车
'''

'''
```

**注意：如果想用当前位置，则打开方式不能是r或r+，需要是rb**

```python
with open('Main.java', 'rb') as f:
    f.seek(1, 1)  # 从当前位置向后移动1个字节
    print(f.read())   # b'i\r\nno\r\n'
```

