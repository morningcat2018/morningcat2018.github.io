---
layout:     post
title:      Neo4j 笔记 之 CQL 基础语法
subtitle:   之 基本节点与关系
date:       2024-03-30
author:     BY morningcat
header-img: img/neo4j/neo4j.png
catalog: true
tags:
    - Neo4j
---


- 关键字不区分大小写


## 创建节点

语法: CREATE (node:label1:label2:...labeln { key1: value, key2: value, ...})

1. 简单节点

> create (n)

> match (n) return n

```json
{
  "identity": 3,
  "labels": [],
  "properties": {

  },
  "elementId": "4:86e2bf13-b7f0-410f-9b0d-17127c944afa:3"
}
{
    //...
}
```

提示: 这里的 n 只是变量的名称,用其他字母或单词替换作用一致

2. 基本节点

> create (n:user {name:"张三",age:26,sex:1})

> match (a:user {name:"张三"}) return a

```json
{
  "identity": 4,
  "labels": [
    "user"
  ],
  "properties": {
    "sex": 1,
    "name": "张三",
    "age": 26
  },
  "elementId": "4:86e2bf13-b7f0-410f-9b0d-17127c944afa:4"
}
```

另一种查询方式

> match (a:user) where a.name="张三" return a

3. 没有标签

> create (n {name:"李四",age:30,sex:1})

> match (n:user) where n.name="李四" return n

使用上面的语句是查询不到结果的,因为它指定了需要在user标签下查询

4. 多标签

> create (n:user:student {name:"王武",age:16,sex:1})

> match (n:user) where n.name="王武" return n

```json
{
  "identity": 7,
  "labels": [
    "student",
    "user"
  ],
  "properties": {
    "sex": 1,
    "name": "王武",
    "age": 16
  },
  "elementId": "4:86e2bf13-b7f0-410f-9b0d-17127c944afa:7"
}
```

5. 多节点

> create (n:student {name:"李晓晓",age:16,sex:0}), (:student {name:"王丽",age:17,sex:0})

查询所有student

> MATCH (n:student) RETURN n 



## 创建关系

语法: CREATE (node1)-[:RelationshipType]->(node2) 

1. 已有节点

```cql
match (a:student),(b:student) where a.name="李晓晓" and b.name="王武"
create (a)-[:boyfriend]->(b)
create (b)-[:gillfriend]->(a)
return a,b
```

注意:此语句不是幂等的,不能重复执行;若需要支持幂等,可以将 create 替换成 merge

```cql
match (a:student),(b:student) where a.name="李晓晓" and b.name="王武"
merge (a)-[:boyfriend]->(b)
merge (b)-[:gillfriend]->(a)
return a,b
```

merge 语句的含义就是当不存在时创建;若已存在,则不再进行创建操作

2. 新建节点

```cql
CREATE (d:student{name: "Shikar"}) 
CREATE (e:student {name: "Inia"})
CREATE (d)-[r:crush]->(e)
return d,e
```

同1所述,使用 merge 替换 create 可以避免重复

```cql
merge (q:student{name: "Midk"}) 
merge (w:student {name: "Inia"})
merge (w)-[r:crush]->(q)
return q,w
```

节点 Inia 已存在,不会重复添加; 节点 Midk 不存在,会自动新建


## 查询节点

1. 查询所有节点

> match (n) return n

2. 查询所有 student 

> match (n:student) return n

3. 按属性查询

> match (a:student {name:"张三"}) return a

> match (a:student) where a.name="张三" return a

> match (a:student) where a.sex is null return a

## 删除节点



## 查询关系

> match p=(:student)-[:crush]->(:student) return p

> match p=(pt:student)-[r:crush]->(up:student) return pt.name,r,up.name

> match (pt:student)-[:crush]->() return pt.name

---

> match (n) where (n)-[:friend]->(:user{name:"nick"}) return n

> match (a:user{name:"nick"})<-[:friend]-(n) return n

上诉含义相同

## 删除关系

> match (a:student{name:"Shikar"})-[r:crush]->(:student) delete r

删除所有 crush 关系

> match (:student)-[r:crush]->(:student) delete r

## set

> match (n:student) where id(n)=8 set n.name="王丽丽" return n

> match (n:student) where n.name="李晓晓" set n.age=18 return n

## 附录

1. 数据类型

- Boolean	布尔文字：真、假
- byte	 8 位整数
- short	 16 位整数
- int	 32 位整数
- long	 64 位整数
- float	 32 位浮点数
- double	 64 位浮点数
- char	 16 位字符
- String	 字符串


2. write 子句

- CREATE	该子句用于创建节点、关系和属性
- MERGE	该子句验证图形中是否存在指定的模式。如果没有，它会创建模式
- SET	该子句用于更新节点上的标签、节点上的属性和关系
- DELETE	该子句用于从图中删除节点和关系或路径等
- REMOVE	此子句用于从节点和关系中删除属性和元素
- FOREACH	此类用于更新列表中的数据
- CREATE UNIQUE	使用子句 CREATE 和 MATCH，您可以通过匹配现有模式并创建缺少的模式来获得唯一模式
- Importing CSV files with Cypher	使用加载 CSV，您可以从 .csv 文件导入数据

3. read 子句

- MATCH	该子句用于搜索具有指定模式的数据
- OPTIONAL MATCH	这与 match 相同，唯一的区别是它可以在缺少模式部分的情况下使用 null
- WHERE	此子句 id 用于向 CQL 查询添加内容
- START	该子句用于通过遗留索引查找起点
- LOAD CSV	该子句用于从 CSV 文件导入数据

- RETURN	此子句用于定义要包含在查询结果集中的内容
- ORDER BY	此子句用于按顺序排列查询的输出。它与从句一起使用RETURN或者WITH
- LIMIT	此子句用于将结果中的行限制为特定值
- SKIP	此子句用于定义从哪一行开始包括输出中的行
- WITH	该子句用于将查询部分链接在一起
- UNWIND	此子句用于将列表扩展为行序列
- UNION	该子句用于组合多个查询的结果
- CALL	该子句用于调用部署在数据库中的过程


4. 布尔运算符

- AND	与
- OR	或
- NOT	非
- XOR	异或

5. 比较运算符

- `=`   “等于”运算符
- `<>`  “不等于”运算符
- `<`   “小于”运算符
- `>`   “大于”运算符
- `<=`  “小于或等于”运算符
- `>=`  “大于或等于”运算符

6. 数学运算符

+, -, *, /, %, ^

7. 其他运算法

- 字符串	+
- 列表	+, IN, [X], [X…..Y]
- 正则表达式	=-
- 字符串匹配	开始于，结束于，约束


8. CQL 函数

- String	它们用于处理字符串文字
- Aggregation	它们用于对 CQL 查询结果执行一些聚合操作
- Relationship	它们用于获取关系的详细信息，例如 startnode、endnode 等

