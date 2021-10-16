---
layout:     post
title:      SpringFramework IOC 之 BeanPostProcessor (一) 简介
subtitle:   Spring bpp 的简介
date:       2019-03-15
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - spring
---




# SpringFramework IOC 之 BeanPostProcessor (一) 简介

## 概念

`BeanPostProcessor接口`定义了您可以实现的回调方法，以提供您自己的（或覆盖容器默认的）实例化逻辑，依赖关系解析逻辑等。如果要在Spring容器完成实例化，配置和初始化bean之后实现某些自定义逻辑，则可以插入一个或多个自定义BeanPostProcessor实现。

您可以配置多个`BeanPostProcessor实例`，并且可以通过设置`order`属性来控制这些BeanPostProcessors的`执行顺序`。仅当BeanPostProcessor实现`Ordered接口`时，才能设置此属性;如果你编写自己的BeanPostProcessor，你也应该考虑实现Ordered接口。

有关更多详细信息，请参阅[BeanPostProcessor和Ordered接口](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-programmatically-registering-beanpostprocessors)的javadoc。另请参阅下面有关BeanPostProcessors编程注册的说明。

- BeanPostProcessors在bean（或对象）实例上运行;也就是说，Spring IoC容器实例化一个bean实例，然后BeanPostProcessors完成它们的工作。
- BeanPostProcessors的范围是每个容器。这仅在您使用容器层次结构时才有意义。如果在一个容器中定义BeanPostProcessor，它将只对该容器中的bean进行后处理。换句话说，在一个容器中定义的bean不会被另一个容器中定义的BeanPostProcessor进行后处理，即使两个容器都是同一层次结构的一部分。
- 要更改实际的bean定义（即定义bean的蓝图），您需要使用BeanFactoryPostProcessor，如第7.8.2节“使用BeanFactoryPostProcessor定制配置元数据”中所述。

`org.springframework.beans.factory.config.BeanPostProcessor`接口由两个回调方法组成。当这样的类被注册为带容器的后处理器时，对于容器创建的每个bean实例，后处理器在容器初始化方法之前从容器中获取回调（例如InitializingBean的afterPropertiesSet()或任何在任何bean初始化回调之后，都会调用声明的init方法。后处理器可以对bean实例执行任何操作，包括完全忽略回调。bean后处理器通常会检查回调接口，或者可以使用代理包装bean。一些Spring AOP基础结构类实现为bean后处理器，以便提供代理包装逻辑。

`ApplicationContext`自动检测在实现`BeanPostProcessor接口`的配置元数据中定义的任何bean。ApplicationContext将这些bean注册为后处理器，以便稍后在创建bean时调用它们。Bean后处理器可以像任何其他bean一样部署在容器中。

请注意，在配置类上使用`@Bean`工厂方法声明`BeanPostProcessor`时，工厂方法的返回类型应该是实现类本身，或者至少是`org.springframework.beans.factory.config.BeanPostProcessor`接口，清楚地表明该bean的后处理器性质。否则，ApplicationContext将无法在完全创建之前按类型自动检测它。由于BeanPostProcessor需要尽早实例化以便应用于上下文中其他bean的初始化，因此这种`早期类型检测`至关重要。

虽然`BeanPostProcessor`注册的推荐方法是通过`ApplicationContext自动检测`（如上所述），但也可以使用`addBeanPostProcessor方法`以编程方式对`ConfigurableBeanFactory`注册它们。当需要在注册之前评估条件逻辑，或者甚至在层次结构中跨上下文复制bean后处理器时，这非常有用。但请注意，以编程方式添加的BeanPostProcessors不遵循Ordered接口。这是注册的顺序，它决定了执行的顺序。另请注意，无论是否有任何显式排序，以编程方式注册的BeanPostProcessors始终在通过自动检测注册的BeanPostProcessors之前处理。

实现`BeanPostProcessor`接口的类是特殊的，容器会对它们进行不同的处理。直接引用的所有BeanPostProcessors和bean都会在启动时实例化，这是ApplicationContext的特殊启动阶段的一部分。接下来，所有BeanPostProcessors都以排序方式注册，并应用于容器中的所有其他bean。因为AOP自动代理是作为BeanPostProcessor本身实现的，所以BeanPostProcessors和它们直接引用的bean都不适合自动代理，因此没有编入方面的方面。对于任何此类bean，您应该看到一条信息性日志消息：“Bean foo不适合所有BeanPostProcessor接口处理（例如：不符合自动代理条件）”。请注意，如果使用自动装配或@Resource将bean连接到BeanPostProcessor（可能会回退到自动装配），Spring可能会在搜索类型匹配依赖项候选项时访问意外的bean，从而使它们不符合自动代理或其他类型的条件豆后处理。例如，如果您有一个使用@Resource注释的依赖项，其中field / setter名称不直接对应于bean的声明名称而且没有使用name属性，那么Spring将访问其他bean以按类型匹配它们。

## 案例

1. 定义一个接口和两个实现类
```java
public interface HelloService {
    void say(String message);
}

public class HelloServiceImpl implements HelloService {
    @Override
    public void say(String message) {
        System.out.println("HelloServiceImpl --- " + message);
    }
}

public class HelloServiceImplTwo implements HelloService {
    @Override
    public void say(String message) {
        System.out.println("HelloServiceImplTwo === " + message);
    }
}
```

2. 自定义`BeanPostProcessor`

```java
public class MyBeanPostProcessor implements BeanPostProcessor, Ordered {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor 初始化之前 --- " + beanName + " - " + bean.toString());
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor 初始化之后 --- " + beanName + " - " + bean.toString());
        return bean;
    }

    @Override
    public int getOrder() {
        return 1;
    }
}

```

3. spring bean配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="morning.cat.bpp.demo1.HelloServiceImpl"></bean>

    <bean id="helloServiceTwo" class="morning.cat.bpp.demo1.HelloServiceImplTwo"></bean>

    <bean class="morning.cat.bpp.demo1.bpp.MyBeanPostProcessor"/>

</beans>
```

4. 测试类

```java
public class Test4 {

    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-ioc.xml");
        HelloService helloService = context.getBean("helloServiceTwo", HelloService.class);
        helloService.say("world");
    }
}

```

测试结果：
```
BeanPostProcessor 初始化之前 --- helloService - morning.cat.bpp.demo1.HelloServiceImpl@26f67b76
BeanPostProcessor 初始化之后 --- helloService - morning.cat.bpp.demo1.HelloServiceImpl@26f67b76
BeanPostProcessor 初始化之前 --- helloServiceTwo - morning.cat.bpp.demo1.HelloServiceImplTwo@153f5a29
BeanPostProcessor 初始化之后 --- helloServiceTwo - morning.cat.bpp.demo1.HelloServiceImplTwo@153f5a29
HelloServiceImplTwo === world
```


## 参考资料

[7.8.1 Customizing beans using a BeanPostProcessor](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-extension-bpp)

---

`打不倒你的，终将使你变得强大。`