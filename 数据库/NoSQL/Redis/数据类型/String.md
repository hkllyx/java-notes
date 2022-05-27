# String

String，即字符串，是一种最基本的Redis值类型。

Redis字符串是二进制安全的，这意味着一个Redis字符串能包含任意类型的数据，例如：一张JPEG格式的图片或者一个序列化的Ruby对象。

一个字符串类型的值最多能存储512M字节的内容。

## 使用场景

- 利用`INCR`命令簇（`INCR`、`DECR`、`INCRBY`）来把字符串当作原子计数器使用。
- 使用`APPEND`命令在字符串后添加内容。
- 将字符串作为`GETRANGE`、`SETRANGE`的随机访问向量。
- 在小空间里编码大量数据，或者使用`GETBIT`和`SETBIT`创建一个Redis支持的布隆过滤器（Bloom Filter）。
  > 布隆过滤器是由一个固定大小的二进制向量或者位图（bitmap）和一系列映射（Hash）函数组成的，主要用于判断一个元素是否在一个集合中。初始位图值全部为0，向集合添加元素时，将元素通过多个映射函数找到位图中的多个点，如果这些点的值都是1，则元素很可能已经存在（因为经过映射函数Hash后，可能多个元素对应的位图点位相同，会有一定的误判率），否则一定不存在。也因为误判率的存在，布隆过滤器无法删除元素。

## 基本命令

### `SET`、`SETNX`、`SETEX`、`PSETEX`、`MSET`、`MSETNX`

```redis
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

设定key的值为指定的字符串。

如果key已经设定了对应的值，那么这个操作会直接覆盖原来的值，并且忽略其原类型。

当`SET`命令执行成功之后，之前设置的过期时间将失效。

选项：

| 选项            | 说明                                   | 相关命令 |
| --------------- | -------------------------------------- | -------- |
| EX seconds      | 设置key的过期时间，单位时秒          | `SETEX`  |
| PX milliseconds | 设置key的过期时间，单位时毫秒        | `PSETEX` |
| NX              | 只有key不存在的时候才会设置key的值 | `SETNX`  |
| XX              | 只有key存在的时候才会设置key的值   |          |

```redis
SETNX key value
SETEX key seconds value
PSETEX key milliseconds value
```

注意：由于`SET`命令加上选项已经可以完全取代`SETNX`、`SETEX`、`PSETEX`的功能，所以在将来的版本中，Redis可能会不推荐使用并且最终抛弃这几个命令。

```redis
MSET key value [key value ...]
MSETNX key value [key value ...]
```

`SET`的批量方法。

同`SET`命令一样，`MSET`会替换key已设定的值，如果不想覆盖则可以使用`MSETNX`，只要存在一个key已经设定了值，`MSETNX`就不会执行。

`MSET`和`MSETNX`都是原子的，所以所有给定的keys是一次性设置的。因此，客户端不可能看到一部分keys被更新而另外一部分没有改变的情况（不排除新旧值一致的情况）。

### `SETRANGE`

```redis
SETRANGE key offset value
```

覆盖key对应的字符串从offset开始到末尾的部分值。

如果offset比当前key对应字符串长，就在后面就补`\x00`以达到offset。不存在的keys被认为是空字符串，所以这个命令可以确保key有一个足够大的字符串能在offset处设置值。

注意：offset最大可以是2^29 - 1（536870911），因为redis字符串限制在512M大小。如果你需要超过这个大小，你可以用多个keys。

警告：当设置最后一个字节并且key还没有一个字符串值或者其值是个比较小的字符串时，Redis需要立即分配所有内存，这有可能会导致服务阻塞一会。注意，一旦第一次内存分配完，后面对同一个key调用`SETRANGE`就不再会预先内存分配。

### `GET`、`MGET`

```redis
GET key
MGET key [key ...]
```

`GET`返回key的值。如果key不存在，返回特殊值`nil`。如果key的值不是字符串，就返回错误，因为`GET`只能获取字符串类型的值。

`MGET`返回所有指定的keys的值。对于每个值不是字符串或者不存在的key，都返回特殊值`nil`。正因为此，这个操作从来不会失败。

### `GETRANGE`

```redis
GETRANGE key start end
```

Reids 2.0之前叫`SUBSTR`。

返回key对应的字符串值的子串，这个子串是由start和end位移（两个位移处的字符都包括在结果中）决定的。

可以用负的位移来表示从字符串尾部开始数的下标。所以-1就是最后一个字符，-2就是倒数第二个，以此类推。

这个函数处理超出范围的请求时，都把结果限制在字符串内（end超出字符串的长度部分忽略）。

### `GETSET`

```redis
GETSET key value
```

原子性地将key对应的值返回，并且设置成新值。如果key存在但是对应的值不是字符串，就返回错误。

### `APPEND`

```redis
APPEND key value
```

将指定值追加到指定key对应的字符串值后面。

如果key已经存在，并且值为字符串，那么这个命令会追加到原值的结尾。如果key不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况`APPEND`将类似于`SET`操作。

### `STRLEN`

```redis
STRLEN key
```

返回key对应的字符串值的长度。如果key对应的值不是字符串，就返回错误。

## 数值相关命令

### `INCR`、`DECR`、`INCRBY`、`DECRBY`

```redis
INCR key
DECR key
```

对key对应的数字（以字符串形式存储）做加或减1操作。

如果key不存在，那么在操作之前，这个key对应的值会被置为0。

如果key有一个错误类型的值或者是一个不能表示成数字的字符串，就返回错误。这个操作最大支持在64位有符号的整型数字。

```redis
INCRBY key increment
DECRBY key decrement
```

类似`INCR`和`DECR`，不同的是将key对应的数字加increment或减decrement。

### `INCRBYFLOAT`

```redis
INCRBYFLOAT key increment
```

将key的对应的数字增加指定浮点数，当对应的值不存在时，先将其值设为0再操作。下面任一情况都会返回错误:

- 非法值（不是字符串）
- 当前的值或者相加后的值不能解析为一个双精度的浮点值（超出精度范围了）

字符串中已存的值或者相加参数可以任意选用指数符号，但相加计算的结果会以科学计数法的格式存储。

如果操作命令成功，相加后的值将替换原值存储在对应的键值上，并以字符串类型返回。无论各计算的内部精度如何，输出精度都固定为小数点后`17`位。
