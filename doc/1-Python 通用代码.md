[TOC]

## Selenium
- 淘宝 chromedriver 下载地址：https://registry.npmmirror.com/binary.html?path=chromedriver/
- 本文档使用 selenium 版本：3.141.0（很老的版本，2018年），最新版有很多用法上的变化，我个人还是习惯用老版本

### 我常用的 selenium 初始化类
```python
import random
import time

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait


class ChromeDriver:
    driver: webdriver.Chrome = None
    chrome_options: webdriver.ChromeOptions = None

    def __init__(
            self,
            chromedriver_path='chromedriver',
            headless=False,
            no_sandbox=True,
            user_data_dir='',
            proxy='',
            ignore_image=False,
            remote_debug=False,
    ):
        """
        :param headless: 无头模式
        :param no_sandbox: 关闭沙盒模式
        :param chromedriver_path: chromedriver 可执行文件路径
        :param user_data_dir: 指定用户数据目录
        :param proxy: 使用代理
        :param ignore_image: 忽略图片
        :param remote_debug: 使用远程 debug
        """
        self.chromedriver_path = chromedriver_path
        self.headless = headless
        self.no_sandbox = no_sandbox
        self.remote_debug = remote_debug
        self.user_data_dir = user_data_dir
        self.proxy = proxy
        self.ignore_image = ignore_image

        self.init_chrome_options()
        self.init_driver()

    @classmethod
    def sleep(cls, seconds: int = 1):
        time.sleep(seconds)

    @classmethod
    def random_sleep(cls, min_seconds=1.5, max_seconds=3):
        time.sleep(random.random() * (max_seconds - min_seconds) + min_seconds)

    def init_chrome_options(self):
        chrome_options = webdriver.ChromeOptions()

        if self.headless:
            chrome_options.add_argument('--headless')  # 设置成无界面

        if self.no_sandbox:
            chrome_options.add_argument('--no-sandbox')  # 禁用沙盒 解决linux下root用户无法运行的问题

        # options.add_argument('user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.198 Safari/537.36')

        if self.user_data_dir:
            chrome_options.add_argument(f'--user-data-dir={self.user_data_dir}')

        if self.proxy:
            chrome_options.add_argument(f'--proxy-server={self.proxy}')

        if self.ignore_image:  # 设置成不加载图片
            prefs = {'profile.managed_default_content_settings.images': 2}
            chrome_options.add_experimental_option('prefs', prefs)

        # 直接远程调式浏览器，和 excludeSwitches 互斥
        if self.remote_debug:
            # mac 命令行启动 chrome： /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
            # 注意要完全退出 chrome 再执行上面的命令启动 chrome，命令启动之后要能访问 http://127.0.0.1:9222/json
            chrome_options.add_experimental_option('debuggerAddress', '127.0.0.1:9222')
        else:
            # 去掉'Chrome正受到自动测试软件的控制'以及防止常见的 webdriver 检测
            chrome_options.add_experimental_option('excludeSwitches', ['enable-automation'])
            chrome_options.add_experimental_option('useAutomationExtension', False)
            chrome_options.add_argument('--disable-blink-features=AutomationControlled')

        # 忽略证书错误
        chrome_options.add_argument('--ignore-certificate-errors')
        chrome_options.add_argument('--ignore-ssl-errors')

        # 有时候需要安装插件
        # chrome_options.add_extension('xxx.crx')

        self.chrome_options = chrome_options

    def init_driver(self) -> webdriver.Chrome:
        self.driver = webdriver.Chrome(
            executable_path=self.chromedriver_path,
            # chrome_options=self.chrome_options
            options=self.chrome_options,
        )

        # 去除 navigator.webdriver 的 js
        """
        script = 'Object.defineProperty(navigator, 'webdriver', {get: () => undefined})'
        self.driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {'source': script})
        """

        # 防止被网站检测为 selenium 的 js，这个文件是 puppeteer 中用于抹去自动化程序特征的，已经包含上面 对 navigator 的 webdriver 处理
        with open('stealth.min.js') as f: # https://github.com/ChouBaoDxs/python-summary/blob/master/doc/assets/js/stealth.min.js
            self.driver.execute_cdp_cmd('Page.addScriptToEvaluateOnNewDocument', {'source': f.read()})

        self.driver.implicitly_wait(20)  # 设置隐式等待时间为 20 秒
        return self.driver


if __name__ == '__main__':
    driver = ChromeDriver().driver
    driver.get('https://www.baidu.com/')

    logo_ele = driver.find_element_by_id('s_lg_img_new')
    logo_url = logo_ele.get_attribute('src')
    print('logo_url：', logo_url)

    input_ele = driver.find_element_by_id('kw')
    button_ele = driver.find_element_by_id('su')
    input_ele.send_keys('python')
    input_ele.send_keys(Keys.ENTER)
    # button_ele.click()
    
    # 显示等待
    wait = WebDriverWait(driver, 10)
    wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#content_left')))

    result_div_list = driver.find_elements_by_css_selector('.result')
    print('result：')
    for index, result_div in enumerate(result_div_list):
        print('-' * 20)
        print(index, result_div.text)

    time.sleep(100)
```

