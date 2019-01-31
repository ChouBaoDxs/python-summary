
[TOC]
##### 1.安装配置supervisor
1-1.安装supervisor
```
sudo pip install supervisor
或者
yum install supervisor
```
##### 2.配置supervisor
2-1. 先创建一个目录，用来存放supervisord管理的程序配置文件
```
mkdir /etc/supervisor
```
2-2. 生成默认supervisor配置文件并配置
```
echo_supervisord_conf > supervisord.conf
```
以上命令会在当前目录生成一个supervisord.conf，使用vim打开该文件修改
```
# [include]
# files = relative/directory/*.ini
# 为（2-1创建的目录)
[include]
files = /etc/supervisor/*.conf
```
2-3. 将修改好的supervisord.conf复制到/etc/目录
```
cp supervisord.conf /etc/
```
##### 3.配置被supervisord管理的tornado
3-1. 在2-1创建的目录/etc/supervisor下用vim编辑一个被supervisord管理的tornado配置文件
```
vim tornado_baidu_music.conf
```
3-2. 填入以下配置信息(根据实际情况修改)：
```
[group:tornadoes]
programs=tornado-8000,tornado-8001,tornado-8002,tornado-8003

[program:tornado-8000]
command=/VirtualenvWorkSpace/env36/bin/python /PythonProjects/tornado/baidu_music/musicapp.py --port=8000
directory=/PythonProjects/tornado/baidu_music
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/PythonProjects/tornado/baidu_music/tornado.log
loglevel=info

[program:tornado-8001]
command=/VirtualenvWorkSpace/env36/bin/python /PythonProjects/tornado/baidu_music/musicapp.py --port=8001
directory=/PythonProjects/tornado/baidu_music
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/PythonProjects/tornado/baidu_music/tornado.log
loglevel=info

[program:tornado-8002]
command=/VirtualenvWorkSpace/env36/bin/python /PythonProjects/tornado/baidu_music/musicapp.py --port=8002
directory=/PythonProjects/tornado/baidu_music
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/PythonProjects/tornado/baidu_music/tornado.log
loglevel=info

[program:tornado-8003]
command=/VirtualenvWorkSpace/env36/bin/python /PythonProjects/tornado/baidu_music/musicapp.py --port=8003
directory=/PythonProjects/tornado/baidu_music
user=root
autorestart=true
redirect_stderr=true
stdout_logfile=/PythonProjects/tornado/baidu_music/tornado.log
loglevel=info
```
##### 4.测试supervisord
4-1. 启动supervisord
```
supervisord -c /etc/supervisord.conf
```
4-2. 查看supervisord是否运行
```
ps aux | grep supervisord
```
4-3. 使用supervisorctl管理supervisor
```
supervisorctl   # 执行之后会列出正在运行的程序
> status    # 查看程序状态
> stop tornadoes:*   # 关闭 tornadoes组 程序
> start tornadoes:*  # 启动 tornadoes组 程序
> restart tornadoes:*    # 重启 tornadoes组 程序
> update    ＃ 重启配置文件修改过的程序
> exit  # 退出交互
```

##### 5.nginx配置(Centos7下通过yum安装nginx见6)
5-1. 编辑nginx配置文件
```
vim /etc/nginx/nginx.conf
```
填写以下配置：
```
upstream tornadoes {
    server 127.0.0.1:8000;
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    listen 9000;    # 要nginx监听的端口
    server_name baidu_music.tornado.com;

    location /static/ {         # 如果有静态资源的话
        # 路径root和alias的区别详见=>nginx笔记.md
        alias /PythonProjects/tornado/baidu_music/static/;
        if ($query_string) {
            expires max;
        }
    }

    location / {
        proxy_pass_header Server;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Scheme $scheme;  # 协议 http https
        proxy_pass http://tornadoes;
    }
}
```
5-2. 重启nginx即可
```
systemctl restart nginx
```
5-3. 注意点
Tornado不知道从哪个版本开始，默认绑定的address就是0.0.0.0了，想要绑定到127.0.0.1，就去修改项目目录下server.py里的main方法：```
```
# http_server.listen(options.port)
# ↓
http_server.bind(options.port, '127.0.0.1')
http_server.start(1)
```

##### 6.Centos7下通过yum安装nginx
参考文章地址：https://blog.csdn.net/oldguncm/article/details/78855000
6-1. 添加CentOS 7 EPEL仓库
```
sudo yum install epel-release
```
6-2. 安装nginx
```
sudo yum install nginx -y
```
安装完后，nginx目录在/etc/nginx下，默认配置文件也在该目录
6-3. nginx常用命令
```
systemctl start nginx  # 启动
systemctl stop nginx  # 停止
systemctl restart nginx  # 重启
```
6-4. 关于防火墙
如果您正在运行防火墙，请运行以下命令以允许HTTP和HTTPS通信：
```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```