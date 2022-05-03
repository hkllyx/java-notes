- [Geohash算法](#geohash-算法)
- [相关命令](#相关命令)
  - [GEOADD](#geoadd)
  - [GEODIST](#geodist)
  - [GEOHASH](#geohash)
  - [GEOPOS](#geopos)
  - [GEORADIUS, GEORADIUSBYMEMBER](#georadius-georadiusbymember)

# Geohash算法

GeoHash将二维的经纬度转换成字符串。
[高效的多维空间点索引算法 —Geohash和Google S2](https://halfrost.com/go_spatial_search/)
[GeoHash核心原理解析](https://www.cnblogs.com/LBSer/p/3310455.html)

# 相关命令

## GEOADD

```
GEOADD key longitude latitude member [longitude latitude member ...]
```
- 将指定的地理空间位置（纬度、经度、名称）添加到指定的key中
- 这些数据将会存储到sorted set，目的是为了方便使用 `GEORADIUS` 或者 `GEORADIUSBYMEMBER` 命令对数据进行半径查询等操作
- 该命令以采用标准格式的参数x, y, 所以经度必须在纬度之前。
- 这些坐标的限制是可以被编入索引的，区域面积可以很接近极点但是不能索引。具体的限制，由EPSG:900913 / EPSG:3785 / OSGEO:41001规定如下：
    - 有效的经度从 -180 ~ 180度。
    - 有效的纬度从 -85.05112878 ~ 85.05112878度。
- 当坐标位置超出上述指定范围时，该命令将会返回一个错误。

## GEODIST

```
GEODIST key member1 member2 [unit]
```
- 返回两个给定位置之间的距离。
- 如果两个位置之间的其中一个不存在， 那么命令返回空值。
- 指定单位的参数unit必须是以下单位的其中一个：

    | 单位 | 说明         |
    | ---- | ------------ |
    | m    | 米，省缺单位 |
    | km   | 千米         |
    | mi   | 英里         |
    | ft   | 英尺         |
- 如果用户没有显式地指定单位参数， 那么GEODIST默认使用米作为单位。
- `GEODIST` 命令在计算距离时会假设地球为完美的球形，在极限情况下，这一假设最大会造成0.5% 的误差。

## GEOHASH

```
GEOHASH key member [member ...]
```
- 返回有效的Geohash字符串，该字符串表示一个或多个元素在表示地理空间索引的Sorted Set中的位置（其中使用 `GEOADD` 添加元素）。
- 通常，Redis使用Geohash技术的变体使用52位整数编码表示元素的位置。由于编码和解码过程中所使用的初始最小和最大坐标不同，编码的编码也不同于标准。
- 命令将返回10个字符的Geohash字符串，所以没有精度Geohash，损失相比，使用内部52位表示。返回的geohashes具有以下特性：
    - 他们可以缩短从右边的字符。它将失去精度，但仍将指向同一地区。
    - 与类似的前缀字符串是附近，但相反的是不正确的，这是可能的，用不同的前缀字符串附近。

## GEOPOS

```
GEOPOS key member [member ...]
```
- 从key里返回所有指定位置元素的位置（经度和纬度）。
- 对于表示地理空间索引的Sorted Set，使用 `GEOADD` 命令填充索引，通常需要获取指定成员的坐标。当通过 `GEOADD` 填充地理空间索引时，坐标被转换为一个52位的Geohash，因此返回的坐标可能与用于添加元素的坐标不完全相同，可能会引发小的错误。
- 因为 `GEOPOS` 命令接受可变数量的位置元素作为输入，所以即使用户只给定了一个位置元素，命令也会返回数组回复。

## GEORADIUS, GEORADIUSBYMEMBER

```
GEORADIUS key longitude latitude radius m|km|ft|mi
    [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
    [ASC|DESC] [STORE key] [STOREDIST key]
```
- 以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定最大距离的所有位置元素。
- 在给定以下可选项时，命令会返回额外的信息：

    | 选项      | 说明                                                                                                        |
    | --------- | ----------------------------------------------------------------------------------------------------------- |
    | WITHDIST  | 将位置元素与中心之间的距离也一并返回，单位和给定范围单位相同                                                |
    | WITHCOORD | 将位置元素的经度和维度也一并返回                                                                            |
    | WITHHASH  | 以52位无符号整数的形式返回原始geohash-encoded Sorted Set Score，主要用于底层应用或者调试，实际中作用不大 |
- 在默认情况下， `GEORADIUS` 命令会返回所有匹配的位置元素。虽然用户可以使用COUNT选项去获取前N个匹配元素，但是因为命令在内部可能会需要对所有被匹配的元素进行处理，所以在对一个非常大的区域进行搜索时，即使只使用COUNT选项去获取少量元素，命令的执行速度也可能会非常慢。但是从另一方面来说，使用COUNT选项去减少需要返回的元素数量，对于减少带宽来说仍然是非常有用的。
- 命令默认返回未排序的位置元素。通过以下两个参数，用户可以指定被返回位置元素的排序方式：

    | 选项 | 说明                                            |
    | ---- | ----------------------------------------------- |
    | 省缺 | 未排序                                          |
    | ASC  | 根据中心的位置， 按照从近到远的方式返回位置元素 |
    | DESC | 根据中心的位置， 按照从远到近的方式返回位置元素 |
- STORE和STOREDIST将返回值保持到Sorted Set中，分别以同WITHHASH和WITHDIST返回的额外值作为Score。
- 注意WITHDIST等返回额外信息的选项不可以和STORE，STOREDIST共用。

```
GEORADIUSBYMEMBER key member radius m|km|ft|mi
    [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count]
    [ASC|DESC] [STORE key] [STOREDIST key]
```
- 这个命令和 `GEORADIUS` 命令一样，都可以找出位于指定范围内的元素
- 但是中心点是由给定的元素决定的， 而不是使用输入的经度和纬度来决定中心点