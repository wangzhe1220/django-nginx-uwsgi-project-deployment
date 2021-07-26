# 使用uWSGI+nginx部署django项目  
>> 由nginx负责路由+处理静态资源，uWSGI负责多进程prefork  

## 安装uWSGI  
- 方法一：pip安装
  `pip3 install uwsgi`
- 方法二：二进制文件安装  
  ```
  wget http://projects.unbit.it/downloads/uwsgi-latest.tar.gz
  tar zxvf uwsgi-latest.tar.gz
  cd uwsgi-latest
  make
  ```
  或
  `curl http://uwsgi.it/install | bash -s default /usr/local/uwsgi`

建议使用pip安装，比较简单
