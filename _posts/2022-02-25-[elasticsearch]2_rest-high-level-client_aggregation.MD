---
layout:     post
title:      Java elasticsearch-rest-high-level-client 学习笔记2
subtitle:   聚合的使用
date:       2022-02-25
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch
---



准备数据来自[elasticsearch系统学习笔记9-聚合分析 Aggregations（补充1）](https://blog.csdn.net/u013837825/article/details/122974702)

## 基本流程

```java
RestHighLevelClient restHighLevelClient = getRestHighLevelClient();
try {
    // 索引名
    String indexName = "cars_transactions";

    // 构建聚合语句
    // 创建terms桶聚合 , 聚合名字 colors_agg , 根据 color 分组
    TermsAggregationBuilder aggregationBuilder = AggregationBuilders.terms("colors_agg").field("color");
    // 嵌套聚合 设置 avg 指标聚合，聚合名字 avg_price, 字段 price，计算平均价格
    aggregationBuilder.subAggregation(AggregationBuilders.avg("avg_price").field("price"));

    // 构建 SearchSourceBuilder
    SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
    sourceBuilder.query(QueryBuilders.matchAllQuery());
    sourceBuilder.size(0);
    sourceBuilder.aggregation(aggregationBuilder); // 聚合

    // 构建 searchRequest
    SearchRequest searchRequest = new SearchRequest();
    searchRequest.source(sourceBuilder);
    searchRequest.indices(indexName);

    // 执行请求
    SearchResponse searchResponse = restHighLevelClient.search(searchRequest);

    // 遍历查询结果
    listAggregations(searchResponse);
} catch (Exception e) {
    e.printStackTrace();
} finally {
    if (restHighLevelClient != null) {
        restHighLevelClient.close();
    }
}
```

```java
public static void listAggregations(SearchResponse searchResponse) {
    // 处理聚合查询结果
    Aggregations aggregations = searchResponse.getAggregations();
    // 根据by_shop名字查询terms聚合结果
    Terms byShopAggregation = aggregations.get("colors_agg");

    // 遍历terms聚合结果
    for (Terms.Bucket bucket : byShopAggregation.getBuckets()) {
        // 因为是根据shop_id分组，因此可以直接将桶的key转换成int类型
        String color = bucket.getKeyAsString();
        long colorCount = bucket.getDocCount();
        System.out.print("颜色 " + color + "\n数量 : " + colorCount);

        // 根据avg_price聚合名字，获取嵌套聚合结果
        Avg avg = bucket.getAggregations().get("avg_price");
        // 获取平均价格
        double avgPrice = avg.getValue();
        System.out.println(" ; 平均价格 : " + avgPrice);
    }
}
```

## 指标

1. 总数和总和
```java
ValueCountAggregationBuilder valueCountAggregationBuilder = AggregationBuilders.count("count_orders").field("order_id");
SumAggregationBuilder sumAggregationBuilder = AggregationBuilders.sum("total_sale").field("price");
// 设置聚合查询,可以设置多个聚合查询条件，只要聚合查询命名不同就行
sourceBuilder.aggregation(valueCountAggregationBuilder);
sourceBuilder.aggregation(sumAggregationBuilder);
// 处理结果
ValueCount valueCount = aggregations.get("count_orders");
Sum totalPrice = aggregations.get("total_sale");
```

2. 最大最小值

```java
MaxAggregationBuilder maxAggregationBuilder = AggregationBuilders.max("max_price").field("price");
MinAggregationBuilder minAggregationBuilder = AggregationBuilders.min("min_price").field("price");
Max maxPrice = aggregations.get("max_price");
Min minPrice = aggregations.get("min_price");         
```

## 桶
1. terms 分桶
```java
TermsAggregationBuilder byShop = AggregationBuilders.terms("by_shop").field("shop_id");
```

2. 等等