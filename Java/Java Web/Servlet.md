# Servlet

Servlet是运行在Web或应用服务器上的程序。一个Servlet就是一个Java类，并且可以通过“请求-响应”编程模型来访问的这个驻留在服务器内存里的Servlet程序。

作为在来自Web浏览器或其他HTTP客户机的请求和在HTTP服务器上的数据库或应用程序的中间层，Servlet可以通过Web页面表单来收集用户的输入，显示从数据库或其他来源的记录，动态地创建Web页面。

## Servlet容器

Servlet容器也叫做Servlet引擎，是Web服务器或应用程序服务器的一部分，用于在发送的请求和 响应之上提供网络服务，解码基于MIME的请求，格式化基于MIME的响应。

Servlet没有`main()`方法，不能独立运行，它必须被部署到Servlet容器中，由容器来实例化和调用Servlet的方法（如`doGet()`和`doPost()`），Servlet容器在Servlet的生命周期内包容和管理Servlet。在JSP技术推出后，管理和运行Servlet/JSP的容器也称为Web容器。

1. 有了Servlet之后，用户通过单击某个链接或者直接在浏览器的地址栏中输入URL来访问Servlet。
2. Web服务器接收到该请求后，并不是将请求直接交给Servlet，而是交给Servlet容器。
3. Servlet容器实例化Servlet，调用Servlet的一个特定方法对请求进行处理，并产生一个响应。
4. 这个响应由Servlet容器返回给Web服务器，Web服务器包装这个响应，以HTTP响应的形式发送给Web浏览器。

## Tomcat

Tomcat是一个免费的开放源代码的Servlet容器，它是Apache软件基金会（Apache Software Foundation）的一个顶级项目，由Apache、Sun和其他一些公司及个人共同开发而成。由于有了Sun的参与和支持，最新的Servlet和JSP规范总是能在Tomcat中得到体现，Tomcat 6支持最新的Servlet 2.5和JSP 2.1规范。因为Tomcat技术先进、性能稳定，而且免费，因而深受Java爱好者的喜爱，并得到了部分软件开发商的认可，成为目前比较流行的Web服务器。

Tomcat和IIS、Apache等Web服务器一样，具有处理HTML页面的功能，另外它还是一个Servlet和JSP容器，独立的Servlet容器是Tomcat的默认模式。不过，Tomcat处理静态HTML的能力不如Apache，我们可以将Apache和Tomcat集成在一起使用，Apache作为HTTP Web服务器，Tomcat作为Web容器。

### Tomcat服务器接受客户请求并做出响应的过程

1. 客户端（通常都是浏览器）访问Web服务器，发送HTTP请求。
2. Web服务器接收到请求后，传递给Servlet容器。
3. Servlet容器加载Servlet，产生Servlet实例后，向其传递表示请求和响应的对象。
4. Servlet实例使用请求对象得到客户端的请求信息，然后进行相应的处理。
Servlet实例将处理结果通过响应对象发送回客户端，容器负责确保响应正确送出，同时将控制返回给Web服务器。

### Tomcat体系结构

Tomcat服务器是由一系列可配置的组件构成的，其中核心组件是Catalina Servlet容器，它是所有其他Tomcat组件的顶层容器。我们可以通过查看Tomcat安装文件夹下的conf文件夹中的server.xml文件 来了解Tomcat各组件之间的层次关系。由于server.xml注释太多，特简化如下：

```xml
<?xml version='1.0' encoding='utf-8'?>
<Server port="8005" shutdown="SHUTDOWN">
    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1"
                   connectionTimeout="20000"
                   redirectPort="8443"
                   URIEncoding="UTF-8"/>

        <Engine name="Catalina" defaultHost="localhost">
            <Host name="localhost">
                <Context path="" docBase="WORK_DIR" reloadable="true"/>
            </Host>
        </Engine>
    </Service>
</Server>
```

其中WORK_DIR为你想要导入的项目路径。

