---
layout:     post
title:      Spring MVC 官网文档阅读笔记（一）
subtitle:   spring 官网文档梳理
date:       2019-03-25
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - spring
---

版本：4.3.22.RELEASE

章节：[22. Web MVC framework](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc)

## 22.1 Introduction to Spring Web MVC framework

SpringWeb模型视图控制器（MVC）框架是围绕`DispatcherServlet`设计的，它将请求发送到处理程序，具有可配置的处理程序映射、视图解析、区域设置、时区和主题解析，并支持上载文件。默认处理程序基于`@Controller`和`@Requestmapping`注解，提供了多种灵活的处理方法。随着Spring3.0的引入，@Controller机制还允许您通过@`Pathvariable`注解和其他功能创建`RESTful网站`和应用程序。

在SpringWebMVC中，可以将任何对象用作命令或表单支持对象；不需要实现特定于框架的接口或基类。Spring的数据绑定非常灵活：例如，它将类型不匹配视为可由应用程序评估的验证错误，而不是系统错误。因此，您不需要在表单对象中将业务对象的属性复制为简单的、未类型化的字符串，而只需简单地处理无效的提交，或者正确地转换字符串。相反，通常最好直接绑定到业务对象。

Spring的视图解析非常灵活。Controller通常负责准备带有数据的模型Map并选择视图名称，但它也可以直接写入响应流并完成请求。通过文件扩展名或Accept头内容类型协商，通过bean名称，属性文件甚至自定义ViewResolver实现，可以高度配置视图名称解析。模型（MVC中的M）是一个Map接口，它允许完全抽象视图技术。您可以直接与基于模板的渲染技术（如JSP，Velocity和Freemarker）集成，也可以直接生成XML，JSON，Atom和许多其他类型的内容。模型Map简单地转换为适当的格式，例如JSP请求属性，Velocity模板模型。


### 22.1.1 Spring Web MVC的特点

Spring的Web模块包括许多独特的Web支持功能：

- 明确角色分离。每个角色-控制器、验证器、命令对象、窗体对象、模型对象、DispatcherServlet、处理程序映射、视图解析器等等-都可以由专门的对象来完成。
- 框架类和应用程序类作为JavaBeans的强大而简单的配置。此配置功能包括跨上下文轻松引用，例如从Web控制器到业务对象和验证器。
- 适应性、非侵入性和灵活性。为给定的场景定义所需的任何控制器方法签名，可能使用参数注释（如@RequestParam, @RequestHeader, @PathVariable等）之一。
- 可重复使用的业务代码，无需重复。将现有业务对象用作命令或表单对象，而不是镜像它们以扩展特定的框架基类。
- 可定制的绑定和验证。键入不匹配作为应用程序级验证错误，这些错误会保留违规值，本地化日期和数字绑定等，而不是通过手动解析和转换为业务对象的仅限字符串的表单对象。
- 可定制的处理程序映射和视图解析。处理程序映射和视图解析策略的范围从简单的基于URL的配置到复杂的，专用的解析策略。Spring比授权特定技术的Web MVC框架更灵活。
- 灵活的模型转移。具有名称/值Map的模型传输支持与任何视图技术的轻松集成。
- 可自定义的区域设置，时区和主题解析，支持带或不带Spring标记库的JSP，支持JSTL，支持Velocity而无需额外的桥接，等等。
- 一个简单但功能强大的JSP标记库，称为Spring标记库，为数据绑定和主题等功能提供支持。自定义标记在标记代码方面具有最大的灵活性。有关标记库描述符的信息，请参阅第43章spring spring Tag Library附录
- Spring 2.0中引入的JSP表单标记库，使得在JSP页面中编写表单变得更加容易。有关标记库描述符的信息，请参阅标题为第44章的弹簧形式JSP标记库的附录
- 生命周期范围限定为当前HTTP请求或HTTP会话的Bean。这不是Spring MVC本身的特定功能，而是`Spring MVC`使用的`WebApplicationContext容器`。第7.5.4节“请求，会话，全局会话，应用程序和WebSocket范围”中介绍了这些Bean作用域。

### 22.1.2其他MVC实现的可插拔性

对于某些项目，非Spring MVC实现是更可取的。许多团队希望利用他们在技能和工具方面的现有投资，例如使用JSF。

如果您不想使用Spring的Web MVC，但打算利用Spring提供的其他解决方案，您`可以轻松地将您选择的Web MVC框架与Spring集成`。只需`通过其ContextLoaderListener启动Spring根应用程序上下文`，并通过其ServletContext属性（或Spring的各自辅助方法）从任何操作对象中访问它。不涉及“插件”，因此不需要专门的集成。从Web层的角度来看，您只需将Spring用作库，将根应用程序上下文实例作为入口点。

即使没有Spring的Web MVC，您的注册bean和Spring的服务也可以触手可及。在这种情况下，Spring不会与其他Web框架竞争。它简单地解决了纯web MVC框架没有的许多方面，从bean配置到数据访问和事务处理。因此，您可以使用Spring中间层和/或数据访问层来丰富您的应用程序，即使您只是想使用JDBC或Hibernate的事务抽象。

## 22.2 The DispatcherServlet

与许多其他Web MVC框架一样，Spring的Web MVC框架是围绕中央Servlet设计的请求驱动的，该Servlet向控制器发送请求并提供便于Web应用程序开发的其他功能。然而，Spring的DispatcherServlet不仅仅是这样。它与Spring IoC容器完全集成，因此允许您使用Spring拥有的所有其他功能。

Spring Web MVC `DispatcherServlet`的请求处理工作流程如下图所示。精通模式的读者会认识到DispatcherServlet是“前端控制器”设计模式的表达（这是Spring Web MVC与许多其他领先的Web框架共享的模式）。
![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/mvc.png)

DispatcherServlet是一个实际的Servlet（它继承自HttpServlet基类），因此在Web应用程序中声明。您需要使用URL映射映射您希望DispatcherServlet处理的请求。以下是Servlet 3.0+环境中的标准Java EE Servlet配置：
```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        ServletRegistration.Dynamic registration = container.addServlet("example", new DispatcherServlet());
        registration.setLoadOnStartup(1);
        registration.addMapping("/example/*");
    }
}
```
在前面的示例中，所有以 /example 开头的请求都将由名为 example 的 DispatcherServlet 实例处理。

`WebApplicationInitializer`是Spring MVC提供的一个接口，可确保检测到基于代码的配置并自动用于初始化任何Servlet 3.x 容器。这个名为AbstractAnnotationConfigDispatcherServletInitializer的接口的抽象基类实现使得通过简单地指定其`servlet映射`和`列出配置类`来注册`DispatcherServlet`变得更加容易 - 甚至是建议设置Spring MVC应用程序的方法。有关更多详细信息，请参阅基于代码的Servlet容器初始化

DispatcherServlet是一个实际的Servlet（它继承自HttpServlet基类），因此在Web应用程序的web.xml中声明。您需要使用同一web.xml文件中的URL映射来映射您希望DispatcherServlet处理的请求。这是标准的Java EE Servlet配置;以下示例显示了这样的DispatcherServlet声明和映射：

下面是基于上面代码示例的web.xml等价物：
```xml
<web-app>
    <servlet>
        <servlet-name>example</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>example</servlet-name>
        <url-pattern>/example/*</url-pattern>
    </servlet-mapping>

</web-app>
```

如第7.15节“ApplicationContext的附加功能”中所述，Spring中的ApplicationContext实例可以作用域。在Web MVC框架中，`每个DispatcherServlet都有自己的WebApplicationContext，`它继承了根WebApplicationContext中已定义的所有bean。根WebApplicationContext应包含应在其他上下文和Servlet实例之间共享的所有基础结构bean。可以在特定于servlet的作用域中重写这些继承的bean，并且可以为给定的Servlet实例定义新的作用域特定的bean。

图22.2 Spring Web MVC中的典型上下文层次结构
![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/mvc-context-hierarchy.png)

在初始化DispatcherServlet时，Spring MVC在Web应用程序的WEB-INF目录中查找名为[servlet-name]-servlet.xml的文件，并创建在那里定义的bean，覆盖使用相同名称定义的任何bean的定义在全球范围内。

