---
layout:     post
title:      Spring IOC 官网文档阅读笔记（二）
subtitle:   spring 官网文档梳理
date:       2019-03-18
author:     BY morningcat
header-img: img/20190314/HelloSpring.jpg
catalog: true
tags:
    - spring
---


版本：4.3.22.RELEASE

章节：[7. The IoC container](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#beans)

## 7.4 依赖

### 7.4.1 依赖注入

依赖注入(Dependency injection, DI)是一个过程，在这个过程中，对象定义它们的依赖项，也就是说，对象只通过构造函数参数、到工厂方法的参数，或者对象实例构造或从工厂方法返回后在对象实例上设置的属性来定义它们所使用的其他对象。然后容器在创建bean时注入这些依赖项。这个过程本质上与bean本身相反，因此称为控制反转(IoC)， bean本身通过直接构造类或服务定位器模式来控制依赖项的实例化或位置。

使用DI原理，代码更干净，当对象具有依赖关系时，去耦更有效。对象不查找其依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖于接口或抽象基类（允许在单元测试中使用存根或模拟实现）时。

#### 依赖注入主要有两个变种

1. 基于构造函数的依赖注入

`基于构造函数的依赖注入`由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。调用具有特定参数的静态工厂方法来构造bean几乎是等效的。

以下示例显示了一个只能通过构造函数注入进行依赖注入的类。
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

- 构造函数参数解析

使用参数的类型进行构造函数参数解析匹配。如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序是在实例化bean时将这些参数提供给适当的构造函数的顺序。
案例：
```java
public class Foo {
    public Foo(Bar bar, Baz baz) {
        // ...
    }
}
```
```xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <constructor-arg ref="bar"/>
        <constructor-arg ref="baz"/>
    </bean>

    <bean id="bar" class="x.y.Bar"/>
    <bean id="baz" class="x.y.Baz"/>
</beans>
```
以上配置工作正常，您无需在<constructor-arg/>元素中显式指定构造函数参数索引 `和/或` 类型。

---

当引用另一个bean时，类型是已知的，并且可以发生匹配（与前面的示例一样）。
案例：
```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```
第一种方式：使用`type属性`显式指定构造函数参数的类型
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

第二种方式：使用`index属性`显式指定构造函数参数的索引。
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义。请注意，索引是基于0的。

第三种方式：使用`构造函数参数名称`进行值消歧：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```
请记住，为了使这项工作开箱即用，必须在启用调试标志的情况下编译代码，以便Spring可以从构造函数中查找参数名称。如果无法使用debug标志（或不想）编译代码，则可以使用@ConstructorProperties JDK批注显式命名构造函数参数。然后，示例类必须如下所示：
```java
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

2. 基于Setter的依赖注入

`基于setter的依赖注入`是在调用无参数构造函数或无参数静态工厂方法来实例化bean之后，通过容器调用bean上的setter方法来完成的。

---

以下示例显示了一个只能使用纯setter注入进行依赖注入的类。
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
    // business logic that actually uses the injected MovieFinder is omitted...
}
```

`ApplicationContext`支持它管理的Bean`基于构造函数`和`基于setter`的依赖注入。也可以先通过构造函数方法注入了一些依赖项之后再基于setter的依赖注入。您可以以`BeanDefinition`的形式配置依赖项，并将其与`PropertyEditor`实例结合使用，以将属性从一种格式转换为另一种格式。
但是，大多数Spring用户不直接使用这些类（即以编程方式），而是使用XML bean定义、带注释的组件（使用@Component，@Controller等注解）、`基于Java的@Bean方法和@Configuration类`。然后，这些源在内部转换为`BeanDefinition`的实例，并用于加载整个Spring IoC容器实例。

---

#### 基于构造函数的依赖注入 和 基于Setter的依赖注入 的选择

由于您可以混合基于构造函数和基于setter的依赖注入，因此将构造函数用于强制依赖项和setter方法或可选依赖项的配置方法是一个很好的经验法则。
请注意，在setter方法上使用`@Required注解`可用于使属性成为`必需的依赖项`。

Spring团队通常提倡构造函数注入，因为它使应用程序组件能够实现为不可变对象，并确保所需的依赖项不为空。此外，构造函数注入的组件始终以完全初始化的状态返回到客户端（调用）代码。作为旁注，大量的构造函数参数是一个糟糕的代码气味，暗示该类可能有太多的责任，应该重构以更好地解决关注点的正确分离。

Setter注入应主要仅用于可在类中指定合理默认值的可选依赖项。否则，必须在代码使用依赖项的任何位置执行非空检查。setter注入的一个好处是setter方法使该类的对象可以在以后重新配置或重新注入。因此，通过JMX MBean进行管理是二次注入的一个引人注目的用例。

使用对特定类最有意义的DI样式。有时，在处理您没有源的第三方类时，会选择您。例如，如果第三方类没有公开任何setter方法，那么构造函数注入可能是唯一可用的DI形式。

#### 依赖性解决过程

容器执行bean依赖性解析，如下所示：
- `ApplicationContext`是使用描述所有bean的配置元数据创建和初始化的。可以通过XML、Java代码或注解指定配置元数据。
- 对于每个bean，如果使用的是依赖于普通构造函数的，那么它的依赖关系将以属性、构造函数参数、`static-factory`方法的参数的形式表示。实际创建bean时，会将这些依赖项提供给bean。
- 每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
- 作为值的每个属性或构造函数参数都从其指定格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，例如int，long，String，boolean等。

Spring容器在创建容器时验证每个bean的配置。但是，在实际创建bean之前，不会设置bean属性本身。创建容器时会创建单例作用域并设置为预先实例化（默认值）的Bean。范围在第7.5节“Bean范围”中定义。否则，仅在请求时才创建bean。创建bean可能会导致创建bean的图形，因为bean的依赖关系及其依赖关系（依此类推）被创建和分配。请注意，这些依赖项之间的分辨率不匹配可能会显示较晚，即首次创建受影响的bean时。

#### 循环依赖

如果您主要使用构造函数注入，则可以创建无法解析的循环依赖关系场景。

例如：类A通过构造函数注入需要类B的实例，而类B通过构造函数注入需要类A的实例。如果为A类和B类配置bean以便相互注入，则Spring IoC容器会在运行时检测此循环引用，并抛出`BeanCurrentlyInCreationException`。

一种可能的解决方案是编辑由setter而不是构造函数配置的某些类的源代码。或者，避免构造函数注入并仅使用setter注入。换句话说，尽管不推荐使用，但您可以使用setter注入配置循环依赖项。

与典型情况（没有循环依赖）不同，bean A和bean B之间的循环依赖强制其中一个bean在完全初始化之前被注入另一个bean（一个经典的鸡/蛋场景）。

---

你通常可以相信Spring做正确的事。它在容器加载时检测配置问题，例如对不存在的bean和循环依赖关系的引用。当实际创建bean时，Spring会尽可能晚地设置属性并解析依赖关系。这意味着，如果在创建该对象或其某个依赖项时出现问题，则在请求对象时，正确加载的Spring容器可以在以后生成异常。例如，bean因缺少属性或无效属性而抛出异常。这可能会延迟一些配置问题的可见性，这就是默认情况下ApplicationContext实现预先实例化单例bean的原因。以实际需要之前创建这些bean的一些前期时间和内存为代价，您会在创建ApplicationContext时发现配置问题，而不是更晚。您仍然可以覆盖此默认行为，以便单例bean将延迟初始化，而不是预先实例化。

如果不存在循环依赖关系，当一个或多个协作bean被注入依赖bean时，每个协作bean在被注入依赖bean之前完全配置。这意味着如果bean A依赖于bean B，Spring IoC容器在调用bean A上的setter方法之前完全配置bean B.换句话说，bean被实例化（如果不是预先实例化的单例），设置依赖项，并调用相关的生命周期方法（如配置的`init方法`或`InitializingBean`回调方法）。


#### 依赖注入的例子

以下示例将基于XML的配置元数据用于基于setter的DI。Spring XML配置文件的一小部分指定了一些bean定义：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明setter与XML文件中指定的属性匹配。
以下示例使用基于构造函数的DI：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

现在考虑这个示例的变体，其中不是使用构造函数，而是告诉Spring调用静态工厂方法来返回对象的实例：
```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```
```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}

