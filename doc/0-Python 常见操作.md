[TOC]

## Pip 源替换
临时使用清华源：
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```
升级 pip 到最新的版本 (>=10.0.0) 后全局配置清华源：
```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade pip
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
可选阿里源：
```
https://mirrors.aliyun.com/pypi/simple/
```

## 生成当前 Python 环境的依赖库清单
`pip freeze > requirements.txt`

## 扫描项目输出依赖库清单
通常同于缺少或丢失`requirements.txt`的项目
1. `pip install pipreqs`
2. 在项目根目录执行命令`pipreqs ./`，会在根目录生成文件`requirements.txt`