### 常用操作
- 执行 js：
    ```python
    # 滚动到页面底部
    driver.execute_script('document.documentElement.scrollTop = document.documentElement.scrollHeight') 
    # y 轴向下滚动 1000 个像素
    driver.execute_script('window.scrollBy(0, 1000)')
    ```
- 屏幕截图：
  ```python
    driver.save_screenshot('screenshot.png')
    driver.get_screenshot_as_file('screenshot.png')
    driver.get_screenshot_as_png()
    driver.get_screenshot_as_base64()
  ```
- 执行一连串动作——ActionChains（动作链），不是很常用，双击、滑动、拖拽时会用到

### 元素常用操作
官方文档： https://www.selenium.dev/selenium/docs/api/py/webdriver_remote/selenium.webdriver.remote.webelement.html

| 方法名                            | 描述                                  |
| -------------------------------- | ------------------------------------ |
| `clear`                          | 清空文本框或文本域中的内容                |
| `click`                          | 点击元素                              |
| `get_attribute`                  | 获取元素的属性值                        |
| `is_displayed`                   | 判断元素对于用户是否可见                 |
| `is_enabled`                     | 判断元素是否处于可用状态                 |
| `is_selected`                    | 判断元素（单选框和复选框）是否被选中       |
| `send_keys`                      | 模拟输入文本                           |
| `submit`                         | 提交表单                              |
| `value_of_css_property`          | 获取指定的CSS属性值                     |
| `find_element` / `find_elements` | 获取单个子元素 / 获取一系列子元素         |
| `screenshot`                     | 为元素生成快照                         |


### 显示等待
- `WebDriverWait(driver, timeout, poll_frequency=0.5, ignored_exceptions=None)`
  - driver：浏览器驱动对象
  - timeout：超时时间，单位秒
  - poll_frequency：检测的间隔步长，默认为 0.5s
  - ignored_exceptions：超时后的抛出的异常信息，默认抛出 NoSuchElementExeception 异常
- 等待方法除了 until，还有 until_not。
- `def until(self, method, message='')`
  - 源码里 method 在死循环里调用：`value = method(self._driver)`
  - method 参数可以是自己写的其他函数，不一定是 expected_conditions 模块下的。比如`wait.until(lambda diver:driver.find_element_by_css_selector('#content_left'))`

**常用条件**
官方文档： https://www.selenium.dev/selenium/docs/api/py/webdriver_support/selenium.webdriver.support.expected_conditions.html

| 等待条件                                   | 具体含义                         |
| ---------------------------------------- | ------------------------------- |
| `title_is / title_contains`              | 标题是指定的内容 / 标题包含指定的内容 |
| `visibility_of`                          | 元素可见                         |
| `presence_of_element_located`            | 定位的元素加载完成                 |
| `visibility_of_element_located`          | 定位的元素变得可见                 |
| `invisibility_of_element_located`        | 定位的元素变得不可见                |
| `presence_of_all_elements_located`       | 定位的所有元素加载完成              |
| `text_to_be_present_in_element`          | 元素包含指定的内容                 |
| `text_to_be_present_in_element_value`    | 元素的`value`属性包含指定的内容     |
| `frame_to_be_available_and_switch_to_it` | 载入并切换到指定的内部窗口           |
| `element_to_be_clickable`                | 元素可点击                        |
| `element_to_be_selected`                 | 元素被选中                        |
| `element_located_to_be_selected`         | 定位的元素被选中                   |
| `alert_is_present`                       | 出现 Alert 弹窗                   |