```
静态工厂方法的参数通过<constructor-arg/>元素提供，与实际使用的构造函数完全相同。工厂方法返回的类的类型不必与包含静态工厂方法的类具有相同的类型，尽管在此示例中它是。实例（非静态）工厂方法将以基本相同的方式使用（除了使用factory-bean属性而不是class属性），因此这里不再讨论细节。



### 7.4.2 依赖关系和配置详细

#### Straight values (primitives, Strings, and so on)

<property />元素的value属性将属性或构造函数参数指定为人类可读的`字符串表示形式`。Spring的转换服务用于将这些值从String转换为属性或参数的实际类型。
```XML
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

以下示例使用p命名空间进行更简洁的XML配置。
```XML
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

您还可以将java.util.Properties实例配置为：
```xml
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
Spring容器通过使用JavaBeans PropertyEditor机制将<value />元素内的文本转换为java.util.Properties实例。这是一个很好的快捷方式，并且是Spring团队支持在值属性样式上使用嵌套<value />元素的一些地方之一。

#### idref元素

`idref元素`只是一种防错方法，可以将容器中另一个bean的id（字符串值 - 而不是引用）传递给<constructor-arg />或<property />元素
```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```
上面的bean定义片段与以下片段完全等效（在运行时）：
```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```
第一种形式比第二种形式更可取，因为使用`idref标签`允许容器在部署时`验证`引用的命名bean实际存在。在第二个变体中，不对传递给客户端bean的targetName属性的值执行验证。当客户端bean实际实例化时，才会发现错别字（最有可能导致致命结果）。如果客户端bean是 prototype bean，则只能在部署容器后很长时间才能发现此错误和产生的异常。

