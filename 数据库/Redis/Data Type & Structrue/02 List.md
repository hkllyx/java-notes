- [概述](#概述)
- [使用场景](#使用场景)
- [常用命令](#常用命令)
  - [LPUSH, LPUSHX, RPUSH, RPUSHX](#lpush-lpushx-rpush-rpushx)
  - [LINSERT](#linsert)
  - [LSET](#lset)
  - [LPOP, RPOP](#lpop-rpop)
  - [RPOPLPUSH](#rpoplpush)
  - [BLPOP, BRPOP, BRPOPLPUSH](#blpop-brpop-brpoplpush)
  - [LREM](#lrem)
  - [LTRIM](#ltrim)
  - [LLEN](#llen)
  - [LINDEX](#lindex)
  - [LRANGE](#lrange)

# 概述

- Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。
- `LPUSH` 命令插入一个新元素到列表头部，而 `RPUSH` 命令插入一个新元素到列表的尾部。当对一个空key执行其中某个命令时，将会创建一个新表。
- 类似的，如果一个操作要清空列表，那么key会从对应的key空间删除。这是个非常便利的语义， 因为如果使用一个不存在的key作为参数，所有的列表命令都会像在对一个空表操作一样。
- 一个列表最多可以包含$2^{32} - 1$个元素（4294967295，每个表超过40亿个元素）。
- 从时间复杂度的角度来看，Redis列表主要的特性就是支持时间常数的 插入和靠近头尾部元素的删除，即使是需要插入上百万的条目。访问列表两端的元素是非常快的，但如果你试着访问一个非常大的列表的中间元素仍然是十分慢的，因为那是一个时间复杂度为O(N) 的操作。

# 使用场景

- 在社交网络中建立一个时间线模型，使用 `LPUSH` 去添加新的元素到用户时间线中，使用 `LRANGE` 去检索一些最近插入的条目。
- 你可以同时使用 `LPUSH` 和 `LTRIM` 去创建一个永远不会超过指定元素数目的列表并同时记住最后的N个元素。
- 列表可以用来当作消息传递的基元（primitive），例如，众所周知的用来创建后台任务的Resque Ruby库。
- 你可以使用列表做更多事，这个数据类型支持许多命令，包括像 `BLPOP` 这样的阻塞命令。

# 常用命令

## LPUSH, LPUSHX, RPUSH, RPUSHX

```
LPUSH key value [value ...]
```
- 将所有指定的值插入到存于key的列表的头部。如果key不存在，那么在进行操作前会创建一个空列表。如果key对应的值不是一个list的话，那么会返回一个错误。
- 可以使用一个命令把多个元素push进入列表，只需在命令末尾加上多个指定的参数。元素是从最左端的到最右端的、一个接一个被插入到list的头部。 所以对于这个命令例子 `LPUSH mylist a b c`，返回的列表是 "c b a"。

```
LPUSHX key value
```
- 只有当key已经存在并且存着一个list的时候，在这个key下面的list的头部插入value。
- 与 `LPUSH` 相反，当key不存在的时候不会进行任何操作。

```
RPUSH key value [value ...]
RPUSHX key value
```
- 类似 `LPUSH` 和 `LPUSHX`，不同的是从尾部插入

## LINSERT

```
LINSERT key BEFORE|AFTER pivot value
```
- 把value插入存于key的列表中在基准值pivot的前面或后面（privot是LIST中的元素）。
- 当key不存在时，这个list会被看作是空list，任何操作都不会发生。
- 当key存在，但保存的不是一个list的时候，会返回error。

## LSET

```
LSET key index value
```
- 设置index位置的list元素的值为value。 更多关于index参数的信息，详见LINDEX。
- 当index超出范围时会返回一个error。

## LPOP, RPOP

```
LPOP key
RPOP key
```
- 移除并且返回key对应的list的第一个 / 最后一个元素。
- 当key不存在时返回nil。

## RPOPLPUSH

```
RPOPLPUSH source destination
```
- 原子性地返回并移除存储在source的列表的最后一个元素（列表尾部元素）， 并把该元素放入存储在destination的列表的第一个元素位置（列表头部）。
- 如果source不存在，那么会返回nil值，并且不会执行任何操作。如果source和destination是同样的，那么这个操作等同于移除列表最后一个元素并且把该元素放在列表头部，所以这个命令也可以当作是一个旋转列表的命令。

## BLPOP, BRPOP, BRPOPLPUSH

```
BLPOP key [key ...] timeout
RPOP key [key ...] timeout
RPOPLPUSH source destination timeout
```
- `BLPOP` 和 `BRPOP` 是 `LPOP` 和 `RPOP` 的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被命令阻塞。当给定多个key参数时，按参数key的先后顺序依次检查各个列表，弹出第一个非空列表的第一个 / 最后一个元素。
- `BRPOPLPUSH` 是 `RPOPLPUSH` 的阻塞版本。 当source包含元素的时候，这个命令表现得跟 `RPOPLPUSH` 一模一样。 当source是空的时候，Redis将会阻塞这个连接，直到另一个客户端push元素进入或者达到timeout时限。
- timeout为0能用于无限期阻塞客户端。
- 更多关于阻塞资料参见 [blpop命令](http://redis.cn/commands/blpop.html)

## LREM

```
LREM key count value
```
- 从存于key的列表里移除前count次出现的值为value的元素。 这个count参数通过下面几种方式影响这个操作：
    - count > 0: 从头往尾移除值为value的元素。
    - count < 0: 从尾往头移除值为value的元素。
    - count = 0: 移除所有值为value的元素。
- 比如，LREM list -2“hello” 会从存于list的列表里移除最后两个出现的 “hello”。
- 需要注意的是，不存在的key会被当作空list处理，这个命令会返回0。

## LTRIM

```
LTRIM key start stop
```
- 修剪 (trim) 一个已存在的list，这样list就会只包含指定范围的指定元素。start和stop都是由0开始计数的， 这里的0是列表里的第一个元素（表头），1是第二个元素，以此类推。
- 超过范围的下标并不会产生错误：如果start超过列表尾部，或者start > end，结果会是列表变成空表（即该key会被移除）。 如果end超过列表尾部，Redis会将其当作列表的最后一个元素。
- `LTRIM` 的一个常见用法是和 `LPUSH` / `RPUSH` 一起使用。 例如：
    ```
    LPUSH mylist someelement
    LTRIM mylist 0 99
    ```
    这一对命令会将一个新的元素push进列表里，并保证该列表不会增长到超过100个元素。这个是很有用的，比如当用Redis来存储日志。 需要特别注意的是，当用这种方式来使用LTRIM的时候，操作的复杂度是O(1) ， 因为平均情况下，每次只有一个元素会被移除。

## LLEN

```
LLEN key
```
- 返回存储在key里的list的长度。
- 如果key不存在，那么就被看作是空list，并且返回长度为0。
- 当存储在key里的值不是一个list的话，会返回error。

## LINDEX

```
LINDEX key index
```
- 返回存储在key的列表里index处的元素。
- index是从0开始的，所以0是表示第一个元素，1表示第二个元素，并以此类推。
- 负数索引用于指定从列表尾部开始索引的元素。在这种方法下，-1表示最后一个元素，-2表示倒数第二个元素，并以此往前推。
- 当key位置的值不是一个列表的时候，会返回一个error。

## LRANGE

```
LRANGE key start stop
```
- 返回存储在key的列表里指定范围内的元素。
- start和end偏移量都是基于0的下标，即list的第一个元素下标是0（list的表头），第二个元素下标是1，以此类推。
- 偏移量也可以是负数，表示偏移量是从list尾部开始计数。 例如， -1表示列表的最后一个元素，-2是倒数第二个，以此类推。
- 当下标超过list范围的时候不会产生error。 如果start比list的尾部下标大的时候，会返回一个空列表。 如果stop比list的实际尾部大的时候，Redis会当它是最后一个元素的下标。