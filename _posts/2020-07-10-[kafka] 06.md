---
layout:     post
title:      深入理解 Kafka 读书笔记 6 -- 生产者拦截器
subtitle:   
date:       2020-07-10
author:     BY morningcat
header-img: img/201904/UNADJUSTEDNONRAW_thumb_665.jpg
catalog: true
tags:
    - Kafka
---



背景：[kafka 生产者架构](https://blog.csdn.net/u013837825/article/details/107321959)

## 概念

- 拦截器是 Kafka 0.10.0.0 引入的功能
- 分类
    - 生产者拦截器
    - 消费者拦截器
- 功能
    - 可以用来在消息发送前做一些准备工作
        - 修改消息、完善消息
        - 过滤不需要发送的消息
    - 可以用来在发送回调逻辑前做一些定制化的需求
        - 统计类工作
- 实现 org.apache.kafka.clients.producer.ProducerInterceptor<K,V> 接口
    - onSend
        - KafkaProducer 在将消息序列化和计算分区之前会调用生产者拦截器的 onSend() 方法来对消息进行相应的定制化操作
    - onAcknowledgement
        - KafkaProducer 会在消息被应答之前或消息发送失败时调用生产者拦截器的 onAcknowledgement() 方法，优先于用户设定的 Callback 之前执行


示例

```java
package morning.cat.kafka.client.interceptor;

import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.header.Headers;

import java.util.Map;

/**
 * @describe: {@link Person} 的生产者拦截器实现
 * @author: morningcat.zhang
 * @date: 2020/7/13 4:25 PM
 * @Version 1.0
 */
public class PersonProducerInterceptor implements ProducerInterceptor<String, Person> {

    /**
     * KafkaProducer 在将消息序列化和计算分区之前会调用生产者拦截器的 onSend() 方法来对消息进行相应的定制化操作
     */
    @Override
    public ProducerRecord<String, Person> onSend(ProducerRecord<String, Person> record) {
        String topic = record.topic(); // 主题
        Integer partition = record.partition(); // 分区
        Long timestamp = record.timestamp(); // 时间戳
        String key = record.key();
        Person value = record.value();
        Headers headers = record.headers(); //

        // 定制化需求
        value.setName("Interceptor-" + value.getName());

        return new ProducerRecord<>(topic, partition, timestamp, key, value, headers);
    }

    /**
     * KafkaProducer 会在消息被应答之前或消息发送失败时调用生产者拦截器的 onAcknowledgement() 方法，优先于用户设定的 Callback 之前执行
     * 此方法运行在 KafkaProducer 的 I/O 线程中
     *
     * @param metadata
     * @param exception
     */
    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        if (exception != null) {

            String topic = metadata.topic(); // 主题
            Integer partition = metadata.partition(); // 分区
            Long offset = metadata.offset(); // 偏移量
            String exceptionMsg = exception.getMessage();
            System.out.println(String.format("%s:%s:%s:%s:%s", "PersonProducerInterceptor:onAcknowledgement", topic, partition, offset, exceptionMsg));
            // logger.error
        }
    }

    /**
     * 关闭拦截器时执行一些资源清理工作
     */
    @Override
    public void close() {
        System.out.println("ProducerInterceptor.close");
    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```

```java
package morning.cat.kafka.client.kryo_test;

import morning.cat.kafka.client.domain.Person;
import morning.cat.kafka.client.interceptor.EmptyProducerInterceptor;
import morning.cat.kafka.client.interceptor.PersonProducerInterceptor;
import morning.cat.kafka.client.serialize.PersonUseKryoSerializer;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Arrays;
import java.util.Properties;
import java.util.stream.Collectors;

public class ProducerTest {

    public static final String TOPIC = "helloTopic5";

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 配置序列化器
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, PersonUseKryoSerializer.class.getName());
        // 配置生产者拦截器
        // 如果配置多个拦截器，则按照顺序依次执行
        properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, Arrays.asList(EmptyProducerInterceptor.class, PersonProducerInterceptor.class).stream().map(t -> t.getName()).collect(Collectors.joining(",")));
        KafkaProducer<String, Person> producer = new KafkaProducer(properties);

        Person person = new Person();
        person.setId(333L);
        person.setName("zhangsan");
        person.setPersonNo("P00046JJ99" + System.currentTimeMillis());
        ProducerRecord<String, Person> record = new ProducerRecord<>(TOPIC, person.getPersonNo(), person);
        producer.send(record);

        producer.close();
    }
}
```

```java
package morning.cat.kafka.client.kryo_test;

import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class ConsumerTest {
    public static final String TOPIC = "helloTopic5";

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put("value.deserializer", "morning.cat.kafka.client.serialize.PersonUseKryoDeserializer");
        properties.put("bootstrap.servers", "localhost:9092");
        properties.put("group.id", "no2"); // 分组
        KafkaConsumer<String, Person> consumer = new KafkaConsumer(properties);
        consumer.subscribe(Collections.singleton(TOPIC));

        while (true) {
            try {
                ConsumerRecords<String, Person> records = consumer.poll(Duration.ofMillis(1000));
                if (!records.isEmpty()) {
                    records.forEach(record -> {
                        String key = record.key();
                        System.out.println(key);
                        Person person = record.value();
                        System.out.println(person);
                    });
                    System.out.println("============");
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```