请考虑以下DispatcherServlet Servlet配置（在web.xml文件中）：
```xml
<web-app>
    <servlet>
        <servlet-name>golfing</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>golfing</servlet-name>
        <url-pattern>/golfing/*</url-pattern>
    </servlet-mapping>
</web-app>
```

使用上面的Servlet配置，您需要在应用程序中有一个名为/WEB-INF/golfing-servlet.xml的文件;此文件将包含所有Spring Web MVC特定组件（bean）。您可以通过Servlet初始化参数更改此配置文件的确切位置（有关详细信息，请参阅下文）。

对于单个DispatcherServlet方案，也可以只有一个根上下文。

图22.3 Spring Web MVC中的单根上下文
![](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/images/mvc-root-context.png)

这可以通过设置一个空的contextConfigLocation servlet init参数来配置，如下所示：
```xml
<web-app>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
</web-app>
```

`WebApplicationContext`是普通ApplicationContext的扩展，它具有Web应用程序所需的一些额外功能。它与普通的ApplicationContext的不同之处在于它能够`解析主题`（参见第22.9节“使用主题”），并且它知道它与哪个Servlet相关联（通过指向ServletContext的链接）。WebApplicationContext绑定在ServletContext中，通过在RequestContextUtils类上使用静态方法，如果需要访问它，可以始终查找WebApplicationContext。

请注意，我们可以使用基于java的配置实现相同的功能：
```java
public class GolfingWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        // GolfingAppConfig defines beans that would be in root-context.xml
        return new Class<?>[] { GolfingAppConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        // GolfingWebConfig defines beans that would be in golfing-servlet.xml
        return new Class<?>[] { GolfingWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/golfing/*" };
    }
}
```

### 22.2.1 WebApplicationContext中的特殊Bean类型

Spring DispatcherServlet使用特殊bean来处理请求并呈现适当的视图。这些bean是Spring MVC的一部分。您可以通过在WebApplicationContext中简单地配置其中的一个或多个来选择要使用的特殊bean。但是，您最初不需要这样做，因为如果您不配置任何默认Bean，则Spring MVC会维护一个默认Bean列表。更多内容将在下一节中介绍。首先看下面的表，列出了DispatcherServlet所依赖的特殊bean类型。

WebApplicationContext中的特殊bean类型：
- HandlerMapping

根据某些标准将`传入请求`映射到`处理程序`以及`预处理器`和`后处理器列表`（处理程序拦截器），其详细信息因`HandlerMapping`实现而异。最流行的实现支持带注解的控制器，但也存在其他实现。

- HandlerAdapter

适配器：
无论实际调用哪个处理程序，都可以帮助`DispatcherServlet`调用映射到请求的处理程序。例如，调用带注解的控制器需要解析各种注解。因此，HandlerAdapter的主要目的是保护DispatcherServlet不受此类细节的影响。

- HandlerExceptionResolver

将异常映射到视图也允许更复杂的异常处理代码。

- ViewResolver

视图解析器 将基于逻辑字符串的逻辑视图名称解析为实际的视图类型。

- LocaleResolver & LocaleContextResolver

解析客户端正在使用的区域设置以及可能的时区，以便能够提供国际化视图

- ThemeResolver

解析Web应用程序可以使用的主题，例如，提供个性化布局

- MultipartResolver

解析多部分请求，例如支持从HTML表单处理文件上载。

- FlashMapManager

存储和检索“输入”和“输出”FlashMap，可用于将属性从一个请求传递到另一个请求，通常是通过重定向。

### 22.2.2默认DispatcherServlet配置

正如上一节中针对每个特殊bean所述，`DispatcherServlet`维护了一个默认使用的实现列表。此信息保存在`org.springframework.web.servlet包中的DispatcherServlet.properties文件`中。

所有特殊bean都有一些`合理的默认值`。迟早虽然你需要定制这些bean提供的一个或多个属性。例如，将`InternalResourceViewResolver`设置其前缀属性配置为视图文件的父位置是很常见的。

无论细节如何，这里要理解的重要概念是，一旦在WebApplicationContext中配置了一个特殊的bean（如InternalResourceViewResolver），就会有效地覆盖那些特殊bean类型本来会使用的默认实现列表。例如，如果配置InternalResourceViewResolver，则会忽略ViewResolver实现的默认列表。

在第22.16节“配置Spring MVC”中，您将了解配置Spring MVC的其他选项，包括MVC Java配置和MVC XML命名空间，这两个选项都提供了一个简单的起点，并且对Spring MVC的工作原理知之甚少。无论您如何选择配置应用程序，本节中介绍的概念都应该对您有所帮助。

### 22.2.3 DispatcherServlet处理序列

设置DispatcherServlet并为该特定DispatcherServlet发出请求后，DispatcherServlet开始处理请求，如下所示：
- 将WebApplicationContext作为控制器和进程中的其他元素可以使用的属性在请求中进行搜索和绑定。它默认绑定在密钥DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE下。
- 语言环境解析器绑定到请求，以使进程中的元素能够解析处理请求时使用的语言环境（呈现视图，准备数据等）。如果您不需要区域设置解析，则不需要它。
- 主题解析器绑定到请求，以允许视图等元素确定要使用的主题。如果您不使用主题，则可以忽略它。
- 如果指定了多部分文件解析程序，则会检查请求的多部分;如果找到多部分，请求将包装在MultipartHttpServletRequest中，以供进程中的其他元素进一步处理。有关多部分处理的更多信息，请参见第22.10节“Spring的多部分（文件上载）支持”。
- 搜索适当的处理程序。如果找到处理程序，则执行与处理程序（预处理程序，后处理程序和控制器）关联的执行链以准备模型或呈现。
- 如果返回模型，则呈现视图。如果没有返回模型（可能是由于预处理器或后处理器拦截请求，可能是出于安全原因），则不会呈现任何视图，因为该请求可能已经完成。

在WebApplicationContext中声明的`处理程序异常解析器`会拾取在处理请求期间引发的异常。使用这些异常解析器可以定义自定义行为以解决异常。

Spring DispatcherServlet还支持返回最后修改日期，如Servlet API所指定。确定特定请求的最后修改日期的过程很简单：DispatcherServlet查找适当的处理程序映射并测试找到的处理程序是否实现`LastModified`接口。如果是这样，LastModified接口的long getLastModified（request）方法的值将返回给客户端。

您可以通过将Servlet初始化参数（init-param元素）添加到web.xml文件中的Servlet声明来自定义各个DispatcherServlet实例。有关支持的参数列表，请参阅下表。

DispatcherServlet初始化参数:
- contextClass

实现`ConfigurableWebApplicationContext`的类，由此Servlet实例化并本地配置。默认情况下，使用XmlWebApplicationContext。

- contextConfigLocation

传递给上下文实例（由contextClass指定）的字符串，用于指示可以找到上下文的位置。该字符串可能包含多个字符串（使用逗号作为分隔符）以支持多个上下文。如果多个上下文位置的bean定义了两次，则最新位置优先。

- namespace

WebApplicationContext的命名空间。默认为[servlet-name]-servlet。

## 22.3 Implementing Controllers

控制器(Controllers)提供对通常通过服务接口定义的应用程序行为的访问。控制器解释用户输入并将其转换为由视图表示给用户的模型。Spring以非常抽象的方式实现控制器，使您可以创建各种控制器。

Spring 2.5为MVC控制器引入了一个基于注解的编程模型，该模型使用注解，如@RequestMapping，@RequestParam，@ModelAttribute等。这个注解支持可用于`Servlet MVC`和`Portlet MVC`。以此样式实现的控制器不必扩展特定的基类或实现特定的接口。此外，它们通常不直接依赖于Servlet或Portlet API，尽管您可以轻松配置对Servlet或Portlet工具的访问。
```java
@Controller
public class HelloWorldController {

    @RequestMapping("/helloWorld")
    public String helloWorld(Model model) {
        model.addAttribute("message", "Hello World!");
        return "helloWorld";
    }
}
```

如您所见，`@Controller`和`@RequestMapping`注解允许灵活的方法名称和签名。在此特定示例中，该方法接受Model并将视图名称作为String返回，但可以使用各种其他方法参数和返回值，如本节后面所述。@Controller和@RequestMapping以及许多其他注释构成了Spring MVC实现的基础。本节介绍了这些注释以及它们在Servlet环境中最常用的方式

### 22.3.1 使用@Controller定义控制器

