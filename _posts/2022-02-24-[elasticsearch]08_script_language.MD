---
layout:     post
title:      elasticsearch系统学习笔记8
subtitle:   脚本语言的简单认识
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---


1. 字段设置成 fielddata 为 true

```
PUT /test
{
  "mappings": {
    "_doc": {
      "properties": {
        "age": {
          "type": "integer"
        },
        "name": {
          "type": "text",
          "fielddata":true
        }
      }
    }
  }
}
```

2. 插入数据

```
POST test/_doc/
{
  "name":"apple",
  "age":25
}
```

自动生成id

2. 脚本查询

```
GET test/_doc/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "a_filed_name": {
      "script": {
        "lang": "painless",
        "source": "doc['name'].value+'_hello'"
      }
    }
  }
}
```

```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "test",
        "_type": "_doc",
        "_id": "KwWH5H4BCls4bvt4nUfz",
        "_score": 1,
        "fields": {
          "a_filed_name": [
            "apple_hello"
          ]
        }
      }
    ]
  }
}
```