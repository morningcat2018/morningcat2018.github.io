---
layout:     post
title:      elasticsearch系统学习笔记9
subtitle:   聚合分析 Aggregations
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---



## 概念

- 桶（Buckets）
    - 满足特定条件的文档的集合；（类似 SQL 中的 group by）
- 指标（Metrics）
    - 对桶内的文档进行统计计算；（类似 SQL 中的统计函数 COUNT() 、 SUM() 、 MAX() 等等）


## 分类

聚合分析的功能主要有：
- 指标聚合
- 桶聚合
- 管道聚合
- 矩阵聚合

## 指标聚合

对一组数据进行统计，例如：求最大值、最小值、计算总数、求平均值、求和等等；
类似 SQL 中的 max、min、count、avg、sum 等统计函数；

#### 数据准备

```
PUT /books
{
    "mappings": {
        "_doc": {
            "properties": {
                "name": {
                    "type": "text",
                    "fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                },
                "price": {
                    "type": "float"
                },
                "type": {
                    "type": "text",
                    "fielddata": true,
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
```

```
POST /books/_doc/_bulk
{"index":{"_id":1}}
{"name":"C语言编程","price":23.5,"type":"c"}
{"index":{"_id":2}}
{"name":"数据结构与算法","price":34.5,"type":"ideas"}
{"index":{"_id":3}}
{"name":"计算机组成原理","price":34.5,"type":"Computer"}
{"index":{"_id":4}}
{"name":"计算机网络","price":32.5,"type":"Computer"}
{"index":{"_id":5}}
{"name":"计算机操作系统","price":44.5,"type":"Computer"}
{"index":{"_id":6}}
{"name":"Java 编程","price":13.5,"type":"java"}
{"index":{"_id":7}}
{"name":"数据库原理","price":36.0,"type":"Database"}
{"index":{"_id":8}}
{"name":"ElasticSearch搜索引擎","price":34.8,"type":"search_engine"}
{"index":{"_id":9}}
{"name":"Lucene 原理","price":29.8,"type":"search_engine"}
{"index":{"_id":10}}
{"name":"JVM 技术","price":34.8,"type":"java"}
{"index":{"_id":11}}
{"name":"设计模式","price":27.8,"type":"ideas"}
```

#### max 统计最大值

```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "max": {
        "field": "price"
      }
    }
  }
}

{
  "took": 20,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "value": 44.5
    }
  }
}
```


#### min 统计最小值


```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "min": {
        "field": "price"
      }
    }
  }
}
```

#### value_count 统计文档数量


```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "value_count": {
        "field": "price"
      }
    }
  }
}

```

#### cardinality 基数统计（统计去重后的文档数量）

类似 SQL 中的 select count(distinct price) from books


```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "cardinality": {
        "field": "price"
      }
    }
  }
}

```

#### avg 计算平均值


```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "avg": {
        "field": "price"
      }
    }
  }
}

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
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "value": 31.472726995294746
    }
  }
}
```

#### sum 计算总和


```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "sum": {
        "field": "price"
      }
    }
  }
}

{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "value": 346.1999969482422
    }
  }
}
```

> 这里发现一个小问题，手动计算总和应为 346.2 ；这里为 346.1999969482422 ；猜测应该是 Java 中关于小数二进制保存不准确导致的；

---


#### stats 基本统计

一次性返回总数，最大值，最小值，平均值，总和的结果

```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "stats": {
        "field": "price"
      }
    }
  }
}

{
  "took": 10,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "count": 11,
      "min": 13.5,
      "max": 44.5,
      "avg": 31.472726995294746,
      "sum": 346.1999969482422
    }
  }
}
```

#### extended_stats 高级统计

包含基本统计的结果，另外还会统计：平方和，方差，标准差，平均值加减两个标准差的区间

```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "extended_stats": {
        "field": "price"
      }
    }
  }
}

{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "count": 11,
      "min": 13.5,
      "max": 44.5,
      "avg": 31.472726995294746,
      "sum": 346.1999969482422,
      "sum_of_squares": 11530.459805908205,
      "variance": 57.691074198573254,
      "std_deviation": 7.595464054195323,
      "std_deviation_bounds": {
        "upper": 46.66365510368539,
        "lower": 16.2817988869041
      }
    }
  }
}
```

#### percentiles 百分位统计

