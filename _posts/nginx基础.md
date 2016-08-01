---
title: nginx基础
date: 2016-07-30 14:13:06
categories: 反向代理
tags: nginx
---
## 特点
1. 处理响应请求很快
在正常的情况下，单次请求会得到更快的响应。在高峰期，Nginx 可以比其它的 Web 服务器更快的响应请求。
 
1. 高并发连接
在互联网快速发展，互联网用户数量不断增加的今天，一些大公司、网站都需要面对高并发请求，如果有一个能够在峰值顶住 10 万以上并发请求的 Server，肯定会得到大家的青睐。理论上，Nginx 支持的并发连接上限取决于你的内存，10 万远未封顶。

<!--more-->

1. 低的内存消耗
在一般的情况下，10000 个非活跃的 HTTP Keep-Alive 连接在 Nginx 中仅消耗 2.5MB 的内存，这也是 Nginx 支持高并发连接的基础。
 
1. 具有很高的可靠性：
Nginx 是一个高可靠性的 Web 服务器，这也是我们为什么选择 Nginx 的基本条件，现在很多的网站都在使用 Nginx，足以说明 Nginx 的可靠性。高可靠性来自其核心框架代码的优秀设计、模块设计的简单性，并且这些模块都非常的稳定。
 
1. 高扩展性
Nginx 的设计极具扩展性，它完全是由多个不同功能、不同层次、不同类型且耦合度极低的模块组成。这种设计造就了 Nginx 庞大的第三方模块。
 
1. 热部署
master 管理进程与 worker 工作进程的分离设计，使得 Nginx 具有热部署的功能，可以在 7×24 小时不间断服务的前提下，升级 Nginx 的可执行文件。也可以在不停止服务的情况下修改配置文件，更换日志文件等功能。

## 安装
1. 获取nginx http://nginx.org/download/nginx-1.11.3.tar.gz
1. 解压缩 `tar -zxvf nginx-1.11.3.tar.gz`
1. 进入解压缩目录,执行 `./configure`
1. make & make install
1. 启动 nginx 之后，浏览器中输入 http://localhost 可以验证是否安装启动成功

##　nginx　配置示例
配置目录 conf 下有以下配置文件，除了 `nginx.conf`，其余配置文件，一般只需要使用默认提供即可
├── fastcgi.conf
├── fastcgi_params
├── koi-utf
├── koi-win
├── mime.types
├── nginx.conf
├── scgi_params
├── uwsgi_params
└── win-utf
nginx.conf是主配置文件，默认配置去掉注释之后的内容如下图所示：
 
```nginx
worker_process      # 表示工作进程的数量，一般设置为cpu的核数
 
worker_connections  # 表示每个工作进程的最大连接数
 
server{}            # 块定义了虚拟主机
 
    listen          # 监听端口
 
    server_name     # 监听域名
 
    location {}     # 是用来为匹配的 URI 进行配置，URI 即语法中的“/uri/”
 
    location /{}    # 匹配任何查询，因为所有请求都以 / 开头
 
        root        # 指定对应uri的资源查找路径，这里html为相对路径，完整路径为
                    # /opt/nginx-1.7.7/html/
 
        index       # 指定首页index文件的名称，可以配置多个，以空格分开。如有多
                    # 个，按配置顺序查找。
```

