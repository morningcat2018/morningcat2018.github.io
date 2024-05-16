---
layout:     post
title:      elasticsearch系统学习笔记4
subtitle:   bool查询
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch系统学习笔记
---

## 布尔查询

在 SQL 中，我们需要` and ， or 和括号`来组合查询条件，在 ES 中使用 `bool 查询`可用做到同样的效果；

布尔查询对应 Lucene 的 BooleanQuery ；基于一个或多个布尔子句的使用，每个自居都有一类事件：
- must --- 必须出现在匹配文档中，并且会影响匹配得分；
- filter --- 必须出现在匹配文档中，不计算匹配得分（所以效率高于 must）；
- should --- 应该出现在匹配文档中；
    - 在当前布尔查询中，如果没有 must 或 filter 子句，文档则最少匹配一个 should 子句；
    - `如果有 must 或 filter 子句，文档则可以不匹配的任意一个 should 子句，当然匹配 should 子句会提高得分`(待定)；
    - 应该匹配的 should 子句的最小数量可以通过 `minimum_should_match` 参数进行设置；
- must_not --- 必须不出现在匹配文档中；



## bool 语法结构

```
GET /{索引名}/_search
{
  "query": {
    "bool": { // bool查询
      "must": [], // must条件，类似SQL中的and, 代表必须匹配条件
      "must_not": [], // must_not条件，跟must相反，必须不匹配条件
      "should": [],  // should条件，类似SQL中or, 代表匹配其中一个条件
      "filter": [] // 不计算得分
    }
  }
}
```

#### must

```
GET /order_v2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "order_no": "202003131209120999"
                    }
                },
                {
                    "term": {
                        "shop_id": 123
                    }
                }
            ]
        }
    }
}
```

```sql
select * from order_v2 
where order_no = "202003131209120999" 
and shop_id = 123
```

#### must_not

```
GET /order_v2/_search
{
    "query": {
        "bool": {
            "must_not": [
                {
                    "term": {
                        "shop_id": 1
                    }
                },
                {
                    "term": {
                        "shop_id": 2
                    }
                }
            ]
        }
    }
}
```

```sql
select * from order_v2 
where shop_id != 1 and shop_id != 2
```

#### should

```
GET /order_v2/_search{
    "query": {
        "bool": {
            "should": [
                {
                    "term": {
                        "order_no": "001"
                    }
                },
                {
                    "term": {
                        "order_no": "002"
                    }
                }
            ]
        }
    }
}
```

> select * from order_v2 where order_no = "001" or order_no = "002"

---

```
GET /order_v2/_search
{
    "query": {
        "bool": {
            "should": [
                {
                    "bool": {
                        "must": [
                            {
                                "term": {
                                    "order_no": "003"
                                }
                            },
                            {
                                "term": {
                                    "shop_id": 333
                                }
                            }
                        ]
                    }
                },
                {
                    "terms": {
                        "tag": [
                            1,
                            2
                        ]
                    }
                }
            ]
        }
    }
}
```

> select * from order_v2 where (order_no = '003' and shop_id = 333) or tag in (1,2)

#### minimum_should_match

```
GET /order_v2/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "shop_id": 20220101
                    }
                }
            ],
            "should": [
                {
                    "term": {
                        "order_no": "001"
                    }
                },
                {
                    "term": {
                        "order_no": "002"
                    }
                }
            ],
            "minimum_should_match" : 1,
            "boost" : 1.0
        }
    }
}
```


## 组合案例1

要做到这个，我们只要将 match 查询变更为 match_phrase 查询即可:

```
GET /index_name/_doc_/_search
{
    "query": {
        "bool": {
            "must": {
                "term": {
                    "cityCode": 001
                }
            },
            "should": [
                {
                    "term": {
                        "houseId": 1087599828743
                    }
                },
                {
                    "match": {
                        "resblockName": "团"
                    }
                }
            ]
        }
    }
}
```

> SELECT * FROM table_a WHERE  (cityCode = 001) AND (houseId = 1087599828743 OR resblockName like "%团%")

