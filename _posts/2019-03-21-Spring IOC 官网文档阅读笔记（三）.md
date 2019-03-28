---
layout:     post
title:      Spring IOC 官网文档阅读笔记（三）
subtitle:   spring 官网文档梳理
date:       2019-03-21
author:     BY morningcat
header-img: img/20190314/HelloSpring.jpg
catalog: true
tags:
    - spring
---

版本：4.3.22.RELEASE

章节：[7. The IoC container](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans)

## 7.5 Bean范围（Scope）

您不仅可以控制要插入到从特定bean定义创建的对象的各种依赖项和配置值，还可以控制从特定bean定义创建的对象的范围。这种方法功能强大且灵活，您可以选择通过配置创建的对象的范围，而不必在Java类级别烘焙对象的范围。可以将Bean定义为部署在多个范围之一中：开箱即用，Spring Framework支持七个范围，其中五个范围仅在您使用Web感知ApplicationContext时才可用。

开箱即用支持以下范围：
- singleton 
（默认）将每个Spring IoC容器的单个bean定义范围限定为单个对象实例。

- prototype 
将单个bean定义范围限定为任意数量的对象实例。

- request
将单个bean定义范围限定为单个HTTP请求的生命周期;也就是说，每个HTTP请求都有自己的bean实例，它是在单个bean定义的后面创建的。仅在Web感知Spring ApplicationContext的上下文中有效。

- session
将单个bean定义范围限定为HTTP会话的生命周期。仅在Web感知Spring ApplicationContext的上下文中有效。

- globalSession
将单个bean定义范围限定为全局HTTP会话的生命周期。通常仅在Portlet上下文中使用时有效。仅在Web感知Spring ApplicationContext的上下文中有效。

- application
将单个bean定义范围限定为ServletContext的生命周期。仅在Web感知Spring ApplicationContext的上下文中有效。

- websocket
将单个bean定义范围限定为WebSocket的生命周期。仅在Web感知Spring ApplicationContext的上下文中有效。

从Spring 3.0开始，线程范围可用，但默认情况下未注册。有关更多信息，请参阅`SimpleThreadScope`的[文档](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-scopes-custom-using)。有关如何注册此范围或任何其他自定义范围的说明，请参阅“使用自定义范围”一节。

---

### 7.5.1 The singleton scope

只管理单个bean的一个共享实例，并且对具有与该bean定义匹配的id或id的bean的所有请求都会导致Spring容器返回一个特定的bean实例。

换句话说，当您定义bean定义并将其作为单一作用域时，Spring IoC容器只创建该bean定义定义的对象的一个​​实例。此单个实例存储在此类单例bean的缓存中，并且该命名Bean的所有后续请求和引用都将返回缓存对象。

![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/singleton.png)

Spring的单例bean概念不同于`Gang of Four（GoF）模式`书中定义的`Singleton模式`。GoF Singleton对对象的范围进行硬编码，使得每个ClassLoader创建一个且只有一个特定类的实例。Spring单例的范围最好按容器和每个bean描述。这意味着如果在单个Spring容器中为特定类定义一个bean，则Spring容器将创建该bean定义所定义的类的一个且仅一个实例。单例范围是Spring中的默认范围。要将bean定义为XML中的单例，您可以编写，例如：
```xml
<bean id="accountService" class="com.foo.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.foo.DefaultAccountService" scope="singleton"/>
```

### 7.5.2 The prototype scope

bean的非单例原型范围部署导致每次发出对该特定bean的请求时都会创建一个新的bean实例。也就是说，bean被注入另一个bean，或者通过对容器的getBean()方法调用来请求它。通常，对所有有状态bean使用`prototype 范围`，对无状态bean使用`singleton 范围`。

下图说明了Spring prototype 范围。数据访问对象（DAO）通常不配置为 prototype，因为典型的DAO不保持任何会话状态

![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/prototype.png)

```xml
<bean id="accountService" class="com.foo.DefaultAccountService" scope="prototype"/>
```

