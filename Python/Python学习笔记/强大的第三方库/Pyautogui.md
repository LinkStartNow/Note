# 又一个神奇的自动化第三方库
老规矩，[博客安利](https://blog.csdn.net/qingfengxd1/article/details/108270159)

左上角为（0，0）点向下为y正方向，向右为x正方向

---
## 常规操作
```python
import pyautogui
 
pyautogui.PAUSE = 1 # 调用在执行动作后暂停的秒数，只能在执行一些pyautogui动作后才能使用，建议用time.sleep
pyautogui.FAILSAFE = True # 启用自动防故障功能，左上角的坐标为（0，0），将鼠标移到屏幕的左上角，来抛出failSafeException异常
 
# 判断(x,y)是否在屏幕上
x, y = 122, 244
pyautogui.onScreen(x, y) # 结果为true
 
width, height = pyautogui.size() # 屏幕的宽度和高度
print(width, height)
```

---
## 鼠标操作
1. `position()`
    获取当前鼠标位置
    ```python
    x, y = pyautogui.position()
    print(x, y)
    ```
1. 移动
    - `moveTo(x, y, duration)`
        把鼠标移动到(x, y)用时duration
        `pyautogui.moveTo(100, 100, 0.25)`
    - `moveRel(x, y, duration)`
        鼠标从当前位置向右移动x向下移动y，用时duration。
        `pyautogui.moveRel(1000, 500, 1)`
2. 拖拽
    - `dragTo(x, y, duration)`
        按住左键把鼠标拖动到(x, y)
    - `dragRel(x, y, duration)`
        同理可得，相对于当前位置拖拽
3. 点击
    - `click(x, y, num_of_clicks, secs_between_click, button)`
        移动到x，y执行操作，num...是点击次数,secs...是点击间隔时间,button是按键，可以选择:'left', 'right', 'middle'
        `click(x + 1000, y + 100, duration=1, clicks=2)`
        ***不填xy则原地点击***
    - 同样还有`doubleClick()`和`tripleClick()`分别是双击和三击，与`click()`的区别就是没有clicks参数，也就是不能设置点击次数了
4. 鼠标的按下与松开
    - `mouseUp(x, y, button, duration)`
        移动到xy再松开事件，其他的参数请举一反三
    - `mouseDown()`
        同上一个，这个是按下函数
5. `scroll(nums, x, y)`
    移动到xy滚动nums圈，正数则向上，负数向下
---
## 键盘操作
1. `typewrite(str, interval)`
    键盘上敲击str字符串 ***(只能是键盘上的键，由于键盘上没有中文所以中文无法敲击)***, interval是敲击单个字符间隔
2. `press(keys, times, intervals)`
    按下并松开key键(key可以是个字符串代表一个按键，或者一个字符串列表，代表多个按键),times是敲击次数，后一个是敲击间隔,列表中的键同时敲击
3. `keyDown(key)`
    按下key键不松开
4. `keyUp(key)`
    松开key键
5. `hotkey()`
    可以传入热键组合如
    `hotkey('ctrl', 'v')`执行粘贴
---
## 弹窗
1. `alert(text, title, button)`
    弹出个标题为title的弹窗，窗内文本为text，点击按钮显示的文字是button，返回值也是button的值
    ```python
    a = pyautogui.alert(text='要开始吗???', title='哈哈哈', button='yes')
    print(a)
    ```
2. `confirm()`
    其他参数与alert一样，button变成了buttons，可以传入一个序列，range也可以
    ```python
    # a = pyautogui.confirm(text='要开始吗???', title='哈哈哈', buttons=['ok', 'no'])
    a = pyautogui.confirm(text='要开始吗???', title='哈哈哈', buttons=range(10))
    print(a)
    ```
3. `prompt(text, title, default)`
    生成一个输入框，带有ok和cancel选项，默认值为default
4. `password()`
    参数和prompt几乎一样，可以添加mask参数（默认为'*'）
---
## 图像操作
1. `screenshot(path, region)`
    path是截屏保存的路径，region传入一个4元组，左上角xy值和宽度高度,不填默认是全屏
    `screenshot(r'd:\1.png', region=(0, 0, 100, 200))`
2. `locateCenterOnScreen(img, confidence)`
    ***切记：图片路径不能有中文（包括图片的名字）***
    ***神技***，获取屏幕上与img图片匹配的位置坐标的中心坐标，confidence是容忍度，默认为1，可能会匹配失败，所以可以适当调低增加匹配的可能性
    **建议直接qq截图生成png**
    ```python
    from pyautogui import *
    from pyscreeze import screenshot
    hotkey('win', 'd')
    sleep(1)
    a = locateCenterOnScreen(r'f:\1.png', confidence=0.8)
    print(a)
    click(a, duration=1, clicks=2)
    ```
