---
layout:     post
title:      深入理解 Kafka 读书笔记 7 -- 生产者分区器
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---



背景：[kafka 生产者架构](https://blog.csdn.net/u013837825/article/details/107321959)

## 概念

org.apache.kafka.clients.producer.Partitioner

org.apache.kafka.clients.producer.internals.DefaultPartitioner

