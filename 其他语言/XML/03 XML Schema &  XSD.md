- [概述](#概述)
  - [作用](#作用)
  - [DTD的继任者](#dtd-的继任者)
- [为什么使用XML Schema](#为什么使用-xml-schema)
  - [XML Schema支持数据类型](#xml-schema-支持数据类型)
  - [XML Schema使用XML语法](#xml-schema-使用-xml-语法)
  - [XML Schema可保护数据通信](#xml-schema-可保护数据通信)
  - [XML Schema可扩展](#xml-schema-可扩展)
  - [形式良好是不够的](#形式良好是不够的)
- [示例及说明](#示例及说明)
- [简单的类型](#简单的类型)
  - [简易元素](#简易元素)
  - [属性](#属性)
  - [限定](#限定)
- [复杂类型](#复杂类型)
  - [复合元素](#复合元素)
  - [符合类型指示器](#符合类型指示器)
  - [any和anyAttribute元素](#any-和-anyattribute-元素)
  - [元素替换](#元素替换)
- [数据类型](#数据类型)
  - [字符串数据类型](#字符串数据类型)
  - [日期及时间数据类型](#日期及时间数据类型)
  - [十进制数值数据类型](#十进制数值数据类型)
  - [杂项数据类型](#杂项数据类型)
- [相关阅读](#相关阅读)

# 概述

- XML Schema是基于XML的DTD替代者。
- XML Schema描述XML文档的结构。
- XML Schema语言也称作XML Schema定义（XML Schema Definition，XSD）。
- XML Schema是W3C标准：XML Schema在2001年5月2日成为W3C标准。

## 作用

XML Schema的作用是定义XML文档的合法构建模块，类似DTD。
- XML Schema:
- 定义可出现在文档中的元素
- 定义可出现在文档中的属性
- 定义哪个元素是子元素
- 定义子元素的次序
- 定义子元素的数目
- 定义元素是否为空，或者是否可包含文本
- 定义元素和属性的数据类型
- 定义元素和属性的默认值以及固定值

## DTD的继任者

XML Schema很快会在大部分网络应用程序中取代DTD。
- XML Schema可针对未来的需求进行扩展
- XML Schema更完善，功能更强大
- XML Schema基于XML编写
- XML Schema支持数据类型
- XML Schema支持命名空间

# 为什么使用XML Schema

## XML Schema支持数据类型

XML Schema最重要的能力之一就是对数据类型的支持。

通过对数据类型的支持：
- 可更容易地描述允许的文档内容
- 可更容易地验证数据的正确性
- 可更容易地与来自数据库的数据一并工作
- 可更容易地定义数据约束（data facets）
    > 数据约束，或称facets，是XML Schema原型中的一个术语，中文可译为 “面”，用来约束数据类型的容许值
- 可更容易地定义数据模型（或称数据格式）
- 可更容易地在不同的数据类型间转换数据



## XML Schema使用XML语法

由XML编写XML Schema有很多好处：
- 不必学习新的语言
- 可使用XML编辑器来编辑Schema文件
- 可使用XML解析器来解析Schema文件
- 可通过XML DOM来处理Schema
- 可通过XSLT来转换Schema

## XML Schema可保护数据通信

当数据从发送方被发送到接受方时，其要点是双方应有关于内容的相同的 “期望值”。

通过XML Schema，发送方可以用一种接受方能够明白的方式来描述数据。

一种数据，比如 "03-11-2004"，在某些国家被解释为11月3日，而在另一些国家为当作3月11日。

但是一个带有数据类型的XML元素，比如：`<date type="date">2004-03-11</date>`，可确保对内容一致的理解，这是因为XML的数据类型 "date" 要求的格式是 "YYYY-MM-DD"。

## XML Schema可扩展

XML Schema是可扩展的，因为它们由XML编写。

通过可扩展的Schema定义，您可以：
- 在其他Schema中重复使用您的Schema
- 创建由标准类型衍生而来的您自己的数据类型
- 在相同的文档中引用多重的Schema

## 形式良好是不够的

我们把符合XML语法的文档称为形式良好的XML文档。

即使文档的形式良好，仍然不能保证它们不会包含错误，并且这些错误可能会产生严重的后果。

请考虑下面的情况：您订购的了5打激光打印机，而不是5台。

通过XML Schema，大部分这样的错误会被您的验证软件捕获到。

# 示例及说明

将DTD教程中的 "node.dtd" 转换成XML Schema格式："node.xsd"
```
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
    targetNamespace="http://www.hkllyx.com"
    xmlns="http://www.hkllyx.com"
    elementFormDefault="qualified">

    <xs:element name="note">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="to" type="xs:string"/>
                <xs:element name="from" type="xs:string"/>
                <xs:element name="heading" type="xs:string"/>
                <xs:element name="body" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```
xs:schema元素是每一个XML Schema的根元素。它有几个属性：
- `xmlns:xs`：定义schema中使用的元素和数据类型属于 "http://www.w3.org/2001/XMLSchema" 命名空间；并指定如果要使用该命名空间中定义的元素和数据类型必须以 `xs:` 开头。
- `targetNamespace`：指定在当前的schema中定义的元素是在这个命名空间中定义的。
- `xmlns`：指定默认的命名空间。
- `elementFormDefault`：指定在XML文件中使用的元素是否是必须是该schema中定义的合法元素。

在XML中引用

```
<?xml version="1.0"?>
<note xmlns="http://www.hkllyx.com"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.hkllyx.com note.xsd">

    <to>George</to>
    <from>John</from>
    <heading>Reminder</heading>
    <body>Don't forget the meeting!</body>
</note>
```
- `xmlns`：规定了默认命名空间的声明。此声明会告知schema验证器，在此XML文档中使用的所有元素都被声明于这个命名空间。
- `xmlns:xsi`：XML Schema实例命名空间
- `xsi:schemaLocation`：此属性有两个值：
    - 第一个值是需要使用的命名空间
    - 第二个值是供命名空间使用的XML Schema的位置

# 简单的类型

## 简易元素

XML Schema可定义XML文件的元素。
- 简易元素指那些只包含文本的元素，它不会包含任何其他的元素或属性。
- “仅包含文本” 这个限定很容易造成误解。文本有很多类型。它可以是XML Schema定义中包括的类型中的一种（布尔、字符串、数据等等），或者它也可以是您自行定义的定制类型。
- 也可向数据类型添加限定（即facets），以此来限制它的内容，或者您可以要求数据匹配某种特定的模式。

语法：
```
<xs:element name="element_name"
            type="data_type"
            [default|fixed="value"]/>

<xs:element ref="element_name">
```
- `[` 和 `]` 之间表示可选的。`|` 表是或，多选一。
- default表示简单元素的省缺值，fixed表示简单元素的固定值。
- 常用的data_type：
    - xs:string
    - xs:decimal
    - xs:integer
    - xs:boolean
    - xs:date
    - xs:time
- 使用ref可以引用一个元素

示例：
```
<xs:element name="color" type="xs:string" default="red"/>
```

## 属性

所有的属性均作为简易类型来声明。
- 简易元素无法拥有属性。假如某个元素拥有属性，它就会被当作某种复合类型。
- 但是属性本身总是作为简易类型被声明的。

语法：
```
<xs:attribute name="attribute_name"
              type="data_type"
              [default|fixed="value"]
              [use="required|optional"]/>
```
- required表示属性为必选。
- optional表示属性为可选。

示例：
```
<xs:attribute name="lang" type="xs:string" fixed="EN" use="required"/>
```

## 限定

限定（restriction）用于为XML元素或者属性定义可接受的值。对XML元素的限定被称为facet。

语法：
```
<xs:element name="element_name" [type="simple_type_name"]>
    <xs:simpleType [name="simple_type_name"]>
        <xs:restriction base="data_type">
            <xs:enumeration | xs:fractionDigits |...>
            ...
        </xs:restriction>
    </xs:simpleType>
</xs:element>
```
数据类型的限定：

| 限定           | 说明                                                       |
| -------------- | ---------------------------------------------------------- |
| enumeration    | 定义可接受值的一个列表，方法是使用多个enumeration限定    |
| fractionDigits | 定义所允许的最大的小数位数。必须大于等于0。               |
| length         | 定义所允许的字符或者列表项目的精确数目。必须大于或等于0。 |
| maxExclusive   | 定义数值的上限。所允许的值必须小于此值。                   |
| maxInclusive   | 定义数值的上限。所允许的值必须小于或等于此值。             |
| maxLength      | 定义所允许的字符或者列表项目的最大数目。必须大于或等于0。 |
| minExclusive   | 定义数值的下限。所允许的值必需大于此值。                   |
| minInclusive   | 定义数值的下限。所允许的值必需大于或等于此值。             |
| minLength      | 定义所允许的字符或者列表项目的最小数目。必须大于或等于0。 |
| pattern        | 使用正则表达式定义可接受的字符的精确序列。                 |
| totalDigits    | 定义所允许的阿拉伯数字的精确位数。必须大于0。             |
| whiteSpace     | 定义空白字符（换行、回车、空格以及制表符）的处理方式。     |

空白字符限定取值及意义：

| 取值     | 说明                                                                                                                                       |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| preserve | XML处理器不会移除任何空白字符                                                                                                             |
| replace  | XML处理器将移除所有空白字符                                                                                                               |
| collapse | XML处理器将移除缩减空白字符（换行、回车、空格、制表符会被替换为空格，开头和结尾的空格会被移除，而多个连续的空格会被缩减为一个单一的空格） |

示例：
```
<xs:element name="car" type="carType"/>
    <xs:simpleType name="carType">
        <xs:restriction base="xs:string">
            <xs:enumeration value="Audi"/>
            <xs:enumeration value="Golf"/>
            <xs:enumeration value="BMW"/>
    </xs:restriction>
</xs:simpleType>
```
注释：在这种情况下，类型 "carType" 可被其他元素使用，因为它不是 "car" 元素的组成部分。

# 复杂类型

## 复合元素

复合元素指包含其他元素及 / 或属性的XML元素。

语法：
```
<xs:element name="element_name">
    <xs:complexType>
        ...
    </xs:complexType>
</xs:element>
```

有四种类型的复合元素：
- 空元素
    ```
    <xs:element name="element_name">
        <xs:complexType>
            <xs:complexContent>
                [<xs:attribute .../> | <xs:restriction ...>..</xs:restriction>]
            </xs:complexContent>
        </xs:complexType>
    </xs:element>
    ```
- 包含其他元素的元素
    ```
    <xs:element name="element_name">
        <xs:complexType>
            <xs:sequence>
                [<xs:element .../>
                 | <xs:attribute .../>
                 | <xs:restriction ...>...</xs:restriction>]
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    ```
- 仅包含文本的元素
    ```
    <xs:element name="element_name">
        <xs:complexType>
            <xs:complexContent>
                <xs:extension base="data_type">
                    <xs:attribute .../>
                </xs:extension>
            </xs:complexContent>
        </xs:complexType>
    </xs:element>
    ```
- 包含元素和文本的元素
    ```
    <xs:element name="element_name">
        <xs:complexType>
            <xs:sequence mixed="true">
                [<xs:element .../>
                 | <xs:attribute .../>
                 | <xs:restriction ...>...</xs:restriction>]
            </xs:sequence>
        </xs:complexType>
    </xs:element>
    ```

注释：上述元素均可包含属性！

## 符合类型指示器

通过指示器，我们可以控制在文档中使用元素的方式。有七种指示器：
- Order指示器：用于定义元素的顺序
    - all：规定子元素可以按照任意顺序出现，且每个子元素必须只出现一次
        > 当使用all指示器时，你可以把minOccurs设置为0或1，而只能把maxOccurs指示器设置为1。
    - choice：规定可出现某个子元素或者可出现另外一个子元素（非此即彼）
        > 如需设置子元素出现任意次数，可将maxOccurs（稍后会讲解）设置为unbounded（无限次）
    - sequence：规定子元素必须按照特定的顺序出现
- Occurrence指示器：定义某个元素出现的频率
    - maxOccurs
    - minOccurs
- Group指示器：定义相关的元素、属性组
    - group
    - attributeGroup

语法：
```
<!-- 使用group指示器定义元素组： -->
<xs:group name="group_name">
    <xs:sequence>
        <xs:element .../>
        <xs:element .../>
        ...
    </xs:sequence>
</xs:group>

<!-- 使用attributeGroup指示器定义属性组： -->
<xs:attributeGroup name="attribute_group_name">
    <xs:sequence>
        <xs:attribute .../>
        <xs:attribute .../>
        ...
    </xs:sequence>
</xs:attributeGroup>

<xs:element name="element_name">
    <xs:complexType>
        <!-- 使用Order指示器 -->
        <xs:all | xs:choice | xs:sequence>
            <xs:element ...
                        <!-- 使用Occurrence指示器 -->
                        [maxOccurs="times" | minOccurs="times"]/>
            <!-- 使用group指示器引用元素组 -->
            <xs:group ref="group_name">
        </xs:all | /xs:choice | /xs:sequence>

        <!-- 使用attributeGroup指示器引用属性组： -->
        <xs:attributeGroup ref="attribute_group_name"/>
    </xs:complexType>
</xs:element>
```

## any和anyAttribute元素

any和anyAttribute元素使我们有能力通过未被schema规定的元素和属性来拓展XML文档。

语法：
```
<xs:element name="element_name">
    <xs:complexType>
        <xs:sequence>
            <xs:any [maxOccurs="times"] [minOccurs="times"]/>*
        </xs:sequence>
    </xs:complexType>
    <xs:anyAttribute/>
</xs:element>
```

## 元素替换

通过XML Schema，一个元素可对另一个元素进行替换（Element Substitution）。

在XML schema中定义一个substitutionGroup：首先，我们声明主元素，然后我们会声明次元素，这些次元素可声明它们能够替换主元素。

语法：
```
<xs:element name="main_element_name" type="data_type"/>
<xs:element name="sub_element_name" substitutionGroup="main_element_name"/>
```

# 数据类型

## 字符串数据类型

- 字符串数据类型用于可包含字符串的值。
- 字符串数据类型可包含字符、换行、回车以及制表符。
- 如果使用字符串数据类型，XML处理器就不会更改其值。

请注意，所有以下的数据类型均衍生于字符串数据类型（除了字符串数据类型本身）！

| 名称             | 描述                                                               |
| ---------------- | ------------------------------------------------------------------ |
| ENTITIES         | ENTITY列表                                                        |
| ENTITY           | 实体引用，是在DTD中声明为未解析实体的NCName                     |
| ID               | 在XML中提交ID属性的字符串 (仅与schema属性一同使用)           |
| IDREF            | 在XML中提交IDREF属性的字符串 (仅与schema属性一同使用)        |
| IDREFS           | language包含合法的语言id的字符串                                |
| Name             | 包含合法XML名称的字符串                                          |
| NCName           | nonqualified name名称，不运行字符串中出现 ":"                     |
| NMTOKEN          | 在XML中提交NMTOKEN属性的字符串 (仅与schema属性一同使用)      |
| NMTOKENS         | NMTOKEN列表                                                       |
| normalizedString | 不包含换行符、回车或制表符的字符串                                 |
| QName            | qualified name的简写，"prefix:localPart" 形式                     |
| string           | 字符串                                                             |
| token            | 不包含换行符、回车或制表符、开头或结尾空格或者多个连续空格的字符串 |

可与字符串数据类型一同使用的限定：
- enumeration
- length
- maxLength
- minLength
- pattern (NMTOKENS、IDREFS以及ENTITIES无法使用此约束)
- whiteSpace

## 日期及时间数据类型

日期及时间数据类型用于包含日期和时间的值。

| 名称       | 描述                 | 格式              |
| ---------- | -------------------- | ----------------- |
| date       | 定义一个日期值       | YY-MM-DD          |
| dateTime   | 定义一个日期和时间值 | YY-MM-DDTHH:mm:ss |
| duration   | 定义一个时间间隔     | -PnYnMnDTnHnMnS   |
| gDay       | 定义天               | DD                |
| gMonth     | 定义月               | MM                |
| gMonthDay  | 定义月和天           | MM-DD             |
| gYear      | 定义年               | YYYY              |
| gYearMonth | 定义年和月           | YYYY-MM           |
| time       | 定义一个时间值       | HH:mm:ss          |
- 对于date、time、dateTime,- 如需规定一个时区：
    - 在日期后加一个 "Z" 表示使用UTC时间。如 "2000-01-01Z"
    - 在日期后添加一个正的或负时间的方法，来规定以UTC时间为准的偏移量。如 "2000-01-01+8:00"
- 对于duration，P表示周期，是必须的，而其他则是可选的，且可以是负的。如 "P5Y"，"PT12H"，"-P5Y"

可与日期数据类型一同使用的限定：
- enumeration
- maxExclusive
- maxInclusive
- minExclusive
- minInclusive
- pattern
- whiteSpace

## 十进制数值数据类型

十进制数值数据类型用于数值。

请注意，下面所有的数据类型均源自于十进制数据类型（除decimal本身以外）！

| 名字               | 描述               |
| ------------------ | ------------------ |
| byte               | 8位整数           |
| decimal            | 十进制数           |
| int                | 32位整数          |
| integer            | 整数值             |
| long               | 64位整数          |
| negativeInteger    | 仅包含负值的整数   |
| nonNegativeInteger | 仅包含非负值的整数 |
| nonPositiveInteger | 仅包含非正值的整数 |
| positiveInteger    | 仅包含正值的整数   |
| short              | 16位整数          |
| unsignedLong       | 无符号64位整数   |
| unsignedInt        | 无符号32位整数   |
| unsignedShort      | 无符号16位整数   |
| unsignedByte       | 无符号8位整数    |

可与数值数据类型一同使用的限定：
- enumeration
- fractionDigits
- maxExclusive
- maxInclusive
- minExclusive
- minInclusive
- pattern
- totalDigits
- whiteSpace

## 杂项数据类型

其他杂项数据类型包括逻辑、base64Binary、十六进制、浮点、双精度、anyURI、anyURI以及NOTATION。


| 名称         | 描述                                              |
| ------------ | ------------------------------------------------- |
| anyURI       | 用于规定URI，使用 "%20" 替换空格（URL编码）     |
| base64Binary | Base64编码的二进制数据                           |
| boolean      | 合法的布尔值是true、false、1（true）、0（false） |
| double       | 双精度浮点值                                      |
| float        | 单精度浮点值                                      |
| hexBinary    | 十六进制编码的二进制数据                          |
| NOTATION     | 引用一个xs:notation元素                         |

# 相关阅读

- [XSD](https://www.w3schools.com/xml/schema_intro.asp)
- [什么是QName](https://blog.csdn.net/Benjieming_Wang/article/details/5959961)
- 《[RELAX NG](http://books.xmlschemata.org/relaxng/page2.html)》by Eric van der Vlist is published by O'Reilly
- [RELAX NG Tutorial](https://relaxng.org/tutorial-20011203.html)