> `4.0 beans xsd`不再支持idref元素的local属性，因为它不再提供常规bean引用的值。升级到4.0架构时，只需将现有的idref本地引用更改为idref bean即可。

`<idref />元素`带来值的常见位置（至少在Spring 2.0之前的版本中）是在`ProxyFactoryBean` bean定义中的`AOP拦截器`的配置中。指定拦截器名称时使用`<idref />元素`可以防止拼写错误的拦截器ID。

#### 引用其他 Bean

通过`<ref />标记`的bean属性指定`目标bean`是最常用的形式，并允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一XML文件中。bean属性的值可以与目标bean的id属性相同，也可以作为目标bean的name属性中的值之一。
```xml
<ref bean="someBean"/>
```

通过`parent属性`指定目标bean会创建对当前容器的父容器中的bean的引用。parent属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的一个值相同，并且目标bean必须位于当前bean的父容器中。您主要在拥有容器层次结构并且希望将现有bean包装在父容器中并使用与父bean具有相同名称的代理时使用此bean引用变体。
```xml
<!-- in the parent context -->
<bean id="accountService" class="com.foo.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```
```xml
<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

#### Inner beans

<property />或<constructor-arg />元素中的<bean />元素定义了一个所谓的内部bean。
```xml
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```
作为极端情况，可以从自定义范围接收销毁回调，例如，对于包含在单例bean中的请求范围的内部bean：内部bean实例的创建将绑定到其包含的bean，但是销毁回调允许它参与请求范围的生命周期。这不是常见的情况;内部bean通常只是共享其包含bean的范围。

#### Collections

在`<list />`，`<set />`，`<map />`和`<props />`元素中，分别设置Java Collection类型List，Set，Map和Properties的属性和参数。
```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list>
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map>
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

#### Collection merging

Spring容器还支持集合的合并。应用程序开发人员可以定义父样式的<list />，<map />，<set />或<props />元素，并具有子样式<list />，<map />，<set />或<props />元素继承并覆盖父集合中的值。也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素覆盖父集合中指定的值。

