# 类相关

> 个人的某些理解：貌似在python中属性和方法的界限十分模糊，几乎可以看成是一样的

---
## 构造函数

```python
def __init__(self, name, age): # self必须为第一个参数（也可以不叫self）
        ''' 构造函数， 类的属性由此创建 '''
        self.name = name # 实例属性
        self.age = age # 实例属性
```
有了构造函数后就可以实现用类名初始化该类的变量了
`a = Student('zly', 19) # 类名加括号就是调用类的构造函数`
这些类中的函数里定义的变量(包括`init`函数)都是实例的属性，也就是每个该类的变量各自都有的属性
这种实例属性在类的外部也可以赋予
```python
a.score = 100 # 同样在外部可以创建新的属性，但只是针对a指向的变量有了新的属性，若再用构造函数生成的新对象是没有这个属性的
```
类的实例变量能调用类的函数，当然类也可以调用
```python
a.say_age()
Student.say_age(a) # 在解释器中是这样执行的，达到的是相同的效果
```
类也是个对象。于是，哈哈哈
```python
class Student: #实则生成了一个类对象

    def __init__(self, age, name):
        self.age = age
        self.name = name

a = Student # 将类对象的地址赋值给a
b = a(19, 'zly') # 便可以实现这种操作
print(b.age)
```

详细代码如下：
```python
# 注意下面讲的属性是实例属性
class Student: # 类名一般首字母大写，多个单词组成的话采用驼峰原则如GoodStudent

    def __init__(self, name, age): # self必须为第一个参数（也可以不叫self）
        ''' 构造函数， 类的属性由此创建 '''
        self.name = name # 实例属性
        self.age = age # 实例属性

    def say_age(self): # self为第一个参数
        print('{0}的年龄是{1}'.format(self.name, self.age))


a = Student('zly', 19) # 类名加括号就是调用类的构造函数
a.say_age()
Student.say_age(a) # 在解释器中是这样执行的，达到的是相同的效果
# 纠正一点，创建对象的函数是__new__(),__init__()只是给对象初始化值
a.score = 100 # 同样在外部可以创建新的属性，但只是针对a指向的变量有了新的属性，若再用构造函数生成的新对象是没有这个属性的
print(a.score)
print(dir(a)) # dir访问a的所有属性
print(a.__dict__) # __dict__函数调用a的所有属性组成字典，没有值的属性不会显示

class man:
    pass # 空语句，直接忽略
print(isinstance(a, Student)) # isinstance函数查询对象是不是属于这个类
print(isinstance(a, man))
```

---
## 类属性
所有该类的变量包括这个类对象都共用这一个变量，于是可以用来记录类的属性~~马德这不是废话，不然怎么叫类属性~~
***调用的时候用类名调用，不然容易出bug***

```python
''' 类属性 '''
class Student:

    count = 0 # 类属性

    def __init__(self, name):
        self.name = name
        Student.count += 1

a = Student
b = a('zly')
print('一共创建了{0}个对象'.format(Student.count))
c = a('hhh')
print('一共创建了{0}个对象'.format(Student.count))
```

---

### 调用

> 调用的时候尽量也是用类名去调用类属性，不然会创建新的属性，也不会变动原来的类属性

```python
class yyds:
    add = 'beijing'

a = yyds()
b = yyds()
print(yyds.add)    # beijing
print(a.add)       # beijing    
print(b.add)       # beijing 

yyds.add = 'shanghai'
print(yyds.add)       # shanghai  
print(a.add)          # shanghai 
print(b.add)          # shanghai

a.add = 'hangzhou'
print(yyds.add)      # shanghai   
print(a.add)         # hangzhou   
print(b.add)         # shanghai  
```

> 就算是+=这种看似可以的操作，也不行

```python
class yyds:
    add = 1

a = yyds()
yyds.add += 1
print(a.add)       # 2     
print(yyds.add)    # 2 

a.add += 1
print(a.add)       # 2     
print(yyds.add)    # 1     
```

> 实际上，在改变的时候，已经生成新对象了

```python
class yyds:
    add = 1

a = yyds()
print(id(a.add))       # 2049369440496
print(id(yyds.add))    # 2049369440496

a.add = 233
print(id(a.add))       # 2049369447920      
print(id(yyds.add))    # 2049369440496
```

