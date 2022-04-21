- [概述](#概述)
  - [CSS 和 XSL](#css-和-xsl)
  - [XSL 包含三部分](#xsl-包含三部分)
- [XPath](#xpath)
  - [Xpath 术语](#xpath-术语)
  - [位置路径表达式](#位置路径表达式)
  - [选取节点](#选取节点)
  - [Xpath 谓语（Predicates）](#xpath-谓语predicates)
  - [`*` 和 `|`](#-和-)
  - [Xpath 轴](#xpath-轴)
  - [XPath 运算符](#xpath-运算符)
  - [Xpath 函数](#xpath-函数)
- [XSLT](#xslt)
  - [XSLT 元素](#xslt-元素)
  - [XSLT 函数](#xslt-函数)
- [XSL-FO](#xsl-fo)
  - [XSL-FO 区域](#xsl-fo-区域)

# 概述

- XSL 指扩展样式表语言（EXtensible Stylesheet Language）。
- 万维网联盟开始发展 XSL 的起因是由于对基于 XML 的样式表语言的需求。
- XSLT 指 XSL 转换（EXtensible Stylesheet Language）。

## CSS 和 XSL

- CSS = HTML 样式表
    - HTML 使用预先定义的标签，标签的意义很容易被理解。
    - HTML 元素中的 `<table>` 元素定义表格 - 并且浏览器清楚如何显示它。
    - 向 HTML 元素添加样式是很容易的。通过 CSS，很容易告知浏览器用特定的字体或颜色显示一个元素。
- XSL = XML 样式表
    - XML 不使用预先定义的标签（我们可以使用任何喜欢的标签名），并且这些标签的意义并不都那么容易被理解。
    - `<table>` 元素意味着一个 HTML 表格，一件家具，或是别的什么东西 - 浏览器不清楚如何显示它。
    - XSL 可描述如何来显示 XML 文档！

## XSL 包含三部分

XSL 不仅仅是样式表语言，它包括三部分：
- XSLT：一种用于转换 XML 文档的语言
- XPath：一种用于在 XML 文档中导航的语言
- XSL-FO：一种用于格式化 XML 文档的语。

# XPath

XPath 是一门在 XML 文档中查找信息的语言。XPath 用于在 XML 文档中通过元素和属性进行导航。
- XPath 使用路径表达式在 XML 文档中进行导航
    - XPath 使用路径表达式来选取 XML 文档中的节点或者节点集。
    - 这些路径表达式和我们在常规的电脑文件系统中看到的表达式非常相似。
- XPath 包含一个标准函数库
    - Path 含有超过 100 个内建的函数。
    - 这些函数用于字符串值、数值、日期和时间比较、节点和 QName 处理、序列处理、逻辑值等等。
- XPath 是 XSLT 中的主要元素
    - XPath 是 XSLT 标准中的主要元素。如果没有 XPath 方面的知识，就无法创建 XSLT 文档。
- XPath 是一个 W3C 标准

## Xpath 术语

- 节点（Node）：在 XPath 中，有七种类型的节点（Node）
    - 元素（element）
    - 属性（attribute）
    - 文本（text）
    - 命名空间（namespace）
    - 处理指令（processing-instruction）
    - 注释（comment）
    - 文档节点 / 根元素（document node / root element）
- 基本值 / 原子值（Atomic value）：无父或无子的节点
- 项目（Item）：基本值或者节点
- 节点关系：
    - 父（Parent）：每个元素以及属性都有一个父
    - 子（Children）：元素节点可有零个、一个或多个子
    - 同胞（Sibling）：拥有相同的父的节点
    - 先辈（Ancestor）：某节点的父、父的父，等等。
    - 后代（Descendant）：某个节点的子，子的子，等等。

## 位置路径表达式

  位置路径可以是绝对的，也可以是相对的。绝对路径起始于正斜杠 (/)，而相对路径不会这样。在两种情况中，位置路径均包括一个或多个步，每个步均被斜杠分割：

绝对位置路径：
```
/step/step/...
```
相对位置路径：
```
step/step/...
```
每个步均根据当前节点集之中的节点来进行计算。

步（step）包括：
```
axes::node-test[predicate]
```
- 轴（axis）：定义所选节点与当前节点之间的树关系
- 节点测试（node-test）：识别某个轴内部的节点
- 谓语（predicate）：零个或多个，更深入地提炼所选的节点集

## 选取节点

- XPath 使用路径表达式来选取 XML 文档中的节点或节点集。
- 节点是通过沿着路径 (path) 或者步 (steps) 来选取的。

| 表达式   | 描述                                                     |
| -------- | -------------------------------------------------------- |
| nodename | 选取此节点的所有子节点                                   |
| /        | 从根节点开始选取                                         |
| //       | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置 |
| .        | 选取当前节点                                             |
| ..       | 选取当前节点的父节点                                     |
| @        | 选取属性                                                 |

示例：

| 路径表达式      | 结果                                                                                           |
| --------------- | ---------------------------------------------------------------------------------------------- |
| bookstore       | 选取 bookstore 元素的所有子节点                                                                |
| /bookstore      | 选取根元素 bookstore。<br>注释：假如路径起始于正斜杠 (/)，则此路径始终代表到某元素的绝对路径！ |
| bookstore/book  | 选取属于 bookstore 的子元素的所有 book 元素                                                    |
| //book          | 选取所有 book 子元素，而不管它们在文档中的位置                                                 |
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置         |
| //@lang         | 选取名为 lang 的所有属性                                                                       |

## Xpath 谓语（Predicates）

- 谓语用来查找某个特定的节点或者包含某个指定的值的节点。
- 谓语被嵌在方括号（"[" 和 "]"）中。

示例：

| 路径表达式                         | 结果                                                                                    |
| ---------------------------------- | --------------------------------------------------------------------------------------- |
| /bookstore/book[1]                 | 选取属于 bookstore 子元素的第一个 book 元素                                             |
| /bookstore/book[last()-1]          | 选取属于 bookstore 子元素的倒数第二个 book 元素                                         |
| /bookstore/book[position()<3]      | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素                                 |
| //title[@lang]                     | 选取所有拥有名为 lang 的属性的 title 元素                                               |
| //title[@lang='eng']               | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性                                |
| /bookstore/book[price>35.00]/title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00 |

## `*` 和 `|`

- `*`：通配符，可用来选取未知的 XML 节点。
    - `*`：匹配任意元素
    - `@*`：匹配任意属性
- `|`：或，可以选取若干个路径中的某一个

## Xpath 轴

轴（Axes）可定义相对于当前节点的节点集。

| 轴名称             | 结果                                                   |
| ------------------ | ------------------------------------------------------ |
| ancestor           | 选取当前节点的所有先辈（父、祖父等）                   |
| ancestor-or-self   | 选取当前节点的所有先辈（父、祖父等）以及当前节点本身   |
| attribute          | 选取当前节点的所有属性                                 |
| child              | 选取当前节点的所有子元素                               |
| descendant         | 选取当前节点的所有后代元素（子、孙等）                 |
| descendant-or-self | 选取当前节点的所有后代元素（子、孙等）以及当前节点本身 |
| following          | 选取文档中当前节点的结束标签之后的所有节点             |
| namespace          | 选取当前节点的所有命名空间节点                         |
| parent             | 选取当前节点的父节点                                   |
| preceding          | 选取文档中当前节点的开始标签之前的所有节点             |
| preceding-sibling  | 选取当前节点之前的所有同级节点                         |
| self               | 选取当前节点                                           |

示例：

| 例子                  | 结果                                     |
| --------------------- | ---------------------------------------- |
| child::book           | 选取所有属于当前节点的子元素的 book 节点 |
| attribute::*          | 选取当前节点的所有属性                   |
| child::text()         | 选取当前节点的所有文本子节点             |
| child::node()         | 选取当前节点的所有子节点                 |
| child::*/child::price | 选取当前节点的所有 price 孙节点          |

## XPath 运算符

XPath 表达式可返回节点集、字符串、逻辑值以及数字。
```
|, +, -, *, div, =, !=, <, >, <=, >=, or, and, mod
```

## Xpath 函数

[XPath、XQuery 以及 XSLT 函数](https://www.w3school.com.cn/xpath/xpath_functions.asp)

# XSLT

XSL 转换 （XSL Transformations, XSTL）是一种用于将 XML 文档转换为 XHTML 文档或其他 XML 文档的语言。
- XSLT 用于将一种 XML 文档转换为另外一种 XML 文档，或者可被浏览器识别的其他类型的文档，比如 HTML 和 XHTML。
    - 通常，XSLT 是通过把每个 XML 元素转换为 (X) HTML 元素来完成这项工作的。
    - 通过 XSLT，您可以向或者从输出文件添加或移除元素和属性。
    - 也可重新排列元素，执行测试并决定隐藏或显示哪个元素，等等。
    - 描述转化过程的一种通常的说法是，XSLT 把 XML 源树转换为 XML 结果树。
- XSLT 使用 XPath 在 XML 文档中查找信息。XPath 被用来通过元素和属性在 XML 文档中进行导航。
- 在转换过程中，XSLT 使用 XPath 来定义源文档中可匹配一个或多个预定义模板的部分。一旦匹配被找到，XSLT 就会把源文档的匹配部分转换为结果文档。

语法：
```
<?xml version="1.0" encoding="ISO-8859-1"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    ...
</xsl:stylesheet>
```
- XSL 样式表本身也是一个 XML 文档，因此它总是由 XML 声明起始
- `<xsl:stylesheet>`，定义此文档是一个 XSLT 样式表文档（连同版本号和 XSLT 命名空间属性）。

## XSLT 元素

XSLT 元素命名空间的默认前缀为 xsl

| 元素                   | 描述                                                                      |
| ---------------------- | ------------------------------------------------------------------------- |
| apply-imports          | 应用来自导入样式表中的模版规则                                            |
| apply-templates        | 向当前元素或当前元素的子元素应用模板                                      |
| attribute              | 向元素添加属性                                                            |
| attribute-set          | 创建命名的属性集                                                          |
| call-template          | 调用一个指定的模板                                                        |
| choose                 | 与 when 以及 otherwise 协同使用，来表达多重条件测试                       |
| comment                | 在结果树中创建注释节点                                                    |
| copy                   | 创建当前节点的一个备份（无子节点及属性）                                  |
| copy-of                | 创建当前节点的一个备份（带有子节点及属性）                                |
| decimal-format         | 定义当通过 format-number() 函数把数字转换为字符串时，所要使用的字符和符号 |
| element                | 在输出文档中创建一个元素节点                                              |
| fallback               | 假如处理器不支持某个 XSLT 元素，规定一段备用代码来运行                    |
| for-each               | 遍历指定的节点集中的每个节点                                              |
| if                     | 包含一个模板，仅当某个指定的条件成立时应用此模板                          |
| import                 | 用于把一个样式表中的内容倒入另一个样式表中                                |
| include                | 把一个样式表中的内容包含到另一个样式表中                                  |
| key                    | 声明一个命名的键                                                          |
| message                | 向输出写一条消息（用于错误报告）                                          |
| namespace-alias        | 把样式表中的命名空间替换为输出中不同的命名空间                            |
| number                 | 测定当前节点的整数位置，并对数字进行格式化                                |
| otherwise              | 规定 choose 元素的默认动作                                                |
| output                 | 定义输出文档的格式                                                        |
| param                  | 声明一个局部或全局参数                                                    |
| preserve-space         | 用于定义保留空白的元素                                                    |
| processing-instruction | 生成处理指令节点                                                          |
| sort                   | 对结果进行排序                                                            |
| strip-space            | 定义应当删除空白字符的元素                                                |
| stylesheet             | 定义样式表的根元素                                                        |
| template               | 当指定的节点被匹配时所应用的规则                                          |
| text                   | 通过样式表生成文本节点                                                    |
| transform              | 定义样式表的根元素                                                        |
| value-of               | 提取选定节点的值                                                          |
| variable               | 声明局部或者全局的变量                                                    |
| when                   | 规定 choose 元素的动作                                                    |
| with-param             | 规定需被传入某个模板的参数的值                                            |
[XSLT 元素参考手册](https://www.w3school.com.cn/xsl/xsl_w3celementref.asp)

## XSLT 函数

XSLT 含有超过 100 个内建的函数。这些函数用于字符串值、数值、日期和时间比较、节点和 QName 操作、序列操作、逻辑值，等等。

XSLT 函数的命名空间的 URI 是：
```
http://www.w3.org/2005/02/xpath-functions
```
函数命名空间的默认前缀是 fn。

提示：函数在被调用时常带有 fn: 前缀，比如 fn:string()。不过，既然 fn: 是命名空间的默认前缀，那么在被调用时，函数的名称不必使用前缀。

您可以在我们的 XPath 教程中访问所有内建的 XSLT 2.0 函数参考。

此外，在此列出了内建的 XSLT 函数：

| 名称                                        | 描述                                                           |
| ------------------------------------------- | -------------------------------------------------------------- |
| current()                                   | 返回当前节点作为唯一成员的节点集，类似 XPath 的 "."            |
| document(uri,node-set?)                     | 用于访问外部 XML 文档中的节点                                  |
| element-available(xsl:element-name)         | 检测 XSLT 处理器是否支持指定的元素                             |
| format-number(number,format,decimalformat?) | 把数字转换为字符串                                             |
| function-available(function-name)           | 检测 XSLT 处理器是否支持指定的函数                             |
| generate-id(node-set?)                      | 返回唯一标识指定节点的字符串值                                 |
| key(node-set, value)                        | 检索以前使用 \<xsl:key> 语句标记的元素                         |
| node-set(string)                            | 将树转换为节点集。产生的节点集总是包含单个节点并且是树的根节点 |
| system-property(xsl:prop-name)              | 返回系统属性的值                                               |
| unparsed-entity-uri(entity-name)            | 返回未解析实体的 URI                                           |
- 参数后加 "?" 表示可选参数
- 对于 format-number 方法的 format 参数

    | 符号 | 描述                                                   |
    | ---- | ------------------------------------------------------ |
    | \#   | (表示数字。例如：####)                                 |
    | 0    | (表示 “.” 字符前面和后面的零。例如：0000.00)           |
    | .    | (小数点的位置。例如：###.##)                           |
    | ,    | (千的组分隔符。例如：###,###.##)                       |
    | %    | (把数字显示为百分比。例如：##%)                        |
    | ;    | (模式分隔符。第一个模式用于正数，第二个模式用于负数。) |
- 对于 system-property 参数

    | 系统属性       | 说明                                                                                         |
    | -------------- | -------------------------------------------------------------------------------------------- |
    | xsl:version    | 提供处理器所实现的 XSLT 版本的数字；如果 XSLT 处理器实现本文档所指定的 XSLT 版本，该数字为 1 |
    | xsl:vendor     | XSLT 处理器的开发商                                                                          |
    | xsl:vendor-url | 标识 XSLT 处理器的开发商的 URL                                                               |

更多函数参见 XPath 函数

# XSL-FO

- XSL-FO 指可扩展样式表语言格式化对象（Extensible Stylesheet Language Formatting Objects），用于格式化供输出的 XML 数据。
- XSL-FO 文档是带有输出信息的 XML 文件。它们包含着有关输出布局以及输出内容的信息。
- XSL-FO 文档存储在以 .fo 或 .fob 为后缀的文件中。以 .xml 为后缀存储的 XSL-FO 文档也很常见，这样做的话可以使 XSL-FO 文档更易被 XML 编辑器存取。

语法：
```
<?xml version="1.0" encoding="ISO-8859-1"?>

<fo:root xmlns:fo="http://www.w3.org/1999/XSL/Format">
    ...
</fo:root>
```
- root 元素是 XSL-FO 文档的根元素。这个根元素也要声明 XSL-FO 的命名空间

## XSL-FO 区域

XSL-FO 使用矩形框（区域）来显示输出。
- XSL 格式化模型定义了一系列的矩形（区域）框来显示输出。
- 所有的输出都会被格式化到这些框中，然后会被显示或打印到某个目标媒介。

区域分类：
- Pages（页面）：
    - XSL-FO 输出会被格式化到页面中。
    - 打印输出通常会进入分为许多分割的页面。
    - 浏览器输出经常会成为一个长的页面。
    - 页面包含区域（Region）。
- Regions（区）
    - 每个 XSL-FO 页面均包含一系列的 Regions（区）：
        - region-body (页面的主体)
        - region-before (页面的页眉)
        - region-after (页面的页脚)
        - region-start (左侧栏)
        - region-end (右侧栏)
    - Regions 包含块区域（Block Area）。
- Block areas（块区域）
    - 块区域可定义小的块元素（通常由一个新行开始），比如段落、表格以及列表。
    - 块区域可包含其他的块区域，不过大多数时候它们包含的是行区域（Line Area）。
- Line areas（行区域）
    - 行区域定义了块区域内部的文本行。
    - 行区域包含行内区域（Inline Area）。
- Inline areas（行内区域）
    - 行内区域定了行内部的文本（着重号、单字符以及图像等等）。

[XSL-FO 参考手册](https://www.w3school.com.cn/xslfo/xslfo_reference.asp)