## 1.上传项目到服务器

![image-20210228162907286](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210228162929.png)

直接用FTP软件上传

## 2. 安装所需要的依赖



## 3. 安装nginx

/etc/nginx/sites-enabled/default 配置如下

![image-20210228163200037](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210228163200.png)

app:app 项目入口程序叫app.py，程序内变量也叫app

## 4. 安装uwsgi

`pip3 install uwsgi`



## 5. 安装

`sudo apt-get install uwsgi-plugin-python3`



## 6. 配置uwsgi.ini

在项目文件夹内创建uwsgi.ini

![image-20210228163821059](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210228163821.png)

## 7. 启动nginx

`sudo /etc/init.d/nginx start`

## 8. 启动uwsgi并挂在后台

`sudo nohup uwsgi uwsgi.ini &`