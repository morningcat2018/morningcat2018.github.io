---
layout:     post
title:      Mac环境下安装MongoDB环境
subtitle:   包括使用SpringData操作MongoDB数据
date:       2018-07-01
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - MongoDB
---


## 安装教程

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/

## 下载

https://www.mongodb.com/download-center/community?jmp=docs

案例下载地址：https://fastdl.mongodb.org/osx/mongodb-osx-ssl-x86_64-4.0.6.tgz

## 配置

> cd ~/software/mongodb-osx-x86_64-4.0.6/

> vim runMongod
```
./bin/mongod --dbpath /Users/kevin/software/mongodb-osx-x86_64-4.0.6/data 
--logpath /Users/kevin/software/mongodb-osx-x86_64-4.0.6/log/mongo.log 
--noauth --logappend --fork 
--port 27017 &
```

> chmod 744 runMongod

### 启动

> ./runMongod

### 停止

> `ps -ef | grep mongod`

> kill #pid

## 测试

> ./mongo

进入客户端

```
use test

db.student.insert({"sid":"S2012211598","name":"cat","age":25})
```

## Springboot测试

#### 1. 新建SpringBoot工程

#### 2. 添加maven依赖

```
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

#### 3. application.properties

```
spring.data.mongodb.host=localhost
spring.data.mongodb.password=
spring.data.mongodb.port=27017
spring.data.mongodb.database=test
spring.data.mongodb.repositories.type=auto
```

#### 4. 添加`EnableMongoRepositories`注解

```
@SpringBootApplication
@EnableMongoRepositories(basePackages = "morning.cat.springdatademo.mongo.repository")
public class SpringdataDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringdataDemoApplication.class, args);
    }
}

```

#### 5. 定义实体类

```
import lombok.Data;
import lombok.experimental.Accessors;
import javax.persistence.Table;
@Table
@Data
@Accessors(chain = true)
public class Student {
    private String sid;
    private String name;
    private Integer age;
}
```

#### 6. 继承`MongoRepository`类

```
public interface StudentRepository extends MongoRepository<Student, Long> {
    Student findBySid(String code);
}
```

#### 7. 测试类

```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class BaseTests {

    @Autowired
    private StudentRepository studentRepository;

    @Test
    public void contextLoads() {
        studentRepository.insert(new Student().setSid("S2018234234").setName("gouzi").setAge(22));
    }

}
```

#### 8. 使用`MongoChef`客户端查看数据

