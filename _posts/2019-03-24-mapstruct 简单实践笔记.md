---
layout:     post
title:      mapstruct 简单实践笔记
subtitle:   
date:       2019-03-24
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java工具
---


# mapstruct 简单实践笔记

## 简介

Java bean映射，简单的方法！

`MapStruct`是一个代码生成器，极大地简化了基于配置的约定的Java bean类型之间映射的实现。
生成的映射代码使用简单的方法调用，因此速度快、类型安全且易于理解。

## 文档

http://mapstruct.org/

http://mapstruct.org/documentation/dev/reference/html/

https://github.com/mapstruct/mapstruct-examples

## 实践

1. 依赖

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-jdk8</artifactId>
    <version>${org.mapstruct.version}</version>
</dependency>

<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>${org.mapstruct.version}</version>
</dependency>
```

2. 定义实体类

User.java
```java
@Data
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class User {

    private Long id;

    private String name;

    private Integer age;

    private Date birthday;

    private LocalDate localDate;
}
```

UserVo.java
```java
@Data
@Builder
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class UserVo {

    private Long id;

    private String name;

    private Integer age;
}
```

3. 定义UserMapper类

格式为:
```java
@Mapper
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    UserVo toUserVo(User user);
}
```

4. 测试
```java
@Test
public void test1() {
    User user = User.builder().id(100L).name("gozi").age(19).birthday(new Date()).build();
    UserVo vo = UserMapper.INSTANCE.toUserVo(user);
    System.out.println(vo);
}
```

5. 时间转换

三种方式：
```java
    @Mapping(source = "birthday", target = "birthday", dateFormat = "yyyy.MM.dd")
    @Mapping(target = "day", expression = "java(morning.cat.util.DateFormat.formatDate(user.getBirthday()))")
    @Mapping(target = "day2", expression = "java(org.apache.commons.lang3.time.DateFormatUtils.format(user.getBirthday(),\"yyyy-MM-dd HH:mm:ss\"))")
    UserVo2 toUserVo2(User user);

    default UserVo2 toUserVo3(User user) {
        return UserVo2.builder().id(user.getId()).birthday(new SimpleDateFormat("yyyy-MM-dd").format(user.getBirthday())).build();
    }
```

第一种：使用 @Mapping 注解的 dateFormat 属性
第二种：使用 @Mapping 注解的 expression 属性（借助外部的静态方法）
第三种：在接口内部使用 default 实现一个默认方法（较为麻烦）

6. 忽略字段
```java
@Mapping(target = "id", ignore = true)
```

7. 字段映射

源实体类与目标实体类字段名不一致时；

```java
@Mapping(source = "name", target = "nameFor")
```

8. List映射

```java

List<UserVo> toUserVos(List<User> user);
```

## 解析

使用`mapstruct框架`，编写使用`@Mapper`注解`UserMapper接口类`后，在编译后会自动生成接口实现类`target/generated-sources/annotations/morning/cat/mapstruct/UserMapperImpl.java`；

以下内容是`mapstruct框架`根据配置自动生成的代码：
```java
package morning.cat.mapstruct;

import java.text.SimpleDateFormat;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.List;
import javax.annotation.Generated;
import morning.cat.entity.User;
import morning.cat.vo.UserVo;
import morning.cat.vo.UserVo.UserVoBuilder;
import morning.cat.vo.UserVo2;
import morning.cat.vo.UserVo2.UserVo2Builder;

@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2019-03-24T14:17:03+0800",
    comments = "version: 1.3.0.Final, compiler: javac, environment: Java 1.8.0_181 (Oracle Corporation)"
)
public class UserMapperImpl implements UserMapper {

    @Override
    public UserVo toUserVo(User user) {
        if ( user == null ) {
            return null;
        }

        UserVoBuilder userVo = UserVo.builder();

        userVo.id( user.getId() );
        userVo.name( user.getName() );
        userVo.age( user.getAge() );

        return userVo.build();
    }

    @Override
    public List<UserVo> toUserVos(List<User> user) {
        if ( user == null ) {
            return null;
        }

        List<UserVo> list = new ArrayList<UserVo>( user.size() );
        for ( User user1 : user ) {
            list.add( toUserVo( user1 ) );
        }

        return list;
    }

    @Override
    public UserVo2 toUserVo2(User user) {
        if ( user == null ) {
            return null;
        }

        UserVo2Builder userVo2 = UserVo2.builder();

        if ( user.getBirthday() != null ) {
            userVo2.birthday( new SimpleDateFormat( "yyyy.MM.dd" ).format( user.getBirthday() ) );
        }
        if ( user.getLocalDate() != null ) {
            userVo2.localDate( DateTimeFormatter.ofPattern( "yyyy.MM.dd" ).format( user.getLocalDate() ) );
        }
        userVo2.nameFor( user.getName() );
        userVo2.age( user.getAge() );

        userVo2.day2( org.apache.commons.lang3.time.DateFormatUtils.format(user.getBirthday(),"yyyy-MM-dd HH:mm:ss") );
        userVo2.day( morning.cat.util.DateFormat.formatDate(user.getBirthday()) );

        return userVo2.build();
    }
}

```

本文源码：git@github.com:morningcat2018/mapstruct-sample.git

## 与Spring配合使用

定义：设置`@Mapper`注解的`componentModel`值为`spring`；
即`UserMapper`及其实现交给Spring容器管理；
```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    UserVo toUserVo(User user);
}
```

使用：直接使用`@Autowired`进行自动注入；

```java
@Autowired
UserMapper userMapper;
```

注意：这种实践需要在Spring环境下使用；

## 更多内容

请参考官网文档：http://mapstruct.org/documentation/dev/reference/html/

网上资料：https://zhuanlan.zhihu.com/p/38103512