---
## 类方法&静态方法
```python
class ssr:

    city = 'hrbust'

    # 类方法和静态方法中不能调用实例属性和实例方法,因为没有传入self参数
    @classmethod # 定义类方法一定要加这个
    def print_city(cls):
        print(cls.city)

    @staticmethod # 定义一个静态方法，静态方法：与类属性无关的方法
    def add(a, b):
        print(a + b)

ssr.print_city()
ssr.add(1, 5)
```

---
## 私有属性
python的私有属性比较特殊属性名前面加两个下划线就是了
```python
class ssr:

    def __init__(self) -> None:
        self.__abc = 666
        self.iii = 789
    def get_abc(self):
        return self.__abc

a = ssr()
print(a.iii)
print(a.get_abc())
# print(a.__abc) # 像这样直接调用就会报错
```
稍微详尽点的代码介绍:
```python
class hhh:

    __school = 'hrbust' # 类私有属性

    def __init__(self, name, age):
        self.name = name # 公有属性
        self.__age = age # 私有属性

    def __work(self): # 私有方法
        print('好好工作')
        print('年龄是{0}'.format(self.__age)) # 内部函数可以直接调用私有属性

a = hhh('zly', 19)

print(a.name)
print(a._hhh__age) # 私有属性的访问要在点后加一个下划线和类名然后再跟两个下划线和私有属性名称
# 像a.age这样的访问是不行的
a._hhh__work() # 私有方法的调用也是如此
print(hhh._hhh__school) # 类私有属性的调用
```

---
## @Property装饰器
可以把方法修饰成属性~~也许比较好看吧~~
```python
class hhh:

    @property
    def work(self):
        print('666')

a = hhh()
a.work # 可以像属性一样调用
```
处于既想要保护私有属性，又想要方便交互，于是可以利用修饰器
```python
class Student:

    def __init__(self, name, age):
        self.__age = age
        self.__name = name

    @property
    def age(self):
        return self.__age

    @age.setter
    def age(self, age):
        if 10 < age < 20:
            self.__age = age
        else:
            print('修改失败,请输入10到20之间是数字')


''' 
    一般的做法 ：
        def get_age(self): # 便于外部调用私有属性
            return self.__age

        def set_age(self, age):
            if 10 < age < 20:
                self.__age = age
            else:
                print('修改失败,请输入10到20之间是数字')
    外部调用部分：
        print(a.get_age())
        a.set_age(21)
'''

# 高级玩法，外部看起来好像无事发生
a = Student('zly', 19)
print(a.age)
a.age = 25
```
---
## 方法
python中的方法无法重载但是可以动态添加或修改，跟属性几乎是一样的
```python
class hhh:

    def work(self):
        print('好好工作')

def play(a):
    print('我要玩')

def work2(a):
    print('努力干活')

a = hhh()
a.work()
hhh.work = work2 # 动态修改类的方法
a.work() # 由于是调用类中的方法，于是类中方法改变了对象的方法也变了
hhh.py = play # 动态添加新方法
a.py()
```
---
## Call方法
```python
class hhh:

    def __init__(self) -> None:
        self.aaa = 132

    def __call__(self, a, b):
        print('hello')
        print(self.aaa)
        return a + b

a = hhh()
print(a(5, 6)) # 这个神奇的call居然让a变成了一个'方法'
```
---
## 析构函数
在变量销毁的时候自动调用，程序结束后变量也会自动销毁的
```python
class hhh:

    tot = 0

    def __init__(self):
        hhh.tot += 1
        self.num = hhh.tot

    def __del__(self):
        print('{0}号对象已被销毁'.format(self.num))

a = hhh()
b = hhh()
c = hhh()

del c

print('程序结束')
print(a.num)
```

---

# 继承

> Python里的继承只需要直接在类名后加小括号就行了
>
> 值得注意的是，Python全都是默认public
>
> 子类可以直接调用父类的公有属性与方法

```python
class Father:
    def __init__(self) -> None:
        self.a = 233
    
    def yyds(self):
        print('yyds')

class Son(Father):
    pass

f = Father()
s = Son()
s.yyds()
```

