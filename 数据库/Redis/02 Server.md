- [概述](#概述)
- [相关命令](#相关命令)
  - [BGREWRITEAOF](#bgrewriteaof)
  - [BGSAVE, SAVE](#bgsave-save)
  - [CLIENT](#client)
    - [CLIENT GETNAME](#client-getname)
    - [CLIENT ID](#client-id)
    - [CLIENT KILL](#client-kill)
    - [CLIENT LIST](#client-list)
    - [CLIENT PAUSE](#client-pause)
    - [CLIENT REPLY](#client-reply)
    - [CLIENT SETNAME](#client-setname)
    - [CLIENT UNBLOCK](#client-unblock)
  - [COMMAND](#command)
    - [COMMAND COUNT](#command-count)
    - [COMMAND GETKEYS](#command-getkeys)
    - [COMMAND INFO](#command-info)
  - [CONFIG](#config)
    - [CONFIG GET](#config-get)
    - [CONFIG RESETSTAT](#config-resetstat)
    - [CONFIG REWRITE](#config-rewrite)
    - [CONFIG SET](#config-set)
  - [DBSIZE](#dbsize)
  - [DEBUG](#debug)
    - [DEBUG OBJECT](#debug-object)
    - [DEBUG SEGFAULT](#debug-segfault)
  - [FLUSHDB, FLUSHALL](#flushdb-flushall)
  - [INFO](#info)
  - [LASTSAVE](#lastsave)
  - [LOLWUT](#lolwut)
  - [MEMORY](#memory)
    - [MEMORY DOCTOR](#memory-doctor)
    - [MEMORY MALLOC-STATS](#memory-malloc-stats)
    - [MEMORY PURGE](#memory-purge)
    - [MEMORY STATS](#memory-stats)
    - [MEMORY USAGE](#memory-usage)
  - [MODULE](#module)
    - [MODULE LIST](#module-list)
    - [MODULE LOAD](#module-load)
    - [MODULE UNLOAD](#module-unload)
  - [MONITOR](#monitor)
  - [PSYNC, SYNC](#psync-sync)
  - [REPLICAOF](#replicaof)
  - [ROLE](#role)
  - [SHUTDOWN](#shutdown)
  - [SLAVEOF](#slaveof)
  - [SLOWLOG](#slowlog)
  - [TIME](#time)

# 概述

Redis 服务器命令主要是用于管理 redis 服务。

# 相关命令

## BGREWRITEAOF

```
BGREWRITEAOF
```
- 用于异步执行一个 AOF（AppendOnly File）文件重写操作。重写会创建一个当前 AOF 文件的体积优化版本。
- 即使 `BGREWRITEAOF` 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 `BGREWRITEAOF` 成功之前不会被修改。
- AOF 重写由 Redis 自行触发， `BGREWRITEAOF` 仅仅用于手动触发重写操作。
- 具体内容:
    - 如果一个子 Redis 是通过磁盘快照创建的，AOF 重写将会在 RDB 终止后才开始保存。这种情况下 `BGREWRITEAOF` 任然会返回 OK 状态码。从 Redis 2.6 起你可以通过 `INFO` 命令查看 AOF 重写执行情况。
    - 如果只在执行的 AOF 重写返回一个错误，AOF 重写将会在稍后一点的时间重新调用。
    - 从 Redis 2.4 开始，AOF 重写由 Redis 自行触发，`BGREWRITEAOF` 仅仅用于手动触发重写操作。

## BGSAVE, SAVE

```
BGSAVE
```
- 后台保存 DB。会立即返回 OK 状态码。
- Redis forks, 父进程继续提供服务以供客户端调用，子进程将 DB 数据保存到磁盘然后退出。
- 如果操作成功，可以通过客户端命令 `LASTSAVE` 来检查操作结果。

```
SAVE
```
- 执行一个同步操作，以 RDB 文件的方式保存所有数据的快照很少在生产环境直接使用 SAVE 命令，因为它会阻塞所有的客户端的请求，可以使用 `BGSAVE` 命令代替。
- 如果在 `BGSAVE` 命令的保存数据的子进程发生错误的时，用 `SAVE` 命令保存最新的数据是最后的手段，详细的说明请参考 [Persistence](Advance/Persistence.md)。

## CLIENT

```
CLIENT <subcommand> arg [arg ...]
```

### CLIENT GETNAME

```
CLIENT GETNAME
```
- 返回当前连接由 `CLIENT SETNAME` 设置的名字。如果没有用 CLIENT SETNAME 设置名字，将返回一个空的回复。

### CLIENT ID

```
CLIENT ID
```
- 该命令返回当前连接的 ID，每个 ID 符合如下约束：
    - 永不重复。当调用命令 `CLIENT ID` 返回相同的值时，调用者可以确认原连接未被断开，只是被重用，因此仍可以认为是同一连接
    - ID 值单调递增。若某一连接的 ID 值比其他连接的 ID 值大，可以确认该连接是较新创建的
- 该命令和同为 Redis 5 新增的命令 `CLIENT UNBLOCK` 一起使用，会有更好的效果。

### CLIENT KILL

```
CLIENT KILL [ip:port] [ID client-id] [normal|slave|pubsub] [ADDR ip:port] [SKIPME yes/no]
```
- 关闭一个指定的连接。在 Redis2.8.11 时可以根据客户端地址关闭指定连接
- 选项：
    - `normal|slave|pubsub`，关闭所有指定类的客户端。请注意被认为是属于 normal 类的客户端将会被 MONITOR 命令监视到。
    - `SKIPME yes/no` 默认情况下，该选项设置为 yes，即调用该命令的客户端不会被杀死，但是将该选项设置为 no 也会杀死调用该命令的客户端
- 当前版本的 Redis Sentinel 可以在实例被重新配置的时候使用 `CLIENT KILL` 杀死客户端。这样可以强制客户端和一个 Sentinel 重新连接并更新自己的配置。
- 因为 Redis 的单线程属性，不可能在客户端执行命令时杀掉它。从客户端的角度看，永远无法杀死一个正在执行命令的连接。但是当客户端发送下一条命令时会意识到连接已被关闭，原因为网络错误。

### CLIENT LIST

```
CLIENT LIST
```
- 返回所有连接到服务器的客户端信息和统计数据。
- 每个已连接客户端对应一行（以 LF 分割），每行字符串由一系列 property=value 形式的字段（field）组成，每个字段之间以空格分开。
- 各字段含义

    | 字段名    | 说明                                                                                                 |
    | --------- | ---------------------------------------------------------------------------------------------------- |
    | id        | 唯一的 64 位的客户端 ID (Redis 2.8.12 加入)                                                          |
    | addr      | 客户端的地址和端口                                                                                   |
    | fd        | 套接字所使用的文件描述符                                                                             |
    | age       | 已连接时长（秒）                                                                                     |
    | idle      | 空闲时长（秒）                                                                                       |
    | flags     | 客户端 flag，normal，master，replica，pubsub 的首字母大写形式                                        |
    | db        | 该客户端正在使用的数据库 ID                                                                          |
    | sub       | 已订阅频道的数量                                                                                     |
    | psub      | 已订阅模式的数量                                                                                     |
    | multi     | 在事务中被执行的命令数量                                                                             |
    | qbuf      | 查询缓冲区的长度（字节， 0 表示没有被分配）                                                          |
    | qbuf-free | 查询缓冲区剩余空间的长度（字节， 0 表示没有剩余空间）                                                |
    | obl       | 输出缓冲区的长度（字节， 0 表示没有被分配）                                                          |
    | oll       | 输出列表包含的对象数量（当输出缓冲区没有剩余空间时，命令回复会以字符串对象的形式被入队到这个队列里） |
    | omem      | 输出缓冲区和输出列表占用的内存总量                                                                   |
    | events    | 文件描述符事件                                                                                       |
    | cmd       | 最近一次执行的命令                                                                                   |

### CLIENT PAUSE

```
CLIENT PAUSE timeout
```
- 连接控制命令，它可以将所有客户端的访问暂停给定的毫秒数
- 该命令执行如下：
    - 它会停止处理所有来自一般客户端或者 pub/sub 客户端的命令。但是和 slaves 的交互命令不受影响。
    - 因为它会尽快返回 OK 给调用者，所以 `CLIENT PAUSE` 不会被自己暂停。
    - 当给定的时间结束，所有的客户端都被解除阻塞：查询缓存里积累的所有命令都会被处理。
- 当该命令可以可控的将客户端从一个 Redis 实例切换至另一个实例。比如，当需要升级一个实例时，管理员可以作如下操作：
    1. 使用 `CLIENT PAUSE` 暂停所有客户端
    2. 等待数秒，让 slaves 节点处理完所有来自 master 的复制命令
    3. 将一个 salve 节点切换为 master
    4. 重配客户端以来接新的 master 节点
- 可以在 `MULTI`/`EXEC` 中一起使用 `CLIENT PAUSE` 和 `INFO replication` 以在阻塞的同时获取当前 master 的偏移量。用这种方法，可以让 slaves 处理至给定的复制偏移节点。

### CLIENT REPLY

```
CLIENT REPLY ON|OFF|SKIP
```
- 当需要完全禁用 redis 服务器对当前客户端的回复时可使用该命令。
- 在如下几种场景，此时消耗服务器时间和带宽回复客户端，是一种资源浪费。
    - 执行 fire and forget 类型的命令（fire and forget 就是发送命令，然后完全不关心最终什么时候完成命令操作）
    - 正在进行大量数据加载
    - 正在建缓存，数据在不断传输过程中，客户端会忽略收到的回复
- `CLIENT REPLY` 可设置服务器是否对客户端的命令进行回复。有如下选项：

    | 选项 | 说明                         |
    | ---- | ---------------------------- |
    | ON   | 默认选项，回复客户端每条命令 |
    | OFF  | 不回复客户端命令             |
    | SKIP | 跳过该命令的回复             |

### CLIENT SETNAME

```
CLIENT SETNAME connection-name
```
- 为当前连接分配一个名字。
- 这个名字会显示在 `CLIENT LIST` 命令的结果中， 用于识别当前正在与服务器进行连接的客户端。
- 举个例子， 在使用 Redis 构建队列（queue）时，可以根据连接负责的任务（role），为信息生产者（producer）和信息消费者（consumer）分别设置不同的名字。
- 名字使用 Redis 的字符串类型来保存， 最大可以占用 512 MB。另外，为了避免和 `CLIENT LIST` 命令的输出格式发生冲突，名字里不允许使用空格。
- 要移除一个连接的名字，可以将连接的名字设为空字符串。
- 新创建的连接默认是没有名字的。
- 提示：在 Redis 应用程序发生连接泄漏时，为连接设置名字是一种很好的 debug 手段。

### CLIENT UNBLOCK

```
CLIENT UNBLOCK client-id [TIMEOUT|ERROR]
```
- 当客户端因为执行具有阻塞功能的命令如 `BRPOP`、`XREAD` 或者 `WAIT` 被阻塞时，该命令可以通过其他连接解除客户端的阻塞
- 默认情况下，当命令设置的 timeout 超时时，客户端会被解除阻塞。当提供了额外 (可选) 的设置，可以确定解除阻塞的类型，可以是 TIMEOUT 或者 ERROR 类型。当时 ERROR 类型时，客户端阻塞被强制解除同时收到如下明确报错信息：
    ```
    -UNBLOCKED client unblocked via CLIENT UNBLOCK
    ```
    注意：当然，通常情况下错误信息不会完全一样，但是错误代码中一定包含 -UNBLOCKED 字样
- 这个命令，在仅能使用较少连接但要监控大量 keys 的时候特别有用。例如使用命令 `XREAD` 和很少连接监控多个消息流，在某个时间点，信息流消费进程需要新增一个消息流 key 的监控，为了避免使用更多连接，最好的方法是从连接池中停掉一个阻塞的连接，增加新的要监控的 key，在重启该阻塞命令即可。
- 我们使用如下操作流程来实现实现这种操作：
    1. 管理进程使用额外的连接 control connection 在必要时执行 `CLIENT UNBLOCK`
    2. 同时，在某一连接执行阻塞命令之前，进程执行 `CLIENT ID` 获取该连接的 ID 值
    3. 当需要监控一个新增的 key 或者取消一个 key 的监控时，通过在 control connection 中执行 `CLIENT UNBLOCK` + 上一步获取 ID 值来解除相关连接的阻塞
    4. 监控 key 调整后再次执行阻塞命令即可。
- 上述例子以 Redis streams 为例介绍了操作流程，该操作流程也可以应用到其他数据结构类型

## COMMAND

```
COMMAND
```
- 以数组的形式返回有关所有 Redis 命令的详细信息。
- 集群客户端必须知道命令中 key 的位置，以便命令可以转到匹配的实例，但是 Redis 命令在接收一个 key，多个 key 甚至由其他数据分隔开的多个 key 之间会有所不同。
- 你可以使用 `COMMAND` 来为每一个命令缓存命令和 key 位置之间的映射关系，以实现命令到集群的精确路由。
- 嵌套结果数组。每一个顶级结果包含了六个嵌套的结果。每一个嵌套结果是：
    - 命令名称：以小写字符串形式返回的命令
    - 命令元数规范，命令元数遵循一个简单的模式：
        - 正数：命令拥有固定数量的必需参数。例如，`GET` 的元数是 2，因为该命令仅接收一个参数，并且命令格式始终是 `GET _key_`。
        - 负数：命令拥有最小数量的必需参数，可以有更多的参数。例如，`MGET` 的元数是 -2，因为该命令接收至少一个参数，但最多可以接收无限数量：`MGET _key1_ [key2] [key3] ...`。
    - 命令标志：是包含一个或多个状态回复的 array-reply

        | 标志            | 说明                                       |
        | --------------- | ------------------------------------------ |
        | write           | 命令可能会导致修改                         |
        | readonly        | 命令永远不会修改键                         |
        | denyoom         | 如果当前发生 OOM，则拒绝该命令             |
        | admin           | 服务器管理命令                             |
        | pubsub          | 发布订阅相关的命令                         |
        | noscript        | 在脚本中将会拒绝此命令                     |
        | random          | 命令具有随机结果，在脚本中使用很危险       |
        | sort_for_script | 如果从脚本调用，则排序输出                 |
        | loading         | 允许在数据库加载时使用此命令               |
        | stale           | 允许在从节点具有陈旧数据时使用此命令       |
        | skip_monitor    | 在 MONITOR 中不会显示此命令                |
        | asking          | 集群相关，即使正在导入数据也接受此命令     |
        | fast            | 命令以常量或 log(N) 时间运行。用于延迟监控 |
        | movablekeys     | 在命令中没有预先确定 key 的位置            |
    - 参数列表中第一个 key 的位置：对大部分命令来说，第一个 key 的位置是 1。位置 0 始终是命令名称本身。
    - 参数列表中最后一个 key 的位置
        - Redis 命令通常可以接收一个 key，两个 key 或者无限数量的 key。
        - 如果命令只接收一个 key，那么第一个 key 和最后一个 key 的位置都是 1。
        - 如果命令接收两个 key（例如：`BRPOPLPUSH`、`SMOVE`、`RENAME` 等），那么最后一个 key 的位置是最后一个 key 在参数列表中的位置。
        - 如果命令接收无限数量的 key，那么最后一个 key 的位置是 - 1。
    - 用于定位重复 key 的步数
        - Key 的步数允许我们在命令中查找 key 的位置，比如 `MSET`，其格式是 `MSET _key1_ _val1_ [key2] [val2] [key3] [val3]`...。
        - 在 `MSET` 的用例中，key 是每隔一个位置出现，所以步数的值是 2。对比 `MGET`，其步数是 1。

### COMMAND COUNT

```
COMMAND COUNT
```
- 返回 Redis 服务器命令的总数

### COMMAND GETKEYS

```
COMMAND GETKEYS command-name arg [arg ...]
```
- 以 array-reply 的形式从完整的 Redis 命令返回 key。
- `COMMAND GETKEYS` 是一个辅助命令，让你可以从完整的 Redis 命令中找到 key。
- `COMMAND` 显示了某些命令拥有可变位置的 key，这意味着必须分析完整的命令才能找到要存储或者检索的 key。
- 你可以使用 `COMMAND GETKEYS` 直接从 Redis 解析命令的方式来发现 key 的位置。
- 示例：
    ```
    127.0.0.1:6379> COMMAND GETKEYS MSET a b c d e f
    1) "a"
    2) "c"
    3) "e"
    ```

### COMMAND INFO

```
COMMAND INFO command-name [command-name ...]
```
- 以 array-reply 的形式返回多个 Redis 命令的详细信息。
- 此命令返回的结果与 `COMMAND` 相同，但是你可以指定返回哪些命令。
- 如果你指定了一些不存在的命令，那么在它们的返回位置将会是 nil。

## CONFIG

- 配置相关。
- 配置参数说明：

    | 配置                              | 取值 & 默认值                    | 说明                                                                                                                                                                                                                                                                     |
    | --------------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | daemonize                         | yes, no*                         | 是否以以守护进程的方式运行                                                                                                                                                                                                                                               |
    | pidfile                           |                                  | 当 Redis 以守护进程方式运行时会把 pid 指定文件                                                                                                                                                                                                                           |
    | port                              | 6379                             | Redis 监听端口                                                                                                                                                                                                                                                           |
    | bind                              | 127.0.0.1                        | 绑定的主机地址                                                                                                                                                                                                                                                           |
    | timeout                           | 0                                | 客户端闲置多长时间后关闭连接。0 表示关闭该功能                                                                                                                                                                                                                           |
    | loglevel                          | debug, verbose, notice*, warning | 日志记录级别                                                                                                                                                                                                                                                             |
    | logfile                           | stdout（标准输出）               | 日志记录路径                                                                                                                                                                                                                                                             |
    | databases                         | 16                               | 数据库的数量，默认数据库为 0，可使用 SELECT 命令更变                                                                                                                                                                                                                     |
    | save <seconds> <changes>          | 3600 1 300 100 60 10000          | 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合                                                                                                                                                                                             |
    | rdbcompression                    | yes*, no                         | 存储至本地数据库时是否压缩数据，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大                                                                                                                                                   |
    | dbfilename                        | dump.rdb                         | 指定本地数据库文件名                                                                                                                                                                                                                                                     |
    | dir                               |                                  | 指定本地数据库存放目录                                                                                                                                                                                                                                                   |
    | slaveof \<masterip> \<masterport> |                                  | 设置当本机为 slav 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步                                                                                                                                                              |
    | masterauth                        |                                  | 当 master 服务设置了密码保护时，slav 服务连接 master 的密码                                                                                                                                                                                                              |
    | requirepass                       |                                  | 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH 命令提供密码                                                                                                                                                                                 |
    | maxclients                        | 4064                             | 设置同一时间最大客户端连接数，0 表示不作限制。当客户端连接数到达限制时                                                                                                                                                                                                   |
    | maxmemory                         | 0                                | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区 |
    | appendonly                        | no                               | 指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no                           |
    | appendfsync                       | no, always, everysec*            | 指定更新日志条件。no 表示等操作系统进行数据缓存同步到磁盘（快），always 表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全），everysec：表示每秒同步一次（折衷，默认值）                                                                                        |
    | activerehashing                   | yes*, no                         | 指定是否激活重置哈希                                                                                                                                                                                                                                                     |
- Redis 版本：redis-cli 5.0.5。不同版本之间可能存在差异
- 取值 & 默认值中 "*" 或只写了单个取值的表示默认值

### CONFIG GET

```
CONFIG GET parameter
```
- `CONFIG GET` 命令用来读取 redis 服务器的配置文件参数，但并不是所有参数都支持。 与之对应的命令是 CONFIG SET 用来设置服务器的配置参数。
- `CONFIG GET` 命令只接受一个参数，所有配置参数都采用 key-value 的形式。
- 通过 `CONFIG GET *` 可以查看所有支持的参数。
- 所有支持的参数都与 redis.conf 里面的一样，除了如下的重要差异：
    - 如果指定了 byte 或其他单位，则不能使用 redis.conf 的缩写形式（10k, 2gb ...），应在配置指令的基本单元中将所有内容指定为格式良好的 64 位整数。
    - save 参数是由空格分隔的整数组成的单个字符串。每对整数代表一个秒/修改阈值。例如，如果数据集有 1 个以上变更，则在 900 秒后保存；如果有 10 个以上变更，则在 300 秒后就保存：
        - 在 redis.conf 中格式为
            ```
            save 900 1
            save 300 10
            ```
        - 在 `CONFIG GET save` 返回值为
            ```
            900 1 300 10
            ```

### CONFIG RESETSTAT

```
CONFIG RESETSTAT
```
- 重置使用 `INFO` 命令 Redis 报告的统计信息。

### CONFIG REWRITE

```
CONFIG REWRITE
```
- 重写服务器启动时指定的 redis.conf 文件，应用所需的最小更改来反应服务器当前使用的配置，由于使用了 `CONFIG SET`，因此与原始配置相比可能有所不同。
- 重写以非常保守的方式执行：
    - 注释和原始 redis.conf 文件的整体结构会尽可能的保留下来。
    - 如果一个选项在旧的 redis.conf 文件中已经存在，那么它会在相同的位置（行号）被重写。
    - 如果某个选项在配置文件中尚不存在，但被设置为了该选项的默认值，那么他将不会被重写进程写入配置文件。
    - 如果某个选项在配置文件中尚不存在，但被设置了一个非默认值，那么它会被追加到文件的末尾。
    - 未使用的行将会留空。例如，如果你之前在配置文件中有多个 save 配置项，但由于你禁用了 RDB 持久化，当前的 save 配置变少了或者变为空，那么所有的那些行将会是空行。
    - 如果原始文件由于某些原因不再存在，`CONFIG REWRITE` 也能够从头开始重写配置文件。但是，如果服务器启动的时候没有指定任何配置文件，则只会返回一个错误。
- 原子重写过程
    - 为了保证 redis.conf 文件始终是一致的，也即，在异常或者崩溃的时候，你的配置文件要么是旧的文件，或者是重写完的新文件。
    - 重写是通过一次具有足够内容的 `write(2)` 调用来执行的，至少和旧的文件一样大。
    - 有时会以注释的形式添加额外的 padding，以确保生成的文件足够大，稍后文件会被截断以删除末尾的 padding。

### CONFIG SET

```
CONFIG SET parameter value
```
- 用于在服务器运行期间重写某些配置，而不用重启 Redis。你可以使用此命令更改不重要的参数或从一个参数切换到另一个持久性选项。
- 所有使用 `CONFIG SET` 设置的配置参数将会立即被 Redis 加载，并从下一个执行的命令开始生效。
- 可以使用 `CONFIG SET` 命令将持久化从 RDB 快照切换到 AOF 文件（或其他相似的方式）。
- 一般来说，你应该知道将 appendonly 参数设置为 yes 将启动后台进程以保存初始 AOF 文件（从内存数据集中获取），并将所有后续命令追加到 AOF 文件，从而达到了与一个 Redis 服务器从一开始就开启了 AOF 选项相同的效果。
- 如果你愿意，可以同时开启 AOF 和 RDB 快照，这两个选项不是互斥的。

## DBSIZE

```
DBSIZE
```
- 返回当前数据里面 keys 的数量。

## DEBUG

###  DEBUG OBJECT

```
DEBUG OBJECT key
```
- 一个不应该被客户端使用的调试命令。
- 具体参考 Key 的 `OBJECT` 命令。

### DEBUG SEGFAULT

```
DEBUG SEGFAULT
```
- 执行在崩溃的 Redis 一个无效的内存访问，它是用来模拟在开发过程中的错误。

## FLUSHDB, FLUSHALL

```
FLUSHDB
```
- 删除当前数据库里面的所有数据。
- 这个命令永远不会出现失败。
- 这个操作的时间复杂度是 O(N),N 是当前数据库的 keys 数量。

```
FLUSHALL
```
- 删除所有数据库里面的所有数据，注意不是当前数据库，而是所有数据库。
- 这个命令永远不会出现失败。
- 这个操作的时间复杂度是 O(N)，N 是数据库的数量。

## INFO

```
INFO [section]
```
- 以一种易于理解和阅读的格式，返回关于 Redis 服务器的各种信息和统计数值。
- 通过给定可选的参数 section ，可以让命令只返回某一部分的信息:

    | section      | 说明                   |
    | ------------ | ---------------------- |
    | server       | Redis 服务器的一般信息 |
    | clients      | 客户端的连接部分       |
    | memory       | 内存消耗相关信息       |
    | persistence  | RDB 和 AOF 相关信息    |
    | stats        | 一般统计               |
    | replication  | 主 / 从复制信息        |
    | cpu          | 统计 CPU 的消耗        |
    | commandstats | Redis 命令统计         |
    | cluster      | Redis 集群信息         |
    | keyspace     | 数据库的相关统计       |
    | all          | 返回所有信息           |
    | default      | 值返回默认设置的信息   |

## LASTSAVE

```
LASTSAVE
```
- 执行成功时返回 UNIX 时间戳。
- 客户端执行 `BGSAVE` 命令时，可以通过每 N 秒发送一个 `LASTSAVE` 命令来查看 `BGSAVE` 命令执行的结果，由 `LASTSAVE` 返回结果的变化可以判断执行结果。

## LOLWUT

```
LOLWUT [VERSION version]
```
- `LOLWUT` 命令显示了 Redis 版本：但是作为这样做的一个副作用，它也创建了一个生成计算机艺术，每个版本的 Redis 都是不同的。
- 这个命令是在 Redis 5 中引入的，并在本文中发布。
- 默认情况下，`LOLWUT` 命令将显示与当前 Redis 版本对应的片段，但是可以使用以下格式显示特定的版本
    ```
    LOLWUT VERSION 5 ... other optional arguments ...
    ```
- 上面的“5”就是一个例子。每个 `LOLWUT` 版本都采用一组不同的参数来更改输出。用户可以通过添加更多的数值参数来了解输出是如何变化的。

## MEMORY

### MEMORY DOCTOR

```
MEMORY DOCTOR
```
- 列出 Redis 服务器遇到的不同类型的内存相关问题，并提供相应的解决建议。

### MEMORY MALLOC-STATS

```
MEMORY MALLOC-STATS
```
- 提供内存分配情况的内部统计报表
- 该命令目前仅实现了 jemalloc 作为内存分配器的内存统计，对其他分配器暂不支持

### MEMORY PURGE

```
MEMORY PURGE
```
- 命令 MEMORY PURGE 尝试清除脏页以便内存分配器回收使用
- 该命令目前仅实现了 jemalloc 作为内存分配器的内存统计，对其他分配器暂不支持

### MEMORY STATS

```
MEMORY STATS
```
- 将服务器的内存使用情况以数组情况返回
- 内存使用信息以指标和相对应值的格式返回，如下指标会被返回
- 返回值详见 [MEMORY STATS](http://redis.cn/commands/memory-stats.html)

### MEMORY USAGE

```
MEMORY USAGE key [SAMPLES count]
```
- 给出一个 key 和它值在 RAM 中占用的字节数
- 返回的结果是 key 的值以及为管理该 key 分配的内存总字节数
- 对于嵌套数据类型，可以使用选项 SAMPLES，其中 count 表示抽样的元素个数，默认值为 5。
- 当需要抽样所有元素时，使用 SAMPLES 0

## MODULE

### MODULE LIST

```
MODULE LIST
```
- 返回有关加载到服务器的模块的信息。

### MODULE LOAD

```
MODULE LOAD path [ arg [arg ...]]
```
- 在运行时从动态库加载模块。
- 该命令从 path 参数指定的动态库中加载并初始化 Redis 模块。
- 该路径应该是库的绝对路径，包括完整的文件名。
- 任何其他参数都将原封不动地传递给模块。
- 注意：也可以在服务器启动时使用 redis.conf 中的 loadmodule 配置指令来加载模块。

### MODULE UNLOAD

```
MODULE UNLOAD name
```
- 卸载一个模块。
- 此命令卸载按名称指定的模块。注意，模块的名称是由 `MODULE LIST` 命令报告的，可能与动态库的文件名不同。
- 已知的限制：无法卸载注册自定义数据类型的模块。

## MONITOR

```
MONITOR
```
- 一个调试命令，返回服务器处理的每一个命令，它能帮助我们了解在数据库上发生了什么操作，可以通过 redis-cli 和 telnet 命令使用
    ```
    $ redis-cli monitor
    ```
- 使用 SIGINT (Ctrl-C) 来停止 通过 redis-cli 使用 `MONITOR` 命令返回的输出
- 使用 `QUIT` 命令来停止通过 telnet 使用 `MONITOR` 返回的输出
- 性能消耗：由于 `MONITOR` 命令返回服务器处理的所有的命令，所以在性能上会有一些消耗

## PSYNC, SYNC

```
PSYNC replicationid offset
```
- 从主服务器启动一个复制流。
- 为了从主服务器启动复制流，Redis 副本调用 `PSYNC` 命令。

```
SYNC
```
- 从主服务器启动一个复制流。
- Redis replicas 调用 `SYNC` 命令来启动来自主服务器的复制流。
- 在较新的 Redis 版本中，它已被 `PSYNC` 取代。
- 有关在 Redis 复制的更多信息，请检查 [Replication](Advance/Replication.md) 页面。

## REPLICAOF

```
REPLICAOF host port
```
- 在线修改当前服务器的复制设置。
- 如果当前服务器已经是副本服务器，命令 `REPLIACOF NO ONE` 会关闭当前服务器的复制并转变为主服务器。执行 `REPLIACOF hostname port` 会将当前服务器转变为某一服务器的副本服务器。
- 如果当前服务器已经是某个主服务器 (master server) 的副本服务器，那么执行 `REPLICAOF hostname port` 将使当前服务器停止对原主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。
- 对一个副本服务器执行命令 `REPLICAOF NO ONE` 将使得这个副本服务器关闭复制，并从副本服务器转变回主服务器，原来同步所得的数据集不会被丢弃。因此，当原主服务器停止服务，可以将该副本服务器切换为主服务器，应用可以使用新主服务器进行读写。原主服务器修复后，可将其设置为新主服务器的副本服务器。

## ROLE

```
ROLE
```
- 通过返回实例当前是 master，slave 还是 sentinel 来提供有关 Redis 实例在复制环境中的角色的信息。
- 此命令还返回有关复制状态（如果角色是 master 或者 slave）或者监听的 master 名称列表（如果角色是 sentinel）的额外信息。

## SHUTDOWN

```
SHUTDOWN [NOSAVE] [SAVE]
```
- 这个命令执行如下操作:
    - 停止所有客户端。
    - 如果配置了 save 策略则执行一个阻塞的 save 命令。
    - 如果开启了 AOF, 则刷新 aof 文件。
    - 关闭 redis 服务进程（redis-server）。
- 如果配置了持久化策略，那么这个命令将能够保证在关闭 redis 服务进程的时候数据不会丢失。如果仅仅在客户端执行 `SAVE` 命令，然后执行 `QUIT` 命令，那么数据的完整性将不会被保证，因为其他客户端可能在执行这两个命令的期间修改数据库的数据。
- 注意：一个没有配置持久化策略的 redis 实例 (没有 aof 配置，没有 `SAVE` 命令) 将不会在执行 `SHUTDOWN` 命令的时候转存一个 rdb 文件，通常情况下你不想让一个仅用于缓存的 rendis 实例宕掉。
- SAVE 和 NOSAVE 修饰符：通过指定一个可选的修饰符可以改变这个命令的表现形式，比如:
    - SHUTDOWN SAVE 能够在即使没有配置持久化的情况下强制数据库存储。
    - SHUTDOWN NOSAVE 能够在配置一个或者多个持久化策略的情况下阻止数据库存储。(你可以假想它为一个中断服务的 `ABORT` 命令)
返回值
- 当发生错误的时候返回状态码。当成功的时候不返回任何值，服务退出，连接关闭。

## SLAVEOF

```
SLAVEOF host port
```
- 可以将当前服务器转变为指定服务器的从属服务器 (slave server)。
- 如果当前服务器已经是某个主服务器 (master server) 的从属服务器，那么执行 `SLAVEOF host port` 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。
- 另外，对一个从属服务器执行命令 `SLAVEOF NO ONE` 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集不会被丢弃。
- 利用『 `SLAVEOF NO ONE` 不会丢弃同步所得数据集』这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。

## SLOWLOG

```
SLOWLOG <subcommand> [arg]
```
- 用于读取和重置 Redis 慢查询日志。
- 详见 [SLOWLOG](http://redis.cn/commands/slowlog.html)

## TIME

```
TIME
```
- 返回当前 Unix 时间戳和这一秒已经过去的微秒数。
- 基本上，该接口非常类似 gettimeofday 系统调用。