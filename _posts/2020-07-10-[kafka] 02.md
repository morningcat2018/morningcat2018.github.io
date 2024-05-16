---
layout:     post
title:      深入理解 Kafka 读书笔记 2 -- 基本概念
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---



## kafka 的功能

1. 消息系统

Kafka 与传统的消息系统（消息中间件）都具备`系统解耦`、`冗余存储`、`流量削峰`、缓冲、`异步通信`、扩展性、可恢复性等功能；

2. 存储系统

Kafka 把消息持久化到磁盘，可以把 Kafka 作为长期的数据存储系统来使用，只需要把对应的数据保留策略设置为永久或启用主题的日志压缩功能；

3. 流式处理平台

为多个流行的流式处理框架提供了可靠的`数据来源`和`流式处理类库`；

## kafka 系统的组成

- 一个 Zookeeper 的集群
    - 元数据管理
    - 控制器的选举
- 若干个 Broker (即 Kafka 服务器)
    - 服务代理节点
    - 将收到的消息存储到磁盘
- 若干个 Producer
    - 创建消息
    - 将消息发送到 Broker
- 若干个 Consumer
    - 采用拉模式（Pull）从 Broker 订阅并消费消息

## 重要概念

1. 主题 Topic

kafka 中的消息以主题为单位进行归类；
生产者将消息发布到特定主题（每一条消息都要指定一个主题）；
消费者订阅主题并消费；


2. 分区 Partition

一个主题可以分为多个分区，一个分区只属于单个主题；（也可以说多个分区构成了一个完整的主题数据）
同一个主题下的不同分区包含的消息是不同的；

分区可以看作是一个`可追加的日志文件`，消息在被追加到分区日志文件上时会分配一个特定的`偏移量（offset）`，offset 是消息在`分区上的唯一标识`；

kafka 通过 offset 来保证消息在分区上的`顺序性`，不过 offset 并不跨分区；（即保证`分区顺序性`，而不是主题顺序性）

同一个主题的不同分区可能分布在不同的 broker 上；（一个主题横跨多个 broker，以此提供更高的性能）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200710161319574.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM4Mzc4MjU=,size_16,color_FFFFFF,t_70)


3. 多副本机制

Kafka 为分区引入了多副本机制（Replica)，提升容灾能力；
同一个分区中的不同副本中保存的是相同的消息（同一时刻，副本中的消息可能不完全相同，与同步效率有关）