- Server：表示整个的Catalina Servlet容器。Tomcat提供了Server接口的一个默认实现，这通常不需要用户自己去实现。在Server容器中，可以包含一个或多个Service组件。
- Service：Service是存活在Server内部的中间组件，它将一个或多个连接器（Connector）组件绑定到一个单独的引擎（Engine）上。在Server中，可以包含一个或多个Service组件。Service也很少由用户定制，Tomcat提供了Service接口的默认实现，而这种实现既简单又能满足应用。
- Connector：连接器（Connector）处理与客户端的通信，它负责接收客户请求，以及向客户返回响应结果。在Tomcat中，有多个连接器可以使用。
- Engine：在Tomcat中，每个Service只能包含一个Servlet引擎（Engine）。引擎表示一个特定的Service的请求处理流水线。作为一个Service可以有多个连接器，引擎从连接器接收和处理所有的请求，将响应返回给适合的连接器，通过连接器传输给用户。用户允许通过实现Engine接口提供自定义的引擎，但通常不需要这么做。
- Host：一个Host表示一个虚拟主机，一个引擎可以包含多个Host。用户通常不需要创建自定义的Host，因为Tomcat给出的Host接口的实现（类StandardHost）提供了重要的附加功能。
- Context：一个Context表示了一个Web应用程序，运行在特定的虚拟主机中。
  - Web应用程序：在Sun公司发布的Java Servlet规范中，对Web应用程序做出了如下的定义：“一个Web应 用程序是由一组Servlet、HTML页面、类，以及其他的资源组成的运行在Web服务器上的完整的应用程序。它可以在多个供应商提供的实现了Servlet规范的Web容器中运行”。
  - 一个Host可以包含多个Context（代表Web应用程序），每一个Context都有一个唯一的路径。
  - 用户通常不需要创建自定义的Context，因为Tomcat给出的Context接口的实现（类`StandardContext`）提供了重要的附加功能。

容器等级：

![Tomcat容器等级](../../resource/img/Java%20Web/Tomcat容器等级.png)

## Servlet工作过程和生命周期

### Servlet生命周期

Servlet的生命周期通过`java.servlet.Servlet`接口中的`init()`、`service()`、和 `destroy()`方法表示。Servlet的生命周期有四个阶段：加载并实例化、初始化、请求处理、销毁。

1. 加载并实例化：Servlet容器负责加载和实例化Servlet。
    - 当Servlet容器启动时，或者在容器检测到需要这个Servlet来响应第一个请求时，创建Servlet实例。
    - 当Servlet容器启动后，Servlet通过类加载器来加载Servlet类，加载完成后再`new`一个Servlet对象来完成实例化。
2. 初始化：在Servlet实例化之后，容器将调用`init()`方法，并传递实现`ServletConfig`接口的对象。在`init()`方法中，Servlet可以部署描述符中读取配置参数，或者执行任何其他一次性活动。在Servlet的整个生命周期类，`init()`方法只被调用一次。
3. 请求处理：当Servlet初始化后，容器就可以准备处理客户机请求了。
    - 当容器收到对这一Servlet的请求，就调用Servlet的`service()`方法，并把请求和响应对象作为参数传递。
    - 当并行的请求到来时，多个`service()`方法能够同时运行在独立的线程中。通过分析`ServletRequest`或者`HttpServletRequest`对象，`service()`方法处理用户的请求，并调用`ServletResponse`或者`HttpServletResponse`对象来响应。
4. 销毁：一旦Servlet容器检测到一个Servlet要被卸载，这可能是因为要回收资源或者因为它正在被关闭，容器会在所有Servlet的`service()`线程之后，调用Servlet的`destroy()`方法。然后，Servlet就可以进行无用存储单元收集清理。这样Servlet对象就被销毁了。

![Servlet生命周期](../../resource/img/Java%20Web/Servlet生命周期.png)

### Servlet工作过程

![Servlet工作过程](../../resource/img/Java%20Web/Servlet工作过程.png)

## JSP生命周期和工作过程

理解JSP底层功能的关键就是去理解它们所遵守的生命周期。

JSP生命周期就是从创建到销毁的整个过程，类似于servlet生命周期，区别在于JSP生命周期还包括将JSP文件编译成`Servlet`。

### JSP生命周期

1. 解析阶段：Servlet容器解析JSP文件代码，如果有语法错误，就会向客户端返回错误信息
2. 翻译阶段：Servlet容器把JSP文件翻译成Servlet源文件
3. 编译阶段：Servlet容器编译Servlet源文件，生成servlet
4. 初始化阶段：加载与JSP对应的Servlet类，创建其实例，并调用它的初始化方法
5. 运行时阶段：调用与JSP对应的Servlet实例的服务方法
6. 销毁阶段：调用与JSP对应的Servlet实例的销毁方法，然后销毁Servlet实例

### JSP工作过程

