---
layout:     post
title:      Spring IOC 官网文档阅读笔记（一）
subtitle:   spring 官网文档梳理
date:       2019-03-14
author:     BY morningcat
header-img: img/20190314/HelloSpring.jpg
catalog: true
tags:
    - spring
---

版本：4.3.22.RELEASE

章节：[7. The IoC container](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans)

## 7.1 Spring IoC 容器和 bean简介

`控制反转（IoC）`是一个过程，通过这个过程，对象定义它们的依赖关系，然后容器在创建bean时注入这些依赖项，这个过程基本上是反向的，因此名称Inversion of Control（IoC）。
bean本身通过使用类的直接构造来控制其依赖关系的实例化或定位，诸如`服务定位器模式`。IoC也称为依赖注入（DI）。

Spring中`org.springframework.beans`和`org.springframework.context`包是Spring框架的IoC容器的基础。
其中`BeanFactory`接口提供了一种能够管理任何类型对象的高级配置机制。 `ApplicationContext`是一个`BeanFactory`的子类，它增加了与Spring的AOP功能的更容易的集成、消息资源处理（用于国际化）、事件发布; 和特定于应用程序层的上下文，例如WebApplicationContext 在Web应用程序中使用的上下文。
简而言之，`BeanFactory`提供了配置框架和基本功能，`ApplicationContext`添加了更多特定于企业的功能。

在Spring中，构成应用程序主干并由`Spring IoC 容器`管理的对象称为`Bean`。`Bean`是一个由Spring IoC容器实例化、组装和管理的对象。Bean及其之间的依赖关系反映在容器使用的`配置元数据`中。

## 7.2 容器概观

接口`org.springframework.context.ApplicationContext`表示`Spring IoC容器`，负责实例化、配置和组装上述bean。
容器通过读取`配置元数据`获取有关要实例化、配置和组装的对象的指令。
配置元数据以`XML`、`Java注解`或`Java代码`表示。它允许您表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。

`ApplicationContext接口`的几个实现是与Spring一起提供的。在独立应用程序中，通常会创建`ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`的实例。虽然XML一直是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明性地支持这些额外的元数据格式，从而指示容器使用`Java注解或代码`作为元数据格式。

下图是Spring工作原理的高级视图:
![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/container-magic.png)


### 7.2.1 配置元数据

基于XML的元数据不是唯一允许的配置元数据形式。Spring IoC容器本身完全与实际编写此配置元数据的格式分离。目前，许多开发人员为其Spring应用程序选择基于Java的配置。

有关在Spring容器中使用其他形式的元数据的信息，请参阅：
- 基于注解的配置：Spring 2.5引入了对基于注解的配置元数据的支持。
- 基于Java的配置：从Spring 3.0开始，Spring JavaConfig项目提供的许多功能成为Spring Framework核心的一部分。因此，您可以使用Java而不是XML文件在应用程序类外部定义bean。
要使用这些新功能，请[参阅](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans-java)`@Configuration`、`@Bean`、`@Import`、`@DependsOn`注释。


1. 基于XML的配置元数据会显示这些bean在顶级<beans/>元素中配置为<bean/>元素。如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
`id属性`是一个字符串，用于标识单个bean定义。`class属性`定义bean的类型并使用完全限定的classname。id属性的值指的是协作对象。

2. Java注解通常在`@Configuration类`中使用`@Bean`注解方法。

---

