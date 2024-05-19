---
layout:     post
title:      Maven 依赖中的 scope 标签含义
subtitle:   
date:       2020-12-21
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - java
---

可参考[官网文档](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#dependency-scope)


大致有 complie(缺省值),test,provided,runtime,system,import  等选项；

## complie

1. 示例

```xml
    <dependency>
      <groupId>com.mycompany.app</groupId>
      <artifactId>my-app</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>compile</scope>
    </dependency>
```
等价于
```xml
    <dependency>
      <groupId>com.mycompany.app</groupId>
      <artifactId>my-app</artifactId>
      <version>1.0-SNAPSHOT</version>
    </dependency>
```

2. 作用范围

build -> test -> runtime -> package



## test

1. 示例

```xml
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
```

2. 作用范围

test

仅在测试阶段起作用


## provided

1. 示例

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
    <scope>provided</scope>
</dependency>
```

2. 作用范围

build -> test 

3. 说明

runtime 阶段并不输出依赖关系，而是由第三方提供相应依赖；
例如 web war 包都不包括 servlet-api.jar，而是由 servlet 容器来提供。

## runtime

1. 示例

```xml
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
			<scope>runtime</scope>
		</dependency>
        <dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
			<scope>runtime</scope>
		</dependency>
```

2. 作用范围

test -> runtime

3. 说明

被依赖项目无需参与项目的编译，不过后期的测试和运行周期需要其参与

## system

1. 示例

```xml
    <dependency>
      <groupId>javax.sql</groupId>
      <artifactId>jdbc-stdext</artifactId>
      <version>2.0</version>
      <scope>system</scope>
      <systemPath>${java.home}/lib/rt.jar</systemPath>
    </dependency>
```

2. 作用

引入本地jar


## import

1. 示例

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>maven</groupId>
            <artifactId>A</artifactId>
            <version>1.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
  </dependencyManagement>
```