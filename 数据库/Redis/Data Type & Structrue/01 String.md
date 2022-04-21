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
- [Bitmap 相关命令](#bitmap-相关命令)
    - [BITCOUNT](#bitcount)
    - [GETBIT](#getbit)
    - [SETBIT](#setbit)
    - [BITOP](#bitop)
    - [BITPOS](#bitpos)
    - [BITFIELD](#bitfield)

# 概述

- 字符串是一种最基本的 Redis 值类型。
- Redis 字符串是二进制安全的，这意味着一个 Redis 字符串能包含任意类型的数据，例如：一张 JPEG 格式的图片或者一个序列化的 Ruby 对象。
- 一个字符串类型的值最多能存储 $512 M$ 字节的内容。

# 使用场景

- 利用 `INCR` 命令簇（`INCR`, `DECR`, `INCRBY`）来把字符串当作原子计数器使用。
- 使用 `APPEND` 命令在字符串后添加内容。
- 将字符串作为 `GETRANGE` 和 `SETRANGE` 的随机访问向量。
- 在小空间里编码大量数据，或者使用 `GETBIT` 和 `SETBIT` 创建一个 Redis 支持的 Bloom 过滤器。

# 基础命令

## SET, SETNX, SETEX, PSETEX, MSET, MSETNX

```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```
- 将 key 设定为指定的 “字符串” 值。
- value 可以是每种类型的 string（包括二进制数据），例如，您可以在值内存储 jpeg 图像。value 不能大于 512 MB。
- 如果 key 已经保存了一个值，那么这个操作会直接覆盖原来的值，并且忽略原始类型。
- 当 `SET` 命令执行成功之后，之前设置的过期时间都将失效。
- 选项：

    | 选项            | 说明                                   | 相关命令 |
    | --------------- | -------------------------------------- | -------- |
    | EX seconds      | 设置 key 的过期时间，单位时秒          | `SETEX`  |
    | PX milliseconds | 设置 key 的过期时间，单位时毫秒        | `PSETEX` |
    | NX              | 只有 key 不存在的时候才会设置 key 的值 | `SETNX`  |
    | XX              | 只有 key 存在的时候才会设置 key 的值   |          |

```
SETNX key value
SETEX key seconds value
PSETEX key milliseconds value
```
- 注意：注意: 由于 `SET` 命令加上选项已经可以完全取代 `SETNX`, `SETEX`, `PSETEX` 的功能，所以在将来的版本中，redis 可能会不推荐使用并且最终抛弃这几个命令。

```
MSET key value [key value ...]
MSETNX key value [key value ...]
```
- `MSET` 给定的 keys 对应到相应的 values，就像普通的 SET 命令一样，`MSET` 会用新的 value 替换已经存在的 value。
- 如果你不想覆盖已经存在的 values，可以使用 `MSETNX`。只要有一个 key 已经存在，`MSETNX` 一个操作都不会执行。
- `MSET` 和 `MSETNX` 是原子的，所以所有给定的 keys 是一次性 set 的。客户端不可能看到这种一部分 keys 被更新而另外的没有改变的情况。

## SETRANGE

```
SETRANGE key offset value
```
- 覆盖 key 对应的 string 的一部分，从指定的 offset 处开始，覆盖 value 的长度。
- 如果 offset 比当前 key 对应 string 还要长，那这个 string 后面就补 `\x00` 以达到 offset。不存在的 keys 被认为是空字符串，所以这个命令可以确保 key 有一个足够大的字符串，能在 offset 处设置 value。
- 注意，offset 最大可以是 $2^{29} - 1$ (536870911), 因为 redis 字符串限制在 512M 大小。如果你需要超过这个大小，你可以用多个 keys。
- 警告 ：当设置最后一个字节并且 key 还没有一个字符串 value 或者其 value 是个比较小的字符串时，Redis 需要立即分配所有内存，这有可能会导致服务阻塞一会。注意，一旦第一次内存分配完，后面对同一个 key 调用 SETRANGE 就不会预先得到内存分配。

## APPEND

```
APPEND key value
```
- 如果 key 已经存在，并且 value 为字符串，那么这个命令会把 value 追加到原 value 的结尾。
- 如果 key 不存在，那么它将首先创建一个空字符串的 key，再执行追加操作，这种情况 `APPEND` 将类似于 `SET` 操作。

## GET, MGET

```
GET key
MGET key [key ...]
```
- `GET` 返回 key 的 value。如果 key 不存在，返回特殊值 nil。如果 key 的 value 不是 string，就返回错误，因为 GET 只处理 string 类型的 values。
- `MGET` 返回所有指定的 key 的 value。对于每个不对应 string 或者不存在的 key，都返回特殊值 nil。正因为此，这个操作从来不会失败。

## GETRANGE

```
GETRANGE key start end
```
- 这个命令是被改成 `GETRANGE` 的，在小于 2.0 的 Redis 版本中叫 `SUBSTR`。
- 返回 key 对应的字符串 value 的子串，这个子串是由 start 和 end 位移（两个位移处的字符都包括在结果中）决定的。
- 可以用负的位移来表示从 string 尾部开始数的下标。所以 -1 就是最后一个字符，-2 就是倒数第二个，以此类推。
- 这个函数处理超出范围的请求时，都把结果限制在 string 内。

## GETSET

```
GETSET key value
```
- 原子性地将 key 对应到 value 并且返回原来 key 对应的 value。
- 如果 key 存在但是对应的 value 不是字符串，就返回错误。

## STRLEN

```
STRLEN key
```
- 返回 key 的 string 类型 value 的长度。如果 key 对应的非 string 类型，就返回错误。

# 数值相关命令

## INCR, DECR, INCRBY, DECRBY

```
INCR key
DECR key
```
- 对 key 对应的数字做加或减 1 操作。
- 如果 key 不存在，那么在操作之前，这个 key 对应的值会被置为 0。
- 如果 key 有一个错误类型的 value 或者是一个不能表示成数字的字符串，就返回错误。这个操作最大支持在 64 位有符号的整型数字。

查看命令 INCR 了解关于增减操作的额外信息。

```
INCRBY key increment
DECRBY key decrement
```
- 类似 `INCR` 和 `DECR`，不同的是将 key 对应的数字加 increment 或减 decrement。

## INCRBYFLOAT

```
INCRBYFLOAT key increment
```
- 通过指定浮点数 key 来增长浮点数 (存放于 string 中) 的值. 当键不存在时，先将其值设为 0 再操作。下面任一情况都会返回错误:
    - key 包含非法值 (不是一个 string).
    - 当前的 key 或者相加后的值不能解析为一个双精度的浮点值.(超出精度范围了)
- 如果操作命令成功，相加后的值将替换原值存储在对应的键值上，并以 string 的类型返回。
- string 中已存的值或者相加参数可以任意选用指数符号，但相加计算的结果会以科学计数法的格式存储。
- 无论各计算的内部精度如何，输出精度都固定为小数点后 17 位.

# Bitmap 相关命令

## BITCOUNT

```
BITCOUNT key [start end]
```
- 统计字符串被设置为 1 的 bit 数。
- 一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。
- start 和 end 参数的设置和 `GETRANGE` 命令类似，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，以此类推。
- 不存在的 key 被当成是空字符串来处理，因此对一个不存在的 key 进行 `BITCOUNT` 操作，结果为 0 。

## GETBIT

```
GETBIT key offset
```
- 返回 key 对应的 string 在 offset 处的 bit 值。
- 当 offset 超出了字符串长度的时候，这个字符串就被假定为由 0 比特填充的连续空间。
- 当 key 不存在的时候，它就认为是一个空字符串，所以 offset 总是超出范围，然后 value 也被认为是由 0 比特填充的连续空间。

## SETBIT

```
SETBIT key offset value
```
- 设置或者清空 key 的 value (字符串) 在 offset 处的 bit 值。
- 那个位置的 bit 要么被设置，要么被清空，这个由 value（只能是 0 或 1）来决定。
- 当 key 不存在的时候，就创建一个新的字符串 value。要确保这个字符串大到在 offset 处有 bit 值。参数 offset 需要大于等于 0，并且小于 $2^{32}$ (限制 bitmap 大小为 512M)。当 key 对应的字符串增大的时候，新增的部分 bit 值都是设置为 0。
- 警告：当设置最后一个 bit (offset 等于 $2^{32} - 1$) 并且 key 还没有一个字符串 value 或者其 value 是个比较小的字符串时，Redis 需要立即分配所有内存，这有可能会导致服务阻塞一会。注意，一旦第一次内存分配完，后面对同一个 key 调用 `SETBIT` 就不会预先得到内存分配。

## BITOP

```
BITOP operation destkey key [key ...]
```
- 对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
- operation 参数：

    | 参数 | 说明                        |
    | ---- | --------------------------- |
    | AND  | 与                          |
    | OR   | 或                          |
    | XOR  | 异或                        |
    | NOT  | 非，只能对一个 key 进行操作 |
- 处理不同长度的字符串
    - 集合中所有比最长 string 短的 string 在尾部补 0 至最长 string 的长度。
    - 不存在的 key 的 value 表示空 string，然后进行补 0。

## BITPOS

```
BITPOS key bit [start] [end]
```
- 返回字符串里面第一个被设置为 1 或者 0 的 bit 位。
- 返回一个位置，把字符串当做一个从左到右的字节数组，其中第一个 byte 的最有效位在位置 0，第二个 byte 的最有效位在位置 8，以此类推。
- 默认情况下整个字符串都会被检索一次，只有在指定 start 和 end 参数 (包括 start 和 end 位)，该范围被解释为一个字节的范围，而不是一系列的位。所以 start=0 并且 end=2 是指前三个字节范围内查找。
- 注意，返回的位的位置始终是从 0 开始的（绝对位置），即使使用了 start 来指定了一个开始字节也是这样。
- 和 `GETRANGE` 命令一样，start 和 end 也可以包含负值，负值将从字符串的末尾开始计算，-1 是字符串的最后一个字节，-2 是倒数第二个，等等。
- 不存在的 key 将会被当做空字符串来处理。
- 返回值
    - 命令返回字符串里面第一个被设置为 1 或者 0 的 bit 位。
    - 如果我们在空字符串或者 0 字节的字符串里面查找 bit 为 1 的内容，那么结果将返回 -1。
    - 如果我们在字符串里面查找 bit 为 0 而且字符串只包含 1 的值时，将返回字符串最右边的第一个空位。如果有一个字符串是三个字节的值为 0xff 的字符串，那么命令 BITPOS key 0 将会返回 24，因为 0 ~ 23 位都是 1。
    - 基本上，我们可以把字符串看成右边有无数个 0。
    - 然而，如果用指定 start 和 end 范围进行查找指定值时，如果该范围内没有对应值，结果将返回 -1。

## BITFIELD

```
BITFIELD key [GET type offset]
             [SET type offset value]
             [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
```
- 本命令会把 Redis 字符串当作位数组，并能对变长位宽和任意未字节对齐的指定整型位域进行寻址。
- 在实践中，可以使用该命令对一个有符号的 5 位整型数（i5）的 1234 位设置指定值，也可以对一个 31 位无符号整型数（u31）的 4567 位进行取值。类似地，在对指定的整数进行自增和自减操作，本命令可以提供有保证的、可配置的上溢和下溢处理操作。
- `BITFIELD` 命令能操作多字节位域，它会执行一系列操作，并返回一个响应数组，在参数列表中每个响应数组匹配相应的操作。
- 例如，下面的命令是对一个 8 位有符号整数偏移 100 位自增 1，并获取 4 位无符号整数的值：
    ```
    > BITFIELD mykey INCRBY i5 100 1 GET u4 0
    1) (integer) 1
    2) (integer) 0
    ```
- 提示：
    - 用 `GET` 指令对超出当前字符串长度的位（含 key 不存在的情况）进行寻址，执行操作的结果会对缺失部分的位（bits）赋值为 0。
    - 用 `SET` 或 `INCRBY` 指令对超出当前字符串长度的位（含 key 不存在的情况）进行寻址，将会扩展字符串并对扩展部分进行补 0，扩展方式包括：按需扩展、按最小长度扩展和按最大寻址能力扩展。
- 支持子命令和整型

    | 子命令                               | 说明                                           |
    | ------------------------------------ | ---------------------------------------------- |
    | `GET <type> <offset>`                | 返回指定的位域                                 |
    | `SET <type> <offset> <value>`        | 设置指定位域的值并返回它的原值                 |
    | `INCRBY <type> <offset> <increment>` | 自增指定位域的值并返回它的新值                 |
    | `OVERFLOW [WRAP|SAT|FAIL]`           | 设置溢出行为来改变调用 `INCRBY` 指令的后序操作 |
    - type
        - 当需要一个整型时，有符号整型需在位数前加 i，无符号在位数前加 u。例如，u8 是一个 8 位的无符号整型，i16 是一个 16 位的有符号整型。
        - 有符号整型最大支持 64 位，而无符号整型最大支持 63 位。对无符号整型的限制，是由于当前 Redis 协议不能在响应消息中返回 64 位无符号整数。
    - offset
        - 如果未定带数字的前缀，将会以字符串的第 0 位作为起始位。
        - 不过，如果偏移量带有 # 前缀，那么指定的偏移量需要乘以整型宽度，如 i8 #2 表示 8 * 2 = 16 位。
    - 溢出控制
        - WRAP（默认）: 回环算法，适用于有符号和无符号整型两种类型。对于无符号整型，回环计数将对整型最大值进行取模操作（C 语言的标准行为）。对于有符号整型，上溢从最负的负数开始取数，下溢则从最大的正数开始取数，例如，如果 i8 整型的值设为 127，自加 1 后的值变为 - 128。
        - SAT: 饱和算法，下溢之后设为最小的整型值，上溢之后设为最大的整数值。例如，i8 整型的值从 120 开始加 10 后，结果是 127，继续增加，结果还是保持为 127。下溢也是同理，但量结果值将会保持在最负的负数值。
        - FAIL: 失败算法，这种模式下，在检测到上溢或下溢时，不做任何操作。相应的返回值会设为 NULL，并返回给调用者。