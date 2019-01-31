安装依赖：
```
pip install gunicorn
pip install gevent
```
先cd到项目启动看看：
```
gunicorn main:app
# 这里main的意思是main.py(你的可能是server.py、manage.py，分别写成server或manage)，app则是这个代码文件中Flask对象的名称
# 指定host和端口：-b 127.0.0.1:8080 
# 指定4个进程：-w 4
# 使用gevent：-k gevent
```
通过指定配置文件启动：`gunicorn -c my_gunicorn.conf main:app`
配置文件简单示例：
```
workers = 4 
bind = '0.0.0.0:5000'
worker_class = 'gevent'
```
然后就可以在Nginx中配置反向代理到这个gunicorn了，注意是用普通的proxy_pass。
注：gunicorn部署django时，一般就是执行`gunicorn myproject.wsgi:application` # applcation可以省略