---

## 函数重载

> Python并不支持函数重载，但是函数会覆盖，也就是说，后定义会覆盖之前的定义

```python
''' 后定义的无参构造成功了，最终使用的是无参的构造函数，有参的被覆盖了 '''
class Father:
    def __init__(self, x) -> None:
        self.a = 233
    
    def __init__(self) -> None:
        self.a = 233
    
    def yyds(self):
        print('yyds')

s = Father()
print(s.a)
```

```python
''' 
    后定义的有参构造成功了，最终使用的是有参的构造函数，无参的被覆盖了 
    于是最终报错了
'''
class Father:
    def __init__(self) -> None:
        self.a = 233
    
    def __init__(self, x) -> None:
        self.a = 233
    
    def yyds(self):
        print('yyds')

s = Father()
print(s.a)
'''
    s = Father()
TypeError: Father.__init__() missing 1 required positional argument: 'x'
'''
```

---

## 构造方法

> 在子类不写构造函数的情况下，解释器会自动调用父类的无参构造函数

```python
class Father:
    def __init__(self) -> None:
        self.a = 233
    
    def yyds(self):
        print('yyds')

class Son(Father):
    # def __init__(self) -> None:
    #     # super().__init__()
    #     pass
    pass

s = Son()
print(s.a)   # 233
```

调用的过程大致为：

```python
def __init__(self) -> None:
    super().__init__()
```

> 但如果父类手写了有参的构造函数，父类将不再拥有无参构造函数，也就是这时解释器不会自动调用了

```python
class Father:
    # def __init__(self) -> None:
    #     self.a = 233

    def __init__(self, x) -> None:
        self.a = 233
    
    def yyds(self):
        print('yyds')

class Son(Father):
    # def __init__(self) -> None:
    #     # super().__init__()
    #     pass
    pass

s = Son()
print(s.a)
'''
   s = Son()
TypeError: Father.__init__() missing 1 required positional argument: 'x'
'''
```

> 此时，我们如果想要调用父类的构造函数，首先得写自己的构造函数，然后在内部手动调用父类的构造函数

```python
class Father:
    def __init__(self, x) -> None:
        self.a = 233
    
    # def __init__(self) -> None:
    #     self.a = 233
    
    def yyds(self):
        print('yyds')

class Son(Father):
    def __init__(self) -> None:
        super().__init__(666)  # 传入了参数x = 666，成功调用有参构造函数
        pass
    pass

s = Son()
print(s.a)
```

## 调用构造方法

> 子类调用父类的构造方法有两种方式：
>
> 1. 将父类当成父类来处理
> 2. 将父类当成普通类处理

---

### 将父类当成父类

```python
class Father:
    def __init__(self) -> None:
        self.a = 233

class Son(Father):
    def __init__(self) -> None:
        super().__init__()

s = Son()
print(s.a)
```

---

### 将父类当成普通类

```python
class Father:
    def __init__(self) -> None:
        self.a = 233

class Son(Father):
    def __init__(self) -> None:
        Father.__init__(self)

s = Son()
print(s.a)
```

---

#### 拓展

> 由此可以看出，我们其实可以调用其他类的构造函数来初始化自身

```python
class Father:
    def __init__(self) -> None:
        self.a = 233

class yyds:
    def __init__(self) -> None:
        self.ssr = 567

class Son(Father):
    def __init__(self) -> None:
        yyds.__init__(self)

s = Son()
print(s.ssr)  # 567
print(s.a)    # AttributeError: 'Son' object has no attribute 'a'
```

**锐评：看来不论有没有血缘关系都没用，继承啥的都是浮云，构造方法调用谁的，谁才是真“父亲”**

**注意：判断起来父亲还是Father哈，别搞混了，虽然他啥也没留给你哈哈哈哈**

```python
class Father:
    def __init__(self) -> None:
        self.a = 233

class yyds:
    def __init__(self) -> None:
        self.ssr = 567

class Son(Father):
    def __init__(self) -> None:
        yyds.__init__(self)

s = Son()
print(isinstance(s, Son))       # True     
print(isinstance(s, yyds))      # False        
print(isinstance(s, Father))    # True          
```

