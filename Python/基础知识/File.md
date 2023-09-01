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

# 当前工作目录

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

# 绝对路径&相对路径

绝对路径：总是从根文件夹开始

相对路径：相对于程序的当前工作目录

## 相对路径

在普通文件夹之外还有`.`和`..`文件夹，他们并不是真正的文件夹，一般用于相对路径

一个点`.`表示当前文件夹

两个点`..`表示父文件夹

**注意：在相对路径开始处的`.\`是可以选的，例如`.\a.txt`和`a.txt`指的是一个文件**

# 创建新文件夹

程序可以用`os.makedirs`函数创建新文件夹（目录）

**注意：该函数会创建完整的路径**

例如：

```python
import os

os.makedirs('sss/abc')
```

上述代码实际上生成的路径是`./sss/abc`

其中当前目录下并没有sss文件夹，也没有abc文件夹，但是运行时并不会报错，而会生成这俩文件夹

# 处理绝对路径&相对路径

## 相对路径转绝对路径

我们都知道，可以直接通过`.`获取当前的路径，而`os.path.abspath`方法就可以将传入的路径转换（一般都是`.`或`..`开头的相对路径）为绝对路径

```python
import os

print(os.path.abspath('.'))     # E:\Desktop\666                    
print(os.path.abspath('..'))    # E:\Desktop             
```

## 判断绝对路径

调用`os.path.isabs`方法就行

```python
import os

print(os.path.isabs('.'))      # False          
print(os.path.isabs('c:\\'))   # True         
```

## 路径寻找

可以通过方法`os.path.relpath`查找

```python
import os

print(os.path.relpath('e:\\'))              # ..\..         
print(os.path.relpath('e:/', 't.java'))     # ..\..\..   
```

## 获取目录、文件名

可以通过`os.path.split`获取一个又目录和文件名组成的元组

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

# 查看文件大小和文件夹内容

## 获取文件大小

可以调用`os.path.getsize(path)`获取路径中的文件字节数

```python
import os

print(os.path.getsize('tst.py')) # ./tst.py文件的大小为1260
```

## 获取目录下的文件（目录）名

可以调用`os.listdir(path)`

**注意：这是在os下的而不是在os.path下的**

```python
import os

# 获取当前路径下的所有文件、目录名
print(os.listdir('.')) # ['.cph', '.vscode', '1.cpp', '1.exe', 'a', 'build', 'CMakeLists.txt', 'head', 'hello.py', 'Main.java', 'src', 'ssr', 'sss', 't.asm', 't.java', 'test.txt', 'tst.java', 'tst.py']
```

# 检查路径的有效性

##   判断是否存在

```python
import os

print(os.path.exists('hello.py'))    # True             
print(os.path.exists('hello.p'))     # False          
```

## 判断是文件还是文件夹

```python
import os

print(os.path.isfile('tst.py'))  # True            
print(os.path.isfile('ssr'))     # False          
print(os.path.isdir('tst.py'))   # False           
print(os.path.isdir('ssr'))      # True           
```

# 文件读写

## 过程

1. 调用`open`方法，返回一个`File`对象
2. 调用`File`对象的`read`或`write`方法
3. 调用`File`对象的`close`方法，关闭该文件

## open打开文件

```python
f = open('tst.py')
```

默认情况下，都是以读纯文本方式打开文件的

也就是说，上面的代码与下面的作用一样

```python
f = open('1.cpp', 'r')
```

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

### readlines读取

`readlines`方法可以读取每一行的内容，分开来存进列表

中间用`\n`分隔

```python
f = open('1.cpp')

s = f.readlines()

print(s)
# ['#include <iostream>\n', '\n', 'using namespace std;\n', '\n', 'int main()\n', '{\n', '    cout << "hello world" << endl;\n', '}']
```

## 写入文件

要写入文件的话，文件的打开方式就得是`w`（写入）或者`a`（追加）

如果文件不存在，则会直接创建

```python
f = open('tst.txt', 'w')

f.write('abc')
```

