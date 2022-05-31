# Sorted Set

Redis有序集合和Redis集合类似，是不包含相同字符串的合集。它们的差别是，每个有序集合的元素都关联着一个评分（score），这个评分用于把有序集合中的成员按最低分到最高分排列。

使用有序集合，你可以非常快地（O(log(N))）完成添加，删除和更新元素的操作。因为元素是在插入时就排好序的，所以很快地通过评分（score）或者位次（position）获得一个范围的元素。

访问有序集合的中间元素同样也是非常快的，因此你可以使用有序集合作为一个没用重复成员的智能列表。在这个列表中，你可以轻易地访问任何你需要的东西：

- 有序的元素
- 快速的存在性测试
- 快速访问集合中间元素

简而言之，使用有序集合你可以很好地完成很多在其他数据库中难以实现的任务。

## 使用场景

- 在一个巨型在线游戏中建立一个排行榜，每当有新的记录产生时，使用`ZADD`来更新它。可以用`ZRANGE`轻松地获取排名靠前的用户；也可以提供一个用户名，然后用`ZRANK`获取他在排行榜中的名次。同时使用`ZRANK`和`ZRANGE`可以获得与指定用户有相同分数的用户名单。所有这些操作都非常迅速。
- 有序集合通常用来索引存储在Redis中的数据。例如：如果你有很多的Hash来表示用户，那么你可以使用一个有序集合，这个集合的年龄字段用来当作评分，用户ID当作值。用`ZRANGEBYSCORE`可以简单快速地检索到给定年龄段的所有用户。

## 相关命令

### `ZADD`

```redis
ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
```

将所有指定分数/元素对添加到key对应的有序集合里面。

如果指定添加的成员已经是有序集合里面的成员，则会更新改成员的分数并更新到正确的排序位置。

如果key不存在，将会创建一个新的有序集合并将添加，就像原来存在一个空的有序集合一样。如果key存在，但是类型不是有序集合，将会返回一个错误应答。

分数值是一个双精度的浮点型数字字符串。`+inf`和`-inf`都是有效值。

选项

| 选项 | 说明                                                    |
| ---- | ------------------------------------------------------- |
| XX   | 仅更新存在的成员，不添加新成员                          |
| NX   | 不更新存在的成员，只添加新成员                          |
| CH   | changed，修改返回值为发生变化的成员总数，不包括分数更新 |
| INCR | 类似`ZINCRBY`命令，对成员的分数进行增加操作             |

### `ZINCRBY`

```redis
ZINCRBY key increment member
```

对key对应的有序集合的元素的评分加上增量increment。

如果有序集合不存在指定的元素，就将该元素添加到结婚证，分数就是increment（就好像它之前的分数是0.0）。如果key不存在，就创建一个只含有指定member成员的有序集合。当key不是有序集类型时，返回一个错误。

score值必须是字符串表示的整数值或双精度浮点数，并且能接受double精度的浮点数。也有可能给一个负数来减少score的值。

### `ZPOPMAX`、`ZPOPMIN`、`BZPOPMAX`、`BZPOPMIN`

```redis
ZPOPMAX key [count]
ZPOPMIN key [count]
```

删除并返回key对应的有序集合中最多count个具有最高/最低分数的成员。如未指定，count的默认值为1。

指定一个大于有序集合的基数的count不会产生错误。当返回多个元素时候，得分最高/最低的元素将是第一个元素，然后是分数较低的元素。

```redis
BZPOPMAX key [key ...] timeout
BZPOPMIN key [key ...] timeout
```

前两个命令的阻塞版本，因为当没有成员可以从任何给定的排序集弹出时，它阻塞了连接。timeout是一个整数，单位秒。0表示无限期阻塞。

### `ZCARD`

```redis
ZCARD key
```

返回key对应的有序集的基数（元素个数）。

### `ZRANGE`、`ZREVRANGE`

```redis
ZRANGE key start stop [WITHSCORES]
ZREVRANGE key start stop [WITHSCORES]
```

`ZRANGE`返回存储在有序集合key中的指定范围的元素。返回的元素可以认为是按分数从最低到最高排列。如果得分相同，将按字典排序。如果想获得相反排序的结果，使用`ZREVRANGE`。

参数start和stop都是基于0的索引，即0是第一个元素，1是第二个元素，以此类推。它们也可以是负数，表示从有序集合的末尾的偏移量，其中-1是有序集合的最后一个元素，-2是倒数第二个元素，等等。

start和stop都是全包含的区间，因此例如`ZRANGE myzset 0 1`将会返回有序集合的第一个和第二个元素。超出范围的索引不会产生错误。如果start参数的值大于有序集合中的最大索引，或者start > stop，将会返回一个空列表。如果stop的值大于有序集合的末尾，Redis会将其视为有序集合的最后一个元素。

可以传递WITHSCORES选项，以便将元素的分数与元素一起返回。客户端类库可以自由地返回更合适的数据类型（建议：具有值和得分的数组或记录）。

### `ZRANGEBYLEX`、`ZREVRANGEBYLEX`、`ZLEXCOUNT`

```redis
ZRANGEBYLEX key min max [LIMIT offset count]
ZREVRANGEBYLEX key max min [LIMIT offset count]
```

