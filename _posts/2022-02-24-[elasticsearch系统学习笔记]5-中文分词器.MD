---
layout:     post
title:      elasticsearch系统学习笔记5
subtitle:   中文分词器
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch系统学习笔记
---

## IK

https://github.com/medcl/elasticsearch-analysis-ik

- Analyzer: ik_smart , ik_max_word
- Tokenizer: ik_smart , ik_max_word

1. 下载

下载地址 https://github.com/medcl/elasticsearch-analysis-ik/releases

本机下载 elasticsearch-analysis-ik-6.3.2.zip

2. 解压

目录名改为 analysis-ik

// 3. 将 analysis-ik/config 文件夹下的内容移动到  {ES_HOME}/config 目录下

4. 将解压后的目录移动到 {ES_HOME}/plugins 目录下

5. 重启 ES 服务

启动窗口加载日志会多出一些加载插件的信息：[YABPFPe] loaded plugin [analysis-ik]

---

测试：
```json
GET /_analyze
{
  "text": "中华人民共和国国歌",
  "analyzer": "ik_smart"
}

{
  "tokens": [
    {
      "token": "中华人民共和国",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "国歌",
      "start_offset": 7,
      "end_offset": 9,
      "type": "CN_WORD",
      "position": 1
    }
  ]
}
```

```json
GET /_analyze
{
  "text": "中华人民共和国国歌",
  "analyzer": "ik_max_word"
}

{
  "tokens": [
    {
      "token": "中华人民共和国",
      "start_offset": 0,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 0
    },
    {
      "token": "中华人民",
      "start_offset": 0,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 1
    },
    {
      "token": "中华",
      "start_offset": 0,
      "end_offset": 2,
      "type": "CN_WORD",
      "position": 2
    },
    {
      "token": "华人",
      "start_offset": 1,
      "end_offset": 3,
      "type": "CN_WORD",
      "position": 3
    },
    {
      "token": "人民共和国",
      "start_offset": 2,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 4
    },
    {
      "token": "人民",
      "start_offset": 2,
      "end_offset": 4,
      "type": "CN_WORD",
      "position": 5
    },
    {
      "token": "共和国",
      "start_offset": 4,
      "end_offset": 7,
      "type": "CN_WORD",
      "position": 6
    },
    {
      "token": "共和",
      "start_offset": 4,
      "end_offset": 6,
      "type": "CN_WORD",
      "position": 7
    },
    {
      "token": "国",
      "start_offset": 6,
      "end_offset": 7,
      "type": "CN_CHAR",
      "position": 8
    },
    {
      "token": "国歌",
      "start_offset": 7,
      "end_offset": 9,
      "type": "CN_WORD",
      "position": 9
    }
  ]
}
```


## analysis-hanlp

https://github.com/KennFalcon/elasticsearch-analysis-hanlp

提供的分词方式
- hanlp: 默认分词
- hanlp_standard: 标准分词
- hanlp_index: 索引分词
- hanlp_nlp: NLP分词
- hanlp_crf: CRF分词
- hanlp_n_short: N-最短路分词
- hanlp_dijkstra: 最短路分词
- hanlp_speed: 极速词典分词

1. 下载

https://github.com/KennFalcon/elasticsearch-analysis-hanlp/releases/tag/v6.3.2

本机下载 elasticsearch-analysis-hanlp-6.3.2.zip

2. 解压

目录名改为 analysis-hanlp

3. 将 analysis-hanlp/data 文件夹下的内容复制到  {ES_HOME}/data 目录下

3. 将 analysis-hanlp/config 文件夹下的内容复制到  {ES_HOME}/config/analysis-hanlp/ 目录下

4. 将解压后的目录移动到 {ES_HOME}/plugins 目录下

5. 重启 ES 服务

启动窗口加载日志会多出一些加载插件的信息：[YABPFPe] loaded plugin [analysis-hanlp]

---

测试：

```json
GET /_analyze
{
  "text": "中华人民共和国国歌",
  "analyzer": "hanlp"
}

{
  "tokens": [
    {
      "token": "中华人民共和国",
      "start_offset": 0,
      "end_offset": 7,
      "type": "ns",
      "position": 0
    },
    {
      "token": "国歌",
      "start_offset": 0,
      "end_offset": 2,
      "type": "n",
      "position": 1
    }
  ]
}
```

```json
GET /_analyze
{
  "text": "中华人民共和国国歌",
  "analyzer": "hanlp_index"
}
```

## elasticsearch-analysis-pinyin

https://github.com/medcl/elasticsearch-analysis-pinyin

- pinyin


1. 下载

本机下载 elasticsearch-analysis-pinyin-6.3.2.zip

2. 解压

目录名改为 analysis-pinyin

3. 将解压后的目录移动到 {ES_HOME}/plugins 目录下

4. 重启 ES 服务

启动窗口加载日志会多出一些加载插件的信息：[YABPFPe] loaded plugin [analysis-pinyin]

```json
GET /_analyze
{
  "text": "中华民族",
  "analyzer": "pinyin"
}

{
  "tokens": [
    {
      "token": "zhong",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 0
    },
    {
      "token": "hua",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 1
    },
    {
      "token": "min",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 2
    },
    {
      "token": "zu",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 3
    },
    {
      "token": "zhmz",
      "start_offset": 0,
      "end_offset": 0,
      "type": "word",
      "position": 3
    }
  ]
}
```

---

在索引中的用法：

1. 创建一个索引，包含自定义的分词器

```
PUT /index3/ 
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
    }
}
```

2. Test Analyzer, analyzing a chinese name, such as 刘德华

```
GET /index3/_analyze
{
  "text": ["刘德华"],
  "analyzer": "pinyin_analyzer"
}
```

3. Create mapping

```
POST /index3/_mapping 
{
    "properties": {
        "name": {
            "type": "keyword",
            "fields": {
                "pinyin": {
                    "type": "text",
                    "store": false,
                    "term_vector": "with_offsets",
                    "analyzer": "pinyin_analyzer",
                    "boost": 10
                }
            }
        }
    }
}
```

4. Indexing

```
POST /medcl/_create/andy
{"name":"刘德华"}
```

5. Let's search

```
curl http://localhost:9200/medcl/_search?q=name:%E5%88%98%E5%BE%B7%E5%8D%8E
curl http://localhost:9200/medcl/_search?q=name.pinyin:%e5%88%98%e5%be%b7
curl http://localhost:9200/medcl/_search?q=name.pinyin:liu
curl http://localhost:9200/medcl/_search?q=name.pinyin:ldh
curl http://localhost:9200/medcl/_search?q=name.pinyin:de+hua
```