---
layout:     post
title:      elasticsearch系统学习笔记3
subtitle:   数据基本操作
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---

## 插入数据

语法：
```
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

例子：
```
PUT /order/_doc/1
{
  "id": 1,
  "shop_id": 1,
  "user_id": 11,
  "create_at": "2019-12-12 12:20:22"
}
```

## 修改数据

更新整个文档语法跟前面介绍的插入文档的语法一模一样，只要ID相同就会直接覆盖之前的文档

虽然说elasticsearch支持通过Api更新文档，但是ES底层文档是不可变的，每次修改文档，本质上是创建一个新的文档，然后把老的文档标记成删除。

## 查询一个文档

语法：
GET /{index}/{type}/{id}

例子：
GET /order/_doc/1

## 删除文档

语法：
DELETE /{index}/{type}/{id}

例子:
DELETE /order/_doc/1


## Query DSL

#### 1. 查询基本语法结构

```
GET /{索引名}/_search
{
	"from" : 0,  // 返回搜索结果的开始位置
    "size" : 10, // 分页大小，一次返回多少数据
    "_source" :[ "field1","field2" ], // 需要返回的字段数组
	"query" : {  }, // query子句
	"aggs" : {  }, // 聚合子句 主要用来编写统计分析语句
	"sort" : {  } // 排序子句 设置排序条件
}
```

{索引名}，支持支持一次搜索多个索引，多个索引使用逗号分隔，例子： GET /order1,order2/_search

按前缀匹配索引名： GET /order*/_search 搜索索引名以order开头的索引


查询语句，返回的JSON数据格式如下
```
{
    "took": 5, // 查询消耗时间，单位毫秒 
    "timed_out": false, // 查询是否超时
    "_shards": { // 本次查询参与的ES分片信息，查询中参与分片的总数，以及这些分片成功了多少个失败了多少个
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": { // hits字段包含我们搜索匹配的结果
        "total": 1, // 匹配到的文档总数
        "max_score": 1.0, // 匹配到的最大分值
        "hits": [
            // 这里就是我们具体的搜索结果，是一个JSON文档数组
        ]
    }
}
```

#### 2.分页

- from 开始位置
- size 当前页大小

```
GET /order/_search
{
  "from": 20,
  "size": 10
}
```

#### 3._source

结果数据只返回 _source 标注的字段值

```
GET /order/_search
{
  "_source": [
    "shop_id",
    "user_id"
  ]
}
```

#### 4.sort

语法：
```
GET /{索引名}/_search
{
  "query": {
    // ...查询条件....
  },
  "sort": [
    {
      "{Field1}": { // 排序字段1
        "order": "desc" // 排序方向，asc或者desc, 升序和降序
      }
    },
    {
      "{Field2}": { // 排序字段2
        "order": "desc" // 排序方向，asc或者desc, 升序和降序
      }
    }
    // ....多个排序字段.....
  ]
}
```

例子:
```
GET /order/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "user_id": {
        "order": "desc"
      }
    },
    {
      "shop_id": {
        "order": "asc"
      }
    }
  ]
}
```

#### 5. 匹配单个字段 match

```
GET /{索引名}/_search
{
    "query": {
        "match": {
            "{FIELD}": "{TEXT}"
        }
    }
}
```

如果 {FIELD} 字段的数据类型是 text 类型，搜索关键词会进行分词处理

#### 6. 精确匹配 term

精确匹配是指等值匹配，不需要进行分词处理，例如：订单号、手机号、时间字段

```
GET /{索引名}/_search
{
    "query": {
        "term": {
            "{FIELD}": "{VALUE}"
        }
    }
}
```

类似： 
```sql
select * from order_v2 
where order_no = "202003131209120999"
```

#### 7. 包含 terms

```
GET /{索引名}/_search
{
    "query": {
        "terms": [
            "{VALUE1}",
            "{VALUE2}"
      ]
    }
}
```

类似： 
```sql
select * from order_v2 
where shop_id in (123,100,300)
```

#### 8. 范围匹配 range

```
GET /{索引名}/_search
{
  "query": {
    "range": {
      "{FIELD}": {
        "gte": 10, 
        "lte": 20
      }
    }
  }
}
```

范围参数如下：
- gt - 大于 （ > ）
- gte - 大于且等于 （ >= ）
- lt - 小于 （ < ）
- lte - 小于且等于 （ <= ）


类似：
```sql
 select * from order_v2 
 where shop_id >= 10 and shop_id <= 20
 ```

#### 9. exists missing

exists 相当于SQL中的 IS NOT NULL
missing 相当于 IS NULL

```
GET /{索引名}/_search
{
    "query": {
        "exists": {
			"field" : "title"
		}
    }
}
```

```sql
select * from table_a 
where title is not null
```

#### 10. multi_match

多值匹配

```
GET /{索引名}/_search
{
    "query": {
        "multi_match": {
			"query" : "团",
			"fields" : ["title" , "desc"]
		}
    }
}
```

```sql
select * from table_a 
where title like "%团%" or desc like "%团%"
```