与其他作用域相比，Spring不管理 prototype bean的完整生命周期：容器实例化、配置、组装原型对象，并将其交给客户端，而不再记录该原型实例。因此，尽管无论范围如何都在所有对象上调用初始化生命周期回调方法，但在原型的情况下，不会调用已配置的销毁生命周期回调。客户端代码必须清理原型范围的对象并释放原型bean所持有的昂贵资源。要使Spring容器释放原型范围的bean所拥有的资源，请尝试使用自定义`bean post-processor`，它包含对需要清理的bean的引用。

在某些方面，Spring容器关于原型范围bean的角色是Java new运算符的替代品。超过该点的所有生命周期管理必须由客户端处理。（有关Spring容器中bean的生命周期的详细信息，请参见第7.6.1节“生命周期回调”。）

### 7.5.3 具有prototype-bean依赖关系的单例bean

当您使用具有依赖于 prototype-bean 的单例作用域bean时，请注意在实例化时解析依赖项。因此，如果依赖项将 prototype 范围的bean注入到单例范围的bean中，则会实例化一个新的原型bean，然后将依赖注入到单例bean中。原型实例是唯一提供给单例范围bean的实例。

但是，假设您希望单例范围的bean在运行时重复获取原型范围的bean的新实例。您不能将原型范围的bean依赖注入到您的单例bean中，因为当Spring容器实例化单例bean并解析和注入其依赖项时，该注入只发生一次。如果您需要在运行时多次使用原型bean的新实例，请[参见第7.4.6节“方法注入”](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-method-injection)

### 7.5.4 Request, session, global session, application, and WebSocket scopes

The request, session, globalSession, application, and websocket scopes 仅当您使用Web感知的Spring ApplicationContext实现（例如XmlWebApplicationContext）时才可用。
如果将这些范围与常规的Spring IoC容器（如ClassPathXmlApplicationContext）一起使用，则会抛出IllegalStateException，抱怨未知的bean范围。


#### 初始Web配置

如果您在Spring Web MVC中访问作用域bean，实际上是在Spring DispatcherServlet或DispatcherPortlet处理的请求中，则不需要特殊设置：DispatcherServlet和DispatcherPortlet已经公开了所有相关状态。

如果您使用Servlet 2.5 Web容器，并且在Spring的DispatcherServlet之外处理请求（例如，使用JSF或Struts时），则需要注册`org.springframework.web.context.request.RequestContextListener ``ServletRequestListener`。对于Servlet 3.0+，可以通过`WebApplicationInitializer`接口以编程方式完成。或者，或者对于旧容器，将以下声明添加到Web应用程序的web.xml文件中：
```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者，如果您的侦听器设置存在问题，请考虑使用Spring的`RequestContextFilter`。过滤器映射取决于周围的Web应用程序配置，因此您必须根据需要进行更改。
```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`，`RequestContextListener`和`RequestContextFilter`都完全相同，即将HTTP请求对象绑定到为该请求提供服务的Thread。这使得请求和会话范围的bean可以在调用链中进一步使用。


#### Request scope

文件配置
```xml
<bean id="loginAction" class="com.foo.LoginAction" scope="request"/>
```
使用注解驱动的组件或Java Config时，可以使用`@RequestScope`注解将组件分配给 Request scope
```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

Spring容器通过对每个HTTP请求使用loginAction bean定义来创建LoginAction bean的新实例。也就是说，loginAction bean的范围是HTTP请求级别。您可以根据需要更改创建的实例的内部状态，因为从同一个loginAction bean定义创建的其他实例将不会在状态中看到这些更改;它们特别针对个人要求。当请求完成处理时，将放弃作用于请求的bean。

#### Session scope

```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

```

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

Spring容器通过在单个HTTP会话的生存期内使用userPreferences bean定义来创建UserPreferences bean的新实例。换句话说，userPreferences bean在HTTP会话级别有效地作用域。与请求范围的bean一样，您可以根据需要更改创建的实例的内部状态，因为知道同样使用从同一userPreferences bean定义创建的实例的其他HTTP Session实例在状态中看不到这些更改，因为它们特定于单个HTTP会话。最终丢弃HTTP会话时，也会丢弃作用于该特定HTTP会话的bean。