@Controller注释指示特定类充当控制器的角色。Spring不要求您扩展任何控制器基类或引用Servlet API。但是，如果需要，您仍可以引用特定于Servlet的功能。

@Controller注释充当带注释的类的构造型，指示其角色。调度程序扫描这些带注释的类以查找映射方法，并检测@RequestMapping注释（请参阅下一节）。

您可以使用调度程序上下文中的标准Spring bean定义显式定义带注释的控制器bean。但是，@Controller的构造型也允许自动检测，与Spring一致支持，以检测类路径中的组件类，并为它们自动注册bean定义。

要`启用此类带注解控制器的自动检测`，请将​​`组件扫描`添加到配置中。使用spring-context架构，如以下XML片段所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.springframework.samples.petclinic.web"/>

    <!-- ... -->

</beans>
```

### 22.3.2使用@RequestMapping映射请求

您可以使用`@RequestMapping`批注将`/appointments`等URL映射到整个类或特定的处理程序方法。通常，类级注释将特定请求路径（或路径模式）映射到表单控制器上，其他方法级注释缩小了特定HTTP方法请求方法（“GET”，“POST”等）的主映射。或HTTP请求参数条件。
Petcare示例中的以下示例显示了使用此批注的Spring MVC应用程序中的控制器：
```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @RequestMapping(method = RequestMethod.GET)
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @RequestMapping(path = "/{day}", method = RequestMethod.GET)
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }

    @RequestMapping(path = "/new", method = RequestMethod.GET)
    public AppointmentForm getNewForm() {
        return new AppointmentForm();
    }

    @RequestMapping(method = RequestMethod.POST)
    public String add(@Valid AppointmentForm appointment, BindingResult result) {
        if (result.hasErrors()) {
            return "appointments/new";
        }
        appointmentBook.addAppointment(appointment);
        return "redirect:/appointments";
    }
}
```
在上面的示例中，@RequestMapping用于许多地方。第一种用法是在类型（类）级别，它指示此控制器中的所有处理程序方法都相对于`/appointments`路径。get（）方法还有一个@RequestMapping细化：它​​只接受GET请求，这意味着`/appointments`的HTTP GET会调用此方法。add（）具有类似的细化，getNewForm（）将HTTP方法和路径的定义合并为一个，以便该方法处理`appointments/new`的GET请求。
getForDay（）方法显示了@RequestMapping的另一种用法：URI模板。

不需要类级别的@RequestMapping。没有它，所有路径都是绝对的，而不是相对的。以下来自PetClinic示例应用程序的示例显示了使用@RequestMapping的多动作控制器：
```java
@Controller
public class ClinicController {

    private final Clinic clinic;

    @Autowired
    public ClinicController(Clinic clinic) {
        this.clinic = clinic;
    }

    @RequestMapping("/")
    public void welcomeHandler() {
    }

    @RequestMapping("/vets")
    public ModelMap vetsHandler() {
        return new ModelMap(this.clinic.getVets());
    }

}
```
上面的示例未指定GET与PUT，POST等，因为@RequestMapping默认映射所有HTTP方法。使用`@RequestMapping(method = GET)`或`@GetMapping`缩小映射范围。

#### 组成的 @RequestMapping 变体

`Spring Framework 4.3 `引入了以下@RequestMapping注释的方法级组合变体，这些变体有助于简化常见HTTP方法的映射，并更好地表达带注释的处理程序方法的语义。例如，@GetMapping可以读作GET @RequestMapping。

- @GetMapping
- @PostMapping
- @PutMapping
- @DeleteMapping
- @PatchMapping

```java
@Controller
@RequestMapping("/appointments")
public class AppointmentsController {

    private final AppointmentBook appointmentBook;

    @Autowired
    public AppointmentsController(AppointmentBook appointmentBook) {
        this.appointmentBook = appointmentBook;
    }

    @GetMapping
    public Map<String, Appointment> get() {
        return appointmentBook.getAppointmentsForToday();
    }

    @GetMapping("/{day}")
    public Map<String, Appointment> getForDay(@PathVariable @DateTimeFormat(iso=ISO.DATE) Date day, Model model) {
        return appointmentBook.getAppointmentsForDay(day);
    }

    @GetMapping("/new")
    public AppointmentForm getNewForm() {
        return new AppointmentForm();
    }

    @PostMapping
    public String add(@Valid AppointmentForm appointment, BindingResult result) {
        if (result.hasErrors()) {
            return "appointments/new";
        }
        appointmentBook.addAppointment(appointment);
        return "redirect:/appointments";
    }
}

```

#### @Controller and AOP Proxying

在某些情况下，控制器可能需要在运行时使用AOP代理进行修饰。一个例子是如果您选择直接在控制器上使用`@Transactional`注释。在这种情况下，对于控制器而言，我们建议使用基于类的代理。这通常是控制器的默认选择。但是，如果控制器必须实现不是Spring Context回调的接口（例如InitializingBean，*Aware等），则可能需要显式配置基于类的代理。例如，使用`<tx：annotation-driven/>`，更改为`<tx：annotation-driven proxy-target-class =“true”/>`。

#### Spring MVC 3.1中@RequestMapping方法的新支持类

Spring 3.1为@RequestMapping方法引入了一组新的支持类，分别称为`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。建议使用它们，甚至需要利用Spring MVC 3.1中的新功能并继续使用。默认情况下，MVC命名空间和MVC Java配置启用新的支持类，但如果两者都不使用，则必须明确配置。本节介绍旧支持类和新支持类之间的一些重要差异。

在Spring 3.1之前，在两个单独的阶段中检查类型和方法级请求映射 - 首先通过`DefaultAnnotationHandlerMapping`选择控制器，然后通过`AnnotationMethodHandlerAdapter`缩小调用的实际方法。

使用Spring 3.1中的新支持类，RequestMappingHandlerMapping是唯一决定应该处理请求的方法的地方。将控制器方法视为唯一端点的集合，其中每个方法的映射都是从类型和方法级别的@RequestMapping信息派生的。

这实现了一些新的可能性。一旦HandlerInterceptor或HandlerExceptionResolver现在可以期望基于对象的处理程序是HandlerMethod，它允许它们检查确切的方法，它的参数和相关的注释。不再需要跨不同的控制器分割URL的处理。

还有一些事情不再可能：
- 首先使用SimpleUrlHandlerMapping或BeanNameUrlHandlerMapping选择控制器，然后根据@RequestMapping注释缩小方法。
- 依赖于方法名作为后退机制来消除两个没有显式路径映射URL路径但是否相同匹配的@RequestMapping方法之间的歧义，例如，通过HTTP方法。在新的支持类中，必须唯一地映射@RequestMapping方法。
- 如果没有其他控制器方法更具体地匹配，则使用单个默认方法（没有显式路径映射）处理请求。在新的支持类中，如果找不到匹配的方法，则会引发404错误。

现有支持类仍支持上述功能。但是，要利用新的Spring MVC 3.1功能，您需要使用新的支持类。

#### URI模板模式

URI模板可用于在@RequestMapping方法中方便地访问URL的选定部分。

URI模板是类似URI的字符串，包含一个或多个变量名称。当您为这些变量替换值时，模板将成为URI。建议的URI模板RFC定义了如何参数化URI。例如，URI模板`http://www.example.com/users/{userId}`包含变量userId。将值fred分配给变量会产生`http://www.example.com/users/fred。`

在Spring MVC中，您可以在方法参数上使用`@PathVariable`批注将其绑定到URI模板变量的值：
```java
@GetMapping("/owners/{ownerId}")
public String findOwner(@PathVariable String ownerId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    model.addAttribute("owner", owner);
    return "displayOwner";
}
```

URI模板`/owners/{ownerId}`指定变量名称ownerId。当控制器处理此请求时，ownerId的值将设置为URI的相应部分中找到的值。例如，当一个请求进入`/owners/fred`时，ownerId的值为fred。

也可以这样：
```java
@GetMapping("/owners/{ownerId}")
public String findOwner(@PathVariable("ownerId") String theOwner, Model model) {
    // implementation omitted
}
```

方法可以包含任意数量的@PathVariable注释：
```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public String findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
    Owner owner = ownerService.findOwner(ownerId);
    Pet pet = owner.getPet(petId);
    model.addAttribute("pet", pet);
    return "displayPet";
}
```

