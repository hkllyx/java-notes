- [概述](#概述)
- [使用场景](#使用场景)
- [相关命令](#相关命令)
  - [HSET, HMSET, HSETNX](#hset-hmset-hsetnx)
  - [HINCRBY, HINCRBYFLOAT](#hincrby-hincrbyfloat)
  - [HGET, HMGET, HGETALL, HVALS](#hget-hmget-hgetall-hvals)
  - [HDEL](#hdel)
  - [HLEN](#hlen)
  - [HSTRLEN](#hstrlen)
  - [HKEYS](#hkeys)
  - [HEXISTS](#hexists)
  - [HSCAN](#hscan)

# 概述

- Redis Hashes 是字符串字段和字符串值之间的映射。
- 一个拥有少量（100 个左右）字段的 hash 需要 很少的空间来存储，所有你可以在一个小型的 Redis 实例中存储上百万的对象。
- 尽管 Hashes 主要用来表示对象，但它们也能够存储许多元素，所以你也可以用 Hashes 来完成许多其他的任务。
- 一个 hash 最多可以包含 $2^{32} - 1$ 个 key-value 键值对（超过 40 亿）。

# 使用场景

- Redis Hashes 是完美的表示对象（eg: 一个有名，姓，年龄等属性的用户）的数据类型。
    ```
    @cli
    HMSET user:1000 username antirez password P1pp0 age 34
    HGETALL user:1000
    HSET user:1000 password 12345
    HGETALL user:1000
    ```

# 相关命令

## HSET, HMSET, HSETNX

```
HSET key field value
```
- 设置 key 指定的哈希集中指定字段的值。
- 如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联。
- 如果字段在哈希集中存在，它将被重写。

```
HMSET key field value [field value ...]
```
- 批量设置 key 指定的哈希集中指定字段的值。
- 该命令将重写所有在哈希集中存在的字段。

```
HSETNX key field value
```
- 只在 key 指定的哈希集中不存在指定的字段时，设置字段的值。
- 如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联。
- 如果字段已存在，该操作无效果。

## HINCRBY, HINCRBYFLOAT

```
HINCRBY key field increment
```
- 增加 key 指定的哈希集中指定字段的数值。
- 如果 key 不存在，会创建一个新的哈希集并与 key 关联。
- 如果字段不存在，则字段的值在该操作执行前被设置为 0。
- 支持的值的范围限定在 64 位 有符号整数。

```
HINCRBYFLOAT key field increment
```
- 为指定 key 的 hash 的 field 字段值执行 float 类型的 increment 加。
- 如果 field 不存在，则在执行该操作前设置为 0. 如果出现下列情况之一，则返回错误：
    - field 的值包含的类型错误 (不是字符串)。
    - 当前 field 或者 increment 不能解析为一个 float 类型。

## HGET, HMGET, HGETALL, HVALS

```
HGET key field
```
- 返回 key 指定的哈希集中该字段所关联的值。

```
HMGET key field [field ...]
```
- 返回 key 指定的哈希集中指定字段的值。
- 对于哈希集中不存在的每个字段，返回 nil 值。因为不存在的 keys 被认为是一个空的哈希集，对一个不存在的 key 执行 `HMGET` 将返回一个只含有 nil 值的列表

```
HGETALL key
```
- 返回 key 指定的哈希集中所有的字段和值。
- 返回值中，每个字段名的下一个是它的值，所以返回值的长度是哈希集大小的两倍。

```
HVALS key
```
- 返回 key 指定的哈希集中所有字段的值。

## HDEL

```
HDEL key field [field ...]
```
- 从 key 指定的哈希集中移除指定的域。在哈希集中不存在的域将被忽略。
- 如果 key 指定的哈希集不存在，它将被认为是一个空的哈希集，该命令将返回 0。

## HLEN

```
HLEN key
```
- 返回 key 指定的哈希集包含的字段的数量。

## HSTRLEN

```
HSTRLEN key field
```
- 返回 hash 指定 field 的 value 的字符串长度，如果 hash 或者 field 不存在，返回 0。

## HKEYS

```
HKEYS key
```
- 返回 key 指定的哈希集中所有字段的名字。

## HEXISTS

```
HEXISTS key field
```
- 返回 hash 里面 field 是否存在。

## HSCAN

```
ZSCAN key cursor [MATCH pattern] [COUNT count]
```
- 参见 Redis Key 中 `SCAN` 命令。