#### Global session scope

```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="globalSession"/>
```

globalSession范围类似于标准HTTP会话范围（如上所述），并且仅适用于基于portlet的Web应用程序的上下文。portlet规范定义了构成单个portlet Web应用程序的所有portlet之间共享的全局会话的概念。在globalSession作用域中定义的Bean的作用域（或绑定）到全局portlet会话的生存期。

如果编写基于Servlet的标准Web应用程序并将一个或多个bean定义为具有globalSession作用域，则使用标准HTTP会话作用域，并且不会引发错误。

#### Application scope

```xml
<bean id="appPreferences" class="com.foo.AppPreferences" scope="application"/>
```

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

Spring容器通过对整个Web应用程序使用appPreferences bean定义一次来创建AppPreferences bean的新实例。也就是说，appPreferences bean的作用域是ServletContext级别，存储为常规的ServletContext属性。这有点类似于Spring单例bean，但在两个重要方面有所不同：它是每个ServletContext的单例，而不是每个Spring的'ApplicationContext'（在任何给定的Web应用程序中可能有几个），它实际上是暴露的，因此作为ServletContext属性可见。

#### Scoped beans as dependencies

Spring IoC容器不仅管理对象（bean）的实例化，还管理协作者（或依赖关系）的连接。如果要将（例如）HTTP请求作用域bean注入到具有较长寿命范围的另一个bean中，您可以选择注入AOP代理来代替作用域bean。也就是说，您需要注入一个代理对象，该对象公开与范围对象相同的公共接口，但也可以从相关范围（例如HTTP请求）中检索真实目标对象，并将方法调用委托给真实对象。

您还可以在作为单例的作用域的bean之间使用<aop：scoped-proxy />，然后引用通过可序列化的中间代理，从而能够在反序列化时重新获取目标单例bean。
当针对范围原型的bean声明<aop：scoped-proxy />时，共享代理上的每个方法调用都将导致创建一个新的目标实例，然后该调用将被转发到该目标实例。
此外，范围代理不是以生命周期安全的方式从较短范围访问bean的唯一方法。您也可以简单地将您的注入点（即构造函数/ setter参数或自动装配字段）声明为ObjectFactory <MyTargetBean>，允许getObject()调用在每次需要时按需检索当前实例 - 而无需保留实例或单独存储它。
作为扩展变体，您可以声明ObjectProvider <MyTargetBean>，它提供了几个额外的访问变体，包括getIfAvailable和getIfUnique。
JSR-330的变体称为Provider，与Provider <MyTargetBean>声明一起使用，并且对每次检索尝试都使用相应的get()调用。有关JSR-330整体的更多详细信息，请参见此处。

以下示例中的配置只有一行，但了解“为什么”以及它背后的“如何”非常重要。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/>
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.foo.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

要创建这样的代理，可以将子<aop：scoped-proxy />元素插入到作用域bean定义中（请参阅“选择要创建的代理类型”一节和第41章基于XML模式的配置）。为什么在request, session, globalSession和自定义范围级别定义bean的定义需要<aop：scoped-proxy />元素？让我们检查下面的单例bean定义，并将其与您需要为上述范围定义的内容进行对比（请注意，以下userPreferences bean定义不完整）。
```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session"/>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```
在前面的示例中，单例bean userManager注入了对HTTP会话范围的bean userPreferences的引用。这里的重点是userManager bean是一个单例：它将在每个容器中实例化一次，并且它的依赖项（在这种情况下只有一个，userPreferences bean）也只注入一次。这意味着userManager bean将仅在完全相同的userPreferences对象上操作，即最初注入的对象。

