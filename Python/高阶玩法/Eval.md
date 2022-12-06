# eval函数
`eval(expression, globals=None, locals=None)`
执行expression中的语句然后返回结果
experssion:表达式，是个字符串
globals:定义后作用域会被限定
locals:同上
***定义后两个参数时一定要用字典不然会出错***
无后两个参数时表达式中的变量会从全局变量中找，限定变量范围后则会从给出的范围中找。当两个参数的字典中都存在需要的变量时则 ***locals优先***

---
## expression示例
```python
a=10;
print(eval("a+1"))
```
打印出11

---
## globals示例
```python
a=10;
g={'a':4}
print(eval("a+1",g))
```
结果为5，因为这个a取的是g中的a

---
## locals示例
```python
a=10
b=20
c=30
g={'a':6,'b':8}
t={'b':100,'c':10}
print(eval('a+b+c',g,t))
```
结果会输出116，b同时存在于g和t中，由于t是locals所以比g优先于是b取的是t中的