- [概述](#概述)
- [使用场景](#使用场景)
- [相关命令](#相关命令)
    - [ZADD](#zadd)
    - [ZCARD](#zcard)
    - [ZINCRYBY](#zincryby)
    - [ZPOPMAX, ZPOPMIN, BZPOPMAX, BZPOPMIN](#zpopmax-zpopmin-bzpopmax-bzpopmin)
    - [ZUNIONSTORE, ZINTERSTORE](#zunionstore-zinterstore)
    - [ZRANGE, ZREVRANGE](#zrange-zrevrange)
    - [ZRANGEBYLEX, ZREVRANGEBYLEX, ZLEXCOUNT](#zrangebylex-zrevrangebylex-zlexcount)
    - [ZRANGEBYSCORE, ZREVRANGEBYSCORE, ZCOUNT](#zrangebyscore-zrevrangebyscore-zcount)
    - [ZRANK, ZREVRANGE](#zrank-zrevrange)
    - [ZSOCRE](#zsocre)
    - [ZREM, ZREMRANGEBYLEX, ZREMRANGEBYSCORE, ZREMRANGEBYRANK](#zrem-zremrangebylex-zremrangebyscore-zremrangebyrank)
    - [ZSCAN](#zscan)

# 概述

有序集合（Sorted sets）
- Redis有序集合和Redis集合类似，是不包含 相同字符串的合集。它们的差别是，每个有序集合的成员都关联着一个评分（score），这个评分用于把有序集合中的成员按最低分到最高分排列。
- 使用有序集合，你可以非常快地（O(log(N))）完成添加，删除和更新元素的操作。 因为元素是在插入时就排好序的，所以很快地通过评分 (score) 或者位次 (position) 获得一个范围的元素。
- 访问有序集合的中间元素同样也是非常快的，因此你可以使用有序集合作为一个没用重复成员的智能列表。在这个列表中，你可以轻易地访问任何你需要的东西：
    - 有序的元素
    - 快速的存在性测试
    - 快速访问集合中间元素
- 简而言之，使用有序集合你可以很好地完成很多在其他数据库中难以实现的任务。

# 使用场景

- 在一个巨型在线游戏中建立一个排行榜，每当有新的记录产生时，使用`ZADD`来更新它。你可以用`ZRANGE`轻松地获取排名靠前的用户，你也可以提供一个用户名，然后用`ZRANK`获取他在排行榜中的名次。 同时使用`ZRANK`和`ZRANGE`你可以获得与指定用户有相同分数的用户名单。所有这些操作都非常迅速。
- 有序集合通常用来索引存储在Redis中的数据。例如：如果你有很多的hash来表示用户，那么你可以使用一个有序集合，这个集合的年龄字段用来当作评分，用户ID当作值。用`ZRANGEBYSCORE`可以简单快速地检索到给定年龄段的所有用户。
- 有序集合或许是最高级的Redis数据类型。

# 相关命令

## ZADD

```
ZADD key [NX|XX] [CH] [INCR] score member [score member ...]
```
- 将所有指定成员添加到键为key有序集合（sorted set）里面。 添加时可以指定多个分数 / 成员（score/member）对。
- 如果指定添加的成员已经是有序集合里面的成员，则会更新改成员的分数（scrore）并更新到正确的排序位置。
- 如果key不存在，将会创建一个新的有序集合（sorted set）并将分数 / 成员（score/member）对添加到有序集合，就像原来存在一个空的有序集合一样。
- 如果key存在，但是类型不是有序集合，将会返回一个错误应答。
- 分数值是一个双精度的浮点型数字字符串。+inf和 -inf都是有效值。
- `ZADD` 选项

    | 选项 | 说明                                                    |
    | ---- | ------------------------------------------------------- |
    | XX   | 仅更新存在的成员，不添加新成员                          |
    | NX   | 不更新存在的成员，只添加新成员                          |
    | CH   | changed，修改返回值为发生变化的成员总数，不包括分数更新 |
    | INCR | 类似`ZINCRBY`命令，对成员的分数进行递增操作           |

## ZCARD

```
ZCARD key
```
- 返回key的有序集元素个数。

## ZINCRYBY

```
ZINCRBY key increment member
```
- 为有序集key的成员member的score值加上增量increment。
- 如果key中不存在member，就在key中添加一个member，score是increment（就好像它之前的score是0.0）。
- 如果key不存在，就创建一个只含有指定member成员的有序集合。
- 当key不是有序集类型时，返回一个错误。
- score值必须是字符串表示的整数值或双精度浮点数，并且能接受double精度的浮点数。也有可能给一个负数来减少score的值。

## ZPOPMAX, ZPOPMIN, BZPOPMAX, BZPOPMIN

```
ZPOPMAX key [count]
ZPOPMIN key [count]
```
- 删除并返回有序集合key中的最多count个具有最高 / 最低得分的成员。
- 如未指定，count的默认值为1。
- 指定一个大于有序集合的基数的count不会产生错误。
- 当返回多个元素时候，得分最高 / 最低的元素将是第一个元素，然后是分数较低的元素。

```
BZPOPMAX key [key ...] timeout
BZPOPMIN key [key ...] timeout
```
- 前两个命令的阻塞版本，因为当没有成员可以从任何给定的排序集弹出时，它阻塞了连接。
- 得分最高 / 最低的成员将从非空的第一个排序集中弹出，并按给定键的顺序检查它们。
- timeout参数被解释为一个整数值，指定要阻塞的最大秒数。0的超时可用于无限期阻塞。

## ZUNIONSTORE, ZINTERSTORE

```
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight] [SUM|MIN|MAX]
```
- 计算给定的numkeys个有序集合的并集，并且把结果放到destination中。
- 在给定要计算的key和其它参数之前，必须先给定key个数 (numberkeys)。默认情况下，结果集中某个成员的score值是所有给定集下该成员score值之和。
- 使用WEIGHTS选项，你可以为每个给定的有序集指定一个乘法因子，意思就是，每个给定有序集的所有成员的score值在传递给聚合函数之前都要先乘以该因子。如果WEIGHTS没有给定，默认就是1。
- 使用AGGREGATE选项，你可以指定并集的结果集的聚合方式。
    - 默认使用的参数SUM，可以将所有集合中某个成员的score值之和作为结果集中该成员的score值。
    - 如果使用参数MIN或者MAX，结果集就是所有集合中元素最小或最大的元素。
- 如果key destination存在，就被覆盖。

```
ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight] [SUM|MIN|MAX]
```
- 计算给定的numkeys个有序集合的交集，并且把结果放到destination中。
- 在给定要计算的key和其它参数之前，必须先给定key个数 (numberkeys)。默认情况下，结果中一个元素的分数是有序集合中该元素分数之和，前提是该元素在这些有序集合中都存在。
- 因为交集要求其成员必须是给定的每个有序集合中的成员，结果集中的每个元素的分数和输入的有序集合个数numkeys相等。
- 对于WEIGHTS和AGGREGATE参数的描述，参见命令 `ZUNIONSTORE`。
- 如果destination存在，就把它覆盖。

## ZRANGE, ZREVRANGE

```
ZRANGE key start stop [WITHSCORES]
```
- 返回存储在有序集合key中的指定范围的元素。返回的元素可以认为是按得分从最低到最高排列。如果得分相同，将按字典排序。
- 当你需要元素从最高分到最低分排列时，请参阅 `ZREVRANGE`（相同的得分将使用字典倒序排序）。
- 参数start和stop都是基于零的索引，即0是第一个元素，1是第二个元素，以此类推。 它们也可以是负数，表示从有序集合的末尾的偏移量，其中 -1是有序集合的最后一个元素，-2是倒数第二个元素，等等。
- start和stop都是全包含的区间，因此例如`ZRANGE myzset 0 1`将会返回有序集合的第一个和第二个元素。
- 超出范围的索引不会产生错误。如果start参数的值大于有序集合中的最大索引，或者start > stop，将会返回一个空列表。 如果stop的值大于有序集合的末尾，Redis会将其视为有序集合的最后一个元素。
- 可以传递`WITHSCORES`选项，以便将元素的分数与元素一起返回。。客户端类库可以自由地返回更合适的数据类型（建议：具有值和得分的数组或记录）。

## ZRANGEBYLEX, ZREVRANGEBYLEX, ZLEXCOUNT

```
ZRANGEBYLEX key min max [LIMIT offset count]
```
- 返回指定成员区间内的成员，按成员字典正序排序，分数必须相同。
- 在某些业务场景中，需要对一个字符串数组按名称的字典顺序进行排序时，可以使用Redis中SortSet这种数据结构来处理。
- 提示:
    - 分数必须相同！如果有序集合中的成员分数有不一致的，返回的结果就不准（会根据分数排序）。
    - 成员字符串作为二进制数组的字节数进行比较。
    - 默认是以ASCII字符集的顺序进行排列。如果成员字符串包含utf-8这类字符集的内容，就会影响返回结果，所以建议不要使用。
    - 默认情况下，“max” 和 “min” 参数前必须加 “[” 符号作为开头。”[” 符号与成员之间不能有空格，返回成员结果集会包含参数 “min” 和 “max” 。
    - “max” 和 “min” 参数前可以加 “(“ 符号作为开头表示小于，“(“ 符号与成员之间不能有空格。返回成员结果集不会包含 “max” 和 “min” 成员。
    - 可以使用 “-“ 和 “+” 表示得分最小值和最大值
    - “min” 和 “max” 不能反，“max” 放前面 “min” 放后面会导致返回结果为空
    - 与`ZRANGEBYLEX`获取顺序相反的指令是 `ZREVRANGEBYLEX`。
    - 源码中采用C语言中`memcmp()`函数，从字符的第0位到最后一位进行排序，如果前面部分相同，那么较长的字符串比较短的字符串排序靠后。

```
ZREVRANGEBYLEX key max min [LIMIT offset count]
```

```
ZLEXCOUNT key min max
```
- 用于计算有序集合中指定成员之间的成员数量。
- 提示：
    - 成员名称前需要加 '[' 符号作为开头，'[' 符号与成员之间不能有空格
    - 可以使用 '-' 和 '+' 表示得分最小值和最大值
    - min和max不能反，max放前面min放后面会导致返回结果为0
    - 计算成员之间的成员数量时，参数min和max的位置也计算在内
    - min和max参数的含义与`ZRANGEBYLEX`命令中所描述的相同

## ZRANGEBYSCORE, ZREVRANGEBYSCORE, ZCOUNT

```
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
```
- 返回key的有序集合中的分数在min和max之间的所有元素（包括分数等于max或者min的元素）。元素被认为是从低分到高分排序的。
- 具有相同分数的元素按字典序排列（这个根据redis对有序集合实现的情况而定，并不需要进一步计算）。
- 可选的LIMIT参数指定返回结果的数量及区间。注意，如果offset太大，定位offset就可能遍历整个有序集合，这会增加O(N) 的复杂度。
- 可选参数WITHSCORES会返回元素和其分数，而不只是元素。这个选项在redis2.0之后的版本都可用。
- 区间及无限
    - min和max可以是 - inf和 + inf，这样一来，你就可以在不知道有序集的最低和最高score值的情况下，使用ZRANGEBYSCORE这类命令。
    - 默认情况下，区间的取值使用闭区间 (小于等于或大于等于)，你也可以通过给参数前增加 (符号来使用可选的开区间 (小于或大于)。

```
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

```
ZCOUNT key min max
```
- 返回有序集key中score值在min和max之间 (默认包括score值等于min或max) 的成员。
- 关于参数min和max的详细使用方法，请参考`ZRANGEBYSCORE`命令。

## ZRANK, ZREVRANGE

```
ZRANK key member
```
- 返回有序集key中成员member的排名。其中有序集成员按score值递增 (从小到大) 顺序排列。排名以0为底，也就是说，score值最小的成员排名为0。
- 使用`ZREVRANK`命令可以获得成员按score值递减 (从大到小) 排列的排名。

```
ZREVRANK key member
```

## ZSOCRE

```
ZSCORE key member
```
- 返回有序集key中，成员member的score值。
- 如果member元素不是有序集key的成员，或key不存在，返回nil。

## ZREM, ZREMRANGEBYLEX, ZREMRANGEBYSCORE, ZREMRANGEBYRANK

```
ZREM key member [member ...]
```
- 从有序集中删除指定成员，成员不存在则被忽略。
- 当key存在，但是其不是有序集合类型，就返回一个错误。

```
ZREMRANGEBYLEX key min max
```
- 删除名称按字典由低到高排序成员之间所有成员。
- 不要在成员分数不同的有序集合中使用此命令，因为它是基于分数一致的有序集合设计的，如果使用，会导致删除的结果不正确。
- 待删除的有序集合中，分数最好相同，否则删除结果会不正常。


```
ZREMRANGEBYSCORE key min max
```
- 移除有序集key中，所有score值介于min和max之间 (包括等于min或max) 的成员。

```
ZREMRANGEBYRANK key start stop
```
- 移除有序集key中，指定排名 (rank) 区间内的所有成员。
- 下标参数start和stop都以0为底，0处是分数最小的那个元素。
- 这些索引也可是负数，表示位移从最高分处开始数。例如，-1是分数最高的元素，-2是分数第二高的，依次类推。

## ZSCAN

```
ZSCAN key cursor [MATCH pattern] [COUNT count]
```
- 参见Redis Key中`SCAN`命令。