- [概述](#概述)
  - [内部 DTD](#内部-dtd)
  - [外部 DTD](#外部-dtd)
    - [私有 DTD](#私有-dtd)
    - [公共 DTD](#公共-dtd)
  - [DTD 的作用](#dtd-的作用)
- [DTD 构建模块](#dtd-构建模块)
  - [元素](#元素)
  - [属性](#属性)
    - [ID 和 IDREF 类型](#id-和-idref-类型)
    - [NMTOKEN 和 NMTOKENS 类型](#nmtoken-和-nmtokens-类型)
    - [NOTATION 类型](#notation-类型)
  - [实体](#实体)
  - [PCDATA](#pcdata)
  - [CDATA](#cdata)
- [参考](#参考)

# 概述

- 文档类型定义（Document Type Definition, DTD）可定义合法的 XML 文档构建模块。
- 它使用一系列合法的元素来定义文档的结构。
- DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。

## 内部 DTD

假如 DTD 被包含在您的 XML 源文件中，它应当通过下面的语法包装在一个 DOCTYPE 声明中：
```

<!DOCTYPE root_element [
  Document Type Definition(DTD):
    elements/attributes/entities/notations/processing instructions/comments/PE references
]>
```

示例：
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE note [                         <!-- 定义此文档是 note 类型的文档 -->
  <!ELEMENT note (to,from,heading,body)> <!-- 定义 note 元素有四个元素："to, from, heading, body" -->
  <!ELEMENT to      (#PCDATA)>           <!-- 定义 to 元素为 "#PCDATA" 类型 -->
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>
```

## 外部 DTD

假如 DTD 位于 XML 源文件的外部，那么它应通过下面的语法被封装在一个 DOCTYPE 定义中：

### 私有 DTD

私有 DTD 使用 SYSTEM 表示，接着是外部 DTD 的相对或绝对 URL。

```
<!DOCTYPE root_element SYSTEM "DTD_location">
```
这个 XML 文档和上面内部 DTD 示例的 XML 文档相同，但是拥有一个外部的 DTD：
```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note>
```
node.dtd，和 XML 文档在同一目录下
```
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```

### 公共 DTD

公共 DTD 则使用 PUBLIC，接着是 DTD 公共名称，接着是 DTD 的相对或绝对 URL。
```
<!DOCTYPE root_element PUBLIC "DTD_name" "DTD_location">

DTD_name ::= "prefix//owner_of_the_DTD//description_of_the_DTD//ISO 639_language_identifier"
```
DTD 名称格式：
- prefix：注册信息

    | 前缀  | 说明                                        |
    | ----- | ------------------------------------------- |
    | `ISO` | 该 DTD 是 ISO 标准。所有 ISO 标准均已通过。 |
    | `+`   | 该 DTD 是经过批准的非 ISO 标准              |
    | `-`   | 该 DTD 是未经过批准的非 ISO 标准            |
- owner_of_the_DTD：组织信息，是 DTD 的拥有者的名称，如 "W3C"
- description_of_the_DTD：描述信息，包括 4 部分
    - 类型信息：一般是 "DTD"
    - 标签：指定公开文本描述，即对所引用的公开文本的唯一描述性名称
    - 标签版本
    - DTD 类型：过渡的（Transitional）、严格的（Strict）和框架的（Frameset）
- ISO 639_language_identifier：是 DTD 语言的 ISO 639 语言标识符，如："EN" 表示英文，"ZH" 表示中文

示例：
```
<!DOCTYPE HTML PUBLIC
    "-//W3C//DTD HTML 4.0 Transitional//EN"
    "http://www.w3.org/TR/REC-html40/loose.dtd">
```

## DTD 的作用

- 通过 DTD，每一个 XML 文件均可携带一个有关其自身格式的描述。
- 通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。
- 应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。
- 还可以使用 DTD 来验证数据。

# DTD 构建模块

所有的 XML 文档（以及 HTML 文档）均由以下简单的构建模块构成：
- Elements（元素）
- Attributes（属性）
- Entities（实体）
- PCDATA
- CDATA

## 元素

- 元素是 XML 以及 HTML 文档的主要构建模块。
- HTML 元素的例子是 `<body></body>`，XML 元素的例子是 `<node></node>`。
- 元素的内容可以是文本、元素或者是空的。
- 空的 HTML 元素可被一个 "/" 关闭，例子是 `<br/>`。

语法：
```
<!ELEMENT element_name allowable_contents>
```

| allowable_contents 类型            | 说明                                   |
| ---------------------------------- | -------------------------------------- |
| `(#CDATE)`                         | 内容为 CDATE                           |
| `(#PCDAT)`                         | 内容为 PCDATE                          |
| `(child)` or `(Child1 child2 ...)` | 内容为子元素，多元素必须按定义顺序声明 |
| `(Child1,child2,...)`              | 内容为子元素，多元素不必按定义顺序声明 |
| `EMPTY`                            | 内容为空                               |
| `ANY`                              | 内容为任意                             |
| `?`, `*`, `+`, `|` 和其他类型共用  | 正则表达式的通配符                     |

示例：
```
<!ELEMENT note (body)>
<!-- "body" 子元素必须在 "note" 元素内出现至少一次 -->
<!ELEMENT note (body+)>
<!-- "note" 元素必须包含 "to" 元素、"from" 元素、"header" 元素，以及非 "message元素既 "body" 元素 -->
<!ELEMENT note (to,from,header,(message|body))>
<!-- "note" 元素可包含出现零次或多次的 PCDATA、"to"、"from"、"header" 或"message" -->
<!ELEMENT note (#PCDATA|to|from|header|message)*>
```

## 属性

- 属性可提供有关元素的额外信息。
- 属性总是被置于某元素的开始标签中。
- 属性总是以 `name="value"` 的形式成对出现的。

语法：
```
<!ATTLIST element_name attribute_name attribute_type "default_value">
```
- attribute_type

    | 类型           | 描述                                 |
    | -------------- | ------------------------------------ |
    | `CDATA`        | 值为字符数据 (character data)        |
    | `(en1|en2|..)` | 此值是枚举列表中的一个值             |
    | `ID`           | 值为唯一的 id                        |
    | `IDREF`        | 值为另外一个元素的 id                |
    | `IDREFS`       | 值为其他 id 的列表                   |
    | `NMTOKEN`      | 值为合法的 XML 名称                  |
    | `NMTOKENS`     | 值为合法的 XML 名称的列表            |
    | `ENTITY`       | 值是一个实体                         |
    | `ENTITIES`     | 值是一个实体列表                     |
    | `NOTATION`     | 值是记号的名称，和 NOTATION 搭配使用 |
    | `xml:`         | 值是一个预定义的 XML 值              |
- default_value

    | 值               | 说明           |
    | ---------------- | -------------- |
    | `value`          | 属性的省缺值   |
    | `#REQUIRED`      | 属性值是必需的 |
    | `#IMPLIED`       | 属性不是必需的 |
    | `#FIXED "value"` | 属性值是固定的 |

### ID 和 IDREF 类型

- IDREF 类型允许一个元素的属性使用文件中的另一个元素。
- 方法就是把那个元素的 ID 标识值作为该属性的取值。

示例：
```
<?xml version = "1.0" encoding="Gb2312" standalone = "yes"?>
<!DOCTYPE contacts [
    <!ELEMENT contacts ANY>
    <!ELEMENT contact(name,email)>
    <!ELEMENT name(#PCDATA)>
    <!ELEMENT email(#PCDATA)>
    <!ATTLIST contact id ID #REQUIRED>
    <!ATTLIST contact boss IDREF #IMPLIED>
]>
<contacts>
    <contact id="1">
        <name>张三</name>
        <EMAIL>zhang@abc.com</EMAIL>
    </contact>
    <contact id="2" boss="1">
        <name>李四</name>
        <EMAIL>li@abc.com</EMAIL>
    </contact>
</contacts>
```

### NMTOKEN 和 NMTOKENS 类型

- 类型 NMTOKEN 和 NMTOKENS 是诸多属性类型中面向处理程序的又一个类型。
- 这两个类型用于指示一个有效的名字。
- 当需要把一个元素和其它的元件，例如一个 JAVA 类或一个安全算法，相联系时，可以让它们助你一臂之力。

示例：
```
<?xml version = "1.0" encoding="Gb2312" standalone = "yes"?>
<!DOCTYPE data [
    <!ELEMENT data (#PCDATA)>
    <!ATTLIST data security (ON|OFF) "OFF"
                   authorized_user NMTOKENS #IMPLIED>
]>
<data security="ON" authorized_user = "Iggieeb SelenaS Guntherb">
    blah blah blah
</data>
```

### NOTATION 类型

- 记号允许属性值为一个 DTD 中声明的符号，这个类型对于使用非 XML 格式的数据非常有用。
- 现实世界中存在着很多无法或不易用 XML 格式组织的数据，例如图象、声音、影象等等。
- 对于这些数据，XML 应用程序常常并不提供直接的应用支持。通过为它们设定 NOTATION 类型的属性，可以向应用程序指定一个外部的处理程序。
- 例如，当你想要为一个给定的文件类型指定一个演示设备时，可以用 NOTATION 类型的属性作为触发。

NOTATION 声明语法：
```
<!NOTATION name SYSTEM "URI">
<!NOTATION name SYSTEM "MIME_type">
<!NOTATION name PUBLIC "public_ID">
<!NOTATION name PUBLIC "public_ID" "URI">
```

示例：
```
<?xml version = "1.0" encoding="Gb2312" standalone = "yes"?>
<!DOCTYPE file [
    <!ELEMENT file ANY>
    <!ELEMENT movie EMPTY>
    <!ATTLIST movie player NOTATION (mp|gif) #REQUIRED>
    <!NOTATION mp SYSTEM "movPlayer.exe">
    <!NOTATION gif SYSTEM "image/gif">
]>
<file>
    <movie player = "mp"/>
</fiel>
```

## 实体

- 实体是用来定义普通文本的变量。
- 实体引用是对实体的引用，总是由 `&` 开始，由 `;` 结束。
- 当文档被 XML 解析器解析时，实体就会被展开。
- 下面的实体在 XML 中被预定义：

    | 实体引用 | 字符 |
    | -------- | ---- |
    | `&lt;	`  | `<`  |
    | `&gt;	`  | `>`  |
    | `&amp;`  | `&`  |
    | `&quot;` | `"`  |
    | `&apos;` | `'`  |

语法：类似 DTD 声明，分为三类
```
<!ENTITY entity_name "entity_value">
<!ENTITY name SYSTEM "URI">
<!ENTITY name PUBLIC "public_ID" "URI">
```

示例：
```
<!ENTITY copyright "Copyright hkllyx.com">
<!ENTITY copyright SYSTEM "http://www.hkllyx.net/copyright.xml">'
<!ENTITY copyright PUBLIC "-//W3C//TEXT copyright//EN" "http://www.w3.org/xmlspec/copyright.xml">
```

## PCDATA

PCDATA 的意思是被解析的字符数据（parsed character data）。
- 可把字符数据想象为 XML 元素的开始标签与结束标签之间的文本。
- PCDATA 是会被解析器解析的文本。这些文本将被解析器检查实体以及标记。
- 文本中的标签会被当作标记来处理，而实体会被展开。
- 不过，被解析的字符数据不应当包含任何 `&`、`<`、`>` 字符；需要使用 `&amp;`、`&lt;` 以及 `&gt;` 实体来分别替换它们。

## CDATA

CDATA 的意思是字符数据（character data）。
- CDATA 是不会被解析器解析的文本。
- 在这些文本中的标签不会被当作标记来对待，其中的实体也不会被展开。

# 参考

- [Dtd](https://www.ibm.com/developerworks/cn/xml/x-cert/part2/index.html)
- [DTD 教程](https://www.w3school.com.cn/dtd/index.asp)