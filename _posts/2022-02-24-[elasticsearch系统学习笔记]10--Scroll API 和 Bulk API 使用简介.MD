---
layout:     post
title:      elasticsearch系统学习笔记10
subtitle:   Scroll API 和 Bulk API 使用简介
date:       2022-02-24
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_643.jpg
catalog: true
tags:
    - elasticsearch系统学习笔记
---



## Scroll API

[Search APIs->Request Body Search->Scroll](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/search-request-scroll.html)

While a search request returns a single “page” of results, 
the scroll API can be used to retrieve large numbers of results (or even all results) 
from a single search request, 
in much the same way as you would use a cursor on a traditional database.


#### scroll 使用案例

使用[elasticsearch系统学习笔记9-聚合分析 Aggregations](https://blog.csdn.net/u013837825/article/details/122904298)中的数据：books 索引

```
POST /books/_search?scroll=1m
{
  "size": 3,
  "query": {
    "match_all": {}
  }
}

```

- 1m 表示1分钟， scroll=1m ：tells Elasticsearch to keep the search context open for another 1m.
- size 每次 scroll 查询返回的数据数量，包含首次查询

```
{
  "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAA2FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAANxZZQUJQRlBlblEyU2pQYVFSUnNRZUJnAAAAAAAAADgWWUFCUEZQZW5RMlNqUGFRUlJzUWVCZwAAAAAAAAA5FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAAOhZZQUJQRlBlblEyU2pQYVFSUnNRZUJn",
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
    "max_score": 1,
    "hits": [
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "5",
        "_score": 1,
        "_source": {
          "name": "计算机操作系统",
          "price": 44.5,
          "type": "Computer"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "8",
        "_score": 1,
        "_source": {
          "name": "ElasticSearch搜索引擎",
          "price": 34.8,
          "type": "search_engine"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "9",
        "_score": 1,
        "_source": {
          "name": "Lucene 原理",
          "price": 29.8,
          "type": "search_engine"
        }
      }
    ]
  }
}
```

- _scroll_id 下面的迭代请求会一直用到
- 会返回首次的结果数据

```
POST /_search/scroll 
{
    "scroll" : "1m", 
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAA2FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAANxZZQUJQRlBlblEyU2pQYVFSUnNRZUJnAAAAAAAAADgWWUFCUEZQZW5RMlNqUGFRUlJzUWVCZwAAAAAAAAA5FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAAOhZZQUJQRlBlblEyU2pQYVFSUnNRZUJn" 
}
```

- scroll 继续保持连接一分钟，超过这个时间再使用 _search/scroll 会报错
- scroll_id 即 上面的 _scroll_id 的结果

```
{
  "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAA2FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAANxZZQUJQRlBlblEyU2pQYVFSUnNRZUJnAAAAAAAAADgWWUFCUEZQZW5RMlNqUGFRUlJzUWVCZwAAAAAAAAA5FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAAOhZZQUJQRlBlblEyU2pQYVFSUnNRZUJn",
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
    "max_score": 1,
    "hits": [
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "10",
        "_score": 1,
        "_source": {
          "name": "JVM 技术",
          "price": 34.8,
          "type": "java"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "2",
        "_score": 1,
        "_source": {
          "name": "数据结构与算法",
          "price": 34.5,
          "type": "ideas"
        }
      },
      {
        "_index": "books",
        "_type": "_doc",
        "_id": "4",
        "_score": 1,
        "_source": {
          "name": "计算机网络",
          "price": 32.5,
          "type": "Computer"
        }
      }
    ]
  }
}
```

- 就这样每次请求都会按照最初给定的 size 返回数据
- 当请求到最后没有数据可返回时，hits 会响应空数组

```
{
  "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAA2FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAANxZZQUJQRlBlblEyU2pQYVFSUnNRZUJnAAAAAAAAADgWWUFCUEZQZW5RMlNqUGFRUlJzUWVCZwAAAAAAAAA5FllBQlBGUGVuUTJTalBhUVJSc1FlQmcAAAAAAAAAOhZZQUJQRlBlblEyU2pQYVFSUnNRZUJn",
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
    "max_score": 1,
    "hits": []
  }
}
```



#### Clear scroll API

scroll 超时时，搜索上下文将自动删除。
但是，保持 scroll 打开是有代价的，
因此一旦不再使用 scroll ，就应使用 clear scroll API 明确清除 scroll ：

```
DELETE /_search/scroll
{
    "scroll_id" : "xxx"
}

// 也支持批量删除
DELETE /_search/scroll
{
    "scroll_id" : [
      "xxx",
      "yyy"
    ]
}

// 
DELETE /_search/scroll/_all
```


#### Java api for Scrol

```java
package es.demo;

import org.elasticsearch.action.search.ClearScrollRequest;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchScrollRequest;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.TimeValue;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.builder.SearchSourceBuilder;

import java.io.IOException;
import java.util.Arrays;
import java.util.Objects;

public class ScrollDemo {

    public static void main(String[] args) throws IOException {
        RestHighLevelClient restHighLevelClient = EsConfig.getRestHighLevelClient();

        try {
            SearchSourceBuilder builder = new SearchSourceBuilder();
            builder.query(QueryBuilders.matchAllQuery());
            builder.size(3);
            SearchRequest searchRequest = new SearchRequest("books");
            searchRequest.source(builder);
            searchRequest.scroll(TimeValue.timeValueMillis(1));
            SearchResponse searchResponse = restHighLevelClient.search(searchRequest);
            Arrays.asList(searchResponse.getHits().getHits()).forEach(hit -> System.out.println(hit.getSourceAsString()));
            String scrollId = searchResponse.getScrollId();
            int i = 0;
            while (true) {
                System.out.println("--->" + (++i));
                SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId);
                scrollRequest.scroll(TimeValue.timeValueMillis(1));
                SearchResponse searchResponse2 = restHighLevelClient.searchScroll(scrollRequest);
                if (Objects.equals(searchResponse2.getHits().getHits().length, 0)) {
                    ClearScrollRequest clearScrollRequest = new ClearScrollRequest();
                    clearScrollRequest.addScrollId(scrollId);
                    restHighLevelClient.clearScroll(clearScrollRequest);
                    break;
                }
                Arrays.asList(searchResponse2.getHits().getHits()).forEach(hit -> System.out.println(hit.getSourceAsString()));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            restHighLevelClient.close();
        }
    }
}

```


---

## Bulk API

可以结合 Bulk API 使用，进行批量插入或更新操作；

[Bulk API 文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/docs-bulk.html)

#### 批量插入

```
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "field1" : "value2" }

POST /test/_doc/_bulk
{ "index" : { "_id" : "1" } }
{ "field1" : "value1" }
{ "index" : { "_id" : "2" } }
{ "field1" : "value2" }

// 自动生成id
POST /test/_doc/_bulk
{ "index" : {} }
{ "field1" : "value1" }
{ "index" : {} }
{ "field1" : "value2" }
```

#### 批量混合操作

```
POST _bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : { "_index" : "test", "_type" : "_doc", "_id" : "1"} }
{ "doc" : {"field2" : "value2"} }
```

- The possible actions are `index`, `create`, `delete` and `update`.
- `index` 和 `create` 都是创建文档，但在`具有相同索引和类型的文档时`存在差异
    - `index` 不存在则创建，存在则更新
    - `create` 不存在则创建，存在则创建失败
- `delete` 删除
- `update` 更新

#### Java api for bulk

```java
package es.demo;

import com.alibaba.fastjson.JSON;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

public class BulkDemo {

    public static void main(String[] args) {
        List<IndexRequest> indexRequests = new ArrayList<>();
        String index = "bulk_test";
        String type = "_doc";
        for (int i = 0; i < 10; i++) {
            IndexRequest indexRequest = new IndexRequest(index, type, String.valueOf(i));
            Node node = Node.builder().name("name" + i).length(i * 14L).publish(new Date()).build();
            indexRequest.source(JSON.toJSONString(node), XContentType.JSON); // alibaba.fastjson
            indexRequests.add(indexRequest);
        }

        try (RestHighLevelClient restHighLevelClient = EsConfig.getRestHighLevelClient()) {
            BulkRequest bulkRequest = new BulkRequest();
            indexRequests.forEach(t -> bulkRequest.add(t));
            restHighLevelClient.bulk(bulkRequest);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Data
    @Builder
    @AllArgsConstructor
    @NoArgsConstructor
    public static class Node {
        private String name;
        private Long length;
        private Date publish;
    }
}

```