`ZRANGEBYLEX`和`ZREVRANGEBYLEX`返回指定元素区间内的元素，按成员字典正序排序，分数必须相同。

在某些业务场景中，需要对一个字符串数组按名称的字典顺序进行排序时，可以使用Redis中有序这种数据结构来处理。

注意:

- 分数必须相同！如果有序集合中的成员分数有不一致的，返回的结果就不准（会根据分数排序）。
- 成员字符串作为二进制数组的字节数进行比较。
- 默认是以ASCII字符集的顺序进行排列。如果成员字符串包含utf-8这类字符集的内容，就会影响返回结果，所以建议不要使用。
- 默认情况下，max和min参数前必须加`[`符号作为开头。`[`符号与参数之间不能有空格，返回结果集会包含参数就会包含min和max。
- max和min参数前也可以加`(`符号作为开头，`(`符号与参数之间不能有空格。返回成员结果集不会包含max和min。
- 可以使用`-`和`+`表示得分最小值和最大值
- min和max位置不能错位，否则会导致返回结果为空
- 源码中采用C语言中`memcmp()`函数，从字符的第0位到最后一位进行排序，如果前面部分相同，那么较长的字符串比较短的字符串排序靠后

```redis
ZLEXCOUNT key min max
```

用于计算有序集合中指定成员之间的成员数量。min、max的语法同上。

### `ZRANGEBYSCORE`、`ZREVRANGEBYSCORE`、`ZCOUNT`

```redis
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

返回分数在min和max之间（闭区间，包括分数等于max或者min的元素）的所有元素。具有相同分数的元素按字典序排列（这个根据redis对有序集合实现的情况而定，并不需要进一步计算）。

LIMIT参数指定返回结果的数量及区间。注意，如果offset太大，定位offset就可能遍历整个有序集合，这会增加O(N)的复杂度。

WITHSCORES参数要求返回元素的分数。Redis2.0之后的版本可用。

区间及无限：

- min和max可以是`-inf`和`+inf`
- 默认情况下，区间的取值使用闭区间，你也可以通过给参数前增加`(`符号来使用可选的开区间

```redis
```

```redis
ZCOUNT key min max
```

返回分数在min和max之间（闭区间）的元素个数。

### `ZRANK`、`ZREVRANGE`

```redis
ZRANK key member
ZREVRANK key member
```

返回元素在key对应的有序集合中的排名。`ZRANK`中元素排名按分数升序，排名0表示该元素的分数最小；`ZREVRANK`则相反。

### `ZSCORE`

```redis
ZSCORE key member
```

返回元素在key对应的有序集合中的分数。如果元素不是有序集key的成员，或key不存在，返回`nil`。

### `ZREM`、`ZREMRANGEBYLEX`、`ZREMRANGEBYSCORE`、`ZREMRANGEBYRANK`

```redis
ZREM key member [member ...]
```

从有序集中删除指定成员，成员不存在则被忽略。当key存在，但是其值不是有序集合则返回错误。

```redis
ZREMRANGEBYLEX key min max
```

删除按字典升序排序min和max元素之间（闭区间）的所有元素。不要在成员分数不同的有序集合中使用此命令，因为它是基于分数一致的有序集合设计的，如果使用，会导致删除的结果不正确。

```redis
ZREMRANGEBYSCORE key min max
```

删除按分数升序排序分数在min和max之间（闭区间）的所有元素。

```redis
ZREMRANGEBYRANK key start stop
```

删除排名（按分数升序排名）在min和max之间（闭区间）的所有元素。下标参数start和stop都从0开始。

这些索引也可是负数，表示位移从最高分处开始数。例如，-1是分数最高的元素，-2是分数第二高的，依次类推。

### `ZUNIONSTORE`、`ZINTERSTORE`

```redis
ZUNIONSTORE destination num-keys key [key ...]
    [WEIGHTS weight]
    [SUM | MIN | MAX]
```

计算给定的num-keys个有序集合的并集，并且把结果放到destination中。

在给定要计算的key和其它参数之前，必须先给定key个数（num-keys）。默认情况下，结果集中某个元素的分数是所有给定集中相同元素分数之和。

使用`WEIGHTS`选项，你可以为每个给定的有序集指定一个乘法因子，意思就是每个给定有序集的所有元素的分数在传递给聚合函数之前都要先乘以该因子。如果`WEIGHTS`没有给定，默认就是1。

使用聚合选项，可以指定并集的结果集的聚合方式。

- 默认使用`SUM`，将所有集合中某个元素分数之和作为结果集中该元素的分数。
- 如果使用`MIN`或者`MAX`，将所有集合中某个元素的最高/最低分数作为结果集中该元素的分数。

如果destination已存在会被覆盖。

```redis
ZINTERSTORE destination numkeys key [key ...]
    [WEIGHTS weight]
    [SUM | MIN | MAX]
```

类似`ZUNIONSTORE`，不同之处时计算所有给定有序集合的交集。

### `ZSCAN`

```redis
ZSCAN key cursor [MATCH pattern] [COUNT count]
```

参见Key中[`SCAN`](../Key.md#scan)命令。
