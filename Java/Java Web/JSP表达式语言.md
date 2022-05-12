# JSP表达式语言

EL全名为Expression Language，它主要用于替换JSP页面中的脚本表达式`<%= expresseion %>`，从各种类型的Web域中检索Java对象、获取数据。它可以很方便地访问Java Bean属性、数组、`List`、`Map`等。

EL表达式借鉴了JavaScript多类型转换无关性的特点，在使用EL从scope中得到参数时可以自动转换类型，因此对于类型的限制更加宽松。

Web服务器对于`request`域中的属性是以`Object`类型来存储的，在得到该属性时使用的Java语言脚本就应该是`(Object)request.getAttribute("xxx")`，它对于实际应用还必须进行强制类型转换。而EL就将用户从这种类型转换的繁琐工作脱离出来，允许用户直接使用EL表达式取得的值，而不用关心它是什么类型的。

【注意】EL表达式是JSP2.0规范中的一门技术，所以，要想正确解析EL表达式，需要使用支持Servlet 2.4/JSP 2.0技术的WEB服务器。Tomcat6以上的服务器都可以解析EL表达式。

## 语法

```jsp
${ expression }
```

expression可以是字符串、整形、浮点型、布尔型等常量，也可以是变量，或者是它们之间的运算

### 运算符

| 操作符        | 功能                              |
| ------------- | --------------------------------- |
| `.`           | 访问一个Bean属性或者一个`Map`条目 |
| `[]`          | 访问一个数组或者`List`的元素      |
| `()`          | 组织一个子表达式以改变优先级      |
| `+`           | 加（不能同Java一样连接字符串）    |
| `-`           | 减或负                            |
| `*`           | 乘                                |
| `/` 或 `div`  | 除                                |
| `%` 或 `mod`  | 取模                              |
| `==` 或 `eq`  | 测试是否相等                      |
| `!=` 或 `ne`  | 测试是否不等                      |
| `<` 或 `lt`   | 测试是否小于                      |
| `>` 或 `gt`   | 测试是否大于                      |
| `<=` 或 `le`  | 测试是否小于等于                  |
| `>=` 或 `ge`  | 测试是否大于等于                  |
| `&&` 或 `and` | 测试逻辑与                        |
| `\|` 或 `or`  | 测试逻辑或                        |
| `!` 或 `not`  | 测试取反                          |
| `empty`       | 测试是否空值                      |
| `? :`         | 三目运算符                        |

### 隐式对象

| 隐式对象         | 说明                                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| pageScope        | 代表page域，可以用来获取page域中的属性                                                                                 |
| reqeustScope     | 代表reqeust域，可以用来获取reqeust域中的属性                                                                           |
| sessionScope     | 代表session域，可以用来获取session域中的属性                                                                           |
| applicationScope | 代表application域，可以用来获取application域中的属性                                                                   |
| pageContext      | 代表`pageContext`对象，注意和pageScope进行区分                                                                         |
| initParam        | web.xml中初始化参数组成的`Map`，返回`String`<br> `${initParam.name}` -> `ServletContext.getInitParameter(String name)` |
| param            | 请求参数组成的`Map`，返回`String`。<br> `${param.name}` -> `request.getParameter(String name)`                         |
| paramValues      | 请求参数组成的`Map`，返回`String[]`<br> `${paramValues.name}` -> `request.getParameterValues(String name)`             |
| header           | 请求头组成的`Map`，返回`String`<br> `${header.name}` -> `request.getHeader(String name)`                               |
| headerValues     | 请求头组成的`Map`，返回`String[]`<br> `${headerValues}` -> `request.getHeaders(String name)`                           |
| cookie           | 获取Cookie组成的`Map`，此map的值是一个cookie对象<br>`${cookie.name.value}`，获取指定name的Cookie的值                   |

## 使用

使用JSP的`page`指令启用EL。

```jsp
<%@ page isELIgnored ="true|false" %>
```

- `true`表示禁用EL
- `false`表示启用EL
- Servlet 2.4（对应JSP 2.0）后，默认值为`false`

### 访问域对象中的变量

```jsp
${ var.name }
${ var["name"] }
```

- var表示定义在域对象中的变量的变量名
- name表示变量的属性，该属性必须定义了`getXxx`方法

### `[]`和`.`运算符

- 一些情况下可以相互替换

  ```jsp
  ${ var.name } 等价于${ var["name"] }
  ```

- 可同时混合使用：`[]`用于数组或`List`定位，`.`用于属性定位

  ```jsp
  ${ list[0].name }
  ```

- 当要存取的属性名称中包含一些特殊字符，比如`.`或者`-`等非字母或数字的符号，就必须要使用`[]`

  ```jsp
  ${ var["user-name"] }
  ```

- 使用`[]`可以动态获取数据

  ```jsp
  ${ var[nameVar] } // nameVar也是一个变量
  ```

### 与标签搭配使用

- 在标签的属性值中使用EL：

  ```jsp
  // HTML标签
  <body class="${ expression }">

  // JSP标签
  <jsp:setProperty value="${ expression }"/>
  ```

  当JSP编译器在属性中见到`${}`格式后，它会生成代码计算这个表达式的值，并且使用该值替代表达式。

- 在标签的主体中使用EL：

  ```jsp
  // HTML标签
  <body>${ expression }</body>

  // JSP标签
  <jsp:text>${ expression }</jsp:text>
  ```

### EL与函数

JSP EL允许您在表达式中使用函数。这些函数必须被定义在自定义标签库中。函数的使用语法如下：

```jsp
${ ns:func(param1, param2, ...) }
```

- `ns`指的是命名空间（namespace）
- `func`指的是函数的名称，param1指的是第一个参数，param2指的是第二个参数，以此类推。

比如，有JSTL函数`fn:length`，可以像下面这样来获取一个字符串的长度：

```jsp
${fn:length("Get my length")}
```

要使用任何标签库中的函数，您需要将这些库安装在服务器中，然后使用`<taglib>`指令在JSP文件中包含这些库。
