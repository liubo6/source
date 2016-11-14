---
title: ElasticStack
date: 2016-11-14 09:56:04
categories:
tags:
---
# elastic stack
![elastic stack](http://liubo.qiniudn.com/elastic_stack.png)

<!--more-->
# Logstash: Collect from diverse inputs
![](http://liubo.qiniudn.com/logstash.png)

# beats
![](http://liubo.qiniudn.com/beats.png)

# Kibana
![](http://liubo.qiniudn.com/kibana.png)

# elasticsearch
![](http://liubo.qiniudn.com/elasticsearch.png)
## Analytics
柱状图、分布、统计、地理…
## 任何数据
能被查询到的数据就能被分析
## 接近实时
按需实时计算，~1s 刷新间隔

# 下载
## E
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.0.zip
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.0.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.0.0.rpm
```
## L
```
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.0.0.zip
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.0.0.tar.gz
wget https://artifacts.elastic.co/downloads/logstash/logstash-5.0.0.rpm
```

## K
```
wget https://artifacts.elastic.co/downloads/kibana/kibana-5.0.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-5.0.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/kibana/kibana-5.0.0-darwin-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-5.0.0-windows-x86.zip

```
## B
Pick a Beat. Any Beat
```
#Filebeat[Real-time insight into log data.]
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.0.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.0.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-5.0.0-windows-x86_64.zip

#Packetbeat[Analyze network packet data.]
wget https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-5.0.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-5.0.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-5.0.0-windows-x86_64.zip

#Winlogbeat[Analyze Windows event logs.]
wget https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-5.0.0-windows-x86_64.zip
wget https://artifacts.elastic.co/downloads/beats/winlogbeat/winlogbeat-5.0.0-windows-x86.zip

#Metricbeat
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.0.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.0.0-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-5.0.0-windows-x86_64.zip

#Topbeat[Insights from infrastructure data]
wget https://download.elastic.co/beats/topbeat/topbeat-1.3.1-x86_64.rpm
wget https://download.elastic.co/beats/topbeat/topbeat-1.3.1-x86_64.tar.gz
wget https://download.elastic.co/beats/topbeat/topbeat-1.3.1-darwin.tgz
wget https://download.elastic.co/beats/topbeat/topbeat-1.3.1-windows.zip
```
![](http://liubo.qiniudn.com/beats_all.png)

# 社区
```
源码 & Issue: http://github.com/elastic/
英文社区: http://discuss.elastic.co
中文社区: http://elasticsearch.cn
官方 QQ 群: 190605846
下载: https://www.elastic.co/downloads
博客: https://www.elastic.co/blog
线下活动: http://elasticsearch.meetup.com/
```

# ES 基本操作
## 索引（插入数据）
```
POST demo/pm2_5/1
{
“city”:“北京” , “date”: “2016-02-08”,
“aq_level”: “严重污染”, “aq_rank”:68, “aqi”: 391, “co”:115.5,
“no2”: 1.888, “o3”:62.2, “pm2_5”: 415.7, “range”: “74~500”,
“so2”: 523.5, “location”: {“lat”:39.92, “lon”:116.46}
}
---
{
"_index": "demo",
"_type": "pm2_5",
"_id": "1",
"_version": 1,
"_shards": {
"total": 2,
"successful": 1,
"failed": 0
},
"created": true
}
```

## 获取数据
```
获取数据
GET demo/pm2_5/1
{
"_index": "demo",
"_type": "pm2_5",
"_id": "1",
"_version": 1,
"found": true,
"_source": {
"aq_level": "严重污染“, "aq_rank": 68, "aqi": 391, "city": "北京“, "co": 115.5, "date": "2016-02-08“, "province": "北京",
"range": "74~500“, "so2": 523.5
}
}
```

## 删除数据
```
删除数据
DELETE demo/pm2_5/1
```
## 搜索
通过GET参数进行搜索
```
GET demo/pm2_5/_search?q=北京
```

查询北京的 PM2.5 数据
```
POST demo/pm2_5/_search
{
"query": {
"match": {
“city”: “北京"
}
}
}
```
## 分析
```
Aggregation
POST twitter/tweet/_search
{
"aggs" : {
"uers_stats" : {
"terms" : { "field" : "user" }
}
}
}
```




