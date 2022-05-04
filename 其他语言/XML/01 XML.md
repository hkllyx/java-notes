- [XML概述](#xml概述)
    - [XML的特性](#xml的特性)
    - [XML的用途](#xml的用途)
    - [XML树结构](#xml树结构)
- [XML结构及其语法](#xml结构及其语法)
    - [XML声明](#xml声明)
    - [标签和元素](#标签和元素)
        - [语法](#语法)
        - [XML命名规范](#xml命名规范)
    - [属性](#属性)
        - [语法](#语法-1)
        - [XML元素vs. XML属性](#xml元素vs-xml属性)
    - [XML引用](#xml引用)
    - [XML文本](#xml文本)
    - [其他](#其他)
        - [XML注释](#xml注释)
        - [XML换行](#xml换行)
- [XML验证](#xml验证)
    - [形式良好的XML文档](#形式良好的xml文档)
    - [验证XML文档](#验证xml文档)
        - [XML DTD](#xml-dtd)
        - [XML Schema](#xml-schema)
- [XML显示样式](#xml显示样式)
    - [使用CSS显示XML](#使用css显示xml)
    - [使用XSLT显示XML](#使用xslt显示xml)
- [XML命名空间](#xml命名空间)
    - [命名冲突](#命名冲突)
    - [使用前缀来避免命名冲突](#使用前缀来避免命名冲突)
    - [使用命名空间（Namespaces）](#使用命名空间namespaces)
    - [XML Namespace (xmlns) 属性](#xml-namespace-xmlns-属性)
    - [默认的命名空间（Default Namespaces）](#默认的命名空间default-namespaces)
- [CDATA区段](#cdata区段)
    - [PCDATA](#pcdata)
    - [CDATA](#cdata)
- [XML & JavaScript](#xml--javascript)
    - [XMLHttpRequest对象](#xmlhttprequest对象)
    - [XML解析器](#xml解析器)
        - [解析XML文档（xml文件）](#解析xml文档xml文件)
- [XML DOM](#xml-dom)
        - [解析XML字符串（字符串对象）](#解析xml字符串字符串对象)

# XML概述

- XML指可扩展标记语言（e**X**tensible **M**arkup **L**anguage）
- XML是一种标记语言，很类似HTML
- XML的设计宗旨是传输数据，而非显示数据
- XML被设计为具有自我描述性
- XML是W3C的推荐标准

## XML的特性

1. 没有任何行为，是不作为的
    - XML被设计用来结构化、存储以及传输信息，没有定义任何行为，不会对数据做任何事情
2. 仅仅是纯文本
    - XML没什么特别的，它仅仅是纯文本而已，有能力处理纯文本的软件都可以处理XML。
    - 不过，能够读懂XML的应用程序可以有针对性地处理XML的标签，标签的功能性意义依赖于应用程序的特性。
3. XML不是对HTML的替代，XML和HTML为不同的目的而设计
    - XML被设计为传输和存储数据，其焦点是数据的内容，HTML被设计用来显示数据，其焦点是数据的外观。
    - HTML旨在显示信息，而XML旨在传输信息。
    - 对XML最好的描述是：XML是独立于软件和硬件的信息传输工具。
4. XML标签没有被预定义，需要自行定义标签

## XML的用途

XML应用于web开发的许多方面，常用于简化数据的存储和共享。
1. XML把数据从HTML分离
    - 如果你需要在HTML文档中显示动态数据，那么每当数据改变时将花费大量的时间来编辑HTML。
    - 通过XML，数据能够存储在独立的XML文件中。这样你就可以专注于使用HTML进行布局和显示，并确保修改底层数据不再需要对HTML进行任何的改变。
    - 通过使用几行JavaScript，你就可以读取一个外部XML文件，然后更新HTML中的数据内容。
2. XML简化数据共享
    - 在真实的世界中，计算机系统和数据使用不兼容的格式来存储数据。
    - XML数据以纯文本格式进行存储，因此提供了一种独立于软件和硬件的数据存储方法。
    - 这让创建不同应用程序可以共享的数据变得更加容易。
3. XML简化数据传输
    - 通过XML，可以在不兼容的系统之间轻松地交换数据。
    - 对开发人员来说，其中一项最费时的挑战一直是在因特网上的不兼容系统之间交换数据。
    - 由于可以通过各种不兼容的应用程序来读取数据，以XML交换数据降低了这种复杂性。
4. XML简化平台的变更
    - 升级到新的系统（硬件或软件平台），总是非常费时的。必须转换大量的数据，不兼容的数据经常会丢失。
    - XML数据以文本格式存储。这使得XML在不损失数据的情况下，更容易扩展或升级到新的操作系统、新应用程序或新的浏览器。
5. XML用于创建新的Internet语言。很多新的Internet语言是通过XML创建的：
    - XHTML - 最新的HTML版本
    - WSDL - 用于描述可用的web service
    - WAP和WML - 用于手持设备的标记语言
    - RSS - 用于RSS feed的语言
    - RDF和OWL - 用于描述资源和本体
    - SMIL - 用于描述针针对web的多媒体

## XML树结构

XML文档形成了一种树结构，它从 “根部” 开始，然后扩展到 “枝叶”。
- 第一行是XML声明。它定义XML的版本和所使用的编码等信息。
- XML文档必须包含根元素。该元素是所有其他元素的父元素。
- XML文档中的元素形成了一棵文档树。这棵树从根部开始，并扩展到树的最底端。

# XML结构及其语法

## XML声明

XML文档可以有一个可选的XML声明。它可以写作如下形式：
```
<?xml version="version_number"
      encoding="encoding_declaration"
      standalone="standalone_status"?>
```

| 参数       | 取值                                       | 参数说明                                                                                    |
| ---------- | ------------------------------------------ | ------------------------------------------------------------------------------------------- |
| version    | 1.0                                        | 指定所用XML标准的版本。                                                                   |
| encoding   | UTF-8, UTF-16, ISO-8859-1 to ISO-8859-9等 | 定义文档中使用的字符编码。默认使用UTF-8编码。                                             |
| Standalone | yes或no                                  | 通知解析器文档是否依赖外部源信息，比如外部DTD、实体引用。默认值为no，表示依赖外部源信息。 |

XML声明应该遵守以下规则：
1. 如果XML声明出现在XML中，必须把它放在这个XML文档的第一行。
2. 如果包含XML声明，就必须包含版本号属性。
3. 参数名和值区分大小写。
4. 放置参数的顺序很重要。正确的顺序是：version，encoding和standalone。
5. 可以使用单引号或双引号。
6. XML声明没有闭合标签，比如 `</?xml>`。
7. 可以使用一个HTTP协议覆盖XML声明中指定的encoding的值。

## 标签和元素

元素：
- XML元素指的是从（且包括）开始标签直到（且包括）结束标签的部分。
- 元素可包含其他元素、文本或者两者的混合物。元素也可以拥有属性。

### 语法

1. XML命名规则
    - 名称可以含字母、数字以及其他的字符
    - 名称不能以数字或者标点符号开始
    - 名称不能以字符 “xml”（或者XML）开始
    - 名称不能包含空格
    - 可使用任何名称，没有保留的字词
2. 所以XML元素都须有关闭标签
    - 在XML中，省略关闭标签是非法的。所有元素都必须有关闭标签。
    - 声明不属于XML本身的组成部分。它不是XML元素，也不需要关闭标签。
3. XML标签对大小写敏感
    - XML元素使用XML标签进行定义。
    - XML标签对大小写敏感。在XML中，标签 `<Letter>` 与标签 `<letter>` 是不同的。
    - 必须使用相同的大小写来编写开始标签和结束标签。
4. XML文档必须有且只有一个根元素
    - XML文档必须有一个元素是所有其他元素的父元素。该元素称为根元素。
5. XML必须正确地嵌套
    - 正确嵌套的意思是：一个非根元素元素C在是在P元素内打开，那么它必须在P元素内关闭。
6. 对于空标签，可以使用 `<name/>` 格式。
7. XML元素是可扩展的
    - XML元素是可扩展，以携带更多的信息（在一个元素内添加子元素等）。

### XML命名规范

- 使名称具有描述性。使用下划线的名称也很不错。
- 名称应当比较简短，比如：`<book_title>`，而不是：`<the_title_of_the_book>`。
- 避免 `-` 字符。如果您按照这样的方式进行命名：`first-name`，一些软件会认为你需要提取第一个单词。
- 避免 `.` 字符。如果您按照这样的方式进行命名：`first.name`，一些软件会认为 `name` 是对象 `first` 的属性。
- 避免 `:` 字符。冒号会被转换为命名空间来使用。
- XML文档经常有一个对应的数据库，其中的字段会对应XML文档中的元素。有一个实用的经验，即使用数据库的名称规则来命名XML文档中的元素。
- 非英语的字母比如 `éòá` 也是合法的XML元素名，不过需要留意当软件开发商不支持这些字符时可能出现的问题。

## 属性

使用名 / 值对给元素指定一个属性（property）。

### 语法

1. 一个XML元素可以有一个或多个属性（attributes）。
2. XML的属性值须加引号
    - 与HTML类似，XML也可拥有属性（名称 / 值的对）。
    - 在XML中，XML的属性值须加引号（单引号或双引号）。
3. XML属性名区分大小写（和HTML不一样）。
    - 也就是说 `HREF` 和 `href` 会被认为是两个不同的XML属性。
4. 同一个元素相同的属性不能有重复。

### XML元素vs. XML属性

没有什么规矩可以告诉我们什么时候该使用属性，而什么时候该使用子元素。在HTML中，属性用起来很便利，但是在XML中，您应该尽量避免使用属性。如果信息感觉起来很像数据，那么尽量子元素。
- 避免XML属性
    - 属性无法包含多重的值（元素可以）
    - 属性无法描述树结构（元素可以）
    - 属性不易扩展（为未来的变化）
    - 属性难以阅读和维护
- 针对元数据的XML属性
    - 有时候会向元素分配ID引用。这些ID索引可用于标识XML元素，它起作用的方式与HTML中ID属性是一样的。
    - 元数据（有关数据的数据）应当存储为属性，而数据本身应当存储为元素。

## XML引用

引用：
- 通常允许我们在XML文档中添加或包含附加的文本。
- XML中有两种类型的引用：
    - 实体引用： 一个实体引用的起始和结束定界符之间包含一个名称。比如 `&amp;`，其中 `amp` 就是名称。这个名称通常指向一个预定义的文本字符串或标记。
    - 字符引用： 这些包含引用比如 `&#65`; 包含一个hash标记（`#`），后面紧跟一个数字。这个数字始终指向一个字符的Unicode码。在这里，65指向字母 `A`。

语法：
- 引用始终以 "&" 开始，这是一个保留字符，以 ";" 结尾。
- 在XML中，一些字符拥有特殊的意义。
    - `&` 和 `<` 只有作为标记定界符，或在注释，处理指令，或CDATA段中时才能以字面形式出现。
    - 如果在其他地方需要用到这两个字符，必须用数值式字符引用来转义或分别用实体引用 `&amp;` 和 `&lt;` 表示。
    - 在XML中，有5个预定义的实体引用

        | 实体引用 | 字符 |
        | -------- | ---- |
        | `&lt;	`  | `<`  |
        | `&gt;	`  | `>`  |
        | `&amp;`  | `&`  |
        | `&quot;` | `"`  |
        | `&apos;` | `'`  |
    - 注释：在XML中，只有字符 `<` 和 `&` 确实是非法的。`>` 是合法的，但是用实体引用来代替它是一个好习惯。

## XML文本

- XML元素和XML属性的名称区分大小写。这意味着元素的开始和结束标签大小写必须一致。
- 为了避免字符编码的问题，所有的XML文件都应该保存为Unicode UTF-8或者UTF-16文件。
- 空白字符，比如空格，制表符以及XML元素和XML属性之间换行符会被忽略。
- 有些字符是XML语法本身保留的。因此，不能直接使用它们。要使用它们，就要使用一些替代实体。

## 其他

### XML注释

在XML中编写注释的语法与HTML的语法很相似。
```
<!-- xml comment -->
```

### XML换行

XML以LF存储换行
- 在Windows应用程序中，换行通常以一对字符来存储：回车符 (CR) 和换行符 (LF)。这对字符与打字机设置新行的动作有相似之处。
- 在Unix应用程序中，新行以LF字符存储。
- 而Macintosh应用程序使用CR来存储新行。

# XML验证

## 形式良好的XML文档

“形式良好” （Well Formed）或 “结构良好” 的XML文档拥有正确的语法，会遵守前几章介绍过的XML语法规则：
- XML文档必须有根元素
- XML文档必须有关闭标签
- XML标签对大小写敏感
- XML元素必须被正确的嵌套
- XML属性必须加引号

## 验证XML文档

### XML DTD

DTD的作用是定义XML文档的结构。它使用一系列合法的元素来定义文档结构。

### XML Schema

W3C支持一种基于XML的DTD代替者。

# XML显示样式

## 使用CSS显示XML

```
<?xml-stylesheet type="text/css" href="css_URL"?>
```

示例：
- catlog.css
    ```
    CATALOG {
        background-color: #ffffff;
        width: 100%;
    }
    CD {
        display: block;
        margin-bottom: 30pt;
        margin-left: 0;
    }
    TITLE {
        color: #FF0000;
        font-size: 20pt;
    }
    ARTIST {
        color: #0000FF;
        font-size: 20pt;
    }
    COUNTRY,PRICE,YEAR,COMPANY {
        display: block;
        color: #000000;
        margin-left: 20pt;
    }
    ```
- catalog.xml
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <?xml-stylesheet type="text/css" href="catalog.css"?>
    <CATALOG>
        <CD>
            <TITLE>Empire Burlesque</TITLE>
            <ARTIST>Bob Dylan</ARTIST>
            <COUNTRY>USA</COUNTRY>
            <COMPANY>Columbia</COMPANY>
            <PRICE>10.90</PRICE>
            <YEAR>1985</YEAR>
        </CD>
    </CATALOG>
    ```

注释：使用CSS格式化XML不是常用的方法，更不能代表XML文档样式化的未来。W3C推荐使用XSLT。

## 使用XSLT显示XML

```
<?xml-stylesheet type="text/xsl" href="xsl_URL"?>
```

通过使用XSLT，您可以向XML文档添加显示信息。
- XSLT是首选的XML样式表语言。
- XSLT (eXtensible Stylesheet Language Transformations) 远比CSS更加完善。
- 使用XSLT的方法之一是在浏览器显示XML文件之前，先把它转换为HTML。

示例：
- simple.xsl
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <html xsl:version="1.0"
          xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
          xmlns="http://www.w3.org/1999/xhtml">

    <body style="font-family:Arial,helvetica,sans-serif;font-size:12pt; background-color:#EEEEEE">
        <xsl:for-each select="breakfast_menu/food">
            <div style="background-color:teal;color:white;padding:4px">
                <span style="font-weight:bold;color:white">
                    <xsl:value-of select="name"/>
                </span>
                -
                <xsl:value-of select="price"/>
            </div>
            <div style="margin-left:20px;margin-bottom:1em;font-size:10pt">
                <xsl:value-of select="description"/>
                <span style="font-style:italic">
                    (<xsl:value-of select="calories"/> calories per serving)
                </span>
            </div>
        </xsl:for-each>
    </body>
    </html>
    ```
- simple.xml
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <?xml-stylesheet type="text/xsl" href="simple.xsl"?>
    <breakfast_menu>
        <food>
            <name>Belgian Waffles</name>
            <price>$5.95</price>
            <description>
                two of our famous Belgian Waffles
            </description>
            <calories>650</calories>
        </food>
    </breakfast_menu>
    ```

# XML命名空间

XML命名空间提供避免元素命名冲突的方法。

## 命名冲突

在XML中，元素名称是由开发者定义的，当两个不同的文档使用相同的元素名时，就会发生命名冲突。

示例：
- 这个XML文档携带着某个表格中的信息：
    ```
    <table>
        <tr>
            <td>Apples</td>
            <td>Bananas</td>
        </tr>
    </table>
    ```
- 这个XML文档携带有关桌子的信息（一件家具）：
    ```
    <table>
        <f:name>African Coffee Table</f:name>
        <width>80</width>
        <length>120</length>
    </table>
    ```
- 假如这两个XML文档被一起使用，由于两个文档都包含带有不同内容和定义的 `<table>` 元素，就会发生命名冲突。
- XML解析器无法确定如何处理这类冲突。

## 使用前缀来避免命名冲突

- 此文档带有某个表格中的信息：
    ```
    <h:table>
        <h:tr>
            <h:td>Apples</h:td>
            <h:td>Bananas</h:td>
        </h:tr>
    </h:table>
    ```
- 此XML文档携带着有关一件家具的信息：
    ```
    <f:table>
        <f:name>African Coffee Table</f:name>
        <f:width>80</f:width>
        <f:length>120</f:length>
    </f:table>
    ```
- 现在，命名冲突不存在了，这是由于两个文档都使用了不同的名称来命名它们的 `<table>` 元素 (`<h:table>` 和 `<f:table>`)。
- 通过使用前缀，我们创建了两种不同类型的 `<table>` 元素。

## 使用命名空间（Namespaces）

- 这个XML文档携带着某个表格中的信息：
    ```
    <h:table xmlns:h="http://www.w3.org/TR/html4/">
        <h:tr>
            <h:td>Apples</h:td>
            <h:td>Bananas</h:td>
        </h:tr>
    </h:table>
    ```
- 此XML文档携带着有关一件家具的信息：
    ```
    <f:table xmlns:f="http://www.hkllyx.com/furniture">
        <f:name>African Coffee Table</f:name>
        <f:width>80</f:width>
        <f:length>120</f:length>
    </f:table>
    ```
- 与仅仅使用前缀不同，我们为 `<table>` 标签添加了一个xmlns属性，这样就为前缀赋予了一个与某个命名空间相关联的限定名称。

## XML Namespace (xmlns) 属性

XML命名空间属性被放置于元素的开始标签之中，并使用以下的语法：
```
xmlns:namespace-prefix="namespaceURI"
```
当命名空间被定义在元素的开始标签中时，所有带有相同前缀的子元素都会与同一个命名空间相关联。

注释：用于标示命名空间的地址不会被解析器用于查找信息。其惟一的作用是赋予命名空间一个惟一的名称。不过，很多公司常常会作为指针来使用命名空间指向实际存在的网页，这个网页包含关于命名空间的信息。

## 默认的命名空间（Default Namespaces）

为元素定义默认的命名空间可以让我们省去在所有的子元素中使用前缀的工作。
```
xmlns="namespaceURI"
```

示例：
- 这个XML文档携带着某个表格中的信息：
    ```
    <table xmlns="http://www.w3.org/TR/html4/">
        <tr>
            <td>Apples</td>
            <td>Bananas</td>
        </tr>
    </table>
    ```
- 此XML文档携带着有关一件家具的信息：
    ```
    <table xmlns="http://www.hkllyx.com/furniture">
        <name>African Coffee Table</name>
        <width>80</width>
        <length>120</length>
    </table>
    ```

# CDATA区段

所有XML文档中的文本均会被解析器解析，只有CDATA区段（CDATA section）中的文本会被解析器忽略。

## PCDATA

PCDATA指的是被解析的字符数据（Parsed Character Data）。
- XML解析器通常会解析XML文档中所有的文本。
- 当某个XML元素被解析时，其标签之间的文本也会被解析：
    ```
    <message>此文本也会被解析</message>
    ```
- 解析器之所以这么做是因为XML元素可包含其他元素：
    ```
    <name><first>Bill</first><last>Gates</last></name>
    ```
    而解析器会把它分解为像这样的子元素：
    ```
    <name>
       <first>Bill</first>
       <last>Gates</last>
    </name>
    ```

## CDATA

术语CDATA指的是不应由XML解析器进行解析的文本数据（Unparsed Character Data）。
- CDATA部分由 "\<![CDATA[" 开始，由 "]]>" 结束：
- CDATA部分中的所有内容都会被解析器忽略。
- 在XML元素中，"<" 和 "&" 是非法的。
    - "<" 会产生错误，因为解析器会把该字符解释为新元素的开始。
    - "&" 也会产生错误，因为解析器会把该字符解释为字符实体的开始。
    - 某些文本，比如JavaScript代码，包含大量 "<" 或 "&" 字符。为了避免错误，可以将脚本代码定义为CDATA。
- CDATA部分不能包含字符串 "]]>"。也不允许嵌套的CDATA部分。标记CDATA部分结尾的 "]]>" 不能包含空格或折行。

# XML & JavaScript

## XMLHttpRequest对象

- XMLHttpRequest对象用于在后台与服务器交换数据。
- XMLHttpRequest对象是开发者的梦想，因为您能够：
    - 在不重新加载页面的情况下更新网页
    - 在页面已加载后从服务器请求数据
    - 在页面已加载后从服务器接收数据
    - 在后台向服务器发送数据
    - 所有现代的浏览器都支持XMLHttpRequest对象

使用XMLHttpRequest对象的语法：
```
if (window.XMLHttpRequest) {
    // code for IE7+, Firefox, Chrome, Opera, Safari
    xmlHttpRequest = new XMLHttpRequest();
} else {
    // code for IE6, IE5
    xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
}
```

## XML解析器

- 所有现代浏览器都内建了供读取和操作XML的XML解析器。
- 解析器把XML转换为XML DOM对象 - 可通过JavaScript操作的对象。

### 解析XML文档（xml文件）

- 方法一：
    ```
    // 获取XMLHttpRequest对象
    xmlHttpRequest = ...
    xmlHttpRequest = xmlhttp.open("GET","xml_url",false);
    xmlHttpRequest.send();
    xmlDoc=xmlHttpRequest.responseXML;
    ```
- 方法二：
    ```
    // IE
    var xmlDoc = new ActiveXObject("Microsoft.XMLDOM");
    // Firefox及其他
    var xmlDoc=document.implementation.createDocument("","",null);

    xmlDoc.async = "false";
    xmlDoc.load("xml_uri");
    ```
    - 创建XML文档对象。
    - 关闭异步加载，这样确保在文档完全加载之前解析器不会继续脚本的执行。
    - 告知解析器加载指定的XML文档。

# XML DOM

DOM（Document Object Model，文档对象模型）定义了访问和操作文档的标准方法。

[XML DOM参考手册](https://www.w3school.com.cn/xmldom/xmldom_reference.asp)

### 解析XML字符串（字符串对象）

```
// 字符串txt
txt = ...;

if (window.DOMParser) {
    parser=new DOMParser();
    xmlDoc=parser.parseFromString(txt,"text/xml");
} else {
    // IE
    xmlDoc=new ActiveXObject("Microsoft.XMLDOM");
    xmlDoc.async="false";
    xmlDoc.loadXML(txt);
}
```
- IE使用loadXML () 方法来解析XML字符串，而其他浏览器使用DOMParser对象。

注释：loadXML() 方法用于加载字符串（文本），load() 用于加载文件。