在Map <String，String>参数上使用@PathVariable批注时，将使用所有URI模板变量填充映射。

可以从类型和方法级别@RequestMapping注释组装URI模板。因此，可以使用诸如`/owners/42/pets/21`之类的URL调用findPet()方法。
```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @RequestMapping("/pets/{petId}")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // implementation omitted
    }

}
```

`@PathVariable参数`可以是任何简单类型，如int，long，Date等。如果无法执行此操作，Spring会`自动转换为适当的类型`或`抛出TypeMismatchException`。您还可以注册支持解析其他数据类型。请参阅“方法参数和类型转换”一节以及“自定义WebDataBinder初始化”一节。

#### 具有正则表达式的URI模板模式

有时您需要更精确地定义URI模板变量。考虑一下URL`/spring-web/spring-web-3.0.5.jar`

@RequestMapping批注支持在URI模板变量中使用正则表达式。语法是{varName:regex}，其中`第一部分定义变量名称`，第二部分`定义正则表达式`。例如：
```java
@RequestMapping("/spring-web/{symbolicName:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{extension:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String extension) {
    // ...
}
```

#### 路径模式

除了URI模板之外，`@RequestMappin`g注释和所有组合的`@RequestMapping变体`也支持Ant样式的路径模式（例如，`/myPath/*.do`）。
还支持URI模板变量和Ant样式globs的组合（例如`/owners/*/pets/{petId}`）。

#### 路径模式比较

当URL匹配多个模式时，使用排序来查找最具体的匹配。

具有较低URI变量和通配符计数的模式被认为更具体。例如`/hotels/{hotel}/*`有1个URI变量和1个外卡，被认为比`/hotels/{hotel}/**`更具体，它们是1个URI变量和2个外卡。

如果两个模式具有相同的计数，则更长的模式被认为更具体。例如`/foo/bar*`比`/foo/*`更长并且被认为更具体。

当两个模式具有相同的计数和长度时，具有较少通配符的模式被认为更具体。例如`/hotels/{hotel}`比`/hotels/*`更具体。

还有一些额外的特殊规则：
- 默认映射模式`/**`不如任何其他模式具体。例如`/api/{a}/{b}/{c}`更具体。
- 诸如`/public/**`之类的前缀模式不如任何其他不包含双通配符的模式特定。例如`/public/path3/{a}/{b}/{c}`}更具体。

有关完整详细信息，请参阅AntPathMatcher中的AntPatternComparator。请注意，PathMatcher可以自定义（请参阅配置Spring MVC一节中的第22.16.11节“路径匹配”）。

#### 带占位符的路径模式

@RequestMapping注释中的模式支持针对本地属性和/或系统属性和环境变量的$ {...}占位符。在控制器映射到的路径可能需要通过配置进行自定义的情况下，这可能很有用。有关占位符的更多信息，请参阅PropertyPlaceholderConfigurer类的javadoc。

#### 后缀模式匹配

默认情况下，Spring MVC执行`".*"`后缀模式匹配，以便映射到`/person`的控制器也隐式映射到`/person.*`。这使得通过URL路径（例如`/person.pdf`,`/person.xml`）请求资源的不同表示变得容易。

可以关闭或限制后缀模式匹配，以便为内容协商目的明确注册一组路径扩展。通常建议使用通用请求映射来最小化歧义，例如/ person / {id}，其中点可能不代表文件扩展名，例如，`/person/joe@email.com` vs `/person/joe@email.com.json`。此外，如下面的注释中所解释的，后缀模式匹配以及内容协商可能在某些情况下用于尝试恶意攻击，并且有充分的理由对其进行有意义的限制。

有关后缀模式匹配配置，请参见第22.16.11节“路径匹配”，对于内容协商配置，请参见第22.16.6节“内容协商”。

#### Suffix Pattern Matching and RFD

https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-requestmapping-rfd

#### 矩阵变量

URI规范[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)定义了在路径段中包含名称 - 值对的可能性。规范中没有使用特定术语。可以应用一般的“URI路径参数”，尽管源自Tim Berners-Lee的旧帖子的更独特的“矩阵URI”也经常被使用并且是众所周知的。在Spring MVC中，这些被称为矩阵变量。

矩阵变量可以出现在任何路径段中，每个矩阵变量用“;”分隔（分号）。例如：`"/cars;color=red;year=2012"`多个值可以是“，”（逗号）分隔`"color=red,green,blue"`或者可以重复变量名称`"color=red;color=green;color=blue`

如果URL预计包含矩阵变量，则请求映射模式必须使用URI模板表示它们。这确保了请求可以正确匹配，无论是否存在矩阵变量以及它们的提供顺序。

下面是提取矩阵变量“q”的示例：
```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11

}
```

由于所有路径段都可能包含矩阵变量，因此在某些情况下，您需要更具体地确定变量的预期位置：
```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22

}
```

矩阵变量可以定义为可选，并指定默认值：
```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1

}
```

所有矩阵变量都可以在Map中获得：
```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId"") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 11, "s" : 23]

}
```

请注意，要启用矩阵变量，`必须将RequestMappingHandlerMapping的removeSemicolonContent属性设置为false`。默认情况下，它设置为true。

---

MVC Java配置和MVC命名空间都提供了启用矩阵变量的选项。如果您使用的是Java配置，则“使用MVC Java配置进行高级自定义”部分介绍了如何自定义RequestMappingHandlerMapping。在MVC命名空间中，<mvc：annotation-driven>元素具有应该设置为true的`enable-matrix-variables属性`。默认情况下，它设置为false。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven enable-matrix-variables="true"/>

