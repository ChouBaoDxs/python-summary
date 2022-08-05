[TOC]

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