通常，不会在容器中配置`细粒度域对象`，因为创建和加载域对象通常由DAO和业务逻辑负责。
但是，您可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。
请参阅[使用AspectJ使用Spring依赖注入域对象](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#aop-atconfigurable)。


### 7.2.2 实例化容器

实例化`Spring IoC容器`非常简单。提供给ApplicationContext构造函数的位置路径实际上是资源字符串，允许容器从各种外部资源（如本地文件系统，Java CLASSPATH等）加载配置元数据。
> ApplicationContext context = new ClassPathXmlApplicationContext("SpringApplicationContext.xml");

以下示例显示了数据访问对象daos.xml文件:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
    </bean>
</beans>
```

以下示例显示了服务层对象services.xml配置文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

在前面的示例中，服务层由PetStoreServiceImpl类和两个JpaAccountDao和JpaItemDao类型的数据访问对象组成（基于JPA对象/关系映射标准）。属性`name元素`引用JavaBean属性的名称，`ref元素`引用另一个bean定义的名称。id和ref元素之间的这种联系表达了协作对象之间的依赖关系。

---

让`bean定义`跨越多个XML文件会很有用。通常，每个单独的XML配置文件都代表架构中的逻辑层或模块。
您可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。此构造函数采用多个Resource位置。
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

或者，使用一个或多个<import/>元素来从另一个或多个文件加载bean定义。例如：
```xml
<beans>
    <import resource="resources/services.xml"/>
    <import resource="resources/daos.xml"/>
</beans>
```
```java
ApplicationContext context = new ClassPathXmlApplicationContext( "xxx.xml");
```

根据`Spring Schema`，正在导入的文件的内容（包括顶级<beans/>元素）必须是有效的XML bean定义。



### 7.2.3 Using the container

`ApplicationContext`使您可以读取bean定义并按如下方式访问它们：
```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```
---

最灵活的变体是`GenericApplicationContext` 与 `Reader` 的组合：
```java
GenericApplicationContext context = new GenericApplicationContext();
new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
context.refresh();
```

然后，您可以使用getBean来检索Bean的实例。ApplicationContext接口有一些其他方法可以检索bean，但理想情况下，您的应用程序代码永远不应该使用它们。实际上，您的应用程序代码根本不应该调用getBean()方法，因此根本不依赖于Spring API。例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF托管bean）提供依赖注入，允许您通过元数据（例如自动装配注释）声明对特定bean的依赖性。

如使用`@Autowired`进行获取实例，而不是通过`调用getBean()方法`；


## 7.3 Bean 概览

Spring IoC容器管理一个或多个bean。这些bean是使用您提供给容器的`配置元数据`创建的。

在容器内部，这些bean定义表示为`BeanDefinition对象`，其中包含（以及其他信息）以下元数据：
- 包限定的类名：通常是正在定义的bean的实际实现类。
- Bean行为配置元素，说明bean在容器中的行为方式（范围，生命周期回调等）。
- 引用bean执行其工作所需的其他bean;这些引用也称为协作者或依赖项。
- 要在新创建的对象中设置的其他配置设置，例如，在管理连接池的Bean中使用的连接数，或池的大小限制。

这些元数据转换为构成每个bean定义的一组属性。

---

Bean的定义包含以下属性：
class、name、scope、constructor arguments、properties、autowiring mode、lazy-initialization mode、initialization method、destruction method


除了包含有关如何创建特定bean的信息的bean定义之外，ApplicationContext实现还`允许用户注册在容器外部创建的现有对象`。这是通过getBeanFactory()方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory实现DefaultListableBeanFactory。DefaultListableBeanFactory通过方法registerSingleton(..)和registerBeanDefinition(..)支持此注册。

```java
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

ConfigurableListableBeanFactory beanFactory = applicationContext.getBeanFactory();

// 将Bean注册到容器中
HelloService helloService = new HelloServiceImplTwo();
beanFactory.registerSingleton("helloService", helloService);
```

需要尽早注册`Bean元数据`和`手动提供的单例实例`，以便容器在自动装配和其他内省步骤期间正确推理它们。虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时注册新bean（与对工厂的实时访问同时）并未得到官方支持，并且可能导致bean容器中的并发访问异常和/或不一致状态。

### 7.3.1 Bean的命名

每个bean都有一个或多个标识符，这些标识符在托管bean的容器中必须是唯一的。
bean通常只有一个标识符，但如果它需要多个标识符，则额外的标识符可以被视为`别名`。

在基于XML的配置元数据中，使用 `id` and/or `name` 属性指定bean标识符。id属性允许您指定一个id。通常，这些名称是字母数字（'myBean'，'fooService'等），但也可能包含特殊字符。如果要向bean引入其他别名，还可以在name属性中指定它们，用逗号（,）、分号（;）、空格分隔。

作为历史记录，在Spring 3.1之前的版本中，id属性被定义为xsd：ID类型，它约束了可能的字符。
从3.1开始，它被定义为xsd：string类型。请注意，bean ID唯一性仍由容器强制执行，但不再由XML解析器强制执行。

您不需要为bean提供名称或ID。如果没有显式提供名称或标识，则容器会为该bean生成唯一的名称。但是，如果要通过名称引用该bean，则必须通过使用ref元素或`Service Locator样式`查找来提供名称。不提供名称的动机与使用内部bean和自动装配协作者有关。

使用类路径中的组件扫描，Spring按照上述规则为未命名的组件生成bean名称：一般情况下，采用简单的类名并将其初始字符转换为小写。但是，在（不常见的）特殊情况下，当有多个字符且第一个和第二个字符都是大写字母时，原始外壳将被保留。这些规则与java.beans.Introspector.decapitalize （Spring在此处使用）中定义的规则相同。

---

在bean定义之外别名bean

在bean定义本身中，您可以为bean提供多个名称，方法是使用`id属性`指定的最多一个名称和`name属性`中的任意数量的其他名称。这些名称可以是同一个bean的等效别名，并且在某些情况下很有用，例如允许应用程序中的每个组件通过使用特定于该组件本身的bean名称来引用公共依赖项。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" name="helloService2" class="morning.cat.bpp.demo1.HelloServiceImpl"></bean>

    <bean id="helloServiceTwo" name="helloServiceTwo2,helloServiceTwo3,helloServiceTwo4"
          class="morning.cat.bpp.demo1.HelloServiceImplTwo"></bean>
</beans>

```

但是，指定实际定义bean的所有别名并不总是足够的。有时需要为其他地方定义的bean引入别名。在大型系统中通常就是这种情况，其中配置在每个子系统之间分配，每个子系统具有其自己的一组对象定义。在基于XML的配置元数据中，您可以使用<alias/>元素来完成此任务。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="morning.cat.bpp.demo1.HelloServiceImpl"></bean>

    <alias name="helloService" alias="MyService"/>
</beans>

```

---

例如，子系统A的配置元数据可以通过subsystemA-dataSource的名称来引用DataSource。子系统B的配置元数据可以通过subsystemB-dataSource的名称引用DataSource。在编写使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用DataSource。要使所有三个名称引用同一对象，可以将以下别名定义添加到配置元数据中：
```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```
现在，每个组件和主应用程序都可以通过一个唯一的名称引用dataSource，并保证不与任何其他定义冲突（有效地创建命名空间），但它们引用相同的bean。


### 7.3.2 实例化Bean

bean定义本质上是用于创建一个或多个对象的配方。容器在被询问时查看命名bean的配方，并使用由该bean定义封装的配置元数据来创建（或获取）实际对象。

如果使用基于XML的配置元数据，则指定要在<bean/>元素的class属性中实例化的对象的类型（或类）。此类属性在内部是`BeanDefinition实例`上的`Class属性`，通常是必需的。您可以通过以下两种方式之一使用Class属性：
- 通常，在容器本身通过反向调用其构造函数直接创建bean的情况下指定要构造的bean类，稍微等同于使用new运算符的Java代码。
- 要指定将被调用以创建对象的静态工厂方法的实际类，在不常见的情况下，容器在类上调用静态工厂方法来创建bean。从静态工厂方法的调用返回的对象类型可以完全是同一个类或另一个类。

`内部类名`。如果要为静态嵌套类配置bean定义，则必须使用嵌套类的二进制名称。例如，如果你在com.example包中有一个名为Foo的类，并且这个Foo类有一个名为Bar的静态嵌套类，那么bean定义中'class'属性的值将是
`com.example.Foo$Bar`
请注意，在名称中使用$字符可以将嵌套类名与外部类名分开。

---

1. 使用构造函数实例化

当您通过构造方法创建bean时，所有普通类都可以使用,并与Spring兼容。也就是说，正在开发的类不需要实现任何特定接口或以特定方式编码。简单地指定bean类就足够了。但是，根据您为该特定bean使用的IoC类型，您可能需要一个`默认（空）构造函数`。

Spring IoC容器几乎可以管理您希望它管理的任何类;它不仅限于管理真正的JavaBeans。大多数Spring用户更喜欢实际的JavaBeans，只有一个默认（无参数）构造函数，并且在容器中的属性之后建模了适当的setter和getter。您还可以在容器中拥有更多异国情调的非bean样式类。例如，如果您需要使用绝对不符合JavaBean规范的旧连接池，那么Spring也可以对其进行管理。

---

2. 使用静态工厂方法实例化

定义使用静态工厂方法创建的bean时，可以使用`class属性`指定包含静态工厂方法的类和`名为factory-method的属性`，以指定工厂方法本身的名称。您应该能够调用此方法（使用后面描述的可选参数）并返回一个活动对象，随后将其视为通过构造函数创建的对象。这种bean定义的一个用途是在遗留代码中调用静态工厂。

以下bean定义指定通过调用`factory-method`创建bean。该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。在此示例中，createInstance（）方法必须是静态方法。
```xml
<bean id="clientService" class="examples.ClientService"
    factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}
    public static ClientService createInstance() {
        return clientService;
    }
}
```

---

3. 使用实例工厂方法实例化

与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用现有bean的非静态方法来创建新bean。要使用此机制，请将class属性保留为空，并在factory-bean属性中，在当前（或父/祖先）容器中指定bean的名称，该容器包含要调用以创建对象的实例方法。使用factory-method属性设置工厂方法本身的名称。

```xml
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如下所示：
```xml
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```
这种方法表明工厂bean本身可以通过依赖注入（DI）进行管理和配置。


在Spring文档中，工厂bean指的是在Spring容器中配置的bean，它将通过实例或静态工厂方法创建对象。相比之下，FactoryBean（注意大小写）是指特定于Spring的FactoryBean。


---

云谁之思？美孟桂矣。

