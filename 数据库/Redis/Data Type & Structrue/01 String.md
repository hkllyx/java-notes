- [概述](#概述)
- [使用场景](#使用场景)
- [基础命令](#基础命令)
    - [SET, SETNX, SETEX, PSETEX, MSET, MSETNX](#set-setnx-setex-psetex-mset-msetnx)
    - [SETRANGE](#setrange)
    - [APPEND](#append)
    - [GET, MGET](#get-mget)
    - [GETRANGE](#getrange)
    - [GETSET](#getset)
    - [STRLEN](#strlen)
- [数值相关命令](#数值相关命令)
    - [INCR, DECR, INCRBY, DECRBY](#incr-decr-incrby-decrby)
    - [INCRBYFLOAT](#incrbyfloat)
- [Bitmap相关命令](#bitmap相关命令)
    - [BITCOUNT](#bitcount)
    - [GETBIT](#getbit)
    - [SETBIT](#setbit)
    - [BITOP](#bitop)
    - [BITPOS](#bitpos)
    - [BITFIELD](#bitfield)

# 概述

- 字符串是一种最基本的Redis值类型。
- Redis字符串是二进制安全的，这意味着一个Redis字符串能包含任意类型的数据，例如：一张JPEG格式的图片或者一个序列化的Ruby对象。
- 一个字符串类型的值最多能存储512M字节的内容。

# 使用场景

- 利用`INCR`命令簇（`INCR`, `DECR`, `INCRBY`）来把字符串当作原子计数器使用。
- 使用`APPEND`命令在字符串后添加内容。
- 将字符串作为`GETRANGE`和`SETRANGE`的随机访问向量。
- 在小空间里编码大量数据，或者使用`GETBIT`和`SETBIT`创建一个Redis支持的Bloom过滤器。

# 基础命令

## SET, SETNX, SETEX, PSETEX, MSET, MSETNX

```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```
- 将key设定为指定的 “字符串” 值。
- value可以是每种类型的string（包括二进制数据），例如，您可以在值内存储jpeg图像。value不能大于512 MB。
- 如果key已经保存了一个值，那么这个操作会直接覆盖原来的值，并且忽略原始类型。
- 当`SET`命令执行成功之后，之前设置的过期时间都将失效。
- 选项：

    | 选项            | 说明                                   | 相关命令 |
    | --------------- | -------------------------------------- | -------- |
    | EX seconds      | 设置key的过期时间，单位时秒          | `SETEX`  |
    | PX milliseconds | 设置key的过期时间，单位时毫秒        | `PSETEX` |
    | NX              | 只有key不存在的时候才会设置key的值 | `SETNX`  |
    | XX              | 只有key存在的时候才会设置key的值   |          |

```
SETNX key value
SETEX key seconds value
PSETEX key milliseconds value
```
- 注意：注意: 由于`SET`命令加上选项已经可以完全取代`SETNX`, `SETEX`, `PSETEX`的功能，所以在将来的版本中，redis可能会不推荐使用并且最终抛弃这几个命令。

```
MSET key value [key value ...]
MSETNX key value [key value ...]
```
- `MSET` 给定的keys对应到相应的values，就像普通的SET命令一样，`MSET` 会用新的value替换已经存在的value。
- 如果你不想覆盖已经存在的values，可以使用`MSETNX`。只要有一个key已经存在，`MSETNX`一个操作都不会执行。
- `MSET` 和`MSETNX`是原子的，所以所有给定的keys是一次性set的。客户端不可能看到这种一部分keys被更新而另外的没有改变的情况。

## SETRANGE

```
SETRANGE key offset value
```
- 覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。
- 如果offset比当前key对应string还要长，那这个string后面就补`\x00`以达到offset。不存在的keys被认为是空字符串，所以这个命令可以确保key有一个足够大的字符串，能在offset处设置value。
- 注意，offset最大可以是2^29 - 1（536870911），因为redis字符串限制在512M大小。如果你需要超过这个大小，你可以用多个keys。
- 警告 ：当设置最后一个字节并且key还没有一个字符串value或者其value是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。注意，一旦第一次内存分配完，后面对同一个key调用SETRANGE就不会预先得到内存分配。

## APPEND

```
APPEND key value
```
- 如果key已经存在，并且value为字符串，那么这个命令会把value追加到原value的结尾。
- 如果key不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况`APPEND`将类似于`SET`操作。

## GET, MGET

```
GET key
MGET key [key ...]
```
- `GET` 返回key的value。如果key不存在，返回特殊值nil。如果key的value不是string，就返回错误，因为GET只处理string类型的values。
- `MGET` 返回所有指定的key的value。对于每个不对应string或者不存在的key，都返回特殊值nil。正因为此，这个操作从来不会失败。

## GETRANGE

```
GETRANGE key start end
```
- 这个命令是被改成`GETRANGE`的，在小于2.0的Redis版本中叫 `SUBSTR`。
- 返回key对应的字符串value的子串，这个子串是由start和end位移（两个位移处的字符都包括在结果中）决定的。
- 可以用负的位移来表示从string尾部开始数的下标。所以 -1就是最后一个字符，-2就是倒数第二个，以此类推。
- 这个函数处理超出范围的请求时，都把结果限制在string内。

## GETSET

```
GETSET key value
```
- 原子性地将key对应到value并且返回原来key对应的value。
- 如果key存在但是对应的value不是字符串，就返回错误。

## STRLEN

```
STRLEN key
```
- 返回key的string类型value的长度。如果key对应的非string类型，就返回错误。

# 数值相关命令

## INCR, DECR, INCRBY, DECRBY

```
INCR key
DECR key
```
- 对key对应的数字做加或减1操作。
- 如果key不存在，那么在操作之前，这个key对应的值会被置为0。
- 如果key有一个错误类型的value或者是一个不能表示成数字的字符串，就返回错误。这个操作最大支持在64位有符号的整型数字。

查看命令INCR了解关于增减操作的额外信息。

```
INCRBY key increment
DECRBY key decrement
```
- 类似`INCR`和 `DECR`，不同的是将key对应的数字加increment或减decrement。

## INCRBYFLOAT

```
INCRBYFLOAT key increment
```
- 通过指定浮点数key来增长浮点数 (存放于string中) 的值. 当键不存在时，先将其值设为0再操作。下面任一情况都会返回错误:
    - key包含非法值 (不是一个string).
    - 当前的key或者相加后的值不能解析为一个双精度的浮点值.(超出精度范围了)
- 如果操作命令成功，相加后的值将替换原值存储在对应的键值上，并以string的类型返回。
- string中已存的值或者相加参数可以任意选用指数符号，但相加计算的结果会以科学计数法的格式存储。
- 无论各计算的内部精度如何，输出精度都固定为小数点后17位.

# Bitmap相关命令

## BITCOUNT

```
BITCOUNT key [start end]
```
- 统计字符串被设置为1的bit数。
- 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的start或end参数，可以让计数只在特定的位上进行。
- start和end参数的设置和`GETRANGE`命令类似，都可以使用负数值：比如 -1表示最后一个位，而 -2表示倒数第二个位，以此类推。
- 不存在的key被当成是空字符串来处理，因此对一个不存在的key进行`BITCOUNT`操作，结果为0。

## GETBIT

```
GETBIT key offset
```
- 返回key对应的string在offset处的bit值。
- 当offset超出了字符串长度的时候，这个字符串就被假定为由0比特填充的连续空间。
- 当key不存在的时候，它就认为是一个空字符串，所以offset总是超出范围，然后value也被认为是由0比特填充的连续空间。

## SETBIT

```
SETBIT key offset value
```
- 设置或者清空key的value (字符串) 在offset处的bit值。
- 那个位置的bit要么被设置，要么被清空，这个由value（只能是0或1）来决定。
- 当key不存在的时候，就创建一个新的字符串value。要确保这个字符串大到在offset处有bit值。参数offset需要大于等于0，并且小于2^32（限制bitmap大小为512M）。当key对应的字符串增大的时候，新增的部分bit值都是设置为0。
- 警告：当设置最后一个bit（offset等于2^32 - 1）并且key还没有一个字符串value或者其value是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。注意，一旦第一次内存分配完，后面对同一个key调用`SETBIT`就不会预先得到内存分配。

## BITOP

```
BITOP operation destkey key [key ...]
```
- 对一个或多个保存二进制位的字符串key进行位元操作，并将结果保存到destkey上。
- operation参数：

    | 参数 | 说明                        |
    | ---- | --------------------------- |
    | AND  | 与                          |
    | OR   | 或                          |
    | XOR  | 异或                        |
    | NOT  | 非，只能对一个key进行操作 |
- 处理不同长度的字符串
    - 集合中所有比最长string短的string在尾部补0至最长string的长度。
    - 不存在的key的value表示空string，然后进行补0。

## BITPOS

```
BITPOS key bit [start] [end]
```
- 返回字符串里面第一个被设置为1或者0的bit位。
- 返回一个位置，把字符串当做一个从左到右的字节数组，其中第一个byte的最有效位在位置0，第二个byte的最有效位在位置8，以此类推。
- 默认情况下整个字符串都会被检索一次，只有在指定start和end参数 (包括start和end位)，该范围被解释为一个字节的范围，而不是一系列的位。所以start=0并且end=2是指前三个字节范围内查找。
- 注意，返回的位的位置始终是从0开始的（绝对位置），即使使用了start来指定了一个开始字节也是这样。
- 和`GETRANGE`命令一样，start和end也可以包含负值，负值将从字符串的末尾开始计算，-1是字符串的最后一个字节，-2是倒数第二个，等等。
- 不存在的key将会被当做空字符串来处理。
- 返回值
    - 命令返回字符串里面第一个被设置为1或者0的bit位。
    - 如果我们在空字符串或者0字节的字符串里面查找bit为1的内容，那么结果将返回 -1。
    - 如果我们在字符串里面查找bit为0而且字符串只包含1的值时，将返回字符串最右边的第一个空位。如果有一个字符串是三个字节的值为0xff的字符串，那么命令BITPOS key 0将会返回24，因为0 ~ 23位都是1。
    - 基本上，我们可以把字符串看成右边有无数个0。
    - 然而，如果用指定start和end范围进行查找指定值时，如果该范围内没有对应值，结果将返回 -1。

## BITFIELD

```
BITFIELD key [GET type offset]
             [SET type offset value]
             [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
```
- 本命令会把Redis字符串当作位数组，并能对变长位宽和任意未字节对齐的指定整型位域进行寻址。
- 在实践中，可以使用该命令对一个有符号的5位整型数（i5）的1234位设置指定值，也可以对一个31位无符号整型数（u31）的4567位进行取值。类似地，在对指定的整数进行自增和自减操作，本命令可以提供有保证的、可配置的上溢和下溢处理操作。
- `BITFIELD` 命令能操作多字节位域，它会执行一系列操作，并返回一个响应数组，在参数列表中每个响应数组匹配相应的操作。
- 例如，下面的命令是对一个8位有符号整数偏移100位自增1，并获取4位无符号整数的值：
    ```
    > BITFIELD mykey INCRBY i5 100 1 GET u4 0
    1) (integer) 1
    2) (integer) 0
    ```
- 提示：
    - 用`GET`指令对超出当前字符串长度的位（含key不存在的情况）进行寻址，执行操作的结果会对缺失部分的位（bits）赋值为0。
    - 用`SET`或`INCRBY`指令对超出当前字符串长度的位（含key不存在的情况）进行寻址，将会扩展字符串并对扩展部分进行补0，扩展方式包括：按需扩展、按最小长度扩展和按最大寻址能力扩展。
- 支持子命令和整型

    | 子命令                               | 说明                                           |
    | ------------------------------------ | ---------------------------------------------- |
    | `GET <type> <offset>`                | 返回指定的位域                                 |
    | `SET <type> <offset> <value>`        | 设置指定位域的值并返回它的原值                 |
    | `INCRBY <type> <offset> <increment>` | 自增指定位域的值并返回它的新值                 |
    | `OVERFLOW [WRAP|SAT|FAIL]`           | 设置溢出行为来改变调用`INCRBY`指令的后序操作 |
    - type
        - 当需要一个整型时，有符号整型需在位数前加i，无符号在位数前加u。例如，u8是一个8位的无符号整型，i16是一个16位的有符号整型。
        - 有符号整型最大支持64位，而无符号整型最大支持63位。对无符号整型的限制，是由于当前Redis协议不能在响应消息中返回64位无符号整数。
    - offset
        - 如果未定带数字的前缀，将会以字符串的第0位作为起始位。
        - 不过，如果偏移量带有 # 前缀，那么指定的偏移量需要乘以整型宽度，如i8 #2表示8 * 2 = 16位。
    - 溢出控制
        - WRAP（默认）: 回环算法，适用于有符号和无符号整型两种类型。对于无符号整型，回环计数将对整型最大值进行取模操作（C语言的标准行为）。对于有符号整型，上溢从最负的负数开始取数，下溢则从最大的正数开始取数，例如，如果i8整型的值设为127，自加1后的值变为 - 128。
        - SAT: 饱和算法，下溢之后设为最小的整型值，上溢之后设为最大的整数值。例如，i8整型的值从120开始加10后，结果是127，继续增加，结果还是保持为127。下溢也是同理，但量结果值将会保持在最负的负数值。
        - FAIL: 失败算法，这种模式下，在检测到上溢或下溢时，不做任何操作。相应的返回值会设为NULL，并返回给调用者。