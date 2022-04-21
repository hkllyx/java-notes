- [概述](#概述)
- [使用场景](#使用场景)
- [相关命令](#相关命令)
  - [SADD](#sadd)
  - [SREM](#srem)
  - [SPOP](#spop)
  - [SRANDMEMBER](#srandmember)
  - [SMOVE](#smove)
  - [SDIFF & SDIFFSTORE](#sdiff--sdiffstore)
  - [SINTER & SINTERSTORE](#sinter--sinterstore)
  - [SUNION & SUNIONSTORE](#sunion--sunionstore)
  - [SCARD](#scard)
  - [SMEMBERS](#smembers)
  - [SISMEMBER](#sismember)
- [SSCAN](#sscan)

# 概述

- Redis 集合是一个无序的字符串合集。你可以以 O(1) 的时间复杂度（无论集合中有多少元素时间复杂度都为常量）完成 添加，删除以及测试元素是否存在的操作。
- Redis 集合有着不允许相同成员存在的优秀特性。向集合中多次添加同一元素，在集合中最终只会存在一个此元素。实际上这就意味着，在添加元素前，你并不需要事先进行检验此元素是否已经存在的操作。
- 一个 Redis 列表十分有趣的事是，它们支持一些服务端的命令从现有的集合出发去进行集合运算。 所以你可以在很短的时间内完成合并（union）, 求交 (intersection), 找出不同元素的操作。
- 一个集合最多可以包含 $2^{32} - 1$ 个元素（4294967295，每个集合超过 40 亿个元素）。

# 使用场景

- 用集合跟踪一个独特的事。想要知道所有访问某个博客文章的独立 IP，只要每次都用 `SADD` 来处理一个页面访问。那么你可以肯定重复的 IP 是不会插入的。
- Redis 集合能很好的表示关系。你可以创建一个 tagging 系统，然后用集合来代表单个 tag。接下来你可以用 `SADD` 命令把所有拥有 tag 的对象的所有 ID 添加进集合，这样来表示这个特定的 tag。如果你想要同时有 3 个不同 tag 的所有对象的所有 ID，那么你需要使用 `SINTER`.
- 使用 `SPOP` 或者 `SRANDMEMBER` 命令随机地获取元素。

# 相关命令

## SADD

```
SADD key member [member ...]
```
- 添加一个或多个指定的 member 元素到集合的 key 中。指定的一个或者多个元素 member 如果已经在集合 key 中存在则忽略。如果集合 key 不存在，则新建集合 key, 并添加 member 元素到集合 key 中。
- 如果 key 的类型不是集合则返回错误。

## SREM

```
SREM key member [member ...]
```
- 在 key 集合中移除指定的元素。
- 如果指定的元素不是 key 集合中的元素则忽略。
- 如果 key 集合不存在则被视为一个空的集合，该命令返回 0。
- 如果 key 的类型不是一个集合，则返回错误。

## SPOP

```
SPOP key [count]
```
- 从存储在 key 的集合中移除并返回一个或多个随机元素。
- 此操作与 `SRANDMEMBER` 类似，它从一个集合中返回一个或多个随机元素，但不删除元素。
- count 参数将在更高版本中提供，但是在 2.6、2.8、3.0 中不可用。
- 如果 count 大于集合内部的元素数量，此命令将会返回整个集合，不会有额外的元素。

## SRANDMEMBER

```
SRANDMEMBER key [count]
```
- 仅提供 key 参数，那么随机返回 key 集合中的一个元素。
- 仅提供 key 参数时，该命令作用类似于 `SPOP` 命令，不同的是 `SPOP` 命令会将被选择的随机元素从集合中移除，而 `SRANDMEMBER` 仅仅是返回该随记元素，而不做任何操作。
- Redis 2.6 开始，可以接受 count 参数：
    - 如果 count 是整数且小于元素的个数，返回含有 count 个不同的元素的数组
    - 如果 count 是个整数且大于集合中元素的个数时，仅返回整个集合的所有元素
    - 如果 count 是负数，则会返回一个包含 count 的绝对值的个数元素的数组
    - 如果 count 的绝对值大于元素的负数，则返回的结果集里会出现一个元素出现多次的情况

## SMOVE

```
SMOVE source destination member
```
- 将 member 从 source 集合移动到 destination 集合中。对于其他的客户端，在特定的时间元素将会作为 source 或者 destination 集合的成员出现。
- 如果 source 集合不存在或者不包含指定的元素，则命令不执行任何操作并且返回 0。否则对象将会从 source 集合中移除，并添加到 destination 集合中去，返回 1。
- 如果 destination 集合已经存在该元素，则命令仅将该元素充 source 集合中移除。
- 如果 source 和 destination 不是集合类型，则返回错误。

## SDIFF & SDIFFSTORE

```
SDIFF key [key ...]
```
- 返回第一个集合与给定的其他集合的差集的元素。
- 不存在的 key 认为是空集。

```
SDIFFSTORE destination key [key ...]
```
- 该命令类似于 `SDIFF`, 不同之处在于该命令不返回结果集，而是将结果存放在 destination 集合中。
- 如果 destination 已经存在，则将其覆盖。

## SINTER & SINTERSTORE

```
SINTER key [key ...]
```
- 返回指定所有的集合的成员的交集。
- 如果 key 不存在则被认为是一个空的集合，当给定的集合为空的时候，结果也为空(一个集合为空，结果一直为空)。

```
SINTERSTORE destination key [key ...]
```
- 这个命令与 `SINTER` 命令类似，但是它并不是直接返回结果集，而是将结果保存在 destination 集合中。
- 如果 destination 集合存在，则会被覆盖。

## SUNION & SUNIONSTORE

```
SUNION key [key ...]
```
- 返回给定的多个集合的并集中的所有成员。

```
SUNIONSTORE destination key [key ...]
```
- 该命令作用类似于 `SUNION` 命令，不同的是它并不返回结果集，而是将结果存储在 destination 集合中。
- 如果 destination 已经存在，则将其覆盖。

## SCARD

```
SCARD key
```
- 返回集合存储的 key 的基数 (集合元素的数量)。
- 如果 key 不存在，则返回 0。

## SMEMBERS

```
SMEMBERS key
```
- 返回 key 集合所有的元素。
- 该命令的作用与使用一个参数的 `SINTER` 命令作用相同。

## SISMEMBER

```
SISMEMBER key member
```
- 返回成员 member 是否是存储的集合 key 的成员。
- 如果 member 元素是集合 key 的成员，则返回 1，如果 member 元素不是 key 的成员，或者集合 key 不存在，则返回 0。

## SSCAN

```
SSCAN cursor [MATCH pattern] [COUNT count]
```
- 参见 Redis Key 中 `SCAN` 命令。