## 读写 excel
读 excel 一般用 xlrd 或 openpyxl，写 excel 一般用 xlwt 或 openpyxl。xlwt 只能生成 xls 格式的 excel（文件保存为 xlsx 后缀并不会报错），xls 行数有 65536 的限制，超出时需要改用 openpyxl。

**xlwt 写入 xls 例子**
```python
from io import BytesIO
import typing

import xlwt

# 表头加粗
default_header_style = xlwt.XFStyle()
default_header_style.font.bold = True

# 表头居中
align = xlwt.Alignment()
align.vert = xlwt.Alignment.VERT_CENTER
align.horz = xlwt.Alignment.HORZ_CENTER
default_header_style.alignment = align

# 自动换行样式
default_wrap_style = xlwt.XFStyle()
default_wrap_style.alignment.wrap = 1


def generate_excel_by_xlwt(
        excel_data: typing.Iterable[typing.Iterable],
        excel_header: typing.Iterable = None,
        save_path: str = '',
        sheet_name: str = '',
        header_style: xlwt.XFStyle = None,
) -> typing.Optional[BytesIO]:
    """
    :param excel_data: 表格数据
    :param excel_header: 表头
    :param save_path: 保存路径，不传的话函数返回 BytesIO，可以用于返回 http 响应
    :param sheet_name: sheet 名称
    :param header_style: 表头样式
    :return: 提供 save_path 时返回 None，否则返回 BytesIO
    """
    sheet_name = sheet_name or '1'
    header_style = header_style or default_header_style

    work_book = xlwt.Workbook(encoding='utf-8')
    work_sheet: xlwt.Worksheet = work_book.add_sheet(sheet_name)

    offset = 0
    # 处理表头
    if excel_header:
        for index, title in enumerate(excel_header):
            work_sheet.write(0, index, title, header_style)
        offset = 1

    # 处理内容
    for row_num, row_data in enumerate(excel_data):
        for index, cell_data in enumerate(row_data):
            work_sheet.write(row_num + offset, index, cell_data, default_wrap_style)

    if not save_path:
        output = BytesIO()
        work_book.save(output)
        output.seek(0)
        return output
    else:
        work_book.save(save_path)


if __name__ == '__main__':
    header = ['id', 'name']
    data = [
        ['1', '张三'],
        ['2', '李四'],
        ['3', '王五'],
        ['4', '赵六'],
    ]

    generate_excel_by_xlwt(data, header, 'excel_xlwt.xls')
```

**xlrd 读取 xls 例子**
```python
import xlrd

work_book: xlrd.Book = xlrd.open_workbook('excel_xlwt.xls')
work_sheet: xlrd.sheet.Sheet = work_book.sheet_by_index(0)
# work_sheet = work_book.sheet_by_name('sheet_name')

# 方式一
for row in range(work_sheet.nrows):
    for col in range(work_sheet.ncols):
        cell: xlrd.sheet.Cell = work_sheet.cell(row, col)
        value = cell.value
        # value  = work_sheet.cell_value(row, col)
        print(value, end='\t')
    print()

# 方式二
for row in work_sheet.get_rows():
    for cell in row:
        print(cell.value, end='\t')
    print()

# 方式三
for row in range(work_sheet.nrows):
    row_values = work_sheet.row_values(row)
    print(row_values)
```

