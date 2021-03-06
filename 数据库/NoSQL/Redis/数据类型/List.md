# List

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

`LPUSH`命令插入一个新元素到列表头部，而`RPUSH`命令插入一个新元素到列表的尾部。当对一个空key执行其中某个命令时，将会创建一个新表。

类似的，如果要清空列表，那么key会从对应的keyspace删除。这是个非常便利的语义，因为如果使用一个不存在的key作为参数，所有的列表命令都会像在对一个空表操作一样。

一个列表最多可以包含2^32 - 1个元素。从时间复杂度的角度来看，Redis列表主要的特性就是支持时间常数的插入和靠近头尾部元素的删除，即使是需要插入上百万的条目。访问列表两端的元素是非常快的，但如果你试着访问一个非常大的列表的中间元素仍然是十分慢的，因为那是一个时间复杂度为O(N)的操作。

## 使用场景

- 在社交网络中建立一个时间线模型，使用`LPUSH`去添加新的元素到用户时间线中，使用`LRANGE`去检索一些最近插入的条目。
- 同时使用`LPUSH`和`LTRIM`去创建一个永远不会超过指定元素数目的列表并同时记住最后的N个元素。
- 列表可以用来当作消息传递的基元（primitive），例如，众所周知的用来创建后台任务的Resque Ruby库。
- 你可以使用列表做更多事，这个数据类型支持许多命令，包括像`BLPOP`这样的阻塞命令。

## 常用命令

### `LPUSH`、`LPUSHX`、`RPUSH`、`RPUSHX`

```redis
LPUSH key value [value ...]
```

将所有指定值插入key对应列表的头部。如果key不存在，那么在进行操作前会创建一个空列表。如果key对应的值不是一个列表的话，那么会返回一个错误。

可以使用一个命令把多个元素加入进入列表，只需在命令末尾指定多个值。元素是从最左端的到最右端的、一个接一个被插入到列表的头部。所以对于这个命令例子 `LPUSH mylist a b c`，返回的列表是"c b a"。

```redis
LPUSHX key value
```

只有当key已经存在并且对应的值是列表时，在对应列表头部插入值。与`LPUSH`相反，当key不存在的时候不会进行任何操作。

```redis
RPUSH key value [value ...]
RPUSHX key value
```

类似`LPUSH`和`LPUSHX`，不同的是从尾部插入值。

### `LINSERT`

```redis
LINSERT key BEFORE | AFTER pivot value
```

把值插入到key对应的列表中基准元素pivot（pivot是列表中的元素）的前面或后面。

当key不存在时，对应的值会被看作是空列表，任何操作都不会发生。当key存在，但保存的不是一个列表的时候，会返回错误。

### `LSET`

```redis
LSET key index value
```

设置index位置的列表元素的值为指定值。更多关于index参数的信息，详见[`LINDEX`](#lindex)。当index超出范围时会返回一个错误。

### `LPOP`、`RPOP`

```redis
LPOP key
RPOP key
```

移除并且返回key对应列表的头部/尾部元素。当key不存在时返回`nil`。

### `RPOPLPUSH`

```redis
RPOPLPUSH source destination
```

原子性地返回并移除存储在source的列表的尾部元素，并把该元素放入插入在destination的列表的头部。

如果source不存在，那么会返回`nil`值，并且不会执行任何操作。如果source和destination是同样的，那么这个操作等同于将列表尾部元素移动到头部，所以这个命令也可以当作是一个旋转列表的命令。

### `BLPOP`、`BRPOP`、`BRPOPLPUSH`

```redis
BLPOP key [key ...] timeout
RPOP key [key ...] timeout
RPOPLPUSH source destination timeout
```

`BLPOP`、`BRPOP`是`LPOP`、`RPOP`的阻塞版本，当给定列表内没有任何元素可出列的时候，连接将被命令阻塞。当给定多个key参数时，按参数key的先后顺序依次检查各个列表，弹出第一个非空列表的头部/为不元素。

`BRPOPLPUSH`是`RPOPLPUSH`的阻塞版本。当source包含元素的时候，这个命令表现得跟`RPOPLPUSH`一模一样。当source是空的时候，Redis将会阻塞这个连接，直到另一个客户端向source插入元素进入或者达到timeout时限。

timeout为0能用于无限期阻塞客户端。

更多关于阻塞资料参见[BLPOP命令](http://redis.cn/commands/blpop.html)

### `LREM`

```redis
LREM key count value
```

从key对应的列表里移除前count个值等于value的元素。 这个count参数通过下面几种方式影响这个操作：

- count > 0: 从头往尾移除值为value的元素。
- count < 0: 从尾往头移除值为value的元素。
- count = 0: 移除所有值为value的元素。

比如，`LREM mylist -2 "hello"`会从存于mylist的列表里移除最后两个出现的"hello"。

需要注意的是，不存在的key会被当作空列表处理，这个命令会返回0。

### `LTRIM`

```redis
LTRIM key start stop
```

修剪（trim）一个key对应的列表，这样列表就会只包含指定范围的指定元素。start和stop都是由0开始计数的，这里的0是列表里的第一个元素（表头），1是第二个元素，以此类推。

超过范围的下标并不会产生错误：如果start超过列表尾部，或者start > end，结果会是列表变成空表（即该key会被移除）。 如果end超过列表尾部，Redis会将其当作列表的最后一个元素。

`LTRIM`的一个常见用法是和`LPUSH`/`RPUSH`一起使用。例如：

```redis
LPUSH mylist someelement
LTRIM mylist 0 99
```

这一对命令会将一个新的元素push进列表里，并保证该列表不会增长到超过100个元素。这个是很有用的，比如当用Redis来存储日志。需要特别注意的是，当用这种方式来使用`LTRIM`的时候，操作的复杂度是O(1)，因为平均情况下，每次只有一个元素会被移除。

### `LLEN`

```redis
LLEN key
```

返回存储在key里的列表的长度。如果key不存在，那么就被看作是空list，并且返回长度为0。当存储在key里的值不是一个列表的话，会返回错误。

### `LINDEX`

```redis
LINDEX key index
```

返回存储在key的列表里index处的元素。

index是从0开始的，所以0是表示第一个元素，1表示第二个元素，并以此类推。负数索引用于指定从列表尾部开始索引的元素。在这种方法下，-1表示最后一个元素，-2表示倒数第二个元素，并以此往前推。

当key位置的值不是一个列表的时候，会返回一个错误。

### `LRANGE`

```redis
LRANGE key start stop
```

返回存储在key的列表里指定范围内的元素。

start和end偏移量都是基于0的下标，即列表的第一个元素下标是0（头），第二个元素下标是1，以此类推。偏移量也可以是负数，表示偏移量是从尾部开始计数。例如，-1表示列表的最后一个元素，-2是倒数第二个，以此类推。

当下标超过列表范围的时候不会产生错误。如果start比List的尾部下标大的时候，会返回一个空列表。如果stop比列表的长度大的时候，Redis会当它是为不元素的下标。
