---
layout:     post
title:      深入理解 Kafka 读书笔记 1 -- 环境搭建与脚本测试
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---


## 安装 JDK

...

## Zookeeper 环境

前提：
- JDK 已安装成功

[Zookeeper 环境搭建笔记](https://blog.csdn.net/u013837825/article/details/97393992)

## Kafka 环境

前提：
- JDK 已安装成功
- Zookeeper 已启动

[下载](http://kafka.apache.org/downloads)

解压

> tar zxvf kafka_2.12-2.5.0.tgz

配置

> vi conf/server.properties

```
# 主要配置项
broker.id=0

# 对外提供服务的入口地址
listeners=PLAINTEXT://localhost:9092

# 每个主题的默认分片数量
# num.partitions=1

log.dirs=/tmp/kafka-logs

zookeeper.connect=localhost:2181/kafka
```

启动

> bin/kafka-server-start.sh config/server.properties

后台启动

> bin/kafka-server-start.sh config/server.properties &

验证服务

> jps -l

35844 kafka.Kafka

出现 kafka.Kafka 表示启动成功，并不代表服务能对外提供服务

## 测试

1. 创建主题

> bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic helloTopic --replication-factor 1 --partitions 4

Created topic helloTopic.

---

--partitions 分区数
--replication-factor 副本因子
--topic 指定主题名称

创建了一个名称为 helloTopic 的主题，这个主题有 4 个分区，每个分区都是只有自身一个副本

---

> bin/kafka-topics.sh --zookeeper localhost:2181/kafka --list

2. 生产者消费者测试脚本

消费者

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic helloTopic

---

生产者

> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic helloTopic

出现 > 后输入一些字符串，消费者端会输出同样的字符串；


