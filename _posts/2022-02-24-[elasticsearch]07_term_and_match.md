---
layout:     post
title:      elasticsearch系统学习笔记7
subtitle:   理解 term 与 match 的使用区别
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---


## 数据准备

```
POST /blog/_doc/_bulk
{"index":{"_id":1}}
{"title":"安徽合肥"}
{"index":{"_id":2}}
{"title":"安徽宣城"}
{"index":{"_id":3}}
{"title":"江苏南京"}
{"index":{"_id":4}}
{"title":"山东肥城"}
{"index":{"_id":5}}
{"title":"陕西西安"}
```

## term 

team 为精确查询

会将需要查询的值完整的去到索引中进行匹配查找

---

#### 案例1

```
GET blog/_doc/_search
{"query":{"term":{"title":"安"}}}

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
    "total": 3,
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "2",
        "_score": 0.6931472,
        "_source": {
          "title": "安徽宣城"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "5",
        "_score": 0.2876821,
        "_source": {
          "title": "陕西西安"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "安徽合肥"
        }
      }
    ]
  }
}
```


因为 `安徽宣城`，`陕西西安`，`安徽合肥` 这些文本在进行文本分析时，都有`安`出现，所以根据倒排索引能查到这三条数据；

```
POST blog/_analyze
{
  "field": "title",
  "text": "安徽合肥"
}

{
  "tokens": [
    {
      "token": "安",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<IDEOGRAPHIC>",
      "position": 0
    },
    {
      "token": "徽",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<IDEOGRAPHIC>",
      "position": 1
    },
    {
      "token": "合",
      "start_offset": 2,
      "end_offset": 3,
      "type": "<IDEOGRAPHIC>",
      "position": 2
    },
    {
      "token": "肥",
      "start_offset": 3,
      "end_offset": 4,
      "type": "<IDEOGRAPHIC>",
      "position": 3
    }
  ]
}
```

#### 案例2

```
GET blog/_doc/_search
{"query":{"term":{"title":"安徽合肥"}}}

{
  "took": 6,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

查看结果会发现一条都没有查询到，有点反常识；但如果结合案例1大概就能明白，文本 `安徽合肥`在分词创建倒排索引时，并没有保留一个完整的文本作为索引，而是四个单独的字作为索引；
而 `team` 为精确查询，所以根据`安徽合肥`到倒排索引中并不会搜索到数据；


> 知识拓展
如果结合[elasticsearch系统学习笔记5-中文分词器](https://blog.csdn.net/u013837825/article/details/122738216)中的知识，
在创建 blog索引 时将 title 字段指定为`ik_max_word`

```
GET /_analyze
{
  "text": "安徽合肥",
  "analyzer": "ik_max_word"
}

{
  "tokens": [
    {
      "token": "安徽",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "合肥",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 1
    }
  ]
}

文本 `安徽合肥`在分词创建倒排索引时，将会被分词成 "安徽" 和 "合肥"
```

#### 案例3

```
GET blog/_doc/_search
{"query":{"term":{"title.keyword":"安徽合肥"}}}

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
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "title": "安徽合肥"
        }
      }
    ]
  }
}
```

在字段名后加上个 `.keyword` 就能查询到，这是因为创建索引 blog 时字段 title 有一个`子属性`就是 keyword ；

```
GET /blog/_mapping 

{
  "blog": {
    "mappings": {
      "_doc": {
        "properties": {
          "title": {
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

## match

https://www.elastic.co/guide/en/elasticsearch/reference/6.3/query-dsl-match-query.html

#### 案例1

```
GET blog/_doc/_search
{"query":{"match":{"title":"安"}}}

{
  "took": 12,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 4,
    "max_score": 1.3862944,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "2",
        "_score": 1.3862944,
        "_source": {
          "title": "安徽宣城"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.1507283,
        "_source": {
          "title": "安徽合肥"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "4",
        "_score": 0.6931472,
        "_source": {
          "title": "山东肥城"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "5",
        "_score": 0.2876821,
        "_source": {
          "title": "陕西西安"
        }
      }
    ]
  }
}
```

如果根据单个字或词来看，term 和 match 并没有什么区别

#### 案例2

```
GET blog/_doc/_search
{"query":{"match":{"title":"安徽合肥"}}}

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
    "total": 4,
    "max_score": 1.3862944,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "2",
        "_score": 1.3862944,
        "_source": {
          "title": "安徽宣城"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.1507283,
        "_source": {
          "title": "安徽合肥"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "4",
        "_score": 0.6931472,
        "_source": {
          "title": "山东肥城"
        }
      },
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "5",
        "_score": 0.2876821,
        "_source": {
          "title": "陕西西安"
        }
      }
    ]
  }
}
```

如果是多个词或字的查询，就与 term 不太一样了，这是因为 match 在查询之前会将 `安徽合肥` 进行分词 ，按照`安`,`徽`,`合`,`肥`分别查询并将结果汇总；

不信你可以使用下面的语句再查询一遍，结果还是一样的
```
GET blog/_doc/_search
{"query":{"match":{"title":"安肥合徽"}}}
```

---

作用类似于
```
GET blog/_doc/_search
{
  "query": {
    "terms": {
      "title": ["安","徽","合","肥"]
    }
  }
}
```

或者
```
GET blog/_doc/_search
{
  "query": {
    "bool": {
      "should": [
        {"match":{"title":"安"}},
        {"match":{"title":"徽"}},
        {"match":{"title":"合"}},
        {"match":{"title":"肥"}}
        ]
    }
  }
}
```

#### 案例3

```
GET blog/_doc/_search
{
  "query": {
    "match": {
      "title": {
        "query": "安徽合肥",
        "operator": "and"
      }
    }
  }
}

{
  "took": 37,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.1507283,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.1507283,
        "_source": {
          "title": "安徽合肥"
        }
      }
    ]
  }
}
```

增加 operator 查询条件，代表一个逻辑语句，这里表示需要同时满足`安`,`徽`,`合`,`肥`这四个条件才可以；

其实只要一个文本里包含这四个字的文档，都会被检索出来；

类似于：
```
GET blog/_doc/_search
{
  "query": {
    "bool": {
      "must": [
        {"match":{"title":"安"}},
        {"match":{"title":"徽"}},
        {"match":{"title":"合"}},
        {"match":{"title":"肥"}}
        ]
    }
  }
}
```


#### 案例4

```
GET blog/_doc/_search
{"query":{"match_phrase":{"title":"安徽合肥"}}}

{
  "took": 16,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1.1507283,
    "hits": [
      {
        "_index": "blog",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.1507283,
        "_source": {
          "title": "安徽合肥"
        }
      }
    ]
  }
}
```

match_phrase 短语匹配

用于全文搜索，当你希望寻找邻近的单词时，match_phrase 查询可以满足要求；

也可以通过配置 slop 指定 match_phrase 查询词条能够相隔多远时仍然将文档视为匹配；（0 是默认值）
```
GET blog/_doc/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "安徽合肥",
        "slop": 0
      }
    }
  }
}
```
