---
layout:     post
title:      elasticsearch系统学习笔记-01
subtitle:   环境搭建
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---


## 本机环境

- MacOSX 12.1
- JDK1.8

```
> source ~/.bash_profile
> jdk8
```

## ElasticSearch 环境

1. 下载

下载地址：https://www.elastic.co/cn/downloads/past-releases#elasticsearch

本机版本：[elasticsearch-6-3-2](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-3-2)

2. 启动

> cd software/elasticsearch-6.3.2 

>  ./bin/elasticsearch

> ps -ef | grep elastic

浏览器访问：http://127.0.0.1:9200/ 验证是否启动正常


---

```
本机报错：
org.elasticsearch.bootstrap.StartupException: ElasticsearchException
[Failed to create native process factories for Machine Learning];

问题解决：
此版本默认启用机器学习功能，此功能使用SSE4.2指令，因此仅适用于CPU 支持 SSE4.2的机器。
如果在旧硬件上运行 Elasticsearch，则必须禁用机器学习
（通过设置xpack.ml.enabled 为false）

编辑配置文件：config\elasticsearch.yml，添加如下内容：
xpack.ml.enabled: false
```

## Kibana 环境

1. 下载

https://www.elastic.co/cn/downloads/past-releases#kibana

https://www.elastic.co/cn/downloads/past-releases/kibana-6-3-2

2. 启动

> cd software/kibana-6.3.2-darwin-x86_64 

> ./bin/kibana

浏览器访问：http://localhost:5601

3. 使用 DSL 操作

浏览器访问：http://localhost:5601/app/kibana#/dev_tools/console

```
GET _search
{
  "query": {
    "match_all": {}
  }
}
```

## 使用 Kibana 开发工具

```
POST test/_doc/1
{
  "name":"沙漠",
  "age":25
}

PUT test/_doc/1
{
  "name":"沙漠33",
  "age":28,
  "isAdmin":true,
  "height":1.68,
  "address":{
    "a":"1",
    "b":2
  }
}

GET test/_doc/1

GET test/_doc/_search
{
  "query": {
    "match_all": {}
  }
}

GET test/_mapping

GET test/_settings

DELETE test
```
