# Print的一丢丢知识

***详情还是可以直接看[大佬的博客](https://blog.csdn.net/sinat_28576553/article/details/81154912?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166454023016782388065745%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166454023016782388065745&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-81154912-null-null.142^v51^pc_rank_34_2,201^v3^control_2&utm_term=python%E7%9A%84print&spm=1018.2226.3001.4187)***
`print(*objects, sep=' ', end='\n', file=sys.stdout)`
这是参数列表，可见print是不限制输出的类型的，不管是啥都能输出
```python
a, b, c = 1, 'abc', 'd'
print(a, b, c, 'hello', '!')
```
当然通过参数还能看出，`sep`和`end`都是可以设置的，也就是说由什么补充间隔和由什么结尾是可以自己设置的。这也解释了为什么每次输出默认都由空格隔开，且会自动回车换行
```python
a, b, c = 1, 'abc', 'd'
print(a, b, c, 'hello', '!', sep='.', end='hello world!')
```

---
## 格式化输出
可以类比c语言的%，前面先是一个字符串，后面紧跟%和一个元组
这个字符串称为 ***格式控制符***，后面的元组是 ***转换说明符***，最终会把元组中的元素按顺序带入前面的占位符
1. %字符
    ```python
    a = 'ac'
    b = 'yes'
    print('我%s了,%s'%(a, b))
    ```
2. 最小子段宽度和精度
    在 ***格式控制符*** 中的%后用来设置输出的样式
    如`%x.yf`说明子段宽度为x，小数点后保留y位
    `print('%6.3f'%3.14)`
    其中的x和y也可以先用`*`占位，最后同样从元组中读出他俩的值，当然x和y也可以不填使用默认值
    `print('%*.*f'%(10, 10, 3.14))`
3. 转换标志
   不知道为啥这些转换标志好像 ***不能叠加使用***
    1. `-`表示左对齐（默认右对齐）
        也就是说有空格什么的填充字符是填充在右侧的
        `print('%-*.*f'%(10, 10, 3.14))`
    2. `+`表示数值前加上正负号（也就正数需要，毕竟负数本来就带着负号）
        `print('%+*.*f'%(10, 10, -3.14))`
    3. `0`表示位数不够用`'0'`填充
        `print('%0*.*f'%(10, 1, 3.14))`