`百分位数`是一个统计术语，如果将一组数据从小到大排序，并计算相应的`累计百分数`，`某一百分位所对应数据的值`就称为这一百分位的`百分位数`。

```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "percentiles": {
        "field": "price"
      }
    }
  }
}

{
  "took": 24,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "values": {
        "1.0": 13.500000000000002,
        "5.0": 14,
        "25.0": 28.299999237060547,
        "50.0": 34.5,
        "75.0": 34.79999923706055,
        "95.0": 44.074999999999996,
        "99.0": 44.5
      }
    }
  }
}
```

## 桶聚合

当聚合开始被执行，每个文档里面的值通过计算来决定符合哪个桶的条件。如果匹配到，文档将放入相应的桶并接着进行聚合操作。

#### terms 分组聚合

类似 select count(*) from books group by price

```
GET books/_search
{
  "size": 0, 
  "aggs": {
    "my_result": {
      "terms": {
        "field": "type"
      }
    }
  }
}

{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "computer",
          "doc_count": 3
        },
        {
          "key": "ideas",
          "doc_count": 2
        },
        {
          "key": "java",
          "doc_count": 2
        },
        {
          "key": "search_engine",
          "doc_count": 2
        },
        {
          "key": "c",
          "doc_count": 1
        },
        {
          "key": "database",
          "doc_count": 1
        }
      ]
    }
  }
}
```

> `精彩的来了，桶聚合与指标聚合可以结合使用，更加丰富了聚合分析的功能`

```
GET books/_search
{
  "size": 0,
  "aggs": {
    "my_result": {
      "terms": {
        "field": "type"
      },
      "aggs": {
        "sum_price": {
          "sum": {
            "field": "price"
          }
        },
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}


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
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "computer",
          "doc_count": 3,
          "avg_price": {
            "value": 37.166666666666664
          },
          "sum_price": {
            "value": 111.5
          }
        },
        {
          "key": "ideas",
          "doc_count": 2,
          "avg_price": {
            "value": 31.149999618530273
          },
          "sum_price": {
            "value": 62.29999923706055
          }
        },
        {
          "key": "java",
          "doc_count": 2,
          "avg_price": {
            "value": 24.149999618530273
          },
          "sum_price": {
            "value": 48.29999923706055
          }
        },
        {
          "key": "search_engine",
          "doc_count": 2,
          "avg_price": {
            "value": 32.29999923706055
          },
          "sum_price": {
            "value": 64.5999984741211
          }
        },
        {
          "key": "c",
          "doc_count": 1,
          "avg_price": {
            "value": 23.5
          },
          "sum_price": {
            "value": 23.5
          }
        },
        {
          "key": "database",
          "doc_count": 1,
          "avg_price": {
            "value": 36
          },
          "sum_price": {
            "value": 36
          }
        }
      ]
    }
  }
}
```

#### filter 过滤器聚合

把符合条件的文档放到`一个桶`里进行统计相关指标；


```
GET books/_search
{
  "size": 0,
  "aggs": {
    "my_result": {
      "filter": {
        "match": {
          "name": "java"
        }
      }
    }
  }
}

{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "doc_count": 1
    }
  }
}
```

#### filters 多过滤器聚合

把符合多个过滤器的文档分到不同的桶里进行统计

```
GET books/_search
{
  "size": 0,
  "aggs": {
    "my_result": {
      "filters": {
        "filters": [
          {
            "match": {
              "name": "java"
            }
          },
          {
            "match": {
              "name": "c"
            }
          }
        ]
      }
    }
  }
}


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
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "buckets": [
        {
          "doc_count": 1
        },
        {
          "doc_count": 1
        }
      ]
    }
  }
}
```

#### missing 空值聚合

把索引中的缺失字段的文档分到一个桶里，类似 select count(*) from books where filedA is null

```
GET books/_search
{
  "size": 0,
  "aggs": {
    "my_result": {
      "missing": {
        "field": "price"
      }
    }
  }
}

{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 11,
    "max_score": 0,
    "hits": []
  },
  "aggregations": {
    "my_result": {
      "doc_count": 0
    }
  }
}
```

#### 组合使用案例1

```
GET books/_search
{
  "size": 0,
  "aggs": {
    "missing_result": {
      "missing": {
        "field": "price"
      }
    },
    "sum_result": {
      "sum": {
        "field": "price"
      }
    }
  }
}
```