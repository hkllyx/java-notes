- [概述](#概述)
- [重要动作](#重要动作)
  - [useBean动作](#usebean-动作)
    - [Java Beans](#java-beans)
      - [编写规范](#编写规范)
      - [编写要求](#编写要求)
      - [作用区域](#作用区域)
      - [Java beans实例查找](#java-beans-实例查找)
    - [useBean执行步骤](#usebean-执行步骤)
    - [useBean总结](#usebean-总结)
  - [setProperty动作](#setproperty-动作)
  - [getProperty动作](#getproperty-动作)
  - [include动作](#include-动作)
    - [与include指令比较](#与-include-指令比较)
  - [forward动作](#forward-动作)
  - [plugin动作](#plugin-动作)
  - [element动作](#element-动作)
  - [attribute动作](#attribute-动作)

# 概述

JSP动作元素（action elements），动作元素为请求处理阶段提供信息。动作元素遵循XML元素的语法，有一个包含元素名的开始标签，可以有属性、可选的内容、与开始标签匹配的结束标签。

- 与存储JavaBean有关的（3个）：
    ```
    <jsp:useBean>
    <jsp:setProperty>
    <jsp:getProperty>
    ```
- JSP 1.2开始有的基本元素（6个）：
    ```
    <jsp:include>
    <jsp:forward>
    <jsp:param>
    <jsp:plugin>
    <jsp:params>
    <jsp:fallback>
    ```
- JSP 2.0新增元素，主要与JSP Document有关（6个）：
    ```
    <jsp:root>
    <jsp:declaration>
    <jsp:scriptlet>
    <jsp:expression>
    <jsp:text>
    <jsp:output>
    ```
- JSP 2.0新增元素，主要用来动态生成XML标签的值（3个）：
    ```
    <jsp:attribute>
    <jsp:body>
    <jsp:element>
    ```
- JSP 2.0新增元素，主要用在Tag File中（2个）：
    ```
    <jsp:invoke>
    <jsp:dobody>
    ```

# 重要动作

## useBean动作

```
<jsp:useBean id="beanInstanceName"
    scope="page|request|session|application"
    [ class="package.class"|
        class="package.class" type="package.class"|
        type="package.class" |
        type="package.class"|
        beanName="package.class | <%= expression%>" ]>

    [<jsp:setProperty.../>]
</jsp:useBean>
```
- **id** 命名引用该Bean的变量。如果能够找到id和scope相同的Bean实例，useBean动作将使用已有的Bean实例而不是创建新的实例。
- **scope** 指定Bean在哪种上下文内可用，可以取下面的四个值之一：page，request，session和application。
- **class** 指定Bean的完整包名。
- **type** 指定引用该对象的变量的类型，它必须是Bean类的名字、超类名字、该类所实现的接口名字之一。 请记住变量的名字是由id属性指定的。
- **beanName** 指定Bean的名字。如果提供了type属性和beanName属性，允许省略class属性。

### Java Beans

Java Beans就是符合某种特定的规范的Java类。使用Java Beans的好处是解决代码重复编写，减少代码冗余，功能区分明确，提高了代码的维护性。

Java Bean是基于Java的组件模型，由属性、方法和事件3部分组成。在该模型中，JavaBean可以被修改或与其他组件结合以生成新组件或完整的程序。它是一种Java类，通过封装成为具有某种功能或者处理某个业务的对象。因此，也可以通过嵌在JSP页面内的Java代码访问Bean及其属性。

Bean的含义是可重复使用的Java组件。所谓组件就是一个由可以自行进行内部管理的一个或几个类所组成、外界不了解其内部信息和运行方式的群体。使用它的对象只能通过接口来操作。

#### 编写规范

- Java Beans实际上是根据JavaBean技术标准所指定Bean的命名和设计规范编写的Java类。
- 这些类遵循一个接口格式，以便于使函数命名、底层行为以及继承或实现的行为，其最大的优点在于可以实现代码的可重用性。Bean并不需要继承特别的基类或实现特定的接口。Bean的编写规范使Bean的容器能够分析一个Java类文件，并将其方法翻译成属性 (Properties)，即把Java类作为一个Bean类使用。Bean的编写规范包括Bean类的构造 方法、定义属性和访问方法编写规则

#### 编写要求

- 所有的JavaBean必须放在一个包 (Package) 中。
- JavaBean必须生成public class类，文件名称应该与类名称一致。
- 所有属性必须封装，一个JavaBean类不应有公共实例变量，类变量都为private。
- 属性值应该通过一组存取方法（getXxx和setXxx）来访问：对于每个属性，应该有一个带 匹配公用getter和setter方法的专用实例变量。
- Java Bean类必须有一个空的构造函数：类中必须有一个不带参数的公用构造器，此构造器 也应该通过调用各个属性的设置方法来设置属性的默认值。

#### 作用区域

- page
    - 与当前页面相对应，JavaBean的生命周期存在于一个页面之中，当页面关闭时JavaBean被销毁。

- request
    - 与JSP的request生命周期相对应，JavaBean的生命周期存在于request对象之中， 当request对象销毁时JavaBean也被销毁。
    - 使用 `<jsp:forward>` 重定向或使用 `<jsp:include>` 动作导入jsp程序时，定义的对象会被传到下一个程序中，下一个程序可以任意调用此对象的内容。
    - 可以通过 `HttpRequest.getAttribute(...)` 方法获取javabean对象。

- session
    - 与JSP的session生命周期相对应，JavaBean的生命周期存在于session会话之中， 当session超时或会话结束时JavaBean被销毁。
    - 可以通过 `HttpSession.getAttribute(...)` 方法获取javabean对象。

- application
    - 与JSP的application生命周期相对应，在各个用户与服务器之间共享，只有当服务器 关闭时JavaBean才被销毁。
    - 可以通过 `ServletContext.getAttribute(...)` 方法获取javabean对象。

#### Java beans实例查找

当JavaBean被创建后，通过 `<jsp:setProperty>` 与 `<jsp:getProperty>` 调用时，将会按照page、request、session和application的顺序来查找这个JavaBean实例， 直至找到一个实例对象为止，如果在这4个范围内都找不到JavaBean实例，则抛出异常。

### useBean执行步骤

元素用来定位或初始化一个JavaBeans组件。首先会尝试定位Bean实例，如果其不存在，则会依据class名称 (class属性指定) 或序列化模板 (beanName属性指定) 进行实例化。进行定位或初始化Bean对象时，<jsp:useBean > 按照以下步骤执行。

1. 尝试在scope属性指定的作用域使用你指定的名称 (id属性值) 定位Bean对象；
2. 使用指定的名称 (id属性值) 定义一个引用类型变量；
3. 假如找到Bean对象，将其引用给步骤2定义的变量。假如你指定类型 (type属性)，赋予该Bean对象该类型；
4. 假如没找到，则实例化一个新的Bean对象，并将其引用给步骤2定义的变量。假如该类名 (由beanName属性指定的类名) 代表的是一个序列化模板 (serialized template)，该Bean对象由 `java.beans.Beans.instantiate()` 方法初始化；
5. 假如useBean此次是实例化Bean对象而不是定位Bean对象，且它有体标记 (body tags) 或元素 (位于useBean标签之间的内容，则执行该体标记，比如 `<jsp:setProperty>`。

### useBean总结

- scope在属性当中非常重要，是因为useBean只有在不存在具有相同id和scope的对象时才 会实例化新的对象；如果已有id和scope都相同的对象则直接使用已有的对象，此时useBean开始标记和结束标记之间的任何内容都将被忽略。
- class和beanName不能同时存在；它们用来指定实例化的类名称或序列化模板；如果确信JavaBean对象已存在，class和beanName属性均可无须指定，可只须指定type属性。
- class可以省去type独自存在，beanName必须和type一起使用；class指定的是类名，beanName指定的是类名或序列化模板的名称；class指定的类名必须包含public、无参的构造方法；在对象已实例化时，beanName指定的名称可以为其接口、父类；
- class或beanName指定的类名必须包括包名，type可以省去包名，通过 `<%@page import="..."%>` 指定所属包亦可。
- 如果JavaBean对象已存在，useBean只是用来定位JavaBean，则只需使用type属性即可，class和beanName这时舍去不影响使用。
- class通过 `new` 创建JavaBean对象；beanName通过 `java.beans.Beans.instantiate()` 方法初始化JavaBean对象。

## setProperty动作

setProperty用来和useBean一起协作。设置已经实例化的Bean对象的属性.

```
<jsp:setProperty name="beanInstanceName"
    property="beanPropertyName"
    [value="beanPropertyValue" |
        param="requestParameter"]
/>
```
- **name** 属性是必需的。它表示要设置属性的是哪个Bean。
- **property** 属性是必需的。它表示要设置哪个属性。有一个特殊用法：如果property的值是 "*"， 表示所有名字和Bean属性名字匹配的请求参数都将被传递给相应的属性set方法。
- **value** 属性是可选的。该属性用来指定Bean属性的值。字符串数据会在目标类中通过标准的 `valueOf(...)` 方法自动转换成数字、boolean、Boolean、byte、Byte、char、Character。
- **param** 是可选的。它指定用哪个请求参数作为Bean属性的值。如果当前请求没有参数，则什么事情 也不做，系统不会把null传递给Bean属性的set方法。因此，你可以让Bean自己提供默认属性值， 只有当请求参数明确指定了新值时才修改默认属性值。
- value和param不能同时使用，但可以使用其中任意一个。value是自定义属性的值，param是将请 求参数（比如前端表单数据）作为值注入到该property中。

## getProperty动作

```
<jsp:getProperty name="beanInstanceName"
    property="beanPropertyName" .../>
```
- **name**：要检索的Bean属性名称。Bean必须已定义。
- **property**：表示要提取Bean属性的值。要求获取该属性值的方法名为 `getBeanPropertyName()`，既get + 属性名（首字母大写）。

## include动作

```
<jsp:include page="RELATIVE-URL" flush="true" />
```

- **page**: 包含在页面中的相对URL地址
- **flush**: 布尔属性，定义在包含资源前是否刷新缓存区

### 与include指令比较

- 编译成servlet区别
    - include指令：我们知道JSP文件本身就是servlet，然而JSP文件需要预先编译成servlet， 也就是class文件，而 `<%@ include %>` 页面jsp在请求之前就已经进行了预编译，所有代码包含进来之后，一起进行处理，把所有代码合在一起，编译成一个servlet！
    - include动作：所有JSP文件代码分别处理，是在页面被请求的时候才进行编译，将多个jsp文 件编译成多个servlet，页面语法相对独立，处理完成之后再将代码的显示结果（处理结果）组合进来。如果被包含文件是动态的，那么就会生成两个Servlet，也就是被包含文件也要经过jsp引擎编译执行生成一个Servlet，两个Servlet通过request和response进行通信。如果被包含的文件是静态的，那么这种情况和 `<%@ include %>` 就很相似，只生成了一个Servlet，但是他们之间没有进行简单的嵌入，而依然是通过request和response进行的通信。

- 用法区别
    - include指令：当JSP转换成Servlet时引入指定文件，`<%@ include file="head.jsp" %>`
    - include动作：当JSP页面被请求时引入指定文件，`<jsp:include page="head.jsp"/>`

- 使用include动作还是include指令
    - 使用include指令，如果被包含的文件发生改变，那么，用到它的所有Jsp页面都需要更新。仅当include动作不能满足要求时，我们才应该使用include指令。有些开发人员认为include指令生成的代码执行起来比使用include动作的代码更快。尽管原则上由可能的确如此，但性能上的差异很小，以致难以测量，同时，include动作在维护上的优势十分巨大，当两种方法都可以使用时，include动作几乎肯定是首选的方法。
    - 对于文件包含，应该尽可能地使用include动作。仅在所包含的文件中定义了主页面要用到的字段或方法，或所包含的文件设置了主页面的响应报头时，才应该使用include指令。
    - include指令比include动作更加强大。include指令允许所包含的文件中含有影响主页面的Jsp代码，比如响应报头的设置和字段、方法的定义。

- 总结
    - include指令可以在JSP页面转换成Servlet之前，将JSP代码插入其中。它的主要优点是功能强大，所包含的代码可以含有总体上影响主页面的JSP构造，比如属性、方法的定义和文档类型的设定。它的缺点是难于维护，只要被包含的页面发生更改，就得更改主页面，这是因为主页面不会自动地查看被包含的页面是否发生更改。
    - include动作是在主页面被请求时，将次级页面的输出包含进来。尽管被包含的页面的输出中不能含有JSP，但这些页面可以是其他资源所产生的结果。服务器按照正常的方式对指向被包含资源的URL进行解释，因而这个URL可以是Servlet或JSP页面。服务器以通常的方式运行被包含的页面，将产生的输出放到主页面中，这种方式与RequestDispatcher类的include方法一致。它的优点是在被包含的页面发生更改时，无须对主页面做出修改。它的缺点是所包含的是次级页面的输出，而非次级页面的实际代码，所以在被包含的页面中不能使用任何有可能在整体上影响主页面的JSP构造。

## forward动作

```
<jsp:forward page="RELATIVE-URL|<%= expression %>">
    [<jsp:param name="paramName" value="paramValue" .../>]
</jsp:forward>
```
- 从该指令处停止当前页面的继续执行，而转向其他的一个JSP页面。
- **page** 是你将要定向的文件或URL. 这个文件可以是JSP, 程序段，或者其它能够处理request对象的文件 (如jsp,cgi,php)。

总结
jsp:forward执行时，用户请求的地址URL依然没有发生变化，但页面内容却完全被forward目标页的内容。
jsp:forward动作转发请求时，客户端的请求参数不会丢失。
jsp:forward动作之后的代码是不会执行的。

## plugin动作

用来产生客户端浏览器的特别标签（object或embed），可以使用它来插入Applet或JavaBean。如果需要的插件不存在，它会下载插件，然后执行Java组件。Java组件可以是一个applet或一个JavaBean。

```
<jsp:plugin
    type="bean|applet"
    code="classFileName"
    codebase="classFileDirectoryName"
    [name="instanceName"]
    [archive="URIToArchive, ..."]
    [align="bottom|top|middle|left|right"]
    [height="displayPixels"]
    [width="displayPixels"]
    [hspace="leftRightPixels"]
    [vspace="topBottomPixels"]
    [jreversion="JREVersionNumber|1.1"]
    [nspluginurl="URLToPlugin"]
    [iepluginurl="URLToPlugin"]>
    [<jsp:params>
        <jsp:param name="parameterName" value="parameterValue|<%= expression%>"/>
    </jsp:params>]
    [<jsp:fallback>text message for user</jsp:fallback>]
</jsp:plugin>
```
- **Width**：Applet在Html页面上的宽度
- **Height**：Applet在Html页面上的高度
- **Name**：Applet在Html页面上的名称，用于区名一个Html页面上的多个Applet
- **Code**：Applet类名，必须带后缀class。当没有属性archive时，直接写类名当有属性archive时，必须带包名。
- **Codebase**：Applet的类相对路径，相对于Html页面位置
- **Archive**：Applet所在Jar包的文件名

## element动作

动态定义XML元素。动态是非常重要的，这就意味着XML元素在编译时是动态生成的而非静态。

```
<jsp:element name="">
    <jsp:attribute>…</jsp:attribute>
    <jsp:body>…</jsp:body>
    ...
</jsp:element>
```
- jsp:element中可以包含jsp:attribute和jsp:body, 它只有一个属性name。name的值就是XML元素标签的名称。

## attribute动作

当使用在jsp:element之中时，设置动态定义的XML元素属性。

```
<jsp:attribute trim="true|false">
    ......
</jsp:attribute >
```

它有两个属性：**name** 和 **trim**。
- 其中name的值就是标签的属性名称。
- trim可为true或false。假若为true时， `<jsp:attribute>` 本体内容的前后空白，将被忽略；反之，若为false，前后空白将不被忽略。trim的默认值为true。