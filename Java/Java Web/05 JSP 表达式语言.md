- [概述](#概述)
  - [语法](#语法)
  - [运算符](#运算符)
  - [隐式对象](#隐式对象)
- [使用](#使用)
  - [访问域对象中的变量](#访问域对象中的变量)
  - [`[]` 和 `.` 运算符](#-和--运算符)
  - [与标签搭配使用](#与标签搭配使用)
  - [EL 与函数](#el-与函数)

# 概述

EL 全名为 Expression Language，它主要用于替换 JSP 页面中的脚本表达式 `<%= expresseion %>`，从各种类型的 Web 域中检索 Java 对象、获取数据。它可以很方便地访问 JavaBean 属性，访问数组，访问 List 集合和 Map 集合等。

EL 表达式借鉴了 JavaScript 多类型转换无关性的特点，在使用 EL 从 scope 中得到参数时可以自动转换类型，因此对于类型的限制更加宽松。Web 服务器对于 request 域中的属性是以 Object 类型来存储的，在得到该属性时使用的 Java 语言脚本就应该是 `(Object)request.getAttribute("xxx")`，它对于实际应用还必须进行强制类型转换。而 EL 就将用户从这种类型转换的繁琐工作脱离出来，允许用户直接使用 EL 表达式取得的值，而不用关心它是什么类型的。

【注意】EL 表达式是 JSP2.0 规范中的一门技术，所以，要想正确解析 EL 表达式，需要使用支持 Servlet2.4/JSP2.0 技术的 WEB 服务器。Tomcat6 以上的服务器都可以解析 EL 表达式。

## 语法

```
${ expression }
```
- expression 可以是字符串、整形、浮点型、布尔型（true、false）等常量，也可以是变量，或者是它们之间的运算

## 运算符

| 操作符        | 功能                                |
| ------------- | ----------------------------------- |
| `.`           | 访问一个 Bean 属性或者一个 Map 条目 |
| `[]`          | 访问一个数组或者 List 的元素        |
| `()`          | 组织一个子表达式以改变优先级        |
| `+`           | 加（不能同 Java 一样连接字符串）    |
| `-`           | 减或负                              |
| `*`           | 乘                                  |
| `/` 或 `div`  | 除                                  |
| `%` 或 `mod`  | 取模                                |
| `==` 或 `eq`  | 测试是否相等                        |
| `!=` 或 `ne`  | 测试是否不等                        |
| `<` 或 `lt`   | 测试是否小于                        |
| `>` 或 `gt`   | 测试是否大于                        |
| `<=` 或 `le`  | 测试是否小于等于                    |
| `>=` 或 `ge`  | 测试是否大于等于                    |
| `&&` 或 `and` | 测试逻辑与                          |
| `|` 或 `or`   | 测试逻辑或                          |
| `!` 或 `not`  | 测试取反                            |
| `empty`       | 测试是否空值                        |
| `? :`         | 三目运算符                          |

## 隐式对象

| 隐式对象         | 说明                                                                                                                                                           |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pageScope        | 代表 page 域，可以用来获取 page 域中的属性                                                                                                                     |
| reqeustScope     | 代表 reqeust 域，可以用来获取 reqeust 域中的属性                                                                                                               |
| sessionScope     | 代表 session 域，可以用来获取 session 域中的属性                                                                                                               |
| applicationScope | 代表 application 域，可以用来获取 application 域中的属性                                                                                                       |
| pageContext      | 代表 pageContext 对象，注意和 pageScope 进行区分                                                                                                               |
| initParam        | 以 map 封装的 web.xml 中配置的整个 web 应用的初始化参数，返回 String 类型<br> `${initParam.name}` $\Rightarrow$ `ServletContext.getInitParameter(String name)` |
| param            | 代表请求参数组成的 map 集合，返回 String 类型。<br> `${param.name}` $\Rightarrow$ `request.getParameter(String name)`                                          |
| paramValues      | 代表请求参数组成的 map 集合，返回 String[] 类型<br> `${paramValues.name}` $\Rightarrow$ `request.getParameterValues(String name)`                              |
| header           | 获取所有 HTTP 请求字段的 map 对象，返回 String 类型 <br> `${header.name}` $\Rightarrow$ `request.getHeader(String name)`                                       |
| headerValues     | 获取请求头组成的 map，返回 String[] 类型<br> `${headerValues}` $\Rightarrow$ `request.getHeaders(String name)`                                                 |
| cookie           | 获取 cookie 组成的 map 对象，此 map 的值是一个 cookie 对象<br>`${cookie.name.value}`，获取指定 name 的 Cookie 的值                                             |

# 使用

使用 JSP 的 page 指令启用 EL。
```
<%@ page isELIgnored ="true|false" %>
```
- true 表示禁用 EL
- false 表示启用 EL
- servlet 2.4（对应 JSP 2.0）后， 默认值为 false

## 访问域对象中的变量

```
${ var.name }

${ var["name"] }
```
- var 表示定义在域对象中的变量的变量名
- name 表示变量的属性，该属性必须定义了 `getXxx` 方法

## `[]` 和 `.` 运算符

- 一些情况下可以相互替换
    ```
    ${ var.name } 等价于 ${ var["name"] }
    ```
- 可同时混合使用：`[]` 用于数组或 List 定位，`.` 用于属性定位
    ```
    ${ list[0].name }
    ```
- 当要存取的属性名称中包含一些特殊字符，比如 `.` 或者 `-` 等非字母或数字的符号，就必须要使用 `[]`
    ```
    ${ var["user-name"] }
    ```
- 使用 `[]` 可以动态获取数据
    ```
    ${ var[nameVar] } // nameVar 也是一个变量
    ```

## 与标签搭配使用

- 在标签的属性值中使用 EL：
    ```
    // HTML 标签
    <body class="${expression}">

    // JSP 标签
    <jsp:setProperty value="${expression}"/>
    ```
    当 JSP 编译器在属性中见到 `${}` 格式后，它会生成代码计算这个表达式的值，并且使用该值替代表达式。

- 在标签的主体中使用 EL：
    ```
    // HTML 标签
    <body> ${expression} </body>

    // JSP 标签
    <jsp:text> ${expression} </jsp:text>
    ```

## EL 与函数

JSP EL 允许您在表达式中使用函数。这些函数必须被定义在自定义标签库中。函数的使用语法如下：
```
${ ns:func(param1, param2, ...) }
```
- ns 指的是命名空间（namespace）
- func 指的是函数的名称，param1 指的是第一个参数，param2 指的是第二个参数，以此类推。

比如，有 JSTL 函数 `fn:length`，可以像下面这样来获取一个字符串的长度：
```
${fn:length("Get my length")}
```
要使用任何标签库中的函数，您需要将这些库安装在服务器中，然后使用 `<taglib>` 指令在 JSP 文件中包含这些库。