## 真实用例
```
#运行用户
user www;   
#启动进程,通常设置成和cpu的数量相等
worker_processes  1;
 
#全局错误日志及PID文件
error_log  /var/log/nginx/error.log;
pid        /var/run/nginx.pid;
 
#工作模式及连接数上限
events {
    use   epoll;             #epoll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
    worker_connections  1024;#单个后台worker process进程的最大并发链接数
    # multi_accept on;
}
 
#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
     #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    access_log    /var/log/nginx/access.log;
 
    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    #tcp_nopush     on;
 
    #连接超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;
    tcp_nodelay        on;
 
    #开启gzip压缩
    gzip  on;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
 
    #设定请求缓冲
    client_header_buffer_size    1k;
    large_client_header_buffers  4 4k;
 
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
 
    #设定负载均衡的服务器列表
     upstream mysvr {
    #weigth参数表示权值，权值越高被分配到的几率越大
    #本机上的Squid开启3128端口
    server 192.168.8.1:3128 weight=5;
    server 192.168.8.2:80  weight=1;
    server 192.168.8.3:80  weight=6;
    }
 
   server {
       #侦听80端口
        listen       80;
        #定义使用www.xx.com访问
        server_name  www.xx.com;
 
        #设定本虚拟主机的访问日志
        access_log  logs/www.xx.com.access.log  main;
 
    #默认请求
    location / {
          root   /root;      #定义服务器的默认网站根目录位置
          index index.php index.html index.htm;   #定义首页索引文件的名称
 
          fastcgi_pass  www.xx.com;
         fastcgi_param  SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
          include /etc/nginx/fastcgi_params;
        }
 
    # 定义错误提示页面
    error_page   500 502 503 504 /50x.html; 
        location = /50x.html {
        root   /root;
    }
 
    #静态文件，nginx自己处理
    location ~ ^/(images|javascript|js|css|flash|media|static)/ {
        root /var/www/virtual/htdocs;
        #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
        expires 30d;
    }
    #PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
    location ~ \.php$ {
        root /root;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME /home/www/www$fastcgi_script_name;
        include fastcgi_params;
    }
    #设定查看Nginx状态的地址
    location /NginxStatus {
        stub_status            on;
        access_log              on;
        auth_basic              "NginxStatus";
        auth_basic_user_file  conf/htpasswd;
    }
    #禁止访问 .htxxx 文件
    location ~ /\.ht {
        deny all;
    }
 
     }
}
```

从配置可以看出，nginx 监听了 80 端口、域名为 www.xx.com、根路径为 /root 文件夹
默认 index 文件为 index.html，index.htm 服务器错误重定向到 50x.html 页面
这也是上面在浏览器中输入 http://www.xx.com，能够显示欢迎页面的原因。实际上访问的是 /root/index.html 文件。


## location 匹配规则
### 语法规则
> location [=|~|~*|^~] /uri/ { … }

|符号|含义|
|:------:|:----------------------|
|=    |开头表示精确匹配|
|^~   |开头表示 uri 以某个常规字符串开头，理解为匹配 url 路径即可。nginx 不对 url 做编码，因此请求为`/static/20%/aa`，可以被规则`^~ /static/ /aa`匹配到（注意是空格）|
|~   |开头表示区分大小写的正则匹配|
|~*   |开头表示不区分大小写的正则匹配|
|/   |通用匹配，任何请求都会匹配到|


多个 location 配置的情况下匹配顺序为
* 首先匹配 `=`
* 其次匹配 `^~`
* 其次是按文件中顺序的正则匹配
* 最后是交给 `/` 通用匹配
* 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

例子，有如下匹配规则：
 
```nginx
location = / {
   #规则A
}
location = /login {
   #规则B
}
location ^~ /static/ {
   #规则C
}
location ~ \.(gif|jpg|png|js|css)$ {
   #规则D
}
location ~* \.png$ {
   #规则E
}
location / {
   #规则F
}
 
```

那么产生的效果如下：
 
* 访问根目录 `/`， 比如 `http://localhost/` 将匹配规则 A
* 访问 `http://localhost/login` 将匹配规则 B，`http://localhost/register` 则匹配规则 F
* 访问 `http://localhost/static/a.html` 将匹配规则 C
* 访问 `http://localhost/a.gif`, `http://localhost/b.jpg` 将匹配规则 D 和规则 E，但是规则 D 顺序优先，规则 E 不起作用，而 `http://localhost/static/c.png` 则优先匹配到规则 C
* 访问 `http://localhost/a.PNG` 则匹配规则 E，而不会匹配规则 D，因为规则 E 不区分大小写。
 