1. JSP编译：当浏览器请求JSP页面时，JSP引擎会首先去检查是否需要编译这个文件。如果这个文件没有被编译过，或者在上次编译后被更改过，则编译这个JSP文件。编译的过程包括三个步骤：
    - 解析JSP文件。
    - 将JSP文件转为servlet。
    - 编译servlet。
2. JSP初始化：容器载入JSP文件后，它会在为请求提供任何服务前调用`JSPInit()`方法。如果您需要执行自定义的JSP初始化任务，复写该方法就行了。一般来讲程序只初始化一次，Servlet也是如此。通常情况下您可以在`JSPInit()`方法中初始化数据库连接、打开文件和创建查询表。
3. JSP执行：这一阶段描述了JSP生命周期中一切与请求相关的交互行为，直到被销毁。当JSP网页完成初始化后，JSP引擎将会调用`_JSPService()`方法。该方法需要一个`HttpServletRequest`对象和一个`HttpServletResponse`对象作为它的参数，`_JSPService()`方法在每个request中被调用一次并且负责产生与之相对应的response，并且它还负责产生所有7个HTTP方法的回应，比如GET、POST、DELETE等等。
4. JSP清理：JSP生命周期的销毁阶段描述了当一个JSP网页从容器中被移除时所发生的一切。`JSPDestroy()` 方法在JSP中等价于servlet中的销毁方法。当您需要执行任何清理工作时复写该方法，比如释放数据库连接或者关闭文件夹等等。

## Servlet类中获取JSP内置对象

req、resp为传入参数，表示请求和响应，方法为基类方法。

- `out` -> `resp.getWriter()`
- `request` -> `req`
- `response` -> `resp`
- `session` -> `req.getSession()`
- `application` -> `getServletContext()`
- `exception` -> `Throwable`
- `page` -> `this`
- `pageConfig` -> `PageContext`
- `config` -> `getServletConfig()`

## Servlet配置

### web.xml配置

```xml
<servlet>
    [<description>message</description>]
    [<display-name>displayName<display-name>]
    <servlet-name>servletName</servlet-name>
    [<servlet-class>package.ClassName</servlet-class>
        |<jsp-file>/demo.jsp</jsp-file>]
    [<init-param>
        <param-name>paramName</param-name>
        <param-value>paramValue</param-value>
    </init-param>]
    [<load-on-startup>int</load-on-startup>]
    [<async-supported>true|false</async-supported>]
</servlet>
<servlet-mapping>
    <servlet-name>servletName</servlet-name>
    <url-pattern>URL</url-pattern>
</servlet-mapping>
```

- `servlet`
  - `servlet-name`：用于为Servlet指定一个名字，该元素的内容不能为空。
  - `servlet-class`：用于指定Servlet的完整限定类名。
  - `jsp-file`：用于指定Servlet的决定路径，开头必须为`/`。
  - `init-param`：指定初始化参数。在Servlet类中，可以使用`ServletConfig`接口对象来访问初始化参数。
    - `param-name`：参数的名字
    - `param-value`：指定参数的值
  - `load-on-startup`：当值为0或者大于0时，表示容器在应用启动时就加载这个Servlet；指定加载顺序，数字越小，优先级越高。当是一个负数时或者没有指定时，则指示容器在该Servlet被选择时才加载。
  - `async-support`：是否支持异步处理
- `servlet-mapping`
  - `servlet-name`：指定Servlet名，同Servlet标签中值
  - `url-pattern`：指定访问Servlet的URL

#### `url-pattern`匹配规则

- 省缺表示`/`，即`http://localhost:8080/ContextPath`的请求URL
- `/demo/index.jsp`/`/servletName`可以精确匹配`http://localhost:8080/ContextPath/demo/index.jsp`/`http://localhost:8080/ContextPath/servletName`的请求URL
- `*`表示通配符时
- 以`/`字符开头，并以`/*`结尾的字符串用于路径匹配，例如`/demo/*`可以匹配类似`http://localhost:8080/ContextPath/demo/index.html`的URL请求。如果可以匹配多个路径，那么以最长的为结果。
- 以`*.`开头的字符串则被用于扩展名匹配。例如`*.jsp`可以匹配扩展名为jsp的任意请求，`*`前面不能有东西，不能和路径匹配一起用，比如`/demo/*.jsp`是非法的。
- 如果请求URL没有匹配到servlet，容器会根据URL选择对应的请求资源。如果应用定义了一个default servlet，则容器会将请求丢给default servlet。

