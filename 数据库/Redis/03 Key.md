- [概述](#概述)
- [规则](#规则)
- [相关命令](#相关命令)
  - [SCAN](#scan)
  - [KEYS](#keys)
  - [RENAME, RENAMENX](#rename-renamenx)
  - [TYPE](#type)
  - [DEL](#del)
  - [UNLINK](#unlink)
  - [EXISTS](#exists)
  - [EXPIRE, PEXPIRE, EXPIREAT, PEXPIREAT](#expire-pexpire-expireat-pexpireat)
  - [TTL, PTTL](#ttl-pttl)
  - [PERSIST](#persist)
  - [DUMP](#dump)
  - [RESTORE](#restore)
  - [RANDOMKEY](#randomkey)
  - [MOVE](#move)
  - [MIGRATE](#migrate)
  - [TOUCH](#touch)
  - [SORT](#sort)
  - [OBJECT](#object)
  - [WAIT](#wait)

# 概述

Redis的key是二进制安全的字符串（详见String数据类型），这意味着您可以使用任何二进制序列作为key，从字符串 "foo" 到JPEG文件的内容。空字符串也是一个有效的key。

# 规则

- 很长的key不是一个好主意。例如，一个1024 bytes的key不仅在内存方面是一个糟糕的主意，而且因为在数据集中查找时key的比较代价也是昂贵的。即使手头的任务是匹配一个大值的存在性，hash它（例如使用SHA1）也是一个更好的主意，特别是从内存和带宽的角度来看。
- 很短的key通常不是一个好主意。如果你可以使用 "uesr:1000:flowers”，那么写成 "u1000flw" 作为key就没有意义了。前者更具可读性，并且与key对象本身和value对象使用的空间相比，添加的空间更小。虽然短key显然会消耗更少的内存，但您的工作是找到正确的平衡。
- 试着坚持一个模式。例如，"object-type:id" 是一个好主意，比如 "user:1000"。点或破折号通常用于多字字段，如 "comment:1234:reply.to" 或 "commet:1234:reply-to"。
- 允许的最大的key大小为512M。

# 相关命令

## SCAN

```
SCAN cursor [MATCH pattern] [COUNT count]
```
- 扫描当前数据库中的key集
- 使用`SCAN`命令和密切相关的命令`SSCAN`、`HSCAN`和 `ZSCAN`，以便对元素集合进行增量迭代
    - `SSCAN` 用于扫描value为SET的key集
    - `HSCAN` 用于扫描value为HASH的key集
    - `ZSCAN` 用于扫描value为ZSET的key集
- `SCAN` 命令是一个基于游标的迭代器。这意味着命令每次被调用都需要使用上一次这个调用返回的游标作为该次调用的游标参数，以此来延续之前的迭代过程
- 当`SCAN`命令的游标参数被设置为0时，服务器将开始一次新的迭代，而当服务器向用户返回值为0的游标时，表示迭代已结束。
- `SCAN` 命令的返回值 是一个包含两个元素的数组，第一个数组元素是用于进行下一次迭代的新游标， 而第二个数组元素则是一个数组，这个数组中包含了所有被迭代的元素。
- 对于增量式迭代命令不保证每次迭代所返回的元素数量，我们可以使用COUNT选项， 对命令的行为进行一定程度上的调整。默认值为10。

## KEYS

```
KEYS pattern
```
- 查找所有符合给定模式pattern（正则表达式）的key。
- 时间复杂度为O (N)，N为数据库里面key的数量。
- `KEYS` 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的Key集， 你最好还是用`SETS`来代替。
- pattern支持的格式
    - `?` 匹配任意字符一次
    - `*` 匹配任意字符任意次
    - `[abc]` 匹配 'a', 'b', 'c' 中任一个一次
    - `[^a]` 匹配 'a' 之外的任意字符一次
    - `[a-c]` 匹配 'a' 到 'c' 中任一个一次

## RENAME, RENAMENX

```
RENAME key newkey
RENAMENX key newkey
```
- 将key重命名为newkey，如果key与newkey相同，将返回一个错误。
- 如果newkey已经存在，对于`RENAME`newkey的value将被覆盖；对于 `RENAMENX`则会忽略。

## TYPE

```
TYPE key
```
- 返回key所存储的value的数据结构类型。
- 它可以返回string, list, set, zset和hash等不同的类型。
- 如果key不存在时返回none。

## DEL

```
DEL key [key ...]
```
- 删除指定的一批keys，如果删除中的某些key不存在，则直接忽略。

## UNLINK

```
UNLINK key [key ...]
```
- 该命令和`DEL`十分相似：删除指定的key(s), 若key不存在则该key被跳过。
- 但是，相比`DEL`会产生阻塞，该命令会在另一个线程中回收内存，因此它是非阻塞的。
- 这也是该命令名字的由来：仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。

## EXISTS

```
EXISTS key [key ...]
```
- 返回key是否存在。
- 返回存在的key的个数。

## EXPIRE, PEXPIRE, EXPIREAT, PEXPIREAT

```
EXPIRE key seconds
PEXPIRE key milliseconds
```
- 设置key的过期时间，超过时间后，将会自动删除该key。在Redis的术语中一个key的相关超时是不确定的。
- 超时后只有对key执行`DEL`命令或者`SET`命令或者`GETSET`时才会清除。 这意味着，从概念上讲所有改变key的值的操作都会使他清除。
- 如果key被`RENAME`命令修改，相关的超时时间会转移到新key上面。如果新key存在则覆盖。
- `PEXPIRE` 类似，只不过他的时间单位是毫秒。

```
EXPIREAT key timestamp
PEXPIREAT key milliseconds-timestamp
```
- `EXPIREAT` 的作用和`EXPIRE`类似，都用于为key设置生存时间。不同在于`EXPIREAT`命令接受的时间参数是UNIX时间戳Unix timestamp。
- `PEXPIREAT` 这个命令和`EXPIREAT`命令类似，但它以毫秒为单位设置key的过期UNIX时间戳，而不是像`EXPIREAT`那样，以秒为单位。

## TTL, PTTL

```
TTL key
```
- 返回key剩余的过期时间。 这种反射能力允许Redis客户端检查指定key在数据集里面剩余的有效期。
- 在Redis 2.6和之前版本，如果key不存在或者已过期时返回 -1。
- 从Redis2.8开始，错误返回值的结果有如下改变：
    - 如果key不存在或者已过期，返回 -2
    - 如果key存在并且没有设置过期时间（永久有效），返回 -1。

```
PTTL key
```
- `PTTL` 命令返回相同的信息，只不过他的时间单位是毫秒。

## PERSIST

```
PERSIST key
```
- 移除给定key的生存时间，将这个key从『易失的』(带生存时间key) 转换成『持久的』(一个不带生存时间、永不过期的key )。

## DUMP

```
DUMP key
```
- 序列化给定key，并返回被序列化的值，使用RESTORE命令可以将这个值反序列化为Redis键。
- 序列化生成的值有以下几个特点：
    - 它带有64位的校验和，用于检测错误，RESTORE在进行反序列化之前会先检查校验和。
    - 值的编码格式和RDB文件保持一致。
    - RDB版本会被编码在序列化值当中，如果因为Redis的版本不同造成RDB格式不兼容，那么Redis会拒绝对这个值进行反序列化操作。
- 序列化的值不包括任何生存时间信息。
- 如果key不存在，那么返回nil。

## RESTORE

```
RESTORE key ttl serialized-value [REPLACE]
```
- 反序列化给定的序列化值，并将它和给定的key关联。
- 参数ttl以毫秒为单位为key设置生存时间；如果ttl为0，那么不设置生存时间。
- `RESTORE` 在执行反序列化之前会先对序列化值的RDB版本和数据校验和进行检查，如果RDB版本不相同或者数据不完整的话，那么`RESTORE`会拒绝进行反序列化，并返回一个错误。

## RANDOMKEY

```
RANDOMKEY
```
- 从当前数据库返回一个随机的key。

## MOVE

```
MOVE key db
```
- 将当前数据库的key移动到给定的数据库db当中。
- 如果当前数据库 (源数据库) 和给定数据库 (目标数据库) 有相同名字的给定key，或者key不存在于当前数据库，那么MOVE没有任何效果。
- 因此，也可以利用这一特性，将`MOVE`当作锁 (locking) 原语 (primitive)。

## MIGRATE

```
MIGRATE host port key destination-db timeout [COPY] [REPLACE]
```
- 将key原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功，key保证会出现在目标实例上，而当前实例上的key会被删除。
- 这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：迁移成功，迁移失败，等到超时。
- 命令的内部实现是这样的：它在当前实例对给定key执行`DUMP`命令，将它序列化，然后传送到目标实例，目标实例再使用`RESTORE`对数据进行反序列化，并将反序列化所得的数据添加到数据库中；当前实例就像目标实例的客户端那样，只要看到`RESTORE`命令返回OK，它就会调用`DEL`删除自己数据库上的key。
- timeout参数以毫秒为格式，指定当前实例和目标实例进行沟通的最大间隔时间。这说明操作并不一定要在timeout毫秒内完成，只是说数据传送的时间不能超过这个timeout数。
- `MIGRATE` 命令需要在给定的时间规定内完成IO操作。如果在传送数据时发生IO错误，或者达到了超时时间，那么命令会停止执行，并返回一个特殊的错误：IOERR。
- 当IOERR出现时，有以下两种可能：
    - key可能存在于两个实例。
    - key可能只存在于当前实例。
- 唯一不可能发生的情况就是丢失key，因此，如果一个客户端执行 `MIGRATE`, 命令，并且不幸遇上IOERR错误，那么这个客户端唯一要做的就是检查自己数据库上的key是否已经被正确地删除。
- 如果有其他错误发生，那么`MIGRATE`保证key只会出现在当前实例中。（当然，目标实例的给定数据库上可能有和key同名的键，不过这和`MIGRATE`命令没有关系）。

## TOUCH

```
TOUCH key [key ...]
```
- 修改指定key(s) 最后访问时间，若key不存在，不做操作。

## SORT

```
SORT key [BY pattern] [LIMIT offset count] [GET pattern] [ASC|DESC] [ALPHA] destination
```
- 返回或存储key的list、set或sorted set中的元素。
- 默认是按照数值类型排序的，并且按照两个元素的双精度浮点数类型值进行比较。
- 包含的是字符串值并且需要按照字典顺序排序，可以使用ALPHA修饰符。假设正确地设置了环境变量LC_COLLATE，Redis可以感知UTF-8编码。
- 可以使用ASC和DESC控制排序方向，默认为ASC。
- 返回元素的数量可以通过LIMIT修饰符限制。此修饰符有一个offset参数，指定了跳过的元素数量；还带有一个count参数，指定了从offset开始返回的元素数量。
- 详见 [sort命令](http://redis.cn/commands/sort.html)

## OBJECT

```
OBJECT <subcommand> [arg [arg ...]]
```
- `OBJECT` 命令可以在内部调试 (debugging) 给出keys的内部对象，它用于检查或者了解你的keys是否用到了特殊编码的数据类型来存储空间z。
- 当redis作为缓存使用的时候，你的应用也可能用到这些由`OBJECT`命令提供的信息来决定应用层的key的驱逐策略 (eviction policies)
- OBJECT支持多个子命令:
    - `OBJECT REFCOUNT` 该命令主要用于调试 (debugging)，它能够返回指定key所对应value被引用的次数.
    - `OBJECT ENCODING` 该命令返回指定key对应value所使用的内部表示 (representation)(译者注：也可以理解为数据的压缩方式).
    - `OBJECT IDLETIME` 该命令返回指定key对应的value自被存储之后空闲的时间，以秒为单位 (没有读写操作的请求) ，这个值返回以10秒为单位的秒级别时间，这一点可能在以后的实现中改善
- 对象可以用多种方式编码:
    - 字符串可以被编码为raw (常规字符串) 或者int (用字符串表示64位无符号整数这种编码方式是为了节省空间).
    - 列表类型可以被编码为ziplist或者linkedlist。ziplist是为了节省较小的列表空间而设计一种特殊编码方式.
    - 集合被编码为intset或者hashtable。intset是为了存储数字的较小集合而设计的一种特殊编码方式.
    - 哈希表可以被编码为zipmap或者hashtable。zipmap是专为了较小的哈希表而设计的一种特殊编码方式
    - 有序集合被编码为ziplist或者skiplist格式。ziplist可以表示较小的有序集合，skiplist表示任意大小多的有序集合
- 一旦你做了一个操作让redis无法再使用那些节省空间的编码方式，它将自动将那些特殊的编码类型转换为普通的编码类型

## WAIT

```
WAIT numslaves timeout
```
- 此命令阻塞当前客户端，直到所有以前的写命令都成功的传输和指定的slaves确认。如果超时，指定以毫秒为单位，即使指定的slaves还没有到达，命令任然返回。
- 命令始终返回之前写命令发送的slaves的数量，无论是在指定slaves的情况还是达到超时。
- 注意点:
    - 当’WAIT’返回时，所有之前的写命令保证接收由WAIT返回的slaves的数量。
    - 如果命令呗当做事务的一部分发送，该命令不阻塞，而是只尽快返回先前写命令的slaves的数量。
    - 如果timeout是0那意味着永远阻塞。
    - 由于`WAIT`返回的是在失败和成功的情况下的slaves的数量。客户端应该检查返回的slaves的数量是等于或更大的复制水平。
- 一致性（Consistency and WAIT）
    - `WAIT` 不能保证Redis强一致：尽管同步复制是复制状态机的一个部分，但是还需要其他条件。不过，在sentinel和Redis群集故障转移中，WAIT能够增强数据的安全性。
    - 如果写操作已经被传送给一个或多个slave节点，当master发生故障我们极大概率 (不保证100%) 提升一个受到写命令的slave节点为master: 不管是Sentinel还是Redis Cluster都会尝试选slave节点中最优 (日志最新) 的节点，提升为master。
    - 尽管是选择最优节点，但是仍然会有丢失一个同步写操作可能行。
- 实现细节
    - 因为引入了部分同步，Redis slave节点在ping主节点时会携带已经处理的复制偏移量。 这被用在多个地方：
        - 检测超时的slaves
        - 断开连接后的部分复制
        - 实现 `WAIT`
    - 在`WAIT`实现的案例中，当客户端执行完一个写命令后，针对每一个复制客户端，Redis会为其记录写命令产生的复制偏移量。
    - 当执行命令`WAIT`时，Redis会检测slaves节点是否已确认完成该操作或更新的操作。