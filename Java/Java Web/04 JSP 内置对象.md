- [概述](#概述)
  - [内置对象特点](#内置对象特点)
- [九大内置对象](#九大内置对象)
  - [out](#out)
  - [request](#request)
  - [response](#response)
  - [session](#session)
  - [pageContext](#pagecontext)
  - [application](#application)
  - [config](#config)
  - [page](#page)
  - [exception](#exception)
- [域对象](#域对象)
  - [application 域](#application-域)
  - [session 域](#session-域)
  - [request 域](#request-域)
  - [page 域](#page-域)
- [JSP 状态管理](#jsp-状态管理)
  - [Session](#session)
    - [生命周期](#生命周期)
    - [设置生命周期](#设置生命周期)
  - [Cookie](#cookie)
    - [Cookie 作用与风险](#cookie-作用与风险)
    - [Cookie 使用](#cookie-使用)
  - [Cookie & Session 对比](#cookie--session-对比)
- [重定向与转发](#重定向与转发)
  - [重定向](#重定向)
  - [转发](#转发)
  - [URL 的异同](#url-的异同)

# 概述

JSP 内置对象，也称为隐式对象，是 Web 容器创建的一组对象，不用 new 关键字创建既可以使用的对象。

## 内置对象特点

- 由 JSP 规范提供，不用编写者实例化。
- 通过 Web 容器实现和管理
- 所有 JSP 页面均可使用
- 只能在脚本或表达式中使用

# 九大内置对象

- 输出输入对象: `request`、`response`、`out`
- 通信控制对象: `pageContext`、`session`、`application`
- Servlet 对象: `page`、`config`
- 错误处理对象: `exception`

## out

数据流，父类为 `javax.servlet.jsp.JspWriter`
用于将内容写入 JSP 页面实例的输出流中，提供了用于响应浏览器输出结果的方法。

| 方法                | 说明                                                                                        |
| ------------------- | ------------------------------------------------------------------------------------------- |
| **print / println** | 输出数据                                                                                    |
| **flush**           | 输出缓冲区数据，清楚缓冲器                                                                  |
| newLine             | 输出换行字符                                                                                |
| close               | 关闭输出流                                                                                  |
| clear               | 清除缓冲区中数据，不输出到客户端（调用后本页面后续内容不会输出）。在 flush 之后调用抛出异常 |
| clearBuffer         | 清除缓冲区中数据，输出到客户端（调用后本页面后续内容会输出）。在 flush 之后调用抛出异常     |
| getBufferSize       | 获取缓冲区大小                                                                              |
| getRemaining        | 获取缓冲区中没有被占用的空间                                                                |
| isAutoFlush         | 是否为自动输出                                                                              |

## request

请求信息，父类为 `javax.servlet.http.HttpServletRequest`
包含了有关浏览器请求的信息。通过该对象可以获得请求中的首部信息、Cookie 和请求参数。

| 方法                     | 说明                                        |
| ------------------------ | ------------------------------------------- |
| **getAttribute**         | 获取指定属性的值，如该属性值不存在返回 Null |
| **removeAttribute**      | 删除请求中的一个属性                        |
| **setAttribute**         | 设置指定名字参数值                          |
| **getParameter**         | 获取指定名字参数值                          |
| **getParameterValues**   | 获取指定名字所有参数值的数组                |
| **getContextPath**       | 获取项目名                                  |
| **getServletPath**       | 获取当前页面所在目录下全名称                |
| **getRequestURL**        | 获取请求的 URL                              |
| **getCookies**           | 获取所有 Cookie 对象                        |
| **getRequestURI**        | 获取发出请求字符串的客户端地址              |
| **setCharacterEncoding** | 设置请求的字符编码方式                      |
| **getSession**           | 获取和请求相关的会话（session）             |
| **getServletContext**    | 获取 ServletContext（application）          |
| getCharacterEncoding     | 获取请求的字符编码方式                      |
| getParameterNames        | 获取所有参数的名字，一个集合                |
| getAttributeNames        | 获取所有属性名的集合                        |
| getHeader                | 获取指定名字报文首部值                      |
| getHeaders               | 获取指定名字报文首部的所有值，一个枚举      |
| isUserInRole             | 判断认证后的用户是否属于某一成员组          |
| getContentLength         | 返回请求正文的长度，如不确定返回 -1         |
| getHeaderNames           | 获取所有报文首部的名字，一个枚举            |
| getInputStream           | 返回请求输入流，获取请求中的数据            |
| getMethod                | 获取客户端向服务器端传送数据的方法	GET      |
| getParameterValues       | 获取指定名字参数的所有值                    |
| getProtocol              | 获取客户端向服务器端传送数据的协议名称      |
| getRemoteAddr            | 获取客户端的 IP 地址                        |
| getRemoteHost            | 获取客户端的名字                            |
| getServerName            | 获取服务器的名字                            |
| getServerPort            | 获取服务器的端口号                          |
| getContentType           | 获取 MIME 类型                              |

## response

响应，父类为 `javax.servlet.http.HttpServletResponse`
作为 JSP 页面处理结果返回给用户的响应存储在该对象中。并提供了设置响应内容、响应首部以及重定向的方法 (如 cookies, 首部信息等)

| 方法                     | 说明                                   |
| ------------------------ | -------------------------------------- |
| **addCookie**            | 添加一个 Cookie 对象                   |
| **sendRedirect**         | 把响应发送到另一个位置进行处理         |
| **setCharacterEncoding** | 指定响应的编码集（MIME 字符集          |
| sendError                | 向客户端发送错误信息                   |
| setHeader                | 设置指定名字的 Http 文件首部信息       |
| addHeader                | 添加 Http 文件指定名字首部信息         |
| setContentType           | 设置响应的 MIME 类型                   |
| containsHeader           | 判断指定名字 Http 文件首部信息是否存在 |
| encodeURL                | 使用 sessionid 封装 URL                |
| flushBuffer              | 强制把当前缓冲区内容发送到客户端       |
| getBufferSize            | 返回缓冲区大小                         |
| getOutputStream          | 返回到客户端的输出字节流对象           |
| getWriter                | 返回到客户端的输出字符流对象           |

## session

会话，父类为 `javax.servlet.http.HttpSession`
会话对象存储有关此会话的信息，也可以将属性赋给一个会话，每个属性都有名称和值。会话对象主要用于存储和检索属性值。

| 方法                   | 说明                                  |
| ---------------------- | ------------------------------------- |
| **getAttribute**       | 获取指定名字的属性                    |
| **getAttributeNames**  | 获取 session 中全部属性名字，一个集合 |
| **removeAttribute**    | 删除指定名字的属性                    |
| **setAttribute**       | 设定指定名字的属性值                  |
| **invalidate**         | 销毁 session 对象                     |
| getCreationTime        | 返回 session 的创建时间               |
| getId                  | 获取会话标识符                        |
| getLastAccessedTime    | 返回最后发送请求的时间                |
| getMaxInactiveInterval | 返回 session 对象的生存时间单位毫秒   |
| getServletContext      | 返回 ServletContext（application）    |
| isNew                  | 每个请求是否会产生新的 session 对象   |

## pageContext

页面上下文，父类为 `javax.servlet.jsp.PageContext`
描述了当前 JSP 页面的运行环境。可以返回 JSP 页面的其他内置对象（域对象）及其属性的访问，另外，它还实现将控制权从当前页面传输至其他页面的方法。

| 方法                | 说明                                                   |
| ------------------- | ------------------------------------------------------ |
| **forward**         | 重定向到另一页面或 Servlet 组件                        |
| **include**         | 包含某一个界面                                         |
| **getAttribute**    | 获取某范围中指定名字的属性值                           |
| **findAttribute**   | 按范围搜索指定名字的属性                               |
| **removeAttribute** | 删除某范围中指定名字的属性                             |
| **setAttribute**    | 设定某范围中指定名字的属性值                           |
| getException        | 返回当前异常对象	（exception）                         |
| getRequest          | 返回当前请求对象	（request）                           |
| getResponse         | 返回当前响应对象	（response）                          |
| getServletConfig    | 返回当前页面的 ServletConfig 对象	（config）           |
| getServletContext   | 返回所有页面共享的 ServletContext 对象	（application） |
| getSession          | 返回当前页面的会话对象（Session）                      |

## application

应用程序，父类为 `javax.servlet.ServletContext`
存储了运行 JSP 页面的 servlet 以及在同一应用程序中的任何 Web 组件的上下文信息。

| 方法                  | 说明                                   |
| --------------------- | -------------------------------------- |
| **getAttribute**      | 获取应用对象中指定名字的属性值         |
| **getAttributeNames** | 获取应用对象中所有属性的名字，一个集合 |
| **getInitParameter**  | 返回应用对象中指定名字的初始参数值     |
| **setAttribute**      | 设置应用对象中指定名字的属性值         |
| getServerInfo         | 返回 Servlet 编译器中当前版本信息      |

## config

Servlet 配置信息，父类为 `javax.servlet.ServletConfig`
该对象用于存取 servlet 实例的初始化参数。

| 方法                  | 说明                                                        |
| --------------------- | ----------------------------------------------------------- |
| **getInitParameter**  | 返回指定名字的初始参数值                                    |
| getServletContext     | 返回所执行的 Servlet 的环境对象（ApplicationContextFacade） |
| getServletName        | 返回所执行的 Servlet 的名字                                 |
| getInitParameterNames | 返回该 JSP 中所有的初始参数名，一个集合                     |

## page

当前 JSP 的实例，父类为 `java.lang.object`
它代表 JSP 被编译成 Servlet, 可以使用它来调用 Servlet 类中所定义的方法

## exception

运行时的异常，父类为 `java.lang.Throwable`
在某个页面抛出异常时，将转发至 JSP 错误页面，提供此对象是为了在 JSP 中处理错误。只有在错误页面中才可使用。
```
<%@page isErrorPage=“true”%>
```

# 域对象

域对象有 4 个，分别为 application, session, request, pageContext。

"作用域" 就是 "信息共享的范围"，也就是说一个信息能够在多大的范围内有效。共有 4 中中作用域
- application 域
- session 域
- request 域
- page 域

可分别对应 pageContext 域对象的四个整形常量
- PageContext.PAGE_SCOPE（对应 page 域）
- PageContext.REQUEST_SCOPE（对应 request 域）
- PageContext.SESSION_SCOPE（对应 session 域）
- PageContext.APPLICATION_SCOPE（对应 application 域）

**注意：** pageContext 只能使用 `setAttribute(String, Object)` 方法设置 page 域对象，而不能使用 `setAttribute(String, Object, int)` 设置任意域对象

## application 域

从服务器开始执行服务，到服务器关闭为止，application 域的范围最大、停留的时间也最久，所以使用时要特别注意不然可能会造成服务器负载越来越重的情况。

具有 application 域的对象被绑定到 `javax.servlet.ServletContext` 对象中。在 Web 应用程序运行期间，所有的页面都可以访问在这个范围内的对象。

## session 域

Web 交互的最基本单位为 HTTP 请求。每个用户从进入网站到离开网站这段过程称为一个 HTTP 会话， 一个服务器的运行过程中会有多个用户访问，就是多个 HTTP 会话。HTTP 会话开始到结束这段时间。Session 的作用范围为一段用户持续和服务器所连接的时间，但与服务器断线，这个属性就无效了。

Session 的开始时刻比较容易判断，它从浏览器发出第一个 HTTP 请求即可认为会话开始。但结束时刻就不好判断了，因为浏览器关闭时并不会通知服务器，所以只能通过如下这种方法判断：如果一定的时间内客户端没有反应，则认为会话结束。Tomcat 的默认值为 120 分钟，但这个值也可以通过 HttpSession 的 setMaxInactiveInterval () 方法来设置。

具有 session 域的对象被绑定到 `javax.servlet.http.HttpSession` 对象中。

## request 域

HTTP 请求开始到结束这段时间。Request 的范围是指在一 JSP 网页发出请求到另一个 JSP 网页之间，否则这个属性就失效。一个 HTTP 请求的处理可能需要多个 Servlet 合作，而这几个 Servlet 之间可以通过某种方式传递信息，但这个信息在请求结束后就无效了。

具有 request 域的对象被绑定到 `javax.servlet.ServletRequest` 对象中。

要注意的是，因为请求对象对于每一个客户请求都是不同的，所以对于每一个新的请求，都要重新创建和删除这个范围内的对象。

## page 域

当前页面从打开到关闭这段时间，它只能在同一个页面中有效。

具有 page 域的对象被绑定到 `javax.servlet.jsp.PageContext` 对象中。

# JSP 状态管理

**HTTP 协议无状态性**：无状态是指，当浏览器发送请求给服务器的时候，服务器响应客户端请求。但是当同一个浏览器再次发送请求给服务器的时候，服务器并不知道它就是刚才那个浏览器。简单的说，就是服务器不会去记得你（无法保存状态），所以就是无状态协议。

**保存用户状态的两大机制**
- Session
- Cookie

## Session

### 生命周期

1. 创建
    - 当客户端第一次访问某个 jsp 或者 Servlet 时候，服务器会为当前会话创建一个 SessionId。
    - 每次客户端向服务端发送请求时，都会将此 SessionId 携带过去，服务端会对此 SessionId 进行校验。
    - 需要注意只有访问 JSP、Servlet 等程序时才会创建 session，只访问 HTML、IMAGE 等静态资源并不会创建 session, 可调用 request.getSession (true) 强制生成 session。

2. 活动
    - 某次会话当中通过超链接打开的新页面属于同一次会话。
    - 只要当前会话页面没有全部关闭，重新打开新的浏览器窗口访问同一项目资源时属于同一次会话。
    - 除非本次会话的所有页面都关闭后再重新访问某个 Jsp 或者 Servlet 将会创建新的会话。
    - 需要注意的是，原有会话还存在，只是这个旧的 SessionId 仍然存在于服务端，只不过再也没有客户端会携带它然后交予服务端校验。

3. 销毁
    - 调用 `session.invalidate()` 方法
    - session 过期。session 的过期时间是从 session 不活动的时候开始计算，如果 session 一直活动，session 就总不会过期。从该 Session 未被访问，开始计时； 一旦 Session 被访问，计时清 0;
    - 服务器重新启动

### 设置生命周期

Tomcat 中 session 的默认失效时间为 20 分钟。

- web.xml 中设置。单位为分钟
    ```
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
    ```
- jsp 中设置。设置单位为秒，设置为 -1 表示永不过期。
    ```
    <% session.setMaxInactiveInterval(-1); %>
    ```
- tomcat 的 server.xml 中设置，单位为秒。
    ```
    <Context ...
        defaultSessionTimeOut="3600"
        ... />
    ```
## Cookie

中文名称为 “小甜饼”，是 web 服务器保存在客户端的一系列文本信息。

典型应用一：判定注册用户是否已经登录网站。
典型医用二：“购物车” 的处理。
典型医用三：系统会自动记录已经浏览过的视频。
典型医用四：记住用户名和密码实现自动登录功能。

### Cookie 作用与风险

- 对待定对象的追踪
- 保存用户网页浏览记录与习惯
- 简化登录
- Cookie 安全风险
- 容易泄露用户信息

### Cookie 使用

```
// 添加或替换
Cookie cookie = new Cookie(String name, String value);
cookie.setMaxAge(int expiry);//设置过期时间，单位为秒
cookie.getName();
cookie.getValue();
cookie.getMaxAge();
response.addCookie(cookie);

// 获取
Cookie[] cookies = request.getCookies();
```

## Cookie & Session 对比

|              | Session                          | Cookie                      |
| :----------- | :------------------------------- | :-------------------------- |
| 数据存储位置 | 在服务器端保存用户信息           | 在客户端保存用户信息        |
| 保存数据类型 | Object 类型                      | String 类型                 |
| 生存时间     | 随会话的结束而将其存储的数据销毁 | cookie 可以长期保存在客户端 |
| 使用情形     | 保存重要的信息                   | 保存不重要的信息            |

# 重定向与转发

## 重定向

客户端行为，相当于客户端重新向服务器发送请求（两次请求），request 中的属性全部失效，并开始一个新的 request 对象。网址栏 url 改变。

过程：
1. 客户发送一个请求到服务器，服务器匹配 servlet，这都和请求转发一样。
2. servlet 处理完之后调用了 response 对象的 `sendRedirect(String location)` 方法，向客户端返回这个响应，响应行告诉客户端你必须要再发送一个请求，去访问 location。
3. 客户端收到这个信息后，立刻自动发出一个新的请求，去访问 location。
    这里两个请求互不干扰，相互独立，在前面 request 里面 setAttribute() 的任何东西，在后面的 request 里面都获得不了。

**整个过程是两个请求，两个响应。**

## 转发

服务器行为，一次请求，转发后请求对象会保存，地址栏 url 不变。

过程：
1. 客户首先发送一个请求到服务器端，服务器端发现匹配的 Servlet，并指定它去执行。
2. 当这个 Servlet 执行完之后，它要调用 request 对象的 `getRequestDispatcher(String path).forward(request, response)` 方法，把请求转发给指定的 path。
    整个流程都是在服务器端完成的，而且是在同一个请求里面完成的，因此两个 Servlet 共享的是同一个 request。

**整个过程是一个请求，一个响应。**

## URL 的异同

- 在重定向时，绝对路径 `/` 表示的是 `localhost:8080/`（假设端口为 8080），所以 URL 需要加上 `/ ProjectName` （项目名)

- 在转发时，绝对路径 `/` 表示 `localhost:8080/ProjectName`。

- 相对路径没有区别。

例：
假设 localhost:8080/JavaWebTutorial 下有三个 jsp 文件，分别为 demo1.jsp、demo2.jsp、demo3.jsp，从 demo1.jsp 重定向至 demo2.jsp， 转发至 demo3.jsp
```
response.sendRedirect("/JavaWebTutorial/demo2.jsp");
response.sendRedirect("demo2.jsp");
request.getRequestDispatcher("/demo3.jsp").forward(request, response);
request.getRequestDispatcher("demo3.jsp").forward(request, response);
```