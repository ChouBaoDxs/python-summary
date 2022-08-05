[TOC]

# Django REST framework

# Django
## 读取数据库表生成 Model 定义
一般用于已有项目（数据库表不是由 django migrate 生成的）上进行 django 开发
- 命令：`python manage.py inspectdb`
- 可以选择输出到文件：`python manage.py inspectdb > models.py`

## 收集静态文件
```python
# settings.py
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```
执行命令：`python manage.py collectstatic`

由于 DEBUG 模式下，django 会通过 STATICFILES_DIRS 帮忙处理静态文件，而 STATICFILES_DIRS 和 STATIC_ROOT 是冲突配置，所以我习惯下面这样，在需要收集静态文件时将 DEBUG 设置为 False： 
```python
# settings.py
if DEBUG:
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]
else:
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```