#### `url-pattern`匹配顺序

精确匹配 -> 路径匹配（长路径 -> 短路径） -> 扩展匹配 -> 默认匹配

对于filter，不会像servlet那样只匹配一个servlet，因为filter的集合是一个链，所以只会有处理的顺序不同，而不会出现只选择一个filter。Filter的处理顺序和filter-mapping在web.xml中定义的顺序相同。

#### `url-pattern`中的`\*` & `\`

- `/*`属于路径匹配，并且可以匹配所有request，由于路径匹配的优先级仅次于精确匹配，所以`/*`会覆盖所有的扩展名匹配，很多404错误均由此引起，所以这是一种特别恶劣的匹配模式，一般只用于filter的url-pattern
- `/`是servlet中特殊的匹配模式，切该模式有且仅有一个实例，优先级最低，不会覆盖其他任何url-pattern，只是会替换servlet容器的内建default servlet，该模式同样会匹配所有request。

[servlet和filter的url-pattern匹配规则详细描述](https://juejin.im/post/5af3b6cf518825671d20939a)

### 使用`@WebServlet`注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface WebServlet {
    String name() default ""; //名称

    String[] value() default {}; // 同urlPatterns，优先级更高

    String[] urlPatterns() default {};

    int loadOnStartup() default -1;

    WebInitParam[] initParams() default {};

    boolean asyncSupported() default false;

    String smallIcon() default "";

    String largeIcon() default "";

    String description() default "";

    String displayName() default "";
}
```

## 异步Servlet

```java
// 获取异步上下文
AsyncContext asyncCtx = req.startAsync();

// 设置异步超时时间（millisecond），默认30000。
// 在指定的时间内没有执行完异步操作，response依然会依据Servlet的结束逻辑
// 后续的异步操作执行完再写回的时候，可能会遇到异常
asyncCtx.setTimeout(10000);

// 异步执行任务：
//     1：AsyncContext调用start方法
//     2：开启线程
asyncCtx.start(Runnable);
```

[servlet3异步原理与实践](https://www.jianshu.com/p/c23ca9d26f64)

## JSP架构模型

### JSP Model1

传统的JSP Model1模型，JSP是独立的，自主完成所有的任务，如图：

![传统Model1](../../resource/img/Java%20Web/传统Model1.png)

改进的JSP Model1模型，JSP页面与JavaBeans共同协作完成任务，如图：

![改进Model1](../../resource/img/Java%20Web/改进Model1.png)

Model 1模式的实现比较简单，适用于快速开发小规模项目。但从工程化的角度看，它的局限性非常明显：JSP页面身兼View和Controller两种角色，将控制逻辑和表现逻辑混杂在一起，从而导致代码的重用性非常低，增加了应用的扩展性和维护的难度。

早期有大量ASP和JSP技术开发出来的Web应用，这些Web应用都采用了Model1架构。

### JSP Model 2

JSP Model2中使用了三种技术JSP、Servlet和Java Beans。

- Java Beans：负责业务逻辑，对数据库的操作。（M）
- JSP：负责生成动态网页，只用做显示页面。（V）
- Servlet：负责流程控制，用来处理各种请求的分派。（C）

![Model2](../../resource/img/Java%20Web/Model2.png)

交互过程：用户通过浏览器向Web应用中的Servlet发送请求，Servlet接受到请求后实例化Java Beans对象，调用Java Beans对象的方法，JavaBeans对象返回从数据库中读取的数据。Servlet选择合适JSP，并且把从数据库中读取的数据通过这个JSP进行显示，最后JSP页面把最终的结果返回给浏览器。

Model2已经是MVC设计思想下的架构，由于引入了MVC模式，使Model2具有组件化的特点，更适用于大规模应用的开发，但也增加了应用开发的复杂程度。

### MVC模式

- 模型层——Model：是应用程序的核心部分。由JavaBean组件来充当，可以是一个实体对象，或者一种业务逻辑，之所以成为模型，是因为它在应用程序中，有更好的重用性和扩展性
- 视图层——View：提供应用程序与用户之间的交互界面。由JSP或者HTML界面充当，在MVC架构中，这一层并不包含任何的业务逻辑，仅仅提供一种与用户相交互的视图
- 控制层——Controller：用于对程序中的请求进行控制。由Servlet来充当，起到一种宏观调控的作用，它可以通知容器选择什么样的视图，什么样的模型组件

[Servlet工作原理解析](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/)