访问 `http://localhost/category/id/1111` 则最终匹配到规则 F，因为以上规则都不匹配，这个时候应该是 nginx 转发请求给后端应用服务器，比如 FastCGI（php），tomcat（jsp），nginx 作为反向代理服务器存在。



通常有三个匹配规则定义，如下：
 
```nginx
# 直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。
# 这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}
 
# 第二个必选规则是处理静态文件请求，这是 nginx 作为 http 服务器的强项
# 有两种配置模式，目录匹配或后缀匹配，任选其一或搭配使用
location ^~ /static/ {
    root /webroot/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
 
# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器
# 非静态文件请求就默认是动态请求，自己根据实际把握
# 毕竟目前的一些框架的流行，带.php、.jsp后缀的情况很少了
location / {
    proxy_pass http://tomcat:8080/
}
```

## Redirect 语法
```nginx
server {
    listen 80;
    server_name start.igrow.cn;
    index index.html index.php;
    root html;
    if ($http_host !~ "^star\.igrow\.cn$") {
        rewrite ^(.*) http://star.igrow.cn$1 redirect;
    }
}
```
## 防盗链
 
```nginx
location ~* \.(gif|jpg|swf)$ {
    valid_referers none blocked start.igrow.cn sta.igrow.cn;
    if ($invalid_referer) {
       rewrite ^/ http://$host/logo.png;
    }
}
```

## 根据文件类型设置过期时间
 
```nginx
location ~* \.(js|css|jpg|jpeg|gif|png|swf)$ {
    if (-f $request_filename) {
        expires 1h;
        break;
    }
}
```

## 禁止访问某个目录
 
```nginx
location ~* \.(txt|doc)${
    root /data/www/wwwroot/linuxtone/test;
    deny all;
}
```

##　Nginx 静态文件服务
最简单的本地静态文件服务配置
```
server {
        listen       80;
        server_name www.test.com;
        charset utf-8;
        root   /data/www.test.com;
        index  index.html index.htm;
       }
```
优化

```
http {
    # 这个将为打开文件指定缓存，默认是没有启用的，max 指定缓存数量，
    # 建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=204800 inactive=20s;
 
    # open_file_cache 指令中的inactive 参数时间内文件的最少使用次数，
    # 如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个
    # 文件在inactive 时间内一次没被使用，它将被移除。
    open_file_cache_min_uses 1;
 
    # 这个是指多长时间检查一次缓存的有效信息
    open_file_cache_valid 30s;
 
    # 默认情况下，Nginx的gzip压缩是关闭的， gzip压缩功能就是可以让你节省不
    # 少带宽，但是会增加服务器CPU的开销哦，Nginx默认只对text/html进行压缩 ，
    # 如果要对html之外的内容进行压缩传输，我们需要手动来设置。
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
 
    server {
            listen       80;
            server_name www.test.com;
            charset utf-8;
            root   /data/www.test.com;
            index  index.html index.htm;
           }
}
```

