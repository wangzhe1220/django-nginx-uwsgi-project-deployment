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
### django静态资源文件夹 
django一般有1~2个静态资源文件夹，分别为static和media。  
在django开发中，一般使用`python manage.py collectstatic`收集静态资源，在settings.py控制静态资源路径，并且使用urls.py来访问静态资源链接。 有了nginx对静态资源的接管，可以省略这些步骤。 
但是我们依然要获得django静态资源文件夹的绝对路径，我的是`/mnt/mol-ci/static-file/static`,`/mnt/mol-ci/static-file/media`  。

