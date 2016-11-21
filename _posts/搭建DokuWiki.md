---
title: 搭建DokuWiki
date: 2016-11-19 11:42:56
categories: wiki
tags: wiki
---
# 安装nginx
```
yum install nginx
```

# 设置开机启动
```
chkconfig nginx on
```

# 启动服务
```
service nginx start
```

<!--more-->
# 安装php
```
yum install php
```
# 安装php组件
```
yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear 
php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt
 libmcrypt-devel php-fpm
```
# 开机启动
```
chkconfig php-fpm on

```
打开nginx的配置文件
```
server {
    listen       80;
    server_name  域名;

    location / {
        root   php文件主目录;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   php文件主目录;
    }

    location ~ \.php$ {
        root           php文件主目录;// /usr/share/nginx/html
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}

```
重启或者reload nginx

# 访问 
```
ip/dokuwiki/install.php
根据提示安装
```

# 安全
```
按照提示设置，完成安装，中文选择右上角
删除install.php
```

# nginx 配置
```
location ~ /(data|conf|bin|inc)/ 
{ 
deny all; 
}  
```


