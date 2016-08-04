---
title: ELK搭建和使用
date: 2016-08-04 11:51:41
categories: 日志
tags: logs
---
# 需求
为什么要说ELK
在日常的运维管理活动,当发现error时可以从日志了解报错及时解决。日志分为系统日志,应用日志,和安全日志,经常的分析日志可以了解服务器的硬件状况,性能以及安全,通常情况下，**日志被分散到不同的存储设备上**，而企业内部的服务器,少则十几台多则成千上百,如果采取最传统的方式登录每台服务器进行查看,对运维来说难度大劳动强度也大,而且不易管理容易出错不易管理,所以需要一个进行**集中化管理日志的解决方案**。

<!--more-->

![](http://ww1.sinaimg.cn/mw690/69045600gw1f6fbqiuk3pj20go08cwf5.jpg)

# ELK是什么
ELK是elasticsearch,logstash,kibana  这三个工具的简称,实时日志处理领域，开源界的第一选择

## elasticsearch
>Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等

## logstash
>Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。

## kibana
>Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。

# 安装配置ELK
## 安装java环境
```
[root@localhost tmp]# rpm -ivh jdk-8u60-linux-x64.rpm
默认安装到了/usr/java
 
配置环境变量
[root@localhost jdk1.8.0_60]# vi + /etc/profile
export JAVA_HOME=/usr/java/jdk1.8.0_60
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH
```

## 安装elasticsearch
```
解压到/opt/ELK目录下
tar -zxvf elasticsearch-2.3.4.tar.gz -C /opt/ELK/
修改配置
vim /opt/ELK/elasticsearch-2.3.4/config/elasticsearch.yml
cluster.name: liubo-cluster //集群名称
node.name: node-liubo-1     //节点名称
path.data: /opt/ELK/data    //数据目录
path.logs: /opt/ELK/logs    //日志目录
network.host: ["127.0.0.1","172.30.251.243"]  //监听地址,默认只能localhost访问,所以配置需要访问的地址
http.port: 9200             //监听端口
```
不能使用root启动elasticsearch,创建用户
```
Exception in thread "main" java.lang.RuntimeException: don't run elasticsearch as root.
```
创建用户 useradd elastic
设置密码 passwd elastic// 输入密码,elastic
目录变更所有者 chown -R elastic:elastic /opt/ELK/ 

访问地址 http://172.30.251.243:9200/,成功则显示如下
```
{
name: "node-1",
cluster_name: "funds-cluster",
version: {
number: "2.3.4",
build_hash: "e455fd0c13dceca8dbbdbb1665d068ae55dabe3f",
build_timestamp: "2016-06-30T11:24:31Z",
build_snapshot: false,
lucene_version: "5.5.0"
},
tagline: "You Know, for Search"
}
```

## 安装logstash
```
解压 tar -zxvf logstash-2.3.4.tar.gz -C /opt/ELK/
启动并测试,输入什么,输出什么,输入（logstash收集）到什么内容，logstach 就会格式化的输出一些内容
[root@ bin]# ./logstash -e 'input{stdin{}} output{stdout{}}'
hello
Settings: Default pipeline workers: 2
Pipeline main started
2016-08-02T06:15:31.220Z 0.0.0.0 hello
welcome
2016-08-02T06:15:36.138Z 0.0.0.0 welcome
#参数解释
-e 参数:执行
input:输入
output:输出
-d参数:daemon模式,后台启动守护进程

```
创建配置文件
```
vim logstash-test.conf
内容
input { stdin { } }
output {
   elasticsearch {hosts => "127.0.0.1" }
   stdout { codec=> rubydebug }
}


以配置文件方式启动logstash
./bin/logstash -f /opt/ELK/logstash-2.3.4/etc/logstash-test.conf
```



## 安装kibana
```
解压 tar -zxvf kibana-4.5.3-linux-x64.tar.gz  -C /opt/ELK/
更改配置 vim /opt/ELK/kibana-4.5.3-linux-x64/config/kibana.yml
server.port: 5601    //端口
server.host: "172.30.251.243" //本地监听地址
elasticsearch.url: "http://localhost:9200" //elasticsearch 地址
kibana.index: ".kibana" //索引
访问页面 http://172.30.251.243:5601
```


# 插件安装
## head插件
es 安装head 插件,提供可视化，bgidesk 插件
```
./bin/plugin install mobz/elasticsearch-head
./bin/plugin install lukas-vlcek/bigdesk
http://172.30.251.243:9200/_plugin/head/ 可以访问界面
```
## kopf集群管理插件
```
./bin/plugin install lmenezes/elasticsearch-kopf

```

# 收集日志,收集多个日志，开启多个logstash进程即可
## 收集本机日志
启动E,K
logstash配置,启动 logstash -f logstash-test.conf
```
input {
        file{
                path => ["/opt/business/logs/business/*.log"]
                type => "business_log"
                start_position => "beginning"
        }
}
output {
   elasticsearch {hosts => "127.0.0.1:9200"
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
        }
}
```
打开K,可以看到日志,可以安装时间排序等等

## 收集远程日志

可以加入redis,list使用,提高性能.
日志->logstash1->redis->logstash2->es
以nginx为例 

```
vi nginx.conf     #设置log_format,去掉注释
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
access_log  logs/host.access.log  main;  #设置access日志，有访问时自动写入此文件
```


### logstash1配置
```
input {
        file {
                type => "nginx_access_log"
                path => ["/usr/local/nginx/logs/host.access.log"]  
        }
}
output {
        redis {
                host => "192.168.1.10" #redis server
                data_type => "list"
                key => "logstash:redis"
        }
}
```
### logstash2配置
```
input {
        redis {
                host => "192.168.1.10"
                data_type => "list"
                key => "logstash:redis"
                type => "redis-input"
        }
}
filter {
        grok {
                type => "nginx_access"
                match => [
                        "message", "%{IPORHOST:http_host} %{IPORHOST:client_ip} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:http_verb} %{NOTSPACE:http_request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:raw_http_request})\" %{NUMBER:http_status_code} (?:%{NUMBER:bytes_read}|-) %{QS:referrer} %{QS:agent} %{NUMBER:time_duration:float} %{NUMBER:time_backend_response:float}",
                        "message", "%{IPORHOST:http_host} %{IPORHOST:client_ip} \[%{HTTPDATE:timestamp}\] \"(?:%{WORD:http_verb} %{NOTSPACE:http_request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:raw_http_request})\" %{NUMBER:http_status_code} (?:%{NUMBER:bytes_read}|-) %{QS:referrer} %{QS:agent} %{NUMBER:time_duration:float}"
                ]
        }
}
output {
        elasticsearch {
                embedded => false
                protocol => "http"
                host => "localhost"
                port => "9200"
        }
}
```