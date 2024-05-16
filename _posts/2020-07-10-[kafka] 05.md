---
layout:     post
title:      深入理解 Kafka 读书笔记 5 -- 序列化器
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

- 生产者需要用序列化器把对象转换成字节数组才能通过网络发送给 Kafka 服务器
- 消费者需要用反序列化器把从 Kafka 服务器中收到的字节数组转换成相应的对象
- 序列化器与反序列化器需要`一一对应`
- 序列化器需要实现 org.apache.kafka.common.serialization.Serializer 接口
- 反序列化器需要实现 org.apache.kafka.common.serialization.Deserializer 接口
- kafka-clients 客户端提供了几种序列化器
    - String StringSerializer StringDeserializer
    - ByteArray 
    - ByteBuffer
    - Bytes
    - Double
    - Float
    - Integer
    - Long
    - UUID
    - Void

---

Serializer 接口有3个主要方法：

```java
void configure(Map<String, ?> configs, boolean isKey) // 配置当前类
byte[] serialize(String topic, T data); // 执行序列化操作
void close() // 关闭序列化器，如果实现此方法则必须确保此方法的幂等性，因为这个方法可能会被KafkaProducer调用多次
```

如果 Kafka 客户端提供的几种序列化器都无法满足需求，则可以选择使用如：Avro、JSON、Thrift、ProtoBuf、Protostuff、Fst、Kryo、Gson、fastjson等通用的序列化工具来实现；

## 示例： 使用 Kryo 序列化框架

```java
package morning.cat.kafka.client.serialize;

import com.esotericsoftware.kryo.Kryo;
import com.esotericsoftware.kryo.io.Input;
import com.esotericsoftware.kryo.io.Output;

import java.io.ByteArrayOutputStream;

/**
 * @describe: TODO 类描述信息
 * @author: morningcat.zhang
 * @date: 2020/7/13 3:10 PM
 * @Version 1.0
 */
public class KryoUtils {
    private static final ThreadLocal<Kryo> kryos = ThreadLocal.withInitial(() -> {
        Kryo kryo = new Kryo();
        kryo.setRegistrationRequired(false);//关闭注册行为
        kryo.setReferences(false);//不支持循环引用
        return kryo;
    });


    public static <T> byte[] serizaber(T t) {
        Kryo kryo = kryos.get();

        try (Output output = new Output(new ByteArrayOutputStream());) {
            kryo.writeObject(output, t);
            return output.getBuffer();
        } finally {
            kryos.remove();
        }
    }

    public static <T> T deserialize(byte[] data, Class<T> tClass) {
        Kryo kryo = kryos.get();

        try (Input input = new Input(data);) {
            T object = kryo.readObject(input, tClass);
            return object;
        } finally {
            kryos.remove();
        }
    }
}
```

```java
package morning.cat.kafka.client.domain;

import lombok.Data;
import java.io.Serializable;
import java.time.LocalDate;

@Data
public class Person implements Serializable {

    private Long id;

    private String personNo;

    private String name;

    private LocalDate birthday;
}
```

```java
package morning.cat.kafka.client.serialize;

import com.esotericsoftware.kryo.Kryo;
import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.common.serialization.Serializer;

/**
 * @describe: {@link Person} 的序列化工具类{@link Kryo}
 * @author: morningcat.zhang
 * @date: 2020/7/13 11:17 AM
 * @Version 1.0
 */
public class PersonUseKryoSerializer implements Serializer<Person> {

    @Override
    public byte[] serialize(String topic, Person data) {
        return KryoUtils.serizaber(data);
    }
}
```

```java
package morning.cat.kafka.client.serialize;

import com.esotericsoftware.kryo.Kryo;
import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.common.serialization.Deserializer;

/**
 * @describe: {@link Person} 的反序列化工具类{@link Kryo}
 * @author: morningcat.zhang
 * @date: 2020/7/13 11:21 AM
 * @Version 1.0
 */
public class PersonUseKryoDeserializer implements Deserializer<Person> {

    @Override
    public Person deserialize(String topic, byte[] data) {
        return KryoUtils.deserialize(data, Person.class);
    }
}
```

## 示例： 自定义序列化实现

```java
package morning.cat.kafka.client.serialize;

import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.common.serialization.Serializer;

import java.nio.ByteBuffer;
import java.nio.charset.Charset;

/**
 * @describe: {@link Person} 的自定义序列化实现
 * @author: morningcat.zhang
 * @date: 2020/7/14 11:47 AM
 * @Version 1.0
 */
public class PersonSerializer implements Serializer<Person> {
    @Override
    public byte[] serialize(String topic, Person data) {
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

        byteBuffer.putLong(data.getId());
        byteBuffer.putInt(data.getPersonNo().length());
        byteBuffer.put(data.getPersonNo().getBytes());

        byte[] bytes = data.getName().getBytes(Charset.forName("UTF-8"));
        byteBuffer.putInt(bytes.length);
        byteBuffer.put(bytes);
        return byteBuffer.array();
    }
}
```


```java
package morning.cat.kafka.client.serialize;

import morning.cat.kafka.client.domain.Person;
import org.apache.kafka.common.serialization.Deserializer;

import java.nio.ByteBuffer;
import java.nio.charset.Charset;

/**
 * @describe: {@link Person} 的自定义反序列化实现
 * @author: morningcat.zhang
 * @date: 2020/7/14 1:57 PM
 * @Version 1.0
 */
public class PersonDeserializer implements Deserializer<Person> {
    @Override
    public Person deserialize(String topic, byte[] data) {
        Person person = new Person();
        ByteBuffer byteBuffer = ByteBuffer.allocate(data.length);
        byteBuffer.put(data);
        byteBuffer.flip();
        person.setId(byteBuffer.getLong());

        int l1 = byteBuffer.getInt();
        byte[] personNoBytes = new byte[l1];
        byteBuffer.get(personNoBytes, 0, l1);
        person.setPersonNo(new String(personNoBytes, 0, l1));

        int l2 = byteBuffer.getInt();
        byte[] nameBytes = new byte[l2];
        byteBuffer.get(nameBytes, 0, l2);
        person.setName(new String(nameBytes, 0, l2, Charset.forName("UTF-8")));

        return person;
    }
}
```