##　安装和配置基础缓存
我们只需要两个命令就可以启用基础缓存： proxy_cache_path 和 proxy_cache 。proxy_cache_path 用来设置缓存的路径和配置，proxy_cache 用来启用缓存。
```
proxy_cache_path/path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m
use_temp_path=off;
 
server {
    ...
    location / {
        proxy_cache my_cache;
        proxy_pass http://my_upstream;
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;//，当 Nginx 收到服务器返回的error，timeout或者其他指定的5xx错误，
                                                               //并且在其缓存中有请求文件的陈旧版本，则会将这些陈旧版本的文件而不是错误信息发送给客户端
    }
 
}
```
```
proxy_cache_path 命令中的参数及对应配置说明如下：
1. 用于缓存的本地磁盘目录是 /path/to/cache/
1. levels 在 /path/to/cache/ 设置了一个两级层次结构的目录。将大量的文件放置在单个目录中会导致文件访问缓慢，所以针对大多数部署，我们推荐使用两级目录层次结构。如果 levels 参数没有配置，则 Nginx 会将所有的文件放到同一个目录中。
1. keys_zone 设置一个共享内存区，该内存区用于存储缓存键和元数据，有些类似计时器的用途。将键的拷贝放入内存可以使 Nginx 在不检索磁盘的情况下快速决定一个请求是HIT还是MISS，这样大大提高了检索速度。一个 1MB 的内存空间可以存储大约 8000个key，那么上面配置的 10MB 内存空间可以存储差不多 80000 个 key。
1. max_size 设置了缓存的上限（在上面的例子中是 10G）。这是一个可选项；如果不指定具体值，那就是允许缓存不断增长，占用所有可用的磁盘空间。当缓存达到这个上线，处理器便调用 cache manager 来移除最近最少被使用的文件，这样把缓存的空间降低至这个限制之下。
1. inactive 指定了项目在不被访问的情况下能够在内存中保持的时间。在上面的例子中，如果一个文件在 60 分钟之内没有被请求，则缓存管理将会自动将其在内存中删除，不管该文件是否过期。该参数默认值为 10 分钟（10m）。注意，非活动内容有别于过期内容。 Nginx 不会自动删除由缓存控制头部指定的过期内容（本例中 Cache-Control:max-age=120）。过期内容只有在 inactive 指定时间内没有被访问的情况下才会被删除。如果过期内容被访问了，那么 Nginx 就会将其从原服务器上刷新，并更新对应的inactive计时器。
1. Nginx 最初会将注定写入缓存的文件先放入一个临时存储区域，use_temp_path=off命令指示 Nginx 将在缓存这些文件时将它们写入同一个目录下。我们强烈建议你将参数设置为off来避免在文件系统中不必要的数据拷贝。use_temp_path在 Nginx 1.7版本和 Nginx Plus R6中有所介绍。
最终，proxy_cache 命令启动缓存那些URL与location部分匹配的内容（本例中，为/）。你同样可以将proxy_cache命令添加到server部分，这将会将缓存应用到所有的那些location中未指定自己的proxy_cache命令的服务中。
```

## 缓存微调
```
proxy_cache_path /path/to/cache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m
use_temp_path=off;
server {
    ...
    location / {
        proxy_cache my_cache;
        proxy_cache_revalidate on;
        proxy_cache_min_uses 3;
        proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
        proxy_cache_lock on;
        proxy_pass http://my_upstream;
    }
}
```
```
这些命令配置了下列的行为：
1. proxy_cache_revalidate 指示 Nginx 在刷新来自服务器的内容时使用 GET 请求。如果客户端的请求项已经被缓存过了，但是在缓存控制头部中定义为过期，那么 Nginx 就会在 GET 请求中包含 If-Modified-Since 字段，发送至服务器端。这项配置可以节约带宽，因为对于 Nginx 已经缓存过的文件，服务器只会在该文件请求头中 Last-Modified 记录的时间内被修改时才将全部文件一起发送。
1. proxy_cache_min_uses 该指令设置同一链接请求达到几次即被缓存，默认值为 1 。当缓存不断被填满时，这项设置便十分有用，因为这确保了只有那些被经常访问的内容会被缓存。
1. proxy_cache_use_stale 中的 updating 参数告知 Nginx 在客户端请求的项目的更新正在原服务器中下载时发送旧内容，而不是向服务器转发重复的请求。第一个请求陈旧文件的用户不得不等待文件在原服务器中更新完毕。陈旧的文件会返回给随后的请求直到更新后的文件被全部下载。
1. 当 proxy_cache_lock 被启用时，当多个客户端请求一个缓存中不存在的文件（或称之为一个 MISS），只有这些请求中的第一个被允许发送至服务器。其他请求在第一个请求得到满意结果之后在缓存中得到文件。如果不启用 proxy_cache_lock，则所有在缓存中找不到文件的请求都会直接与服务器通信。
```

