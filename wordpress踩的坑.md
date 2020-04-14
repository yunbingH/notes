## 升级PHP

1. 首先移除当前php包

   `yum remove php`

2. 安装PHP

   ```
      rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm         #更新源
   
      rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
   
      yum install -y php72w php72w-opcache php72w-xml php72w-mcrypt php72w-gd php72w-devel php72w-mysql php72w-intl      php72w-mbstring  php70w-fpm #安装依赖包
   
   ```

3. 开启服务

   `systemctl start php-fpm`

## 配置nginx

```
server {
    listen       80 default_server;
    #listen       [::]:80 ipv6only=on;#ipv6不用可以注释掉
    server_name  netest.club;
    root         /usr/share/wordpress;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
        root   /usr/share/wordpress;
        index  index.php index.html index.htm;#这里
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
    location ~ \.php$ {
    root            /usr/share/wordpress;
    fastcgi_pass    127.0.0.1:9000;
    fastcgi_index   index.php;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include         fastcgi_params;
    }
}

```

重启服务

`nginx -s reload`

---

## 配置mariaDB

`yum install -y mariadb`

`systemctl start mariadb.service`

`mysqladmin -uroot password '******'`

`create database wprdpress`

然后在wp-config-sample.php中配置数据库信息，再把文件名改为wp-config.php.



vim /usr/share/nginx/html/wordpress/wp-contents/你的主题名称/footer.php
在最后一行上添加，也可以在最大范围的 中的最后一行添加这句

\<div style="text-align: center;">\<a href="www.miitbeian.gov.cn" target="_blank">填写你的备案号</a\></div\>

------------------------------------------------
或在/usr/share/wordpress/wp-config.php里加

```
define ('WP_ZH_CN_ICP_NUM',true);
```

##　wordpress无法建立目录“wp-contents/uploads/***/***没有上级目录的写权限

`chmod 777 /usr/share/wordpress`