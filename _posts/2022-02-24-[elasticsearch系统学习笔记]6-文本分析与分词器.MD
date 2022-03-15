---
layout:     post
title:      elasticsearch系统学习笔记6
subtitle:   文本分析与分词器
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch系统学习笔记
---


## 概念

- Analysis(文本分析) 是把`全文本`转换成一系列单词（term/token）的过程，也叫分词；
- Analysis(文本分析) 是通过 Analyzer(分词器) 来实现的；
- 可使用内置的分词器，也可以`定制化`分词器；

## Analyzer 的组成

分词器一般由三部分组成（也可以只包含部分组件）：
- Character Filters - 对原始文本进行一些处理操作；比如转换与过滤（去除）
- Tokenizer - 按照一定规则对文本进行切分；
- Token Filters - 将切分后的单词进行一些处理；如转换，增加（同义词...）与删除（结尾词...）


## 内置分词器
- Standard Analyzer - 默认分词器，按词切分，小写处理
- Simple Analyzer - 按照非字母切分（符号被过滤），小写处理
- Stop Analyzer - 小写处理，停用词过滤 the a is
- Whitespace Analyzer - 按照空格切分，不转小写
- Keyword Analyzer - 不分词，直接将输入当作输出
- Patter Analyzer - 正则匹配，默认 \W+ (非字符切分)
- Language - 提供常用语言的分词器

[资料](https://www.bilibili.com/video/BV1vF411v7Qf?p=13&spm_id_from=pageDriver)

---
## 举例一些分词器的构成

1. standard analyzer

- Character Filters
- Tokenizer
    - standard
- Token Filters
    - standard
    - Lower Case （小写转换）
    - Stop (默认关闭)


2. simple analyzer

- Character Filters
- Tokenizer
    - Lower Case
- Token Filters

---

[Language Analyzers](https://www.elastic.co/guide/en/elasticsearch/reference/6.8/analysis-lang-analyzer.html)

3. ICU analyzer

- https://www.elastic.co/guide/en/elasticsearch/plugins/current/analysis-icu.html
- https://icu.unicode.org/

> ./bin/elasticsearch-plugin install analysis-icu

- Character Filters
    - Normalization
- Tokenizer
    - ICU Tokenizer
- Token Filters
    - Normalization
    - Folding
    - Collation
    - Transform

v6.x测试
```
POST /_analyze
{
  "char_filter": [
    "icu_normalizer"
  ],
  "tokenizer": "icu_tokenizer",
  "filter": [
    "icu_normalizer",
    "icu_folding",
    "icu_transform"
  ],
  "text": "他说的确实在理"
}
```

v7.x测试
```
GET /_analyze
{
  "analyzer": "icu_analyzer",
  "text": "他说的确实在理"
}
```

4. THULAC（THU Lexical Analyzer for Chinese）

由清华大学自然语言处理与社会人文计算实验室研制推出的一套中文词法分析工具包，具有中文分词和词性标注功能

https://nlp.csai.tsinghua.edu.cn/project/thulac/



## _analyzer api

1. 指定 Analyzer 进行测试

```
GET /_analyze
{
  "analyzer": "standard",
  "text": "Elasticsearch in Action"
}
```


2. 指定索引的字段上所使用的分词器进行测试

```
POST user/_analyze
{
  "field": "name",
  "text": "Elasticsearch in action"
}
```

3. 定制化分词器进行测试

```
POST /_analyze
{
  "tokenizer": "standard",
  "filter": ["uppercase"],
  "text": "Elasticsearch in action"
}

```