当将一个寿命较短的scoped bean注入一个寿命较长的scoped bean时，这不是你想要的行为，例如将一个HTTP Session-scoped合作bean作为依赖注入singleton bean。相反，您需要一个userManager对象，并且在HTTP会话的生命周期中，您需要一个特定于所述HTTP会话的userPreferences对象。因此，容器创建一个对象，该对象公开与UserPreferences类（理想情况下是UserPreferences实例的对象）完全相同的公共接口，该对象可以从作用域机制（HTTP请求，会话等）获取真实的UserPreferences对象。容器将此代理对象注入userManager bean，该bean不知道此UserPreferences引用是代理。在此示例中，当UserManager实例在依赖注入的UserPreferences对象上调用方法时，它实际上是在代理上调用方法。然后，代理从（在这种情况下）HTTP会话中获取真实的UserPreferences对象，并将方法调用委托给检索到的真实UserPreferences对象。

因此，在将request-，session-和globalSession-scoped bean注入协作对象时，您需要以下正确和完整的配置：
```xml
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.foo.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

#### 选择要创建的代理类型

默认情况下，当Spring容器为使用<aop：scoped-proxy />元素标记的bean创建代理时，将创建基于CGLIB的类代理。

CGLIB代理只拦截公共方法调用！不要在这样的代理上调用非公开方法;它们不会被委托给实际的作用域目标对象。

或者，您可以通过为<aop：scoped-proxy />元素的`proxy-target-class属性`的值指定false，将Spring容器配置为为此类作用域bean创建`基于JDK接口的标准代理`。使用基于JDK接口的代理意味着您不需要在应用程序类路径中使用其他库来实现此类代理。但是，这也意味着scoped bean的类必须至少实现一个接口，并且注入scoped bean的所有协作者必须通过其中一个接口引用bean。

有关选择基于类或基于接口的代理的更多详细信息，[请参见第11.6节“代理机制”](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#aop-proxying)。

### 7.5.5自定义范围

#### 创建自定义范围

要将自定义作用域集成到Spring容器中，需要实现`org.springframework.beans.factory.config.Scope接口`，本节将对此进行介绍。

`Scope接口`有四种方法可以从作用域中获取对象，从作用域中删除它们，并允许它们被销毁。

以下方法从基础范围返回对象。例如，会话范围实现返回会话范围的bean（如果它不存在，则该方法在将其绑定到会话以供将来参考之后返回该bean的新实例）。
> Object get(String name, ObjectFactory objectFactory)

以下方法从基础范围中删除对象。例如，会话范围实现从基础会话中删除会话范围的bean。应返回该对象，但如果找不到具有指定名称的对象，则可以返回null。
> Object remove(String name)

以下方法注册范围应在销毁时或在范围中指定的对象被销毁时应执行的回调。有关销毁回调的更多信息，请参阅javadocs或Spring作用域实现。
> void registerDestructionCallback(String name, Runnable destructionCallback)

以下方法获取基础范围的对话标识符。每个范围的标识符都不同。对于会话范围的实现，该标识符可以是会话标识符
> String getConversationId()

#### 使用自定义范围

在编写并测试一个或多个自定义Scope实现之后，您需要让Spring容器知道您的新范围。以下方法是使用Spring容器注册新Scope的核心方法：
> void registerScope(String scopeName, Scope scope);

此方法在`ConfigurableBeanFactory接口`上声明，该接口在通过BeanFactory属性随Spring提供的大多数具体ApplicationContext实现上可用。

> 下面的示例使用Spring附带的`SimpleThreadScope`，但`默认情况下未注册`。对于您自己的自定义Scope实现，过程是相同的。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后创建符合自定义作用域的作用域规则的bean定义：
```xml
<bean id="..." class="..." scope="thread">
```

---

使用自定义Scope实现，您不仅可以使用编程方式注册。还可以使用`CustomScopeConfigurer`类以声明方式执行Scope注册：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="bar" class="x.y.Bar" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="foo" class="x.y.Foo">
        <property name="bar" ref="bar"/>
    </bean>

</beans>
```

> 当您在FactoryBean实现中放置<aop：scoped-proxy />时，它是作用域的工厂bean本身，而不是从getObject())返回的对象。

---

云谁之思？美孟桂矣。
