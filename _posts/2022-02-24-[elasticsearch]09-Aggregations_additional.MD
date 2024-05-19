---
layout:     post
title:      elasticsearch系统学习笔记9-补充1
subtitle:   聚合分析 Aggregations
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---


[资料来源](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_aggregation_test_drive.html)

## 1. 准备数据
```
PUT /cars_transactions
{
    "mappings": {
        "_doc": {
            "properties": {
                "color": {
                    "type": "keyword"
                },
                "make": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                },
                "price": {
                    "type": "long"
                },
                "sold": {
                    "type": "date"
                }
            }
        }
    }
}
```

```
POST /cars_transactions/_doc/_bulk
{ "index": {}}
{ "price" : 10000, "color" : "red", "make" : "honda", "sold" : "2014-10-28" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 30000, "color" : "green", "make" : "ford", "sold" : "2014-05-18" }
{ "index": {}}
{ "price" : 15000, "color" : "blue", "make" : "toyota", "sold" : "2014-07-02" }
{ "index": {}}
{ "price" : 12000, "color" : "green", "make" : "toyota", "sold" : "2014-08-19" }
{ "index": {}}
{ "price" : 20000, "color" : "red", "make" : "honda", "sold" : "2014-11-05" }
{ "index": {}}
{ "price" : 80000, "color" : "red", "make" : "bmw", "sold" : "2014-01-01" }
{ "index": {}}
{ "price" : 25000, "color" : "blue", "make" : "ford", "sold" : "2014-02-12" }
```

## 2. 基本聚合

```
GET /cars_transactions/_doc/_search
{
    "size" : 0,
    "aggs" : { 
        "popular_colors" : { 
            "terms" : { 
              "field" : "color"
            }
        }
    }
}
```

	
1. 聚合操作被置于顶层参数 `aggs` 之下（如果你愿意，完整形式 `aggregations` 同样有效）
2. 然后，可以为聚合指定一个我们想要名称，本例中是： `popular_colors` ；名字的选择取决于使用者，响应的结果会以我们定义的名字为标签
3. 最后，定义单个桶的类型 `terms` ；这个 terms 桶会为每个碰到的唯一词项动态创建新的桶。 因为我们告诉它使用 color 字段，所以 terms 桶会为每个颜色动态创建新桶
4. 聚合是在特定搜索结果背景下执行的， 这也就是说它只是查询请求的另外一个顶层参数（例如，使用 /_search 端点）。 聚合可以与查询结对
5. 可能会注意到我们将 size 设置成 0 。我们并不关心搜索结果的具体内容，所以将返回记录数设置为 0 来提高查询速度

```
{
  "took": 7,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "popular_colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4
        },
        {
          "key": "blue",
          "doc_count": 2
        },
        {
          "key": "green",
          "doc_count": 2
        }
      ]
    }
  }
}
```

1. 因为我们设置了 size 参数，所以不会有 hits 搜索结果返回。
2. popular_colors 聚合是作为 aggregations 字段的一部分被返回的。
3. 每个桶的 key 都与 color 字段里找到的唯一词对应。它总会包含 doc_count 字段，告诉我们包含该词项的文档数量。
4. 每个桶的数量代表该颜色的文档数量。
5. 前面的这个例子完全是实时执行的：一旦文档可以被搜到，它就能被聚合。这也就意味着我们可以直接将聚合的结果源源不断的传入图形库，然后生成实时的仪表盘


## 3. 添加度量指标

```
GET /cars_transactions/_doc/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": { 
            "avg_price": { 
               "avg": {
                  "field": "price" 
               }
            }
         }
      }
   }
}
```

	
1. 为度量新增 aggs 层。
2. 为度量指定名字： avg_price 。
3. 最后，为 price 字段定义 avg 度量。

```
{
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "avg_price": {
            "value": 32500
          }
        },
        {
          "key": "blue",
          "doc_count": 2,
          "avg_price": {
            "value": 20000
          }
        },
        {
          "key": "green",
          "doc_count": 2,
          "avg_price": {
            "value": 21000
          }
        }
      ]
    }
  }
}
```

##  4. `嵌套桶`
> 在我们使用不同的嵌套方案时，聚合的力量才能真正得以显现
令人激动的分析来自于将桶嵌套进 另外一个桶 所能得到的结果

```
GET /cars_transactions/_doc/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": { 
               "avg": {
                  "field": "price"
               }
            },
            "make_aggs": { 
                "terms": {
                    "field": "make.keyword" 
                }
            }
         }
      }
   }
}
```
	
1. 注意前例中的 avg_price 度量仍然保持原位。
2. 另一个聚合 make_aggs 被加入到了 color 颜色桶中。
3. 这个聚合是 terms 桶，它会为每个汽车制造商生成唯一的桶。
4. `一个聚合的每个 层级 都可以有多个 度量 或 桶`

```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 8,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "colors": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "red",
          "doc_count": 4,
          "make_aggs": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "honda",
                "doc_count": 3
              },
              {
                "key": "bmw",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 32500
          }
        },
        {
          "key": "blue",
          "doc_count": 2,
          "make_aggs": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "ford",
                "doc_count": 1
              },
              {
                "key": "toyota",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 20000
          }
        },
        {
          "key": "green",
          "doc_count": 2,
          "make_aggs": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
              {
                "key": "ford",
                "doc_count": 1
              },
              {
                "key": "toyota",
                "doc_count": 1
              }
            ]
          },
          "avg_price": {
            "value": 21000
          }
        }
      ]
    }
  }
}
```

1. 正如期望的那样，新的聚合嵌入在每个颜色桶中。
2. 现在我们看见按不同制造商分解的每种颜色下车辆信息。
3. 最终，我们看到前例中的 avg_price 度量仍然维持不变。

## 5. 最后的修改

```
GET /cars_transactions/_doc/_search
{
   "size" : 0,
   "aggs": {
      "colors": {
         "terms": {
            "field": "color"
         },
         "aggs": {
            "avg_price": { "avg": { "field": "price" }
            },
            "make" : {
                "terms" : {
                    "field" : "make.keyword"
                },
                "aggs" : { 
                    "min_price" : { "min": { "field": "price"} }, 
                    "max_price" : { "max": { "field": "price"} } 
                }
            }
         }
      }
   }
}
```

## 总结
灵活的嵌套方显`聚合`的强大