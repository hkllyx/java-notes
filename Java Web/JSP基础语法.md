# JSP基础语法

## 注释

HTML注释：在客户端查看源代码时能看见注释。

```jsp
<!-- html comments -->
```

JSP注释：JSP引擎在将JSP页面翻译成Servlet程序时，忽略JSP页面中被注释的内容。 注释虽然写在JSP程序中，但不会发送给客户。

```jsp
<%-- jsp comments --%>
```

JAVA注释：客户端不可见，只能出现在Java代码区中，不允许直接出现在页面中

```jsp
// java single-line comments
/* java multiline comments */
```

## JSP表达式

是对数据的表示，系统将其作为一个值进行计算。

语法：`<%= expression%>`，例如：`<%= user.getName()%>`

本质：在将JSP页面转换成Servlet后，使用`out.print()`将表达式的值输出。

注意：

- 如果表达式是调用一个方法，那么这个方法必须要有返回值，而不应是`void`，也就是说 `void getName()`这样的方法是不能被调用的。
- 在方法的后面不能有分号；例如`<%= getName();%>`这是不允许的。

## JSP脚本

就是在`<%%>`里嵌入Java代码，这里的Java代码和我们一般的Java代码没有什么区别，所以每一条语句同样要以`;`结束，这和表达式是不相同的

语法：`<% code %>`，例如：

```jsp
<%
    out.print("java web");
%>
```

本质：就是将代码插入到Servlet的`service()`方法中。

## JSP声明

就是允许用户定义Servlet中的成员变量字段、方法。

Servlet: JSP页面会编译成一个Servlet类，每个Servlet在容器中只有一个实例（单例模式）；在JSP中声明的变量成员变量，成员变量只在创建时初始化（非`final`变量可修改值），该变量的值将一直保存，直到实例销毁。

语法：`<%! code %>`，例如：

```java
<%!
    // 定义字段
    String s = "JAVA";

    // 定义方法
    int method() {
        return 0;
    }
%>
```

本质：其实就是将声明的变量成为Servlet的成员变量（字段）和方法。

## 脚本和声明的对比

访问控制符：

- 声明：由于JSP声明语法定义的变量和方法对应Servlet类的成员变量的方法，所以JSP声明部分定义得额变量和方法可以使用`private`，`public`等访问控制修饰，也可以使用`static`修饰，将其变成类属性和类方法。
- 脚本：脚本中声明的变量都是`service()`函数中的局部变量，所以是不能使用`private`，`public`等访问控制符的。

方法声明：

- 声明：在声明中，可以声明方法，因为实际是在声明Servlet类的方法；
- 脚本：在脚本中，不可以声明方法，因为在Java的service的方法中是不能声明的方法。

编译结果：

- 声明：将声明的变量和方法，作为Servlet类的变量和方法。
- 脚本：将代码插入到Servlet的`service()`方法中。
