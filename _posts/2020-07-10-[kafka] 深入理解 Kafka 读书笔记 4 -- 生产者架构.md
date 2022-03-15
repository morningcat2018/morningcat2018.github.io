---
layout:     post
title:      深入理解 Kafka 读书笔记 4 -- 生产者架构
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---



## kafka 生产者架构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200713162238318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTM4Mzc4MjU=,size_16,color_FFFFFF,t_70)


- 消息在由 KafkaProducer 发往 Kafka 服务器（Broker or Kafka Cluster）之前，需要经历可能的`拦截器`、`序列化器`、`分区器`等一系列的作用；
- 生产者客户端由两个线程协调运行
- 主线程
    - 由 KafkaProducer 创建消息，然后通过可能的拦截器、序列化器和分区器的作用之后缓存到`消息累加器（ RecordAccumulator ，也称消息收集器）`
    - RecordAccumulator 主要用来`缓存消息`以便 Sender 线程可以批量发送，进而减少网络传输的资源消耗以提升性能
    - RecordAccumulator 缓存的大小可以通过生产者客户端参数 `buffer.memory` 配置，默认值为 32 * 1024 * 1024，即 32M
    - 若生产者发送消息的速度超过 Sender 线程发送到 Kafka 服务器的速度，则会导致 RecordAccumulator 空间不足，这时 KafkaProducer 的 send() 方法的调用要么阻塞要么抛出异常（取决于参数 `max.block.ms` 的设置 ，默认值为 60 * 1000 即 60s，不超过这个时间则阻塞，超过则抛出异常）
    - RecordAccumulator 的内部为每个分区都维护了一个`双端队列`，队列中的节点为 `ProducerBatch`
    - 消息写入缓存时，追加到队列的尾部，Sender 读取消息时，从队列头部开始读取
        - ProducerRecord 是生产者创建的消息
        - ProducerBatch 是一个消息批次，`ProducerBatch 中可以包含一至多个 ProducerRecord`
        - 将多个 ProducerRecord 拼凑成一个 ProducerBatch 可以减少网络请求的次数以提升整体的吞吐量
    - 如果生产者客户端需要向多个分区发送消息，可以将 buffer.memory 参数适当调大以增加整体的吞吐量
- Sender 线程（发送线程）
    - 负责从 RecordAccumulator 获取消息并将其发送到 Kafka 服务器中