</beans>
```

#### Consumable Media Types

您可以通过指定`可使用`的介质类型列表来缩小主映射。仅当`Content-Type请求头`与指定的媒体类型匹配时，才会匹配请求。例如：
```java
@PostMapping(path = "/pets", consumes = "application/json")
public void addPet(@RequestBody Pet pet, Model model) {
    // implementation omitted
}
```
Consumable media type 表达式也可以否定为 !text/plain 匹配除了那些以外的所有请求 Content-Type of text/plain
还要考虑使用`MediaType`中提供的常量，例如 `APPLICATION_JSON_VALUE`和`APPLICATION_JSON_UTF8_VALUE`。

> The consumes condition is supported on the type and on the method level. Unlike most other conditions, when used at the type level, method-level consumable types override rather than extend type-level consumable types.

#### Producible Media Types

您可以通过指定`可生成`的媒体类型列表来缩小主映射。仅当`Accept请求头`与其中一个值匹配时，才会匹配请求。此外，使用生成条件可确保用于生成响应的实际内容类型遵循生成条件中指定的媒体类型。例如：
```java
@GetMapping(path = "/pets/{petId}", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public Pet getPet(@PathVariable String petId, Model model) {
    // implementation omitted
}
```

> 请注意，生成条件中指定的介质类型也可以选择指定字符集。例如，在上面的代码片段中，我们指定的媒体类型与`MappingJackson2HttpMessageConverter`中配置的默认媒体类型相同，包括UTF-8字符集。

就像使用消费一样，可生成的媒体类型表达式可以在`text/plain`中被否定，以匹配除了具有`text/plain`的`Accept标头`值的请求之外的所有请求。
还要考虑使用`MediaType`中提供的常量，例如`APPLICATION_JSON_VALUE`和`APPLICATION_JSON_UTF8_VALUE`。

> 类型和方法级别支持生成条件。与大多数其他条件不同，在类型级别使用时，方法级可生成类型会覆盖而不是扩展类型级可生成类型。

#### Request Parameters and Header Values

您可以通过请求参数条件缩小请求匹配，例如“myParam”，“!myParam”或​​“myParam = myValue”。前两个测试请求参数存在/不存在，第三个测试特定参数值。以下是请求参数值条件的示例：
```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @GetMapping(path = "/pets/{petId}", params = "myParam=myValue")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // implementation omitted
    }

}
```

可以执行相同的操作来测试请求标头的存在/不存在，或者根据特定的请求标头值进行匹配：
```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

    @GetMapping(path = "/pets", headers = "myHeader=myValue")
    public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {
        // implementation omitted
    }

}
```

> 虽然您可以使用媒体类型通配符匹配`Content-Type`和`Accept标头值`（例如“`content-type = text/*`”将匹配“`text/plain`”和“`text/html`”），但建议使用分别代替消费和生产条件。它们专门用于此目的。

#### HTTP HEAD and HTTP OPTIONS

映射到“GET”的@RequestMapping方法也隐式映射到“`HEAD`”，即不需要显式声明“HEAD”。处理HTTP HEAD请求就像它是HTTP GET一样，除了不写入主体，只计算字节数并设置“Content-Length”标头。

@RequestMapping方法内置了对`HTTP OPTIONS`的支持。默认情况下，通过将“`Allow`”响应标头设置为在具有匹配URL模式的所有@RequestMapping方法上显式声明的HTTP方法来处理HTTP OPTIONS请求。当没有显式声明HTTP方法时，“Allow”标头设置为“GET，HEAD，POST，PUT，PATCH，DELETE，OPTIONS”。理想情况下，始终声明@RequestMapping方法要处理的HTTP方法，或者使用专用的@RequestMapping变体之一（请参阅“Composed @RequestMapping Variants”一节）。

虽然没有必要，但@RequestMapping方法可以映射到并处理HTTP HEAD或HTTP OPTIONS，或两者兼而有之。

### 22.3.3 定义@RequestMapping处理器方法

@RequestMapping处理程序方法可以具有非常灵活的签名。支持的方法参数和返回值将在下一节中介绍。大多数参数可以按任意顺序使用，唯一的例外是`BindingResult`参数。这将在下一节中介绍。

> Spring 3.1为@RequestMapping方法引入了一组新的支持类，分别称为`RequestMappingHandlerMapping`和`RequestMappingHandlerAdapter`。建议使用它们，甚至需要利用Spring MVC 3.1中的新功能并继续使用。默认情况下，从MVC命名空间启用新的支持类，并使用MVC Java配置，但如果不使用，则必须明确配置。

#### 支持的方法参数类型

以下是受支持的方法参数：
- 请求或响应对象（Servlet API）。选择任何特定的请求或响应类型，例如ServletRequest或HttpServletRequest。
- 会话对象（Servlet API）：类型为HttpSession。此类型的参数强制存在相应的会话。因此，这样的论证永远不会是空的。

> 会话访问可能不是线程安全的，特别是在Servlet环境中。如果允许多个请求同时访问会话，请考虑将RequestMappingHandlerAdapter的“synchronizeOnSession”标志设置为“true”。

详见[资料](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-arguments)

Errors或BindingResult参数必须遵循立即绑定的模型对象，因为方法签名可能有多个模型对象，Spring将为每个模型对象创建一个单独的BindingResult实例，因此以下示例将不起作用：

BindingResult和@ModelAttribute的排序无效。

```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, Model model, BindingResult result) { ... }

```

注意，Pet和BindingResult之间有一个Model参数。要使其正常工作，您必须按如下方式重新排序参数：
```java
@PostMapping
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result, Model model) { ... }

```

支持JDK 1.8的java.util.Optional作为方法参数类型，其注释具有`required属性`（例如@RequestParam，@RequestHeader等。在这些情况下使用java.util.Optional等同于具有required=false。

#### 支持的方法返回类型

以下是支持的返回类型：

https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-return-types

#### 使用@RequestParam将请求参数绑定到方法参数

以下代码段显示了用法：
```java
@Controller
@RequestMapping("/pets")
@SessionAttributes("pet")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, ModelMap model) {
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```
默认情况下，使用此注解的参数是必需的，但您可以通过将@RequestParam的`required属性`设置为`false`来指定参数是可选的
(e.g., @RequestParam(name="id", required=false)).

如果目标方法参数类型不是String，则自动应用类型转换。请参阅“[方法参数和类型转换](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-typeconversion)”一节。

在`Map<String，String>`或`MultiValueMap<String，String>`参数上使用@RequestParam注释时，将使用所有请求参数填充Map。


#### 使用@RequestBody注解映射请求正文

@RequestBody方法参数注释指示应将方法参数绑定到HTTP请求正文的值。例如：
```java
@PutMapping("/something")
public void handle(@RequestBody String body, Writer writer) throws IOException {
    writer.write(body);
}
```

您可以使用`HttpMessageConverter`将请求主体转换为方法参数。HttpMessageConverter负责从HTTP请求消息转换为对象并从对象转换为HTTP响应主体。`RequestMappingHandlerAdapter`支持带有以下默认`HttpMessageConverters`的`@RequestBody`注解：
- ByteArrayHttpMessageConverter 
- StringHttpMessageConverter 
- FormHttpMessageConverter  -- 将表单数据转换为MultiValueMap <String，String>。
- SourceHttpMessageConverter  --  转换为/从javax.xml.transform.Source转换
有关这些转换器的更多信息，请参阅[消息转换器](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#rest-message-conversion)。另请注意，如果使用MVC命名空间或MVC Java配置，则默认情况下会注册更多范围的消息转换器。有关更多信息，[请参见第22.16.1节“启用MVC Java配置或MVC XML命名空间”](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-config-enable)。

如果您打算读写XML，则需要使用`org.springframework.oxm`包中的特定Marshaller和Unmarshaller实现配置`MarshallingHttpMessageConverter`。下面的示例显示了如何在配置中直接执行此操作，但如果您的应用程序是通过MVC命名空间或MVC Java配置配置的，[请参见第22.16.1节“启用MVC Java配置或MVC XML命名空间”](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-config-enable)。
```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <util:list id="beanList">
            <ref bean="stringHttpMessageConverter"/>
            <ref bean="marshallingHttpMessageConverter"/>
        </util:list>
    </property
</bean>

<bean id="stringHttpMessageConverter"
        class="org.springframework.http.converter.StringHttpMessageConverter"/>

<bean id="marshallingHttpMessageConverter"
        class="org.springframework.http.converter.xml.MarshallingHttpMessageConverter">
    <property name="marshaller" ref="castorMarshaller"/>
    <property name="unmarshaller" ref="castorMarshaller"/>
</bean>

<bean id="castorMarshaller" class="org.springframework.oxm.castor.CastorMarshaller"/>

```

@RequestBody方法参数可以使用`@Valid`注释，在这种情况下，它将使用配置的`Validator实例进行验证`。使用MVC命名空间或MVC Java配置时，假设JSR-303实现在类路径上可用，则会自动配置JSR-303验证器。

与@ModelAttribute参数一样，可以使用`Errors`参数来检查错误。如果未声明此类参数，则将引发`MethodArgumentNotValidException`。异常在`DefaultHandlerExceptionResolver`中处理，它将400错误发送回客户端。

> 有关通过MVC命名空间或MVC Java配置配置消息转换器和验证器的信息，另请参见第22.16.1节“启用MVC Java配置或MVC XML命名空间”。

#### 使用@ResponseBody注释映射响应正文

@ResponseBody注释类似于@RequestBody。此注释可以放在方法上，并指示应将返回类型直接写入HTTP响应主体（而不是放在模型中，或解释为视图名称）。例如：
```java
@GetMapping("/something")
@ResponseBody
public String helloWorld() {
    return "Hello World";
}
```
上面的示例将导致文本Hello World被写入HTTP响应流。

与@RequestBody一样，Spring使用`HttpMessageConverter`将返回的对象转换为响应主体。有关这些转换器的更多信息，请参阅上一节和消息转换器。

#### 使用@RestController批注创建REST控制器

控制器实现REST API是一个非常常见的用例，因此只提供JSON，XML或自定义MediaType内容。为方便起见，您可以使用`@RestController`注释控制器类，而不是使用@ResponseBody注释所有@RequestMapping方法。

@RestController是一个结合@ResponseBody和@Controller的构造型注解。更重要的是，它为您的Controller提供了更多的意义，并且可能在框架的未来版本中带来额外的语义。

与常规@Controllers一样，`@ControlAdtroller`或`@RestControllerAdvice` bean可以协助`@RestController`。有关更多详细信息，请参阅“[使用@ControllerAdvice和@RestControllerAdvice建议控制器](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#mvc-ann-controller-advice)”一节。

#### 使用HttpEntity

HttpEntity类似于@RequestBody和@ResponseBody。除了访问请求和响应主体之外，HttpEntity（以及特定于响应的子类ResponseEntity）还允许访问请求和响应头，如下所示：
```java
@RequestMapping("/something")
public ResponseEntity<String> handle(HttpEntity<byte[]> requestEntity) throws UnsupportedEncodingException {
    String requestHeader = requestEntity.getHeaders().getFirst("MyRequestHeader"));
    byte[] requestBody = requestEntity.getBody();

    // do something with request header and body

    HttpHeaders responseHeaders = new HttpHeaders();
    responseHeaders.set("MyResponseHeader", "MyValue");
    return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED);
}
```

上面的示例获取MyRequestHeader请求标头的值，并将主体作为字节数组读取。它将MyResponseHeader添加到响应中，将Hello World写入响应流，并将响应状态代码设置为201（已创建）。

与@RequestBody和@ResponseBody一样，Spring使用HttpMessageConverter来转换请求和响应流。有关这些转换器的更多信息，请参阅上一节和消息转换器。

#### 在方法上使用@ModelAttribute

@ModelAttribute批注可用于方法或方法参数。本节解释了它在方法上的用法，下一节解释了它在方法参数上的用法。

方法上的`@ModelAttribute`指示该方法的目的是添加一个或多个模型属性。此类方法支持与@RequestMapping方法相同的参数类型，但不能直接映射到请求。而是在同一个控制器中的@RequestMapping方法之前调用控制器中的@ModelAttribute方法。几个例子：
```java
// Add one attribute
// The return value of the method is added to the model under the name "account"
// You can customize the name via @ModelAttribute("myAccount")

@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountManager.findAccount(number);
}

// Add multiple attributes

@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountManager.findAccount(number));
    // add more ...
}
```

@ModelAttribute方法用于使用常用属性填充模型，例如填充状态或宠物类型的下拉列表，或者检索像Account这样的命令对象，以便使用它来表示HTML表单上的数据。后一种情况将在下一节进一步讨论。

请注意两种样式的@ModelAttribute方法。在第一个中，该方法通过返回隐式添加属性。在第二种方法中，该方法接受一个Model并向其添加任意数量的模型属性。您可以根据需要在两种样式之间进行选择。

控制器可以包含任意数量的@ModelAttribute方法。在同一控制器的@RequestMapping方法之前调用所有这些方法。

@ModelAttribute方法也可以在@ControllerAdvice-annotated类中定义，这些方法适用于许多控制器。有关更多详细信息，请参阅“使用@ControllerAdvice和@RestControllerAdvice建议控制器”一节。

> 未明确指定模型属性名称时会发生什么？在这种情况下，会根据模型属性的类型为模型属性分配默认名称。例如，如果方法返回Account类型的对象，则使用的默认名称为“account”。您可以通过@ModelAttribute注释的值更改它。如果直接向Model添加属性，请使用适当的重载addAttribute（..）方法 - 即使用或不使用属性名称。

@ModelAttribute注释也可用于@RequestMapping方法。在这种情况下，@RequestMapping方法的返回值被解释为模型属性而不是视图名称。然后根据视图名称约定派生视图名称，就像返回void的方法一样 - 请参见第22.13.3节“默认视图名称”。

#### 在方法参数上使用@ModelAttribute

如前一节所述，@ModelAttribute可用于方法或方法参数。本节介绍其在方法参数上的用法。

方法参数上的@ModelAttribute指示应从模型中检索参数。如果模型中不存在，则应首先实例化参数，然后将其添加到模型中。一旦出现在模型中，参数的字段应该从具有匹配名称的所有请求参数中填充。这在Spring MVC中称为数据绑定，这是一种非常有用的机制，可以使您不必单独解析每个表单字段。
```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { }
```

鉴于以上示例，Pet实例可以来自何处？有几种选择：
- 由于使用@SessionAttributes，它可能已经在模型中 - 请参阅“使用@SessionAttributes在请求之间的HTTP会话中存储模型属性”一节。
- 由于同一控制器中的@ModelAttribute方法，它可能已经在模型中 - 如上一节中所述。
- 可以基于URI模板变量和类型转换器来检索它（下面更详细地解释）。
- 它可以使用其默认构造函数进行实例化。

@ModelAttribute方法是从数据库中检索属性的常用方法，可以选择通过使用@SessionAttributes在请求之间存储该属性。在某些情况下，使用URI模板变量和类型转换器检索属性可能很方便。这是一个例子：
```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```
在此示例中，模型属性的名称（即“account”）与URI模板变量的名称匹配。如果您注册可以将String帐户值转换为Account实例的Converter <String，Account>，那么上面的示例将无需@ModelAttribute方法即可运行。

下一步是数据绑定。`WebDataBinder`类将请求参数名称（包括查询字符串参数和表单字段）与名称模型属性字段进行匹配。在必要时应用类型转换（从String到目标字段类型）后填充匹配字段。第9章“验证，数据绑定和类型转换”中介绍了数据绑定和验证。自定义控制器级别的数据绑定过程在“自定义WebDataBinder初始化”一节中介绍。

由于数据绑定，可能存在错误，例如缺少必填字段或类型转换错误。要检查此类错误，请在`@ModelAttribute参数`后面添加一个`BindingResult参数`：
```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```

使用BindingResult，您可以检查是否发现了错误，在这种情况下，通常会在Spring的<errors>表单标记的帮助下呈现错误。

请注意，在某些情况下，在没有数据绑定的情况下访问模型中的属性可能很有用。对于这种情况，您可以将模型注入控制器，或者在注释上使用绑定标志：
```java
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountUpdateForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) {

    // ...
}
```

除了数据绑定之外，您还可以使用自己的自定义验证程序调用验证，该验证程序传递用于记录数据绑定错误的相同BindingResult。这允许数据绑定和验证错误在一个地方累积，然后报告给用户：
```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) {

    new PetValidator().validate(pet, result);
    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```
或者，您可以通过添加JSR-303 @Valid注释自动调用验证：

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) {

    if (result.hasErrors()) {
        return "petForm";
    }

    // ...

}
```
有关如何配置和使用验证的详细信息，请参见第9.8节“Spring验证”和第9章，验证，数据绑定和类型转换。

#### 使用@SessionAttributes在请求之间的HTTP会话中存储模型属性

类型级`@SessionAttributes`注释声明特定处理程序使用的会话属性。这通常会列出模型属性的名称或模型属性的类型，这些属性应该透明地存储在会话或某些会话存储中，作为后续请求之间的表单支持bean。

以下代码段显示了此批注的用法，指定了模型属性名称：
```java
@Controller
@RequestMapping("/editPet.do")
@SessionAttributes("pet")
public class EditPetForm {
    // ...
}
```

#### 使用@SessionAttribute访问预先存在的全局会话属性

如果您需要访问全局管理的预先存在的会话属性，即在控制器外部（例如通过过滤器），并且可能存在或不存在，请在方法参数上使用@SessionAttribute注释：
```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) {
    // ...
}
```
对于需要添加或删除会话属性的用例，请考虑将`org.springframework.web.context.request.WebRequest`或`javax.servlet.http.HttpSession`注入控制器方法。

对于作为控制器工作流的一部分在会话中临时存储模型属性，请考虑使用SessionAttributes，如“使用@SessionAttributes在请求之间的HTTP会话中存储模型属性”一节中所述。

#### 使用@RequestAttribute访问请求属性

与@SessionAttribute类似，@RequestAttribute注释可用于访问由过滤器或拦截器创建的预先存在的请求属性：
```java
@RequestMapping("/")
public String handle(@RequestAttribute Client client) {
    // ...
}
```


#### 使用“application / x-www-form-urlencoded”数据

前面几节介绍了使用@ModelAttribute来支持来自浏览器客户端的表单提交请求。建议将相同的注释用于非浏览器客户端的请求。但是，在处理HTTP PUT请求时，有一个显着的区别。浏览器可以通过HTTP GET或HTTP POST提交表单数据。非浏览器客户端也可以通过HTTP PUT提交表单。这提出了一个挑战，因为Servlet规范要求ServletRequest.getParameter *（）系列方法仅支持HTTP POST的表单字段访问，而不支持HTTP PUT。

为了支持HTTP PUT和PATCH请求，spring-web模块提供了过滤器HttpPutFormContentFilter，可以在web.xml中配置：
```xml
<filter>
    <filter-name>httpPutFormFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpPutFormFilter</filter-name>
    <servlet-name>dispatcherServlet</servlet-name>
</filter-mapping>
```
上面的过滤器截取内容类型为`application/x-www-form-urlencoded`的HTTP PUT和PATCH请求，从请求正文中读取表单数据，并包装ServletRequest以便通过ServletRequest.getParameter使表单数据可用*（）方法系列。
> 由于HttpPutFormContentFilter使用请求的主体，因此不应为依赖于`application/x-www-form-urlencoded`的其他转换器的PUT或PATCH URL配置它。这包括@RequestBody MultiValueMap <String，String>和HttpEntity <MultiValueMap <String，String >>。

#### 使用@CookieValue注释映射cookie值

@CookieValue批注允许将方法参数绑定到HTTP cookie的值。

让我们考虑以下cookie已收到http请求：
```
JSESSIONID=415A4AC178C59DACE0B2C9CA727CDD84
```

以下代码示例演示了如何获取JSESSIONID cookie的值：
```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@CookieValue("JSESSIONID") String cookie) {
    //...
}
```

如果目标方法参数类型不是String，则自动应用类型转换。请参阅“方法参数和类型转换”一节。

Servlet和Portlet环境中带注释的处理程序方法支持此注释。

#### 使用@RequestHeader注释映射请求头属性

@RequestHeader注释允许将方法参数绑定到请求标头。这是一个示例请求标头：
```
Host                    localhost:8080
Accept                  text/html,application/xhtml+xml,application/xml;q=0.9
Accept-Language         fr,en-gb;q=0.7,en;q=0.3
Accept-Encoding         gzip,deflate
Accept-Charset          ISO-8859-1,utf-8;q=0.7,*;q=0.7
Keep-Alive              300

```

以下代码示例演示了如何获取Accept-Encoding和Keep-Alive标头的值：
```java
@RequestMapping("/displayHeaderInfo.do")
public void displayHeaderInfo(@RequestHeader("Accept-Encoding") String encoding,
        @RequestHeader("Keep-Alive") long keepAlive) {
    //...
}
```

如果method参数不是String，则自动应用类型转换。请参阅“方法参数和类型转换”一节。

在Map <String，String>，MultiValueMap <String，String>或HttpHeaders参数上使用@RequestHeader注释时，将使用所有标头值填充映射。

内置支持可用于将逗号分隔的字符串转换为字符串数组/集合或类型转换系统已知的其他类型。例如，使用@RequestHeader（“Accept”）注释的方法参数可以是String类型，也可以是String []或List <String>。

Servlet和Portlet环境中带注释的处理程序方法支持此注释。

#### 方法参数和类型转换

从请求中提取的基于字符串的值（包括请求参数，路径变量，请求标头和cookie值）可能需要转换为方法参数或字段的目标类型（例如，将请求参数绑定到@ModelAttribute中的字段参数）他们受到约束。如果目标类型不是String，则Spring会自动转换为适当的类型。支持所有简单类型，如int，long，Date等。您可以通过WebDataBinder进一步自定义转换过程（请参阅“自定义WebDataBinder初始化”一节）或使用FormattingConversionService注册Formatters（请参见第9.6节“Spring Field Formatting”）。

#### 自定义WebDataBinder初始化

要通过Spring的`WebDataBinder`自定义与PropertyEditors的请求参数绑定，您可以在控制器中使用`@InitBinder`-annotated方法，在@ControllerAdvice类中使用@InitBinder方法，或者提供自定义WebBindingInitializer。有关更多详细信息，请参阅“使用@ControllerAdvice和@RestControllerAdvice建议控制器”一节。

#### 使用@InitBinder自定义数据绑定

使用@InitBinder注释控制器方法允许您直接在控制器类中配置Web数据绑定。@InitBinder标识初始化WebDataBinder的方法，该方法将用于填充带注释的处理程序方法的命令和表单对象参数。

这样的init-binder方法支持@RequestMapping方法支持的所有参数，命令/表单对象和相应的验证结果对象除外。Init-binder方法不能有返回值。因此，它们通常被宣布为无效。典型的参数包括WebDataBinder与WebRequest或java.util.Locale的组合，允许代码注册特定于上下文的编辑器。

以下示例演示如何使用@InitBinder为所有java.util.Date表单属性配置CustomDateEditor。
```java
@Controller
public class MyFormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

或者，从Spring 4.2开始，考虑使用addCustomFormatter来指定Formatter实现而不是PropertyEditor实例。如果您碰巧在共享的FormattingConversionService中具有基于Formatter的设置，则此功能特别有用，并且可以将相同的方法重用于特定于控制器的绑定规则调整。
```java
@Controller
public class MyFormController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

#### 配置自定义WebBindingInitializer

要外部化数据绑定初始化，您可以提供WebBindingInitializer接口的自定义实现，然后通过为AnnotationMethodHandlerAdapter提供自定义Bean配置来启用它，从而覆盖默认配置。

以下来自PetClinic应用程序的示例显示了使用WebBindingInitializer接口的自定义实现的配置，org.springframework.samples.petclinic.web.ClinicBindingInitializer，它配置了几个PetClinic控制器所需的PropertyEditors。
```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="cacheSeconds" value="0"/>
    <property name="webBindingInitializer">
        <bean class="org.springframework.samples.petclinic.web.ClinicBindingInitializer"/>
    </property>
</bean>
```

@InitBinder方法也可以在@ControllerAdvice-annotated类中定义，在这种情况下，它们适用于匹配的控制器。这提供了使用WebBindingInitializer的替代方法。有关更多详细信息，请参阅“使用@ControllerAdvice和@RestControllerAdvice建议控制器”一节。

#### 使用@ControllerAdvice和@RestControllerAdvice为控制器提供建议

@ControllerAdvice注释是一个组件注释，允许通过类路径扫描自动检测实现类。使用MVC命名空间或MVC Java配置时会自动启用它。

使用`@ControllerAdvice`注释的类可以包含`@ExceptionHandler`，`@InitBinder`和`@ModelAttribute`带注释的方法，这些方法将应用于跨所有控制器层次结构的@RequestMapping方法，而不是声明它们的控制器层次结构。

@RestControllerAdvice是一种替代方法，其中@ExceptionHandler方法默认采用@ResponseBody语义。

@ControllerAdvice和@RestControllerAdvice都可以定位控制器的子集：
```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class AnnotationAdvice {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class BasePackageAdvice {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class AssignableTypesAdvice {}
```
查看@ControllerAdvice文档以获取更多详细信息。

#### Jackson 序列化视图支持

有时可以将上下文中将序列化为HTTP响应主体的对象进行过滤。为了提供这样的功能，Spring MVC内置支持使用Jackson的序列化视图进行渲染。

要将它与@ResponseBody控制器方法或返回ResponseEntity的控制器方法一起使用，只需添加带有类参数的@JsonView批注，该类参数指定要使用的视图类或接口：
```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

> 请注意，尽管@JsonView允许指定多个类，但控制器方法的使用仅支持一个类参数。如果需要启用多个视图，请考虑使用复合接口。

对于依赖于视图分辨率的控制器，只需将序列化视图类添加到模型中：
```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

#### Jackson JSONP Support

为了对@ResponseBody和ResponseEntity方法启用JSONP支持，声明一个扩展AbstractJsonpResponseBodyAdvice的@ControllerAdvice bean，如下所示，其中构造函数参数指示JSONP查询参数名称：
```java
@ControllerAdvice
public class JsonpAdvice extends AbstractJsonpResponseBodyAdvice {

    public JsonpAdvice() {
        super("callback");
    }
}
```

对于依赖于视图解析的控制器，当请求具有名为jsonp或callback的查询参数时，将自动启用JSONP。这些名称可以通过jsonpParameterNames属性进行自定义。

> 从Spring Framework 4.3.18开始，不推荐使用JSONP支持，从Spring Framework 5.1开始，将删除JSONP支持，而应使用CORS。

### 22.3.4异步请求处理

Spring MVC 3.2引入了基于Servlet 3的异步请求处理。像往常一样，控制器方法现在可以返回`java.util.concurrent.Callable`而不是返回值，并从Spring MVC托管线程生成返回值。同时，主要的Servlet容器线程被退出并释放，并允许处理其他请求。Spring MVC在TaskExecutor的帮助下在一个单独的线程中调用Callable，当Callable返回时，请求被调度回Servlet容器，以使用Callable返回的值恢复处理。以下是此类控制器方法的示例：
```java
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };

}
```

另一种选择是控制器方法返回DeferredResult的实例。在这种情况下，返回值也将从任何线程产生，即不由Spring MVC管理的线程。例如，可以响应于诸如JMS消息，计划任务等的一些外部事件来产生结果。以下是此类控制器方法的示例：
```java
@RequestMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// In some other thread...
deferredResult.setResult(data);
```

如果不了解Servlet 3.0异步请求处理功能，这可能很难理解。阅读这一点肯定会有所帮助。以下是有关基础机制的一些基本事实：
- 可以通过调用request.startAsync（）将ServletRequest置于异步模式。这样做的主要作用是Servlet以及任何过滤器都可以退出，但响应将保持打开状态以允许稍后完成处理。
- 对request.startAsync（）的调用返回AsyncContext，可用于进一步控制异步处理。例如，它提供方法dispatch，类似于来自Servlet API的转发，但它允许应用程序在Servlet容器线程上恢复请求处理。
- ServletRequest提供对当前DispatcherType的访问，该DispatcherType可用于区分处理初始请求，异步调度，转发和其他调度程序类型。

考虑到上述情况，以下是使用Callable进行异步请求处理的事件序列：
- Controller返回Callable。
- Spring MVC启动异步处理并将Callable提交给TaskExecutor，以便在单独的线程中进行处理。
- DispatcherServlet和所有Filter都退出Servlet容器线程，但响应仍保持打开状态。
- Callable生成一个结果，Spring MVC将请求调度回Servlet容器以恢复处理。
- 再次调用DispatcherServlet，并使用Callable的异步生成结果继续处理。

DeferredResult的序列非常相似，除了应用程序从任何线程产生异步结果：
- Controller返回DeferredResult并将其保存在可以访问它的某个内存中队列或列表中。
- Spring MVC启动异步处理。
- DispatcherServlet和所有已配置的Filter都退出请求处理线程，但响应仍保持打开状态。
- 应用程序从某个线程设置DeferredResult，Spring MVC将请求调度回Servlet容器。
- 再次调用DispatcherServlet，并使用异步生成的结果继续处理。

有关异步请求处理的动机和何时或为何使用它的进一步背景，请阅读[此博客文章系列](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support)。

#### 异步请求的异常处理

如果从控制器方法返回的Callable在执行时引发异常会发生什么？简短回答与控制器方法引发异常时发生的情况相同。它通过常规异常处理机制。更长的解释是，当一个Callable引发一个Exception Spring MVC调度到Servlet容器时，Exception作为结果，并导致使用Exception而不是控制器方法返回值继续请求处理。使用DeferredResult时，您可以选择是否使用Exception实例调用setResult或setErrorResult。

#### 拦截异步请求

HandlerInterceptor还可以实现AsyncHandlerInterceptor以实现afterConcurrentHandlingStarted回调，当异步处理开始时，该回调被调用而不是postHandle和afterCompletion。

HandlerInterceptor还可以注册CallableProcessingInterceptor或DeferredResultProcessingInterceptor，以便更深入地集成异步请求的生命周期，例如处理超时事件。有关更多详细信息，请参阅AsyncHandlerInterceptor的Javadoc。

DeferredResult类型还提供诸如onTimeout（Runnable）和onCompletion（Runnable）之类的方法。有关更多详细信息，请参阅DeferredResult的Javadoc。

使用Callable时，您可以使用WebAsyncTask实例来包装它，该实例还提供了超时和完成的注册方法。

#### HTTP流媒体

控制器方法可以使用DeferredResult和Callable异步生成其返回值，并且可以用于实现诸如长轮询之类的技术，其中服务器可以尽快将事件推送到客户端。

如果您想在单个HTTP响应上推送多个事件，该怎么办？这是与“长轮询”相关的技术，称为“HTTP流”。Spring MVC通过`ResponseBodyEmitter`返回值类型实现了这一点，该类型可用于发送多个对象，而不是像@ResponseBody那样通常的情况，其中发送的每个Object都使用HttpMessageConverter写入响应。

这是一个例子：
```java
@RequestMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

请注意，ResponseBodyEmitter也可以用作ResponseEntity中的主体，以便自定义响应的状态和标头。

#### 使用服务器发送事件的HTTP流式传输

`SseEmitter`是`ResponseBodyEmitter`的子类，为Server-Sent Events提供支持。服务器发送的事件是同一“HTTP Streaming”技术的另一个变体，除了从服务器推送的事件是根据W3C服务器发送事件规范格式化的。

服务器发送事件可用于其预期目的，即将事件从服务器推送到客户端。在Spring MVC中很容易做到，只需要返回一个SseEmitter类型的值。

但请注意，Internet Explorer不支持服务器发送事件，对于更高级的Web应用程序消息传递方案（如在线游戏，协作，财务应用程序等），最好考虑Spring的WebSocket支持，包括SockJS样式的WebSocket仿真，可以追溯到各种各样的浏览器（包括Internet Explorer）以及更高级别的消息传递模式，用于通过更多以消息传递为中心的体系结构中的发布 - 订阅模型与客户端进行交互。有关此问题的进一步背景，请[参阅以下博客文章](https://blog.pivotal.io/pivotal/products/websocket-architecture-in-spring-4-0)。

#### HTTP流直接到OutputStream

ResponseBodyEmitter允许通过HttpMessageConverter将对象写入响应来发送事件。这可能是最常见的情况，例如在编写JSON数据时。但是，有时绕过消息转换并直接写入响应OutputStream（例如文件下载）很有用。这可以在StreamingResponseBody返回值类型的帮助下完成。

```java
@RequestMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

请注意，StreamingResponseBody也可以用作ResponseEntity中的主体，以便自定义响应的状态和标头。

#### 配置异步请求处理



#### Servlet容器配置

对于使用web.xml配置的应用程序，请确保更新到3.0版：
```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    ...

</web-app>
```

必须通过web.xml中的`<async-supported>true</async-supported>`子元素在DispatcherServlet上启用异步支持。此外，任何参与asyncrequest处理的Filter都必须配置为支持ASYNC调度程序类型。为Spring Framework提供的所有过滤器启用ASYNC调度程序类型应该是安全的，因为它们通常扩展OncePerRequestFilter并且具有运行时检查过滤器是否需要参与异步调度。

下面是一些web.xml配置示例：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
            http://java.sun.com/xml/ns/javaee
            http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <filter>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <filter-class>org.springframework.~.OpenEntityManagerInViewFilter</filter-class>
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>Spring OpenEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>REQUEST</dispatcher>
        <dispatcher>ASYNC</dispatcher>
    </filter-mapping>

</web-app>
```

如果使用Servlet 3，基于Java的配置，例如通过WebApplicationInitializer，您还需要设置“asyncSupported”标志以及ASYNC调度程序类型，就像使用web.xml一样。要简化所有这些配置，请考虑扩展AbstractDispatcherServletInitializer或更好的AbstractAnnotationConfigDispatcherServletInitializer，它会自动设置这些选项并使注册Filter实例变得非常容易。


#### Spring MVC配置

MVC Java配置和MVC命名空间提供了配置异步请求处理的选项。WebMvcConfigurer具有configureAsyncSupport方法，而`<mvc:annotation-driven>`具有`<async-support>`子元素。

这些允许您配置用于异步请求的默认超时值，如果未设置，则取决于底层Servlet容器（例如，Tomcat上的10秒）。您还可以配置AsyncTaskExecutor以用于执行从控制器方法返回的Callable实例。强烈建议配置此属性，因为默认情况下Spring MVC使用SimpleAsyncTaskExecutor。MVC Java配置和MVC命名空间还允许您注册CallableProcessingInterceptor和DeferredResultProcessingInterceptor实例。

如果需要覆盖特定DeferredResult的默认超时值，则可以使用适当的类构造函数来执行此操作。类似地，对于Callable，您可以将其包装在WebAsyncTask中，并使用适当的类构造函数来自定义超时值。WebAsyncTask的类构造函数还允许提供AsyncTaskExecutor。

### 22.3.5 Testing Controllers

The spring-test module offers first class support for testing annotated controllers. See [Section 15.6, “Spring MVC Test Framework”](https://docs.spring.io/spring/docs/4.3.22.RELEASE/spring-framework-reference/htmlsingle/#spring-mvc-test-framework).

---

云谁之思？美孟桂矣。
