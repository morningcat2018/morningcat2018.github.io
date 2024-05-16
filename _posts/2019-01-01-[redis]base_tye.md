---
layout:     post
title:      Redis 五种基本类型及操作
subtitle:   
date:       2020-05-11
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Redis
---

- strings : valus is string
- hashes : value is hashmap
- lists : value is 双向链表
- sets : value is 无序集合
- sorted set : value is 有序集合


参考：https://my.oschina.net/mengzhang6/blog/2875820

## string

#### 1. set

> SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]

作用

- 设置键以保存字符串值。
- 如果键已经包含一个值，则无论其类型如何，它都会被`覆盖`。
- 成功进行SET操作后，与密钥关联的任何先前`生存时间`将被丢弃。

参数

- EX seconds -- 设置指定的到期时间, 单位：秒
- PX milliseconds -- 设置指定的到期时间, 单位：毫秒
- NX -- 仅在key不存在的情况下设置密钥
- XX --仅在key存在的情况下覆盖密钥
- KEEPTTL -- 保留与密钥关联的生存时间。

返回值

简单字符串回复：如果SET执行正确，则确定。空答复：如果由于用户指定了 NX 或 XX 选项但不满足条件而未执行 SET 操作，则返回空批量答复。

历史记录

- >= 2.6.12: Added the EX, PX, NX and XX options.
- >= 6.0: Added the KEEPTTL option.

实践

```
127.0.0.1:6379> set name "Www" EX 3
OK
127.0.0.1:6379> get name
"Www"
127.0.0.1:6379> get name
(nil)

127.0.0.1:6379> set name "Hello" NX
(nil)

127.0.0.1:6379> set name "Hello" XX
OK
```

模式

使用命令 `SET resource-name anystring NX EX max-lock-time`

#### 2. get

> GET key

the value of key, or nil when key does not exist.


#### 3. getset

> GETSET key value

GETSET可 与 INCR 一起用于原子复位计数。例如：某个事件每次发生时，进程可能会针对密钥 mycounter 调用INCR，但有时我们需要获取计数器的值并将其自动重置为零。这可以使用GETSET mycounter“ 0”来完成

```
redis> SET mykey "Hello"
"OK"
redis> GETSET mykey "World"
"Hello"
redis> GET mykey
"World"
```

#### 4. mset

> MSET key value [key value ...]

将给定的键设置为其各自的值。与常规SET一样，MSET用新值替换现有值。如果您`不想覆盖`现有值，请参见`MSETNX`。MSET是原子的，因此所有给定的密钥都将立即设置。客户看不到某些键已更新，而其他键未更改。

```
redis> MSET key1 "Hello" key2 "World"
"OK"
redis> GET key1
"Hello"
redis> GET key2
"World"
```

#### 5. mget

> MGET key [key ...]

返回所有指定键的值。对于每个不包含字符串值或不存在的键，将返回特殊值nil。因此，操作永远不会失败。

```
redis> SET key1 "Hello"
"OK"
redis> SET key2 "World"
"OK"
redis> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
```

#### 6. incr

> INCR key

- 将键处存储的数字加1。
- 如果密钥不存在，则在执行操作之前将其设置为0。
- 如果键包含错误类型的值或包含不能表示为整数的字符串，则返回错误。此
- 操作仅限于64位带符号整数。注意：这是字符串操作，因为Redis没有专用的整数类型。存储在密钥处的字符串被解释为以10为基数的64位带符号整数来执行该操作。
- Redis以其整数表示形式存储整数，因此对于实际持有整数的字符串值，存储该整数的字符串表示形式没有开销。

```
127.0.0.1:6379> SET mykey "10"
OK
127.0.0.1:6379> incr mykey
(integer) 11
127.0.0.1:6379> get mykey
"11"
127.0.0.1:6379> incr mykey2
(integer) 1
127.0.0.1:6379> get mykey2
"1"
```


## hash

#### 1. hset

> HSET key field value [field value ...]

将存储在键处的哈希中的字段设置为value。如果密钥不存在，则会创建一个包含哈希的新密钥。如果哈希中已经存在该字段，则将其覆盖。
从Redis 4.0.0开始，HSET是可变的，并且允许多个字段/值对。

```
127.0.0.1:6379> HSET myhash field1 "Hello"
(integer) 1
127.0.0.1:6379> HSET myhash field2 "morning"
(integer) 1
127.0.0.1:6379> HSET myhash field3 "kafka"
(integer) 1
127.0.0.1:6379> HSET myhash field4 "mysql"
(integer) 1
127.0.0.1:6379> HGET myhash field3
"kafka"
```

#### 2. hget

> HGET key field

#### 3. hgetall

> HGETALL key

返回存储在键处的哈希的所有字段和值。在返回的值中，每个字段名后面都跟着它的值，因此答复的长度是哈希值的两倍。

```
127.0.0.1:6379> hgetall myhash
1) "field1"
2) "Hello"
3) "field2"
4) "morning"
5) "field3"
6) "kafka"
7) "field4"
8) "mysql"
```

#### 4. hkeys

> HKEYS key

返回键处存储的哈希中的所有字段名称。

```
127.0.0.1:6379> hkeys myhash
1) "field1"
2) "field2"
3) "field3"
4) "field4"
```

#### 5. hexists

> HEXISTS key field

- 1 if the hash contains field.
- 0 if the hash does not contain field, or key does not exist.

```
127.0.0.1:6379> hexists myhash field2
(integer) 1
127.0.0.1:6379> hexists myhash field22
(integer) 0
```

#### 6. hmset

> HMSET key field value [field value ...]

#### 7. hmget

> HMGET key field [field ...]

```
127.0.0.1:6379> hmset myhash field1 111 field5 555 field6 666
OK
127.0.0.1:6379> hmget myhash field1 field2 field6 field7
1) "111"
2) "morning"
3) "666"
4) (nil)

```

更多：https://redis.io/commands/hmget

## list

#### 1. lpush （队头插入）

- 将所有指定的值插入存储在key的列表的开头。
- 如果键不存在，则在执行推入操作之前将其创建为空列表。
- 当key保留的值不是列表时，将返回错误。
- 只需在命令末尾指定多个参数，就可以使用单个命令调用来推送多个元素。
- 元素从最左边的元素到最右边的元素一个接一个地插入列表的开头。因此，例如命令LPUSH mylist a b c将导致一个包含c作为第一元素，b作为第二元素和a作为第三元素的列表。

```
127.0.0.1:6379> lpush mylist v1
(integer) 1
127.0.0.1:6379> lpush mylist v2 v3 v4 v5
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1
1) "v5"
2) "v4"
3) "v3"
4) "v2"
5) "v1"
```

#### 2. lpop （队头弹出）

> LPOP key

```
127.0.0.1:6379> lpop mylist
"v5"
127.0.0.1:6379> lpop mylist
"v4"
```

#### 3. llen

> LLEN key

#### 4. lrange

> LRANGE key start stop

```
127.0.0.1:6379> llen mylist
(integer) 3
127.0.0.1:6379> lrange mylist 1 2
1) "v2"
2) "v1"
127.0.0.1:6379> lrange mylist 0 2
1) "v3"
2) "v2"
3) "v1"
```

#### 5. rpush 队尾插入

> RPUSH key element [element ...]

#### 6. rpop 队尾弹出

> RPOP key

```
127.0.0.1:6379> rpush mylist a1 a2 a3
(integer) 6
127.0.0.1:6379> lrange mylist 0 -1
1) "v3"
2) "v2"
3) "v1"
4) "a1"
5) "a2"
6) "a3"
127.0.0.1:6379> rpop mylist
"a3"
```

#### 7. rpoplpush

> RPOPLPUSH source destination

```
127.0.0.1:6379> rpoplpush mylist mylist2
"a2"
127.0.0.1:6379> rpoplpush mylist mylist2
"a1"
127.0.0.1:6379> rpoplpush mylist mylist2
"v1"
127.0.0.1:6379> lrange mylist 0 -1
1) "v3"
2) "v2"
127.0.0.1:6379> lrange mylist2 0 -1
1) "v1"
2) "a1"
3) "a2"
```

更多：https://redis.io/commands/rpoplpush

## set 

#### 1. sadd 

> SADD key member [member ...]

```
127.0.0.1:6379> sadd myset s1 s2 s3 s4 s5 
(integer) 5
127.0.0.1:6379> smembers myset
1) "s2"
2) "s1"
3) "s4"
4) "s3"
5) "s5"
```

#### 2. spop

> SPOP key [count]

随机弹出数据节点

```
127.0.0.1:6379> spop myset
"s3"
127.0.0.1:6379> spop myset 2
1) "s5"
2) "s4"
```

#### 3. srandmember

> SRANDMEMBER key [count]

取出数据节点，但不移除集合

```
127.0.0.1:6379> sadd myset s6 s7 s8 s9 
(integer) 4
127.0.0.1:6379> smembers myset
1) "s1"
2) "s2"
3) "s7"
4) "s6"
5) "s9"
6) "s8"
127.0.0.1:6379> srandmember myset
"s2"
127.0.0.1:6379> srandmember myset 3
1) "s1"
2) "s2"
3) "s7"
127.0.0.1:6379> smembers myset
1) "s1"
2) "s2"
3) "s7"
4) "s6"
5) "s9"
6) "s8"
```

更多：https://redis.io/commands/srandmember

## sortset 

有序集合

#### 1. zadd

> ZADD key [NX|XX] [CH] [INCR] score member [score member ...]

https://redis.io/commands/zadd

得分值应为双精度浮点数的字符串表示形式。+inf 和 -inf 值也是有效值。

```
127.0.0.1:6379> zadd myzset 1 java
(integer) 1
127.0.0.1:6379> zadd myzset 1 C++
(integer) 1
127.0.0.1:6379> zadd myzset 8 php
(integer) 1
127.0.0.1:6379> zadd myzset 2 go
(integer) 1
127.0.0.1:6379> zrange myzset 0 -1
1) "C++"
2) "java"
3) "go"
4) "php"
```


XX: 仅更新已经存在的元素。切勿添加元素。
NX: 不要更新现有元素。始终添加新元素。
CH: 修改返回值，从添加的新元素数到更改的元素总数（CH是change的缩写）。更改的元素是添加的新元素以及已为其更新分数的现有元素。因此，命令行中指定的具有与过去相同分数的元素将不计算在内。注意：通常ZADD的返回值仅计算添加的新元素的数量
INCR: 指定此选项后，ZADD的行为类似于ZINCRBY。在此模式下只能指定一对得分元素。