以下示例演示了集合合并：
```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
注意在子bean定义的adminEmails属性的<props />元素上使用merge = true属性。当容器解析并实例化子bean时，生成的实例有一个adminEmails属性集合，其中包含子项的adminEmails集合与父项的adminEmails集合的合并结果。

administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk

子Properties集合的值集继承父<props />的所有属性元素，子值的支持值覆盖父集合中的值。

此合并行为同样适用于<list />，<map />和<set />集合类型。在<list />元素的特定情况下，保持与List集合类型相关联的语义，即有序的值集合的概念;父级的值位于所有子级列表的值之前。对于Map，Set和Properties集合类型，不存在排序。因此，对于作为容器内部使用的关联Map，Set和Properties实现类型的基础的集合类型，没有排序语义有效。

#### Limitations of collection merging

您不能合并不同的集合类型（例如Map和List），如果您尝试这样做，则会抛出相应的Exception。必须在较低的继承子定义上指定merge属性;在父集合定义上指定merge属性是多余的，不会导致所需的合并。

#### Strongly-typed collection

通过在Java 5中引入泛型类型，您可以使用强类型集合。也就是说，可以声明Collection类型，使其只能包含String元素（例如）。如果您使用Spring依赖注入一个强类型的Collection到bean中，您可以利用Spring的类型转换支持，这样强类型Collection实例的元素在被添加到之前就会转换为适当的类型集合。
```java
public class Foo {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```
```xml
<beans>
    <bean id="foo" class="x.y.Foo">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```
当foo bean的accounts属性准备好进行注入时，可以通过反射获得有关强类型Map <String，Float>的元素类型的泛型信息。因此，Spring的类型转换基础结构将各种值元素识别为Float类型，并将字符串值9.99,2.75和3.99转换为实际的Float类型。

#### Null and empty string values

Spring将属性等的空参数视为空字符串。以下基于XML的配置元数据片段将email属性设置为空String值（“”）。
```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```
The preceding example is equivalent to the following Java code:
> exampleBean.setEmail("");

`<null/>元素`处理空值。例如：
```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```
以上配置等同于以下Java代码：
> exampleBean.setEmail(null);

#### 带有p命名空间的XML快捷方式

#### XML shortcut with the c-namespace

#### 复合属性名称

---

### 7.4.3 使用 depends-on

如果bean是另一个bean的依赖项，通常意味着将一个bean设置为另一个bean的属性。通常，您可以使用基于XML的配置元数据中的`<ref />元素`来完成此操作。
但是，有时bean之间的依赖关系`不那么直接`;例如，需要触发类中的静态初始化程序，例如数据库驱动程序注册。在初始化使用此元素的bean之前，`depends-on属性`可以显式强制初始化一个或多个bean。以下示例使用depends-on属性表示对单个bean的依赖关系：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个bean的依赖关系，请提供bean名称列表作为depends-on属性的值，使用逗号，空格和分号作为有效分隔符：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

bean定义中的`depends-on属性`既可以指定初始化时间依赖关系，也可以指定仅限单例bean的相应销毁时间依赖关系。在给定的bean本身被销毁之前，首先销毁定义与给定bean的依赖关系的从属bean。因此，依赖也可以控制关​​机顺序。

### 7.4.4 Lazy-initialized beans (延迟初始化)

默认情况下，`ApplicationContext实现`会急切地创建和配置所有单例bean，作为初始化过程的一部分。通常，这种预先实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后。如果不希望出现这种情况，可以通过将bean定义标记为`延迟初始化`来阻止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，此行为由<bean />元素上的lazy-init属性控制;例如：
```xml
<bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.foo.AnotherBean"/>
```

当ApplicationContext使用前面的配置时，在ApplicationContext启动时，不会急切地预先实例化名为lazy的bean，而是非常预先实例化not.lazy bean。

但是，当延迟初始化的bean是未进行延迟初始化的单例bean的依赖项时，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。惰性初始化的bean被注入到其他地方的单独的bean中，而这个bean并不是惰性初始化的。

您还可以使用<beans />元素上的`default-lazy-init属性`在容器级别控制延迟初始化;例如：
```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```

### 7.4.5 自动装配合作者

自动装配具有以下优点：
- 自动装配可以显着减少指定属性或构造函数参数的需要。（在本章其他地方讨论的其他机制，如bean模板，在这方面也很有价值。）
- 自动装配可以随着对象的发展更新配置。例如，如果需要向类添加依赖项，则可以自动满足该依赖项，而无需修改配置。因此，自动装配在开发期间尤其有用，而不会在代码库变得更稳定时否定切换到显式布线的选项。

---

使用基于XML的配置元数据时，可以使用<bean />元素的autowire属性为bean定义指定autowire模式。自动装配功能有四种模式。您指定每个bean的自动装配，因此可以选择要自动装配的那些。

Autowiring modes
1. no
（默认）无自动装配。必须通过ref元素定义Bean引用。不建议对较大的部署更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。
2. byName
按属性名称自动装配。Spring查找与需要自动装配的属性同名的bean。例如，如果bean定义按名称设置为autowire，并且它包含master属性（即，它具有setMaster（..）方法），则Spring会查找名为master的bean定义，并使用它来设置属性。
3. byType
如果容器中只存在一个属性类型的bean，则允许自动装配属性。如果存在多个，则抛出致命异常，这表示您不能对该bean使用byType自动装配。如果没有匹配的bean，则没有任何反应;该物业未设定。
4. constructor
类似于byType，但适用于构造函数参数。如果容器中没有构造函数参数类型的一个bean，则会引发致命错误。

使用byType或构造函数自动装配模式，您可以连接数组和类型集合。在这种情况下，提供容器内与预期类型匹配的所有autowire候选者以满足依赖性。如果预期的键类型为String，则可以自动装配强类型映射。自动装配的Maps值将包含与预期类型匹配的所有Bean实例，而Maps键将包含相应的bean名称。

您可以将autowire行为与依赖性检查相结合，这将在自动装配完成后执行。

#### 自动装配的局限和缺点

当在整个项目中一致地使用自动装配时，自动装配效果最佳。如果一般不使用自动装配，那么开发人员使用它来连接一个或两个bean定义可能会让人感到困惑。

考虑自动装配的局限和缺点：
- property和constructor-arg设置中的显式依赖项始终覆盖自动装配。您无法自动装配所谓的简单属性，例如基元，字符串和类（以及此类简单属性的数组）。这种限制是按设计的。
- 自动装配不如显式布线精确。尽管如上表所示，Spring会小心避免在可能产生意外结果的歧义的情况下进行猜测，但不再明确记录Spring管理对象之间的关系。
- 可能无法为可能从Spring容器生成文档的工具提供连线信息。
- 容器中的多个bean定义可以匹配setter方法或构造函数参数指定的类型以进行自动装配。对于数组，集合或地图，这不一定是个问题。但是，对于期望单个值的依赖关系，这种模糊性不是任意解决的。如果没有可用的唯一bean定义，则抛出异常。

在后一种情况下，您有几种选择：
- 放弃自动装配，支持显式布线。
- 通过将`autowire-candidate属性`设置为false，避免对bean定义进行自动装配，如下一节所述。
- 通过将其<bean />元素的`primary属性`设置为true，将单个bean定义指定为主要候选者。
- 使用基于注解的配置实现更细粒度的控件


#### 从自动装配中排除 Bean

在每个bean的基础上，您可以从自动装配中排除bean。在Spring的XML格式中，将<bean />元素的`autowire-candidate属性`设置为`false`;容器使特定的bean定义对自动装配基础结构不可用（包括注解样式配置，如@Autowired）。

`autowire-candidate属性`旨在仅影响`基于类型的自动装配`。它不会影响`名称的显式引用`，即使指定的bean未标记为autowire候选，也会解析它。因此，如果名称匹配，按名称自动装配将注入bean。

您还可以根据与bean名称的模式匹配来限制autowire候选者。顶级<beans />元素在其`default-autowire-candidates属性`中接受一个或多个模式。例如，要将autowire候选状态限制为名称以Repository结尾的任何bean，请提供值*Repository。要提供多个模式，请在逗号分隔的列表中定义它们。bean定义autowire-candidate属性的显式值true或false始终优先，对于此类bean，模式匹配规则不适用。

这些技术对于您永远不希望通过自动装配注入其他bean的bean非常有用。这并不意味着排除的bean本身不能使用自动装配进行配置。相反，bean本身不是自动装配其他bean的候选者。

### 7.4.6方法注入

在大多数应用程序场景中，容器中的大多数bean都是单例。当单例bean需要与另一个单例bean协作，或者非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean生命周期不同时会出现问题。假设单例bean A需要使用非单例（prototype）bean B，可能是在A上的每个方法调用上。容器只创建一次单例bean A，因此只有一次机会来设置属性。每次需要时，容器都不能为bean A提供bean B的新实例。

解决方案是放弃一些控制反转。您可以通过实现`ApplicationContextAware接口`使bean A 知道 容器，并通过对容器进行getBean（“B”）调用，每次bean A需要时都要求（通常是新的）bean B实例。以下是此方法的示例：
```java
// a class that uses a stateful Command-style class to perform some processing
package fiona.apple;

// Spring-API imports
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
前面的内容是不可取的，因为业务代码知道并耦合到Spring Framework。Method Injection是Spring IoC容器的一个先进功能，它允许以干净的方式处理这个用例。

您可以在[此博客条目](https://spring.io/blog/2004/08/06/method-injection/)中阅读有关Method Injection的动机的更多信息。

#### Lookup 方法注入

Lookup方法注入是容器覆盖容器托管bean上的方法的能力，以返回容器中另一个命名bean的查找结果。查找通常涉及 `prototype bean`，如上一节中描述的场景。Spring Framework通过使用CGLIB库中的字节码生成来实现此方法注入，以动态生成覆盖该方法的子类。

- 为了使这个动态子类化起作用，Spring bean容器将子类化的类不能是final，并且要重写的方法也不能是final。
- 对具有抽象方法的类进行单元测试需要您自己对类进行子类化，并提供抽象方法的存根实现。
- 组件扫描也需要具体的方法，这需要具体的类别来获取。
- 另一个关键限制是查找方法不适用于工厂方法，特别是配置类中的@Bean方法，因为容器在这种情况下不负责创建实例，因此无法创建运行时生成的子类在飞行中。

查看前面代码片段中的CommandManager类，您会看到Spring容器将动态覆盖createCommand（）方法的实现。您的CommandManager类将没有任何Spring依赖项，如重新编写的示例中所示：
```java
package fiona.apple;

// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

在包含要注入的方法的客户端类（在本例中为CommandManager）中，要注入的方法需要以下形式的签名：
> <public|protected> [abstract] <return-type> theMethodName(no-arguments);

如果方法是抽象的，则动态生成的子类实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。例如：
```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
标识为commandManager的bean在需要myCommand bean的新实例时调用自己的方法createCommand（）。您必须小心将myCommand bean部署为`prototype`，如果这实际上是需要的话。如果它是一个单例，则每次都返回myCommand bean的相同实例。

---

或者，在基于注解的组件模型中，您可以通过`@Lookup`注解声明查找方法：
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

或者，更具惯用性，您可以依赖于针对查找方法的声明返回类型解析目标bean：
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```
请注意，您通常会使用具体的存根实现来声明这种带注解的查找方法，以使它们与Spring的组件扫描规则兼容，默认情况下抽象类被忽略。此限制不适用于显式注册或显式导入的bean类。

> 访问不同范围的目标bean的另一种方法是ObjectFactory / Provider注入点。查看名为“Scoped beans as dependencies”的部分。
> 感兴趣的读者也可以找到`ServiceLocatorFactoryBean`（在org.springframework.beans.factory.config包中）。

#### 任意方法更换

与查找方法注入相比，一种不太有用的方法注入形式是能够使用另一个方法实现替换托管bean中的任意方法。

使用基于XML的配置元数据，您可以使用被替换的方法元素将已有的方法实现替换为已部署的bean。考虑以下类，使用方法computeValue，我们要覆盖它：
```java
public class MyValueCalculator {

    public String computeValue(String input) {
        // some real code...
    }
    // some other methods...
}
```
实现`org.springframework.beans.factory.support.MethodReplacer`接口的类提供了新的方法定义。
```java
public class ReplacementComputeValue implements MethodReplacer {

    public Object reimplement(Object o, Method m, Object[] args) throws Throwable {
        // get the input value, work with it, and return a computed result
        String input = (String) args[0];
        ...
        return ...;
    }
}
```
部署原始类并指定方法覆盖的bean定义如下所示：
```xml
<bean id="myValueCalculator" class="x.y.z.MyValueCalculator">
    <!-- arbitrary method replacement -->
    <replaced-method name="computeValue" replacer="replacementComputeValue">
        <arg-type>String</arg-type>
    </replaced-method>
</bean>

<bean id="replacementComputeValue" class="a.b.c.ReplacementComputeValue"/>
```
您可以在<replacement-method />元素中使用一个或多个包含的<arg-type />元素来指示被覆盖的方法的方法签名。仅当方法重载且类中存在多个变体时，才需要参数的签名。为方便起见，参数的类型字符串可以是完全限定类型名称的子字符串。例如，以下所有匹配java.lang.String：
```
java.lang.String
String
Str
```

因为参数的数量通常足以区分每个可能的选择，所以通过允许您只键入与参数类型匹配的最短字符串，此快捷方式可以节省大量的输入。

---

云谁之思？美孟桂矣。