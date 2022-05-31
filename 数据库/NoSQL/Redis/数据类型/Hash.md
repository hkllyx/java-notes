# Hash

Redis Hash是字符串字段和字符串值之间的映射。

一个拥有少量（100个左右）字段的Hash只很少的空间来存储，所有你可以在一个小型的Redis实例中存储上百万的对象。

尽管Hash主要用来表示对象，但它们也能够存储许多元素，所以你也可以用Hash来完成许多其他的任务。

一个hash最多可以包含2^32 - 1个键值对（超过40亿）。

## 使用场景

Hash是完美的表示对象（eg：一个有名，姓，年龄等属性的用户）的数据类型。

```redis
@cli
HMSET user:1000 username antirez password P1pp0 age 34
HGETALL user:1000
HSET user:1000 password 12345
HGETALL user:1000
```

## 相关命令

### `HSET`、`HMSET`、`HSETNX`

```redis
HSET key field value
```

设置key对应Hash中指定字段的值。如果key对应的Hash不存在，会创建一个新的Hash并与key关联；如果字段在Hash中存在，它将被重写。

```redis
HMSET key field value [field value ...]
```

批量设置key对应的Hash中指定字段的值。该命令将重写所有在Hash中存在的字段。

```redis
HSETNX key field value
```

只在key对应的Hash中不存在指定的字段时才设置字段的值。如果key对应的Hash不存在，会创建一个新的Hash并与key关联；如果字段已存在，该操作无效果。

### `HINCRBY`、`HINCRBYFLOAT`

```redis
HINCRBY key field increment
HINCRBYFLOAT key field increment
```

类似String的`INCRBY`、`INCRBYFLOAT`命令。

### `HGET`、`HMGET`、`HGETALL`、`HVALS`

```redis
HGET key field
```

返回key对应的Hash中指定字段对应的值。

```redis
HMGET key field [field ...]
```

`HGET`的批量形式。

对于Hash中不存在的每个字段，返回`nil`值。因为不存在的keys被认为是一个空的Hash，对一个不存在的key执行`HMGET`将返回一个只含有`nil`值的列表

```redis
HGETALL key
```

返回key对应的Hash中所有的字段和值。返回值中，每个字段名的下一个是它的值，所以返回值的长度是Hash大小的两倍。

```redis
HVALS key
```

返回key对应的Hash中所有字段的值。

### `HDEL`

```redis
HDEL key field [field ...]
```

从key对应的Hash中移除指定字段。在Hash中不存在的域将被忽略。

如果key对应的Hash不存在，它将被认为是一个空的Hash，该命令将返回0。

### `HLEN`

```redis
HLEN key
```

返回key对应的Hash包含的字段的数量。

### `HSTRLEN`

```redis
HSTRLEN key field
```

返回key对应的Hash中指定字段的值的字符串长度，如果Hash或者字段不存在则返回0。

### `HKEYS`

```redis
HKEYS key
```

返回key对应的Hash中所有字段的名字。

### `HEXISTS`

```redis
HEXISTS key field
```

返回key对应的Hash是否存在指定字段。

### `ZSCAN`

```redis
ZSCAN key cursor [MATCH pattern] [COUNT count]
```

参见Key中[`SCAN`](../Key.md#scan)命令。