**openpyxl 写入 xlsx 例子**
```python
from io import BytesIO
import typing

import openpyxl
from openpyxl.worksheet.worksheet import Worksheet


def generate_excel_by_openpyxl(
        excel_data: typing.Iterable[typing.Iterable],
        excel_header: typing.Iterable = None,
        save_path: str = '',
        sheet_name: str = '',
) -> typing.Optional[BytesIO]:
    """
    :param excel_data: 表格数据
    :param excel_header: 表头
    :param save_path: 保存路径，不传的话函数返回 BytesIO，可以用于返回 http 响应
    :param sheet_name: sheet 名称
    :return: 提供 save_path 时返回 None，否则返回 BytesIO
    """
    sheet_name = sheet_name or '1'
    wb = openpyxl.Workbook()

    ws: Worksheet = wb.active
    ws.title = sheet_name

    #  openpyxl 的行号和列号从 1 开始

    offset = 0
    # 处理表头
    if excel_header:
        for index, title in enumerate(excel_header):
            # 第一种写入方式：
            # one_cell = ws.cell(1, index)
            # one_cell.value = data

            # 第二种写入方式：
            ws.cell(1, index + 1, value=title)
        offset = 1

    for row_num, row_data in enumerate(excel_data):
        for col_num, cell_data in enumerate(row_data):
            ws.cell(1 + row_num + offset, col_num + 1, value=cell_data)

    if not save_path:
        output = BytesIO()
        wb.save(output)
        output.seek(0)
        return output
    else:
        wb.save(save_path)


if __name__ == '__main__':
    header = ['id', 'name']
    data = [
        ['1', '张三'],
        ['2', '李四'],
        ['3', '王五'],
        ['4', '赵六'],
    ]

    generate_excel_by_openpyxl(data, header, 'excel_openpyxl.xlsx')
```

**openpyxl 读取 xlsx 例子**
```python
import openpyxl
from openpyxl.worksheet.worksheet import Worksheet

wb = openpyxl.load_workbook('excel_openpyxl.xlsx')
ws: Worksheet = wb.active
print(ws.dimensions)  # A1:B5
print(ws.max_row, ws.max_column)  # 5 2

# 获取指定单元格的值
print(ws.cell(2, 2).value)  # 张三
print(ws['B2'].value)  # 张三
print(ws['A1:B2'])  # ((<Cell '1'.A1>, <Cell '1'.B1>), (<Cell '1'.A2>, <Cell '1'.B2>))

# openpyxl 的行号和列号从 1 开始
for row in range(1, ws.max_row + 1):
    for col in range(1, ws.max_column + 1):
        cell = ws.cell(row, col)
        print(cell.value, end='\t')
    print()
```

## 读写 csv
**通过列表写入**
```python
import csv

header = ['id', 'name']
data = [
    ['1', '张三'],
    ['2', '李四'],
    ['3', '王五'],
    ['4', '赵六'],
]

with open('csv_list_demo.csv', 'w') as f:
    writer = csv.writer(f)
    writer.writerow(header)
    writer.writerows(data)
```

**通过字典写入**
```python
import csv

header = ['id', 'name']
data = [
    {'id': '1', 'name': '张三'},
    {'id': '2', 'name': '李四'},
    {'id': '3', 'name': '王五'},
    {'id': '4', 'name': '赵六'},
]

with open('csv_dict_demo.csv', 'w') as f:
    writer = csv.DictWriter(f, fieldnames=header)
    writer.writeheader()
    writer.writerows(data)
```

**通过列表读取**
```python
import csv

with open('csv_list_demo.csv', 'r') as f:
    reader = csv.reader(f)
    header = next(reader)
    print(header)
    for index, row in enumerate(reader):
        print(index, row)
    
    """
    ['id', 'name']
    0 ['1', '张三']
    1 ['2', '李四']
    2 ['3', '王五']
    3 ['4', '赵六']
    """
```

**通过字典读取**
```python
import csv

with open('csv_dict_demo.csv', 'r')as f:
    reader = csv.DictReader(f)
    for index, row in enumerate(reader):
        print(index, row)

    """
    0 {'id': '1', 'name': '张三'}
    1 {'id': '2', 'name': '李四'}
    2 {'id': '3', 'name': '王五'}
    3 {'id': '4', 'name': '赵六'}
    """
```

**通过 pandas 读写**
```python
import pandas as pd

data = [
    {'id': '1', 'name': '张三'},
    {'id': '2', 'name': '李四'},
    {'id': '3', 'name': '王五'},
    {'id': '4', 'name': '赵六'},
]
df = pd.DataFrame(data)
df.to_csv(
    'csv_pandas_demo.csv',
    index=False,  # 去掉 DataFrame 第一列索引列
)

df = pd.read_csv('csv_pandas_demo.csv')
print(df)
"""
   id name
0   1   张三
1   2   李四
2   3   王五
3   4   赵六
"""
```