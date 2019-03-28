---
layout:     post
title:      Spring ApplicationEvent 入门实践记录
subtitle:   
date:       2019-03-04
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - spring
---


## Spring ApplicationEvent 入门实践记录
参考文档：https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#context-functionality-events

### 实践过程

1. 事件发布器
发布器类`XxxEventPublisher`实现`ApplicationEventPublisherAware接口`获取`ApplicationEventPublisher`实例；
```java
package morning.cat.springboot.demo.config;

import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.stereotype.Component;

@Component
public class EventPublisherTwo implements ApplicationEventPublisherAware {

    private ApplicationEventPublisher applicationEventPublisher;

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

    /**
     * 自定义发布event的方法
     *
     * @param event
     */
    public void publishEvent(ApplicationEvent event) {
        applicationEventPublisher.publishEvent(event);
    }
}

```

2. 自定义事件
继承`ApplicationEvent`类实现自定义事件
```java
package morning.cat.springboot.demo.event;

import lombok.Getter;
import lombok.Setter;
import org.springframework.context.ApplicationEvent;

public class BaseEvent<T> extends ApplicationEvent {

    @Setter
    @Getter
    private T param;

    public BaseEvent(Object source, T param) {
        super(source);
        this.param = param;
    }

}
```

```java
public class HelloApplicationEvent extends BaseEvent<String> {
    public HelloApplicationEvent(Object source, String t) {
        super(source,t);
    }
}


public class WorldApplicationEvent extends BaseEvent<User> {
    public WorldApplicationEvent(Object source, User t) {
        super(source, t);
    }
}
```

3. 监听事件

第一种方式:实现`ApplicationListener`接口；
```java
package morning.cat.springboot.demo.listener;

import morning.cat.springboot.demo.event.WorldApplicationEvent;
import morning.cat.springboot.demo.entity.User;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class WorldApplicationListener implements ApplicationListener<WorldApplicationEvent> {
    @Override
    public void onApplicationEvent(WorldApplicationEvent event) {
        User user = event.getParam();
        System.out.println(user);
    }
}

```

第二种方式:使用`@EventListener`注解
```java
package morning.cat.springboot.demo.listener;

import morning.cat.springboot.demo.event.HelloApplicationEvent;
import org.springframework.context.event.EventListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Component
public class DemoEventListener {

    /**
     * 监听器
     *
     * @param event
     */
    @Async
    @EventListener
    public void listenerHelloApplicationEvent(HelloApplicationEvent event) {
        String param = event.getParam();
        System.out.println("-------111-------" + param);
    }

}
```

4.测试demo

```java
package morning.cat.springboot.demo.controller;

import morning.cat.springboot.demo.config.EventPublisher;
import morning.cat.springboot.demo.config.EventPublisherTwo;
import morning.cat.springboot.demo.event.HelloApplicationEvent;
import morning.cat.springboot.demo.event.WorldApplicationEvent;
import morning.cat.springboot.demo.entity.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/")
public class HelloController {

    @Autowired
    private EventPublisherTwo eventPublisherTwo;

    @GetMapping("/hello1")
    public String hello1() {
        HelloApplicationEvent event = new HelloApplicationEvent(this, "Hello");
        eventPublisherTwo.publishEvent(event);
        return "Hello";
    }

    @GetMapping("/hello2")
    public String hello2() {
        User user = new User().setId(1L).setName("gouzi");
        WorldApplicationEvent event = new WorldApplicationEvent(this, user);
        eventPublisherTwo.publishEvent(event);
        return "Hello";
    }

   
}

```

[代码下载](https://gitee.com/mengzhang6/springboot-event-demo.git)

---

### 问题记录1：@Async 没有异步调用

解决方案: 使用`@EnableAsync`注解启用异步调用的功能

```java
import com.alibaba.fastjson.JSON;
import com.xxx.cloud.commons.ServiceException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.aop.interceptor.AsyncUncaughtExceptionHandler;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurer;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.lang.reflect.Method;
import java.util.concurrent.Executor;

@Configuration
@EnableAsync
@Slf4j
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(5000);
        executor.setThreadNamePrefix("FansZixun-AsyncThread-");
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new AsyncUncaughtExceptionHandler() {
            @Override
            public void handleUncaughtException(Throwable ex, Method method, Object... params) {
                log.error(
                        "Async method: {} has uncaught exception,params:{}",
                        method.getName(),
                        JSON.toJSONString(params));

                if (ex instanceof ServiceException) {
                    ServiceException exception = (ServiceException) ex;
                    log.error(
                            "service asyncException: code={}, message={}", exception.getStatus(), exception.getStatus());
                } else {
                    log.error("other asyncException: ", ex);
                }
            }
        };
    }
}
```

也可以阅读[这篇博客](https://www.cnblogs.com/wihainan/p/6516858.html)了解一些其他信息。





