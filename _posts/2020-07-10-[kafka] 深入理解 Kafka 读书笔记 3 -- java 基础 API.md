---
layout:     post
title:      深入理解 Kafka 读书笔记 3 -- java 基础 API
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---


## 基础 API


```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.5.0</version>
</dependency>
```

```java
package morning.cat.kafka;

import morning.cat.spring.bean.Person;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class KafkaProducerTest {

    public static final String TOPIC = "helloTopic";

    public static void main(String[] args) {
        // Properties properties = new Properties();
        // properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // properties.put("bootstrap.servers", "localhost:9092");
        // KafkaProducer<String, String> producer = new KafkaProducer(properties);
        Map<String, String> map = new HashMap<>();
        map.put("bootstrap.servers", "localhost:9092");
        StringSerializer stringSerializer = new StringSerializer();
        KafkaProducer<String, String> producer = new KafkaProducer(map, stringSerializer, stringSerializer);

        ProducerRecord<String, String> record = new ProducerRecord<>(TOPIC, "my key", "my message");
        producer.send(record);

        producer.close();
    }
}
```

```java
package morning.cat.kafka;

import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;

public class KafkaConsumerTest {
    public static final String TOPIC = "helloTopic";

    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("bootstrap.servers", "localhost:9092");
        map.put("group.id", "no1"); // 分组
        StringDeserializer stringDeserializer = new StringDeserializer();
        KafkaConsumer<String, String> consumer = new KafkaConsumer(map, stringDeserializer, stringDeserializer);
        consumer.subscribe(Collections.singleton(TOPIC));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            records.forEach(record -> System.out.println(record.key() + " " + record.value()));
        }

    }
}
```