## 日志
Nginx 日志主要有两种：access_log(访问日志) 和 error_log(错误日志)。
###  access_log 访问日志
access_log 主要记录客户端访问 Nginx 的每一个请求，格式可以自定义。通过 access_log 你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息。
log_format 指令用于定义日志的格式，语法: log_format name string; 其中 name 表示格式名称，string 表示定义的格式字符串。log_format 有一个默认的无需设置的组合日志格式。
```
#默认的无需设置的组合日志格式
log_format combined '$remote_addr - $remote_user  [$time_local]  '
                    ' "$request"  $status  $body_bytes_sent  '
                    ' "$http_referer"  "$http_user_agent" ';

```

access_log 指令用来指定访问日志文件的存放路径（包含日志文件名）、格式和缓存大小，语法：access_log path [format_name [buffer=size | off]]; 其中 path 表示访问日志存放路径，format_name 表示访问日志格式名称，buffer 表示缓存大小，off 表示关闭访问日志。
```
log_format 使用：在 access.log 中记录客户端 IP 地址、请求状态和请求时间
log_format myformat '$remote_addr  $status  $time_local';
access_log logs/access.log  myformat;
```
需要注意的是：log_format 配置必须放在 http 内，否则会出现警告。Nginx 进程设置的用户和组必须对日志路径有创建文件的权限，否则，会报错。

定义日志使用的字段及其作用
|字段|作用|
|----|----|
|$remote_addr与$http_x_forwarded_for | 记录客户端IP地址 |
|$remote_user| 记录客户端用户名称 |
|$request| 记录请求的URI和HTTP协议 |
|$status | 记录请求状态 |
|$body_bytes_sent | 发送给客户端的字节数，不包括响应头的大小 |
|$bytes_sent| 发送给客户端的总字节数 |
|$connection | 连接的序列号 |
|$connection_requests | 当前通过一个连接获得的请求数量 |
|$msec | 日志写入时间。单位为秒，精度是毫秒 |
|$pipe | 如果请求是通过HTTP流水线(pipelined)发送，pipe值为“p”，否则为“.” |
|$http_referer | 记录从哪个页面链接访问过来的 |
|$http_user_agent | 记录客户端浏览器相关信息 |
|$request_length | 请求的长度（包括请求行，请求头和请求正文）|
|$request_time | 请求处理时间，单位为秒，精度毫秒 |
|$time_iso8601 | ISO8601标准格式下的本地时间 |
|$time_local | 记录访问时间与时区 |


## error_log 错误日志
error_log 主要记录客户端访问 Nginx 出错时的日志，格式不支持自定义。通过查看错误日志，你可以得到系统某个服务或 server 的性能瓶颈等。因此，将日志利用好，你可以得到很多有价值的信息。
 
error_log 指令用来指定错误日志，语法: `error_log path level`; 其中 path 表示错误日志存放路径，level 表示错误日志等级，日志等级包括 debug、info、notice、warn、error、crit，从左至右，日志详细程度逐级递减，即 debug 最详细，crit 最少，默认为 crit。
 
注意：`error_log off` 并不能关闭错误日志记录，此时日志信息会被写入到文件名为 off 的文件当中。如果要关闭错误日志记录，可以使用如下配置：
 
>Linux 系统把存储位置设置为空设备
 
```nginx
 
error_log /dev/null;
 
http {
    # ...
}
```
 
>Windows 系统把存储位置设置为空设备
 
```nginx
 
error_log nul;
 
http {
    # ...
}
```
 
另外 Linux 系统可以使用 tail 命令方便的查阅正在改变的文件,`tail -f filename`会把 filename 里最尾部的内容显示在屏幕上,并且不断刷新,使你看到最新的文件内容。Windows 系统没有这个命令，你可以在网上找到动态查看文件的工具。
