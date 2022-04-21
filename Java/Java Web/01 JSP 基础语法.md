- [注释](#注释)
- [JSP 表达式](#jsp-表达式)
- [JSP 脚本](#jsp-脚本)
- [JSP声明](#jsp声明)
- [脚本和声明的对比：](#脚本和声明的对比)

# 注释

1. HTML 注释：在客户端查看源代码时能看见注释。
    ```
    <!-- html comments -->
    ```
2. JSP 注释：JSP 引擎在将 JSP 页面翻译成 Servlet 程序时，忽略 JSP 页面中被注释的内容。 注释虽然写在 JSP 程序中，但不会发送给客户。
    ```
    <%-- jsp comments --%>
    ```
3. JAVA 注释：客户端不可见，只能出现在 Java 代码区中，不允许直接出现在页面中
    ```
    // java single-line comments
    /* java multiline comments */
    ```

# JSP 表达式

是对数据的表示，系统将其作为一个值进行计算。

- 语法：`<%= expression %>`
    - 例如：`<%= user.getName() %>`
- 本质：在将 JSP 页面转换成 Servlet 后，使用 `out.print()` 将表达式的值输出。
- 注意：
    - 如果表达式是调用一个方法，那么这个方法必须要有返回值，而不应是 `void`，也就是说 `void getName()` 这样的方法是不能被调用的。
    - 在方法的后面不能有分号；例如 `<%= getName();%>` 这是不允许的。

# JSP 脚本

就是在 `<%%>` 里嵌入 Java 代码，这里的 Java 代码和我们一般的 Java 代码没有什么区别，所以每一条语句同样要以 `;` 结束，这和表达式是不相同的

- 语法：`<% code %>`
    - 例如：
    ```
    <%
        out.print("java web");
        if (true) {
            out.print("java");
        } else {
            out.print("web");
        }
    %>
    ```
- 本质：就是将代码插入到 Servlet 的 `service()` 方法中。

# JSP声明

就是允许用户定义 Servlet 中的变量、方法

- Servlet: JSP 页面会编译成一个 Servlet 类，每个 Servlet 在容器中只有一个实例(单例模式)；在 JSP 中声明的变量好是成员变量，成员变量只在 创建时初始化， 该变量的值将一直保存，直到实例销毁。
- 语法：`<%! code %>`
    - 例如：
    ```
    <%!
        String s = "JAVA";
        int method() {
            return 0;
        }
    %>
    ```
- 本质：其实就是将声明的变量加入到 Servlet 类(字段 Field)，方法就成了 Servlet 的方法。

# 脚本和声明的对比：

1. 访问控制符：
    - 声明：由于 JSP 声明语法定义的变量和方法对应 Servlet 类的成员变量的方法， 所以JSP声明部分定义得额变量和方法可以使用 private, public 等访问控制修饰， 也可以使用 static 修饰， 将其变成类属性和类方法。但不能使用 abstract 修饰声明部分的方法， 因为 abstract 导致 JSP 对应的 Servlet 变成抽象类，从而导致无法实例化。
    - 脚本：脚本中声明的变量都是 `service()` 函数中的局部变量，所以是不能使用 private， public 等访问控制符的。

1. 方法声明：
    - 声明：在声明中，可以声明方法， 因为实际是在声明 Servlet 类的方法；
    - 脚本：在脚本中，不可以声明方法，因为在 Java 的 service 的方法中是不能声明的方法。

1. 编译结果：
    - 声明：将声明的变量和方法，作为 Servlet 类的变量和方法。
    - 脚本：将代码插入到 Servlet 的 `service()` 方法中。