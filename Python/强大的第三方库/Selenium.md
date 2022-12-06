# Web自动化测试
***先安利一个大佬的[博客](https://blog.csdn.net/qq_43965708/article/details/120658713?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166529847416782425122973%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=166529847416782425122973&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-120658713-null-null.142^v52^pc_rank_34_2,201^v3^control_2&utm_term=selenium&spm=1018.2226.3001.4187)***
自动化测试可以配合time模块的sleep函数
`sleep(second)`传入停留的时间单位为秒

---
## 打开网页
- `get(url)`
    打开浏览器，但只有一个页面，重复`get`会把之前的**顶掉**
    ```python
    from selenium import webdriver
    b = webdriver.Edge()
    b.get('https://www.4399.com') # 打开4399
    b.quit() # 退出
    ```
- `execute_script(ssr)`
    ssr是字符串类型的指令，`exe...`函数就是用来执行指令的，要打开新标签只需要执行一个新命令即可
    ```python
    from selenium import webdriver
    from time import sleep

    # Edge浏览器
    driver = webdriver.Edge()
    driver.get('https://www.csdn.net/')
    driver.maximize_window()
    sleep(1)
    ssr = 'window.open("https://4399.com")'
    driver.execute_script(ssr)
    ```
---
## 设置窗口大小
1. `set_window_size(x, y)`
    x为水平长度，y为竖直宽度
    如：`driver.set_window_size(1000, 800)`
2. `maxsize_window()`
    浏览器驱动调用该函数实现全屏
    如：`driver.maximize_window()`

---
## 浏览器的前进和后退
- `back()`
    后退函数， `drive.back()`
- `forward()`
    前进函数, `drive.forward()`

---
## 浏览器刷新
使用`refresh()`函数即可,如`driver.refresh()`

---
## 窗口切换
没有特别改变情况下，当前窗口永远是不变的，也就是第一个，于是要操作其他窗口就需要改变当前窗口

首先，通过 `driver.window_handles` 获得所有窗口的句柄（一般按照时间顺序，后打开的窗口在列表后面）。
然后用`switch_to.window(句柄)`来转换当前窗口
```python
# encoding:utf-8
from selenium import webdriver
from selenium.webdriver.common.by import By
from time import sleep
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

# Edge浏览器
driver = webdriver.Edge()
driver.get('https://www.baidu.com')
driver.maximize_window()
into = driver.find_element(By.XPATH, '//*[@id="kw"]')
into.send_keys('蔡徐坤', Keys.ENTER)
js = 'window.open("https://4399.com")'
driver.execute_script(js)
driver.switch_to.window(driver.window_handles[-1])
sleep(2)
print(driver.title)
driver.get_screenshot_as_file(r'e:\桌面\1.jpg')
driver.quit()
```


---
## 定位元素
首先要引入`By`
`from selenium.webdriver.common.by import By`
然后通过`find_element()`查询
```python
如下：
from selenium import webdriver
from selenium.webdriver.common.by import By
from time import sleep

# Edge浏览器
driver = webdriver.Edge()
driver.get('https://www.baidu.com')
driver.maximize_window()
text_label = driver.find_element(By.ID, 'kw')
text_label.send_keys('吃了吗')
```
---
## 常见操作
1. 模拟输入
    `text_label.send_keys(str)`,`str`放输入的内容,`text_label`为输入框对象
2. 清空文本内容
    `text_label.clear()`很简单
3. 判断元素是否可见
    `obj.is_displayed()`
4. 获得标签属性值
    `obj.get_attribute('key')`,`key`里填想要获取的属性，例如`into.get_attribute('value')`
5. 元素的尺寸与文本
    尺寸为`size`, 文本为`text`，调用为`obj.size`
---
## 鼠标操作
1. 单击左键
    ***唯一一个不需要引入新类的操作***
    `button.click()`,`button`为执行对象
2. 单击右键
    首先，需要引入`from selenium.webdriver.common.action_chains import ActionChains`。定位完毕后，`ActionChains(driver).context_click(button).perform()`来执行右键操作
3. 双击
    引入与上面相同,`ActionChains(driver).double_click(button).perform()`
4. 拖动
    ```python
    # 定位要拖动的元素
    source = driver.find_element_by_xpath('xxx')
    # 定位目标元素
    target = driver.find_element_by_xpath('xxx')
    # 执行拖动动作
    ActionChains(driver).drag_and_drop(source, target).perform()
    ```
5. 鼠标悬停
    `ActionChains(driver).move_to_element(o).perform()`
    `o`为悬停的元素
---
## 键盘控制
`send_keys()`的括号中不止能直接输入文本，还可以模拟键盘按键,需要先引入`from selenium.webdriver.common.keys import Keys`
例如：
```python
# encoding:utf-8
from selenium import webdriver
from selenium.webdriver.common.by import By
from time import sleep
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

# Edge浏览器
driver = webdriver.Edge()
driver.get('https://www.baidu.com')
driver.maximize_window()
into = driver.find_element(By.XPATH, '//*[@id="kw"]')
into.send_keys('蔡徐坤')
into.send_keys(Keys.ENTER)
```
同样的括号内的字符串可以拼接
简单的‘ctrl + v’哈哈哈`into.send_keys(Keys.CONTROL, 'v')`
其他的按键一般都可以在键盘上找到，如果是上下左右这样的方向键，则应该是`Keys.ARROW_DOWN`这样的全大写

---
## 调用JavaScript
- 打开新页面
    ```python
    from selenium import webdriver
    from time import sleep

    # Edge浏览器
    driver = webdriver.Edge()
    driver.get('https://www.csdn.net/')
    driver.maximize_window()
    sleep(1)
    ssr = 'window.open("https://4399.com")'
    driver.execute_script(ssr)
    ```
- 滑动滚动条
    - 通过xy坐标滑动，左上角为（0，0）
        ```python
        js = 'window.scrollTo(0, 1000)'
        driver.execute_script(js)
        ```
    - 通过参照标签滑动
        ```python
        taget = driver.find_element(By.XPATH, '//*[@id="skinbody"]/div[10]/div[1]/div[2]/ul/li[1]/a/img')
        driver.execute_script('arguments[0].scrollIntoView();', taget)
        ```
---
## 其他
- `quit()`
    直接关闭所有窗口,`driver.quit()`
- `close()`
    关闭当前窗口,`driver.close()`
- `get_screenshot_as_file()`
    对当前窗口截屏
    `driver.get_screenshot_as_file(r'e:\桌面\1.png')`,括号中填保存路径
- 其他常用方法
    ```python
    # 获取当前页面url
    driver.current_url

    # 获取当前html源码
    driver.page_source

    # 获取当前页面标题
    driver.title

    # 获取浏览器名称(chrome)
    driver.name

    # 对页面进行截图，返回二进制数据
    driver.get_screenshot_as_png()

    # 设置浏览器尺寸
    driver.get_window_size()

    # 获取浏览器尺寸，位置
    driver.get_window_rect()

    # 获取浏览器位置(左上角)
    driver.get_window_position()

    # 设置浏览器尺寸
    driver.set_window_size(width=1000, height=600)

    # 设置浏览器位置(左上角)
    driver.set_window_position(x=500, y=600)

    # 设置浏览器的尺寸，位置
    driver.set_window_rect(x=200, y=400, width=1000, height=600)
    ```
---
## 等待
1. 强制等待
    直接用`sleep(delay)`死等delay的时间，单位为秒
2. 隐式等待
    一次设置全局生效,`implicitly_wait（delay）`



