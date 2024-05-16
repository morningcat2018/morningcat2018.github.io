---
layout:     post
title:      elasticsearch系统学习笔记2
subtitle:   简介
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---

## 简介

- Elasticsearch 是一个分布式、RESTful 风格的`搜索和数据分析引擎`
- 使用java 开发
- 底层基于 Lucene 全文检索引擎工具包

## 基本概念

1. index (索引)

`文档`数据的集合

2. type (文档类型)

在 Elasticsearch 老的版本中文档类型，代表一类文档的集合；
在 Elasticsearch7.0 以后的版本，已经废弃文档类型这个概念；

3. document (文档)

- Elasticsearch 是`面向文档的数据库`，文档是最基本的存储单元
- 文档使用` json 格式`存储
- 文档中的任何` json 字段`都可以作为查询条件
- 文档的 json 格式没有严格限制，可以随意增加、减少字段，甚至每一个文档的格式都不一样
    - 虽然文档的格式没有限制，可以随便存储任意格式数据，但是实际业务中不会这么干，通常一个索引只会存储格式相同的数据

4. Field (文档字段)

文档由多个字段（Field）组成

基本的字段类型有：
- text 分词处理，支持全文搜索
- keyword 不支持全文搜索；作为一个整体进行匹配，不需要分词处理
- long 数值类型
- float 浮点型
- boolean 布尔型
- date 日期类型

## 文档元数据

指的是插入JSON文档的时候系统为当前数据`自动生成的系统字段`，元数据的字段名都是以下划线开头的

常见的元数据如下：
- _index 索引名字
- _type 文档类型，虽然新版ES废弃了type的用法，但是元数据还是可以看到。
- _id  文档唯一 Id, 如果我们没有为文档指定 id，系统会自动生成
- _version 文档的版本号，每修改一次文档数据，字段就会加1， 这个字段新版的ES已经不使用了
- _seq_no  文档的版本号, 替代老的 _version 字段
- _primary_term  文档所在主分区，这个可以跟 _seq_no 字段搭配实现乐观锁
- _source 数据主体内容

```
// es6.3.2 文档
{
  "_index": "test",
  "_type": "_doc",
  "_id": "1",
  "_version": 11,
  "found": true,
  "_source": {
    "name": "沙漠33",
    "age": 28,
    "isAdmin": true,
    "height": 1.68,
    "address": {
      "a": "1",
      "b": 2
    }
  }
}
```

## 映射 (mapping)

类似 MySQL 数据库中的表结构定义 （schema） ，描述了文档包含哪些字段 、每个字段的数据类型是什么。

1. 查询映射规则
> GET test/_mapping

```
{
  "test": {
    "mappings": {
      "_doc": {
        "properties": {
          "address": {
            "properties": {
              "a": {
                "type": "text",
                "fields": {
                  "keyword": {
                    "type": "keyword",
                    "ignore_above": 256
                  }
                }
              },
              "b": {
                "type": "long"
              }
            }
          },
          "age": {
            "type": "long"
          },
          "height": {
            "type": "float"
          },
          "isAdmin": {
            "type": "boolean"
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```


2. 自动映射

当我们没有预先定义文档的映射，在插入数据时系统会自动检测我们插入的数据类型，相当于自动定义文档类型(mapping)；

```
POST test2/_doc/1
{
  "name": "zhang san",
  "age": 19
}

result:
{
  "_index": "test2",
  "_type": "_doc",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}

GET test2/_mapping

result:
{
  "test2": {
    "mappings": {
      "_doc": {
        "properties": {
          "age": {
            "type": "long"
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}

GET test2/_doc/1

result:
{
  "_index": "test2",
  "_type": "_doc",
  "_id": "1",
  "_version": 1,
  "found": true,
  "_source": {
    "name": "zhang san",
    "age": 19
  }
}

```

自动映射的缺点就是会出现ES映射的数据类型，不是我们想要的类型；

实际项目中，我们通常会预先定义好ES的映射规则，也就是类似MYSQL一样，提前定义好表结构。

3. 自定义文档的数据类型

创建ES索引

```
PUT /order
{
  "mappings": {
    "_doc": {
      "properties": {
        "id": {
          "type": "integer"
        },
        "shop_id": {
          "type": "integer"
        },
        "user_id": {
          "type": "integer"
        },
        "order_no": {
          "type": "keyword"
        },
        "create_at": {
          "type": "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        },
        "phone": {
          "type": "keyword"
        },
        "address": {
          "type": "text"
        }
      }
    }
  }
}
```

