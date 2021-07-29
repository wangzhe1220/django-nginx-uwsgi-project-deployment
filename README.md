# 使用uWSGI+nginx部署django项目  
>> 由nginx负责路由+处理静态资源，uWSGI负责多进程prefork  

## 安装软件准备
### 安装uWSGI  
- 方法一：pip安装
  `pip3 install uwsgi`
- 方法二：二进制文件安装  
  ```
  curl http://uwsgi.it/install | bash -s default /usr/local/uwsgi
  wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
  tar zxvf uwsgi-latest.tar.gz
  cd uwsgi-latest
  make
  ```
建议使用pip安装，比较简单

### 安装nginx
- yum直接安装（我是centOS）  
  `yum install nginx`
### django项目
自备

## 进行整合  
### 测试django项目能否正确运行  
运行失败的项目是无法整合的，使用`python manage.py runserver 0.0.0.0:8000`命令尝试在8000端口运行一下，如果浏览器输入地址+:8000可以正常访问，即使静态资源全部404，我们依然可以认为运行成功。  
### django uWSGI app  
uWSGI在django以app的方式被识别，app文件代码（这个文件新建项目时应该就会自带）：  
```
import os

from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'project.settings')

application = get_wsgi_application()

```
django settings.py uwsgi配置：
`WSGI_APPLICATION = 'project.wsgi.application'`  
### 配置uWSGI 
uWSGI的配置非常简单，一个uwsgi.ini文件可以完成所有常用参数的配置。建议将uwsgi.ini放在django项目中，便于版本管理。 
uwsgi.ini 示例：  
```
# uwsig使用配置文件启动
[uwsgi]
# 项目所在的根目录
chdir=/mnt/demo/python-code
confdir=/mnt/demo
# 指定项目的application,区别于启动命令--wsgi-filemysite/wsgi.py
module=project.wsgi:application
#the local unix socket file than commnuincate to Nginx
# 指定sock的文件路径，这个sock文件会在nginx的uwsgi_pass配置，用来nginx与uwsgi通信
# 支持ip+port模式以及socket file模式
#socket=%(chdir)/uwsgi_confs/uwsgi.sock
# socket=127.0.0.1:9001
# 进程个数
processes = 8
# 每个进程worker数
workers=5
procname-prefix-spaced=mol                # uwsgi的进程名称前缀
py-autoreload=1                              # py文件修改，自动加载

# 指定IP端口，web访问入口
http=0.0.0.0:8002

# 指定多个静态文件：static目录和media目录,也可以不用指定该静态文件，在nginx中配置静态文件目录
# uwsgi有自己的配置语法，详细可参考官网，无需写绝对路径，可以用循环、判断等高级配置语法
# for =static media
# static-map=/static=%(chdir)/%(_)
# endfor =

# 启动uwsgi的用户名和用户组
uid=root
gid=root

# 启用主进程
master=true
# 自动移除unix Socket和pid文件当服务停止的时候
vacuum=true

# 序列化接受的内容，如果可能的话
thunder-lock=true
# 启用线程
enable-threads=true
# 设置一个超时，用于中断那些超过服务器请求上限的额外请求
harakiri=30
# 设置缓冲
post-buffering=4096

# 设置日志目录
daemonize=%(confdir)/uwsgi_confs/uwsgi.log
# uWSGI进程号存放
pidfile=%(confdir)/uwsgi_confs/uwsgi.pid
#monitor uwsgi status  通过该端口可以监控 uwsgi 的负载情况
# 支持ip+port模式以及socket file模式
# stats=%(confdir)/uwsgi_conf/uwsgi.status
# stats = 127.0.0.1:9001
```
>> 注意!confdir需要事先新建，用来存放uwsgi运行的log和pid文件。我这里使用的是http方式启动uWSGI，所以nginx的配置和常见nginx+uWSGI有区别。
### django静态资源文件夹 
django一般有1~2个静态资源文件夹，分别为static和media。  
在django开发中，一般使用`python manage.py collectstatic`收集静态资源，在settings.py控制静态资源路径，并且使用urls.py来访问静态资源链接。 有了nginx对静态资源的接管，可以省略这些步骤。   
但是我们依然要获得django静态资源文件夹的绝对路径，我的是`/mnt/mol-ci/static-file/static`,`/mnt/mol-ci/static-file/media`  。  
### nginx配置  
/etc/conf.d文件夹下建立一个配置文件test.conf:  
```
server {
  listen       80;
  listen       [::]:80;
  server_name  yourip.test.com;


  # 项目是前后端分离，所以根目录直接选前端首页
   location / {
    root  /usr/share/nginx/html/master;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
  }
  
  # 根据项目url分发方式在这里设置RESTful api转发
    location /api/ {
    proxy_pass http://localhost:8002;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  # 静态文件static
  location /static/ {
    alias /mnt/mol-ci/static-file/staticp/;
  }
  # 静态文件media
  location /media/ {
    alias /mnt/mol-ci/static-file/media/;
  }

```
### 启动项目
启动uWSGI: `uwsgi --ini /yourpath/uwsgi.ini`  
启动nginx: `systemctl start nginx`（刚装上从未启动过） `nginx -s reload`（以前就装过） 



