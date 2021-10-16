---
layout:     post
title:      分布式任务调度 xxl-job 入门实践记录
subtitle:   入门实践记录
date:       2019-03-04
author:     BY morningcat
header-img: img/20190304/post-bg-unix-linux.jpg
catalog: true
tags:
    - java框架
---

## xxl-job 入门实践记录

### 下载安装包

[下载地址](https://github.com/xuxueli/xxl-job/releases)

或：git clone https://github.com/xuxueli/xxl-job.git

### MySQL数据库

新建数据库及相关表结构：
执行`xxl-job/doc/db/tables_xxl_job.sql`数据库脚本；

### 修改配置

修改`/xxl-job-admin/src/main/resources/xxl-job-admin.properties`脚本中关于数据库的配置：
```
xxl.job.db.driverClass=com.mysql.jdbc.Driver
xxl.job.db.url=jdbc:mysql://localhost:3306/xxl-job?useUnicode=true&characterEncoding=UTF-8
xxl.job.db.user=root
xxl.job.db.password=
```

### 运行xxl-job-admin服务

1. 打成war包然后放到tomcat容器中执行；

> mvn clean package -U

2. 若是调试，则可以直接在IDEA中配置一下tomcat，然后直接运行

启动完成后在浏览器输入：`http://localhost:8080/xxl-job-admin/toLogin`
默认账户：admin/123456
可在`/xxl-job-admin/src/main/resources/xxl-job-admin.properties`脚本中进行修改配置；


### 任务调度测试

选用`/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot`demo作为客户端测试demo;

1. 新建`JobHandler`任务
```java
package com.xxl.job.executor.service.jobhandler;

import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.handler.IJobHandler;
import com.xxl.job.core.handler.annotation.JobHandler;
import com.xxl.job.core.log.XxlJobLogger;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.concurrent.TimeUnit;


/**
 * 任务Handler示例（Bean模式）
 * <p>
 * 开发步骤：
 * 1、继承"IJobHandler"：“com.xxl.job.core.handler.IJobHandler”；
 * 2、注册到Spring容器：添加“@Component”注解，被Spring容器扫描为Bean实例；
 * 3、注册到执行器工厂：添加“@JobHandler(value="自定义jobhandler名称")”注解，注解value值对应的是调度中心新建任务的JobHandler属性的值。
 * 4、执行日志：需要通过 "XxlJobLogger.log" 打印执行日志；
 */
@JobHandler(value = "helloJobHandler")
@Component
public class HelloJobHandler extends IJobHandler {

    @Override
    public ReturnT<String> execute(String param) {
        String now = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
        System.out.println(now + "XXL-JOB, Hello World.");
        return SUCCESS;
    }

}

```

2. 配置

配置脚本`/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/src/main/resources/application.properties`
```
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
```

3. 构建打包

> cd /xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/

> mvn clean package -U

> cd target

4. 启动多个客户端服务

> java -jar xxl-job-executor-sample-springboot-1.9.2.jar --server.port=8901 --xxl.job.executor.port=8801

> java -jar xxl-job-executor-sample-springboot-1.9.2.jar --server.port=8902 --xxl.job.executor.port=8802

5. 在后台管理页面配置任务

- 执行器

执行器管理 -> 新建执行器

- 任务

任务管理 -> 新建任务

选择任务管理器 : 
路由策略 : 
Cron : cron表达式，如 */5 * * * * ?
运行模式 : 
JobHandler : @JobHandler(value = "helloJobHandler") 中配置的value
阻塞处理策略 : 

以上配置参考文档：[官方文档](http://www.xuxueli.com/xxl-job/#/?id=%E9%85%8D%E7%BD%AE%E5%B1%9E%E6%80%A7%E8%AF%A6%E7%BB%86%E8%AF%B4%E6%98%8E%EF%BC%9A)

点击`执行`

## 参考文档

- [http://www.xuxueli.com/xxl-job/#/](http://www.xuxueli.com/xxl-job/#/)

- [https://github.com/xuxueli/xxl-job/](https://github.com/xuxueli/xxl-job/)

- [https://github.com/xuxueli/xxl-job/blob/master/doc/XXL-JOB%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3.md](https://github.com/xuxueli/xxl-job/blob/master/doc/XXL-JOB%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3.md)