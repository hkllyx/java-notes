- [注释](#注释)
- [JSP表达式](#jsp表达式)
- [JSP脚本](#jsp脚本)
- [JSP声明](#jsp声明)
- [脚本和声明的对比：](#脚本和声明的对比)

# 注释

1. HTML注释：在客户端查看源代码时能看见注释。
    ```
    <!-- html comments -->
    ```
2. JSP注释：JSP引擎在将JSP页面翻译成Servlet程序时，忽略JSP页面中被注释的内容。 注释虽然写在JSP程序中，但不会发送给客户。
    ```
    <%-- jsp comments --%>
    ```
3. JAVA注释：客户端不可见，只能出现在Java代码区中，不允许直接出现在页面中
    ```
    // java single-line comments
    /* java multiline comments */
    ```

# JSP表达式

是对数据的表示，系统将其作为一个值进行计算。

- 语法：`<%= expression %>`
    - 例如：`<%= user.getName() %>`
- 本质：在将JSP页面转换成Servlet后，使用 `out.print()` 将表达式的值输出。
- 注意：
    - 如果表达式是调用一个方法，那么这个方法必须要有返回值，而不应是 `void`，也就是说 `void getName()` 这样的方法是不能被调用的。
    - 在方法的后面不能有分号；例如 `<%= getName();%>` 这是不允许的。

# JSP脚本

就是在 `<%%>` 里嵌入Java代码，这里的Java代码和我们一般的Java代码没有什么区别，所以每一条语句同样要以 `;` 结束，这和表达式是不相同的

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
- 本质：就是将代码插入到Servlet的 `service()` 方法中。

# JSP声明

就是允许用户定义Servlet中的变量、方法

- Servlet: JSP页面会编译成一个Servlet类，每个Servlet在容器中只有一个实例（单例模式）；在JSP中声明的变量好是成员变量，成员变量只在 创建时初始化， 该变量的值将一直保存，直到实例销毁。
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
- 本质：其实就是将声明的变量加入到Servlet类（字段Field），方法就成了Servlet的方法。

# 脚本和声明的对比：

1. 访问控制符：
    - 声明：由于JSP声明语法定义的变量和方法对应Servlet类的成员变量的方法， 所以JSP声明部分定义得额变量和方法可以使用private, public等访问控制修饰， 也可以使用static修饰， 将其变成类属性和类方法。但不能使用abstract修饰声明部分的方法， 因为abstract导致JSP对应的Servlet变成抽象类，从而导致无法实例化。
    - 脚本：脚本中声明的变量都是 `service()` 函数中的局部变量，所以是不能使用private，public等访问控制符的。

1. 方法声明：
    - 声明：在声明中，可以声明方法， 因为实际是在声明Servlet类的方法；
    - 脚本：在脚本中，不可以声明方法，因为在Java的service的方法中是不能声明的方法。

1. 编译结果：
    - 声明：将声明的变量和方法，作为Servlet类的变量和方法。
    - 脚本：将代码插入到Servlet的 `service()` 方法中。