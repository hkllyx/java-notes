# Geo

主要用于存储地理位置信息，并对存储的信息进行操作，该功能在Redis 3.2新增。

## Geohash算法

GeoHash将二维的经纬度转换成字符串。

[高效的多维空间点索引算法 —Geohash和Google S2](https://halfrost.com/go_spatial_search/)

[GeoHash核心原理解析](https://www.cnblogs.com/LBSer/p/3310455.html)

## 相关命令

### `GEOADD`

```redis
GEOADD key longitude latitude member [longitude latitude member ...]
```

将指定的地理空间位置（纬度、经度、名称）添加到指定的key中，这些数据将会存储到有序集合（Sorted Set），目的是为了方便使用`GEORADIUS`或者`GEORADIUSBYMEMBER`命令对数据进行半径查询等操作

该命令采用标准格式，所以经度必须在纬度之前。且经纬度的限制由EPSG:900913/EPSG:3785/OSGEO:41001规定如下：

- 有效的经度：-180 ~ 180度。
- 有效的纬度：-85.05112878 ~ 85.05112878度。

当坐标位置超出上述指定范围时，该命令将会返回一个错误。

### `GEOHASH`

```redis
GEOHASH key member [member ...]
```

返回有效的Geohash字符串，该字符串表示元素在表示地理空间索引的有序集合中的位置（索引在使用`GEOADD`时产生）。

通常，Redis使用Geohash技术（52位整数编码）的变体来表示元素的位置。由于编码和解码过程中所使用的初始最小和最大坐标不同，编码也不同于标准。

命令将返回11个字符长度的Geohash字符串，相比于使用52位整数编码它没有精度损失。返回的Geohashes具有以下特性：

- 可以从尾部缩短字符串，它将失去精度，但仍将指向同一地区
- 类似前缀的Geohash字符串是相近的，但相近位置的Geohash字符串前缀不一定类似

### `GEOPOS`

```redis
GEOPOS key member [member ...]
```

从key里返回所有指定位置元素的位置（经度和纬度）。

当通过`GEOADD`将坐标添加到地理空间索引时，坐标被转换为一个52位的Geohash，因此返回的坐标可能与用于添加元素的坐标不完全相同，可能会引发小的错误。

`GEOPOS`命令接受可变数量的位置元素作为输入，所以即使用户只给定了一个位置元素，命令也会返回数组回复。

### `GEODIST`

```redis
GEODIST key member1 member2 [unit]
```

返回两个给定位置之间的距离。如果两个位置之间的其中一个不存在，那么命令返回空值。

指定单位的参数unit必须是以下单位的其中一个：

| 单位 | 说明         |
| ---- | ------------ |
| m    | 米，省缺单位 |
| km   | 千米         |
| ft   | 英尺         |
| mi   | 英里         |

注意：`GEODIST`命令在计算距离时会假设地球为完美的球形，在极限情况下，这一假设最大会造成0.5%的误差。

### `GEORADIUS`、`GEORADIUSBYMEMBER`

```redis
GEORADIUS key longitude latitude radius m | km | ft | mi
    [WITHCOORD] [WITHDIST] [WITHHASH]
    [COUNT count]
    [ASC|DESC]
    [STORE key] [STOREDIST key]
```

以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定最大距离的所有位置元素。

在给定以下可选项时，命令会返回额外的信息：

| 选项      | 说明                                                                      |
| --------- | ------------------------------------------------------------------------- |
| `WITHDIST`  | 将位置元素与中心之间的距离也一并返回，单位和给定范围单位相同              |
| `WITHCOORD` | 将位置元素的经度和维度也一并返回                                          |
| `WITHHASH`  | 返回52位无符号整数形式的Geohash，主要用于底层应用或者调试，实际中作用不大 |

在默认情况下，`GEORADIUS`命令会返回所有匹配的位置元素。虽然用户可以使用`COUNT`选项去获取前N个匹配元素，但是因为命令在内部可能会需要对所有被匹配的元素进行处理，所以在对一个非常大的区域进行搜索时，即使只使用`COUNT`选项去获取少量元素，命令的执行速度也可能会非常慢。

但是从另一方面来说，使用`COUNT`选项去减少需要返回的元素数量，对于减少带宽来说仍然是非常有用的。

命令默认返回未排序的位置元素。通过以下两个参数，用户可以指定被返回位置元素的排序方式：

| 选项 | 说明                                           |
| ---- | ---------------------------------------------- |
| 省缺   | 未排序                                         |
| `ASC`  | 根据中心的位置，按照从近到远的方式返回位置元素 |
| `DESC` | 根据中心的位置，按照从远到近的方式返回位置元素 |

`STORE`和`STOREDIST`将返回值保持到有序集合（ZSet）中，分别以同`WITHHASH`和`WITHDIST`返回的额外值作为分数。

注意：`WITHDIST`等返回额外信息的选项不可以和`STORE`，`STOREDIST`共用。

```redis
GEORADIUSBYMEMBER key member radius m|km|ft|mi
    [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
    [ASC|DESC] [STORE key] [STOREDIST key]
```

这个命令和`GEORADIUS`命令一样，但是中心点是由给定的元素决定的，而不是使用输入的经度和纬度来决定中心点。

注意：在Redis 6.20开始被弃用，代替的是`GEOSEARCH`和`GEORADIUSBYMEMBER`。

### `GEOSEARCH`

```redis
GEOSEARCH key FROMMEMBER member | FROMLONLAT longitude latitude
    BYRADIUS radius m | km | ft | mi | BYBOX width height m|km|ft|mi
    [ASC | DESC]
    [ COUNT count [ANY]]
    [WITHCOORD] [WITHDIST] [WITHHASH]

GEOSEARCHSTORE destination source FROMMEMBER member | FROMLONLAT longitude latitude
    BYRADIUS radius m|km|ft|mi | BYBOX width height m | km | ft | mi
    [ASC | DESC]
    [ COUNT count [ANY]]
    [STOREDIST]
```

用于查找指定形状（圆形/方形）区域内的坐标。

中心点选择：

- `FROMMEMBER`：指定Geo内的某个元素为中心点
- `FROMLONLAT`：指定经纬度为中心点

查找区域形状：

- `BYRADIUS`：类似`GEORADIUS`，指定半径内查找
- `BYBOX`：对称矩形，即在横坐标距离中心点小于width，纵坐标距离中心点小于height

COUNT选项同`GEORADIUS`，返回前指定个坐标，但加上`ANY`后，会尽快得返回符合条件的坐标，这些坐标距离中心点的距离不一定是最近的几个。`ANY`省区了查找全部符合条件坐标后再排序的过程，可以大大提高查询效率。

使用`GEOSEARCHSTORE`是添加`STOREDIST`选项将返回坐标到中心点的距离，单位是命令指定的单位，然后存储到有序集合中。
