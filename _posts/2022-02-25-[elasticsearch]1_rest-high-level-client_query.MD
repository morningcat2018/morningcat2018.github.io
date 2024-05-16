---
layout:     post
title:      Java elasticsearch-rest-high-level-client 学习笔记1
subtitle:   查询
date:       2022-02-25
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---


## maven GVA

```xml
<!-- https://mvnrepository.com/artifact/org.elasticsearch.client/elasticsearch-rest-high-level-client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>6.3.2</version>
</dependency>

```

## 构建 RestHighLevelClient

```java
    public static RestHighLevelClient getRestHighLevelClient() {
        RestClientBuilder restClientBuilder = RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")
        );

        RestHighLevelClient client = new RestHighLevelClient(restClientBuilder);
        return client;
    }
```

## 查询流程


```java
    RestHighLevelClient restHighLevelClient = getRestHighLevelClient();
    try {
        // 索引名
        String indexName = "blog";

        // 构建查询语句
        QueryBuilder queryBuilder = QueryBuilders.termQuery("title", "安");

        // 构建 SearchSourceBuilder
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.query(queryBuilder); // 设置ES查询条件
        sourceBuilder.from(0); // 设置分页参数
        sourceBuilder.size(5);
        //sourceBuilder.sort("title.keyword", SortOrder.DESC); // 排序
        //sourceBuilder.aggregation(); // 聚合
        System.out.println(sourceBuilder.toString());

        // 构建 searchRequest
        SearchRequest searchRequest = new SearchRequest();
        searchRequest.source(sourceBuilder);
        searchRequest.indices(indexName);

        // 执行请求
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest);

        // 遍历查询结果
        listResult(searchResponse);
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (restHighLevelClient != null) {
            restHighLevelClient.close();
        }
    }
```

遍历结果
```java
    public static void listResult(SearchResponse searchResponse) throws IOException {
        RestStatus status = searchResponse.status(); // ES请求状态
        if (status != RestStatus.OK) {
            System.out.println("未查询出结果");
            return;
        }
        TimeValue took = searchResponse.getTook();
        System.out.println("执行时间" + took);
        //boolean timedOut = searchResponse.isTimedOut(); // 是否超时

        // 获取hits，SearchHits对象包含搜索结果
        SearchHits hits = searchResponse.getHits();
        // 搜索结果总数
        //long numHits = hits.getTotalHits();
        // 遍历搜索结果
        SearchHit[] searchHits = hits.getHits();
        for (SearchHit hit : searchHits) {
            hit.getId();
            hit.getIndex();
            hit.getType();
            hit.getScore();

            // 获取文档内容，json字符串格式
            String sourceAsString = hit.getSourceAsString();
            System.out.println(sourceAsString);
            // 获取文档内容，Map对象格式
            //Map<String, Object> sourceAsMap = hit.getSourceAsMap();
        }
    }
```

## 查询API

#### match

```java
QueryBuilders.matchAllQuery();
QueryBuilders.matchQuery("title", "安");
QueryBuilders.matchPhraseQuery("title", "安徽");
QueryBuilders.matchPhrasePrefixQuery("title", "安徽");
QueryBuilders.multiMatchQuery("安", "title", "name", "desc");
```

#### term

```java
QueryBuilders.termQuery("title", "安");
QueryBuilders.termsQuery("title", "安", "京");
```

#### range

```java
QueryBuilders.rangeQuery("price").gte(100).lte(200);

// 等价SQL: price >= 100 and price <= 200
RangeQueryBuilder rangeQueryBuilder = new RangeQueryBuilder("price");
rangeQueryBuilder.gte(100);
rangeQueryBuilder.lte(200);
```

#### exists

```java
QueryBuilders.existsQuery("title");
```

#### wildcard

```java
QueryBuilders.wildcardQuery("title", "*徽*");
```

#### bool

```java
// 等价SQL： shop_id=100 and status=3 or (price >= 100 and price <= 300)
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery()
        .must(QueryBuilders.termQuery("shop_id", 100)) // 通过must设置term子查询
        .must(QueryBuilders.termQuery("status", 3)) // 通过must设置term子查询
        .should(QueryBuilders.rangeQuery("price").gte(100).lte(300)) // 通过should设置range子查询
        ;
```

随意组装

```java
BoolQueryBuilder boolQueryBuilder2 = QueryBuilders.boolQuery()
        .must(QueryBuilders.rangeQuery("shop_id").gte(100).lte(300))
        .must(QueryBuilders.termQuery("name", "es"))
        .should(QueryBuilders.termQuery("title", "Hello"))
        .should(QueryBuilders.termQuery("desc", "Name"))
        .minimumShouldMatch(1)
        .mustNot(QueryBuilders.termQuery("shop_id", 100))
        .mustNot(QueryBuilders.termQuery("shop_id", 100))
        .filter(QueryBuilders.termQuery("shop_id", 100))
        .filter(boolQueryBuilder);
```

#### constant_score

不考虑得分

```java
QueryBuilder queryBuilder = QueryBuilders.termQuery("title", "安");
ConstantScoreQueryBuilder constantScoreQueryBuilder = QueryBuilders.constantScoreQuery(queryBuilder); // 不计算得分
// {"query":{"constant_score":{"filter":{"term":{"title":{"value":"安","boost":1.0}}},"boost":1.0}}}
```


