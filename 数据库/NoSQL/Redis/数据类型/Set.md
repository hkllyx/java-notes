# Set

Redis集合是一个无序的字符串合集。你可以以O(1)的时间复杂度（无论集合中有多少元素时间复杂度都为常量）完成添加、删除以及检查元素是否存在的操作。

Redis集合有着不允许相同成员存在的优秀特性。向集合中多次添加同一元素，在集合中最终只会存在一个此元素。实际上这就意味着，在添加元素前，你并不需要事先进行检验此元素是否已经存在的操作。

一个Redis列表支持一些集合运算的服务端命令。所以你可以在很短的时间内完成合并（union）、求交（intersection）、找出不同元素的操作。

一个集合最多可以包含2^32 - 1个元素。

## 使用场景

- 用集合跟踪一个独特的事。想要知道所有访问某个博客文章的独立IP，只要每次都用`SADD`来处理一个页面访问。那么你可以肯定重复的IP是不会插入的。
- Redis集合能很好的表示关系。你可以创建一个tagging系统，然后用一个集合来代表一个tag。接下来用`SADD`命令把所有拥有此tag的对象的ID添加进此集合，这样来表示哪些对象包含此tag。如果你想找出同时有3个不同tag的所有对象的所有ID，那么你需要使用`SINTER`（取交集）。
- 使用`SPOP`或者`SRANDMEMBER`命令随机地获取元素。

## 相关命令

### `SADD`

```redis
SADD key member [member ...]
```

向key对应的集合添加一个或多个元素。添加的元素如果已经在集合中存在则会被忽略。

如果key不存在，则新建一个集合, 并添加元素到集合中。如果key对应值的类型不是集合则返回错误。

### `SREM`

```redis
SREM key member [member ...]
```

从key对应的集合中移除指定元素。

如果指定的元素不是集合中的元素则忽略。如果key不存在则被视为一个空的集合，该命令返回0。如果key的类型不是一个集合，则返回错误。

### `SPOP`

```redis
SPOP key [count]
```

从key对应的集合中移除并返回一个或多个随机元素。

count参数将在更高版本中提供，但是在2.6、2.8、3.0中不可用。如果count大于集合内部的元素数量，此命令将会返回整个集合，但不会有额外的元素。

### `SRANDMEMBER`

```redis
SRANDMEMBER key [count]
```

如果仅提供key参数，那么随机返回key集合中的一个元素。

Redis 2.6开始，可以接受count参数：

- 如果count是整数且小于元素的个数，返回含有count个不同的元素的数组
- 如果count是个整数且大于集合中元素的个数时，仅返回整个集合的所有元素
- 如果count是负数，则会返回一个包含count的绝对值的个数元素的数组
- **如果count的绝对值大于元素的负数，则返回的结果集里会出现一个元素出现多次的情况**

### `SMOVE`

```redis
SMOVE source destination member
```

将指定元素从source集合移动到destination集合中。对于其他的客户端，在特定的时间元素将会作为source或者destination集合的成员出现。

如果source集合不存在或者不包含指定的元素，则命令不执行任何操作并返回0。否则对象将会从source集合中移除，并添加到destination集合中去并返回1。如果destination集合已经存在该元素，则命令仅将该元素从source集合中移除。

如果source和destination不是集合类型，则返回错误。

### `SCARD`

```redis
SCARD key
```

返回key对应集合的基数（集合元素的数量）。如果key不存在，则返回0。

### `SMEMBERS`

```redis
SMEMBERS key
```

返回key对应集合所有的元素。该命令的作用与使用一个参数的`SINTER`命令作用相同。

### `SISMEMBER`

```redis
SISMEMBER key member
```

返回指定元素是否是key对应集合的成员。如果member元素是集合key的成员，则返回1，否则返回0。

### `SDIFF`、`SDIFFSTORE`

```redis
SDIFF key [key ...]
```

返回第一个key对应的集合与其他key对应的集合的差集。如果key不存在则被认为值是一个空集。

```redis
SDIFFSTORE destination key [key ...]
```

该命令类似于`SDIFF`，不同之处在于该命令不返回结果集，而是将结果存放在destination集合中。如果destination已经存在，则将其覆盖。

### `SINTER`、`SINTERSTORE`

```redis
SINTER key [key ...]
```

返回指定所有key对应的集合的交集。如果key不存在则被认为值是一个空集。

```redis
SINTERSTORE destination key [key ...]
```

该命令类似于`SINTER`，不同之处在于该命令不返回结果集，而是将结果存放在destination集合中。如果destination已经存在，则将其覆盖。

### `SUNION`、`SUNIONSTORE`

```redis
SUNION key [key ...]
```

返回指定的多个key对应的集合的并集。如果key不存在则被认为值是一个空集。

```redis
SUNIONSTORE destination key [key ...]
```

该命令类似于`SUNION`，不同之处在于该命令不返回结果集，而是将结果存放在destination集合中。如果destination已经存在，则将其覆盖。

### `SSCAN`

```redis
SSCAN cursor [MATCH pattern] [COUNT count]
```

参见Key中[`SCAN`](../Key.md#scan)命令。
