- [Redis概述](#redis-概述)
  - [特点](#特点)
  - [优势](#优势)
  - [与其他key-value存储的不同](#与其他-key-value-存储的不同)
- [启动命令](#启动命令)
  - [服务器端](#服务器端)
  - [客户端](#客户端)
- [连接（Connection）](#连接connection)
  - [PING](#ping)
  - [ECHO](#echo)
  - [QUIT](#quit)
  - [AUTH](#auth)
  - [SELECT](#select)
  - [SWAPDB](#swapdb)
- [帮助（HELP）](#帮助help)
  - [无子命令](#无子命令)
  - [有子命令](#有子命令)

# Redis概述

Redis是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

## 特点

Redis与其他key - value缓存产品有以下三个特点：
- Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

## 优势

- 性能极高 –Redis能读的速度是110000次 /s, 写的速度是81000次 /s。
- 丰富的数据类型 –Redis支持二进制案例的Strings, Lists, Hashes, Sets及Ordered Sets数据类型操作。
- 原子 –Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
- 丰富的特性 –Redis还支持publish/subscribe, 通知，key过期等等特性。

## 与其他key-value存储的不同

- Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
- Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，应为数据量不能大于硬件内存。在内存数据库方面的另一个优点是， 相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。 同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

# 启动命令

## 服务器端

```
$ ./redis-server [/path/to/redis.conf] [options]
```
- 配置路径可选
- 常用选项

    | 选项              | 说明     |
    | ----------------- | -------- |
    | `--help`, `-h`    | 查看帮助 |
    | `--version`, `-v` | 查看版本 |

## 客户端

```
$ redis-cli [OPTIONS] [cmd [arg [arg ...]]]
```
- 常用选项

    | 选项            | 说明                                  |
    | --------------- | ------------------------------------- |
    | `--help`        | 查看帮助                              |
    | `--version`     | 查看版本                              |
    | `-h <hostname>` | 服务器主机名（127.0.0.1）               |
    | `-p <port>`     | 服务器端口（6379）                      |
    | `-s <socket>`   | 服务器套接字（127.0.0.1 6379）          |
    | `-a <password>` | 链接服务器的密码                      |
    | `-r <repeat>`   | 重复执行                              |
    | `-i <interval>` | 每次执行cmd时等待指定秒，可以为小数 |
    | `-n <db>`       | 指定数据库                            |
    | `-x`            | 从STDIN中读取参数。                 |
    - `-x` 示例：
        - 从 /etc/services读取并设为foo的value
            ```
            $ redis-cli -x set foo < /etc/services
            ```
        - 执行一系列cmd
            ```
            $ cat /tmp/commands.txt
            set foo 100
            incr foo
            append foo xxx
            get foo

            $ cat /tmp/commands.txt | redis-cli
            OK
            (integer) 101
            (integer) 6
            "101xxx"
            ```
- 可以直接在后面使用cmd执行操作

# 连接（Connection）

## PING

```
PING [message]
```
- 如果没有提供参数，则返回 "PONG"，否则将该参数的副本作为一个块返回。
- 此命令通常用于测试连接是否仍然处于活动状态，或测量延迟。

## ECHO

```
ECHO [message]
```
- 发送一个信息

## QUIT

```
QUIT
```
- 请求服务器关闭连接。
- 一旦将所有等待的答复写入客户机，连接就会关闭。

## AUTH

```
AUTH password
```
- 在受密码保护的Redis服务器中请求身份验证。可以指示Redis在允许客户端执行命令之前要求输入密码。这是使用配置文件中的requirepass指令完成的。
- 如果password与配置文件中的密码匹配，则服务器将以OK状态代码答复并开始接受命令。否则，将返回错误，并且客户端需要尝试新密码。
- 注意：由于Redis的高性能，可以在很短的时间内并行尝试许多密码，因此请确保生成一个很长且很长的密码，这样才能避免这种攻击。

## SELECT

```
SELECT index
```
- 选择具有指定的从0开始的数字索引的Redis逻辑数据库。
- 新连接总是使用数据库0。

## SWAPDB

```
SWAPDB index index
```
- 该命令可以交换同一Redis服务器上的两个DATABASE，可以实现连接某一数据库的连接立即访问到其他DATABASE的数据。
- 访问交换前其他database的连接也可以访问到该DATABASE的数据。

# 帮助（HELP）

## 无子命令

```
HELP command-name
```

## 有子命令

```
super-command-name HELP
```