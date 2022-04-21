- [概述](#概述)
- [相关命令](#相关命令)
  - [CLUSTER](#cluster)
    - [CLUSTER ADDSLOTS](#cluster-addslots)
    - [CLUSTER BUMPEPOCH](#cluster-bumpepoch)
    - [CLUSTER COUNT-FAILURE-REPORTS](#cluster-count-failure-reports)
    - [CLUSTER COUNTKEYSINSLOT](#cluster-countkeysinslot)
    - [CLUSTER DELSLOTS](#cluster-delslots)
    - [CLUSTER FAILOVER](#cluster-failover)
    - [CLUSTER FLUSHSLOTS](#cluster-flushslots)
    - [CLUSTER FORGET](#cluster-forget)
    - [CLUSTER GETKEYSINSLOT](#cluster-getkeysinslot)
    - [CLUSTER INFO](#cluster-info)
    - [CLUSTER KEYSLOT](#cluster-keyslot)
    - [CLUSTER MEET](#cluster-meet)
    - [CLUSTER MYID](#cluster-myid)
    - [CLUSTER NODES](#cluster-nodes)
    - [CLUSTER REPLICAS](#cluster-replicas)
    - [CLUSTER REPLICATE](#cluster-replicate)
    - [CLUSTER RESET](#cluster-reset)
    - [CLUSTER SAVECONFIG](#cluster-saveconfig)
    - [CLUSTER SET-CONFIG-EPOCH](#cluster-set-config-epoch)
    - [CLUSTER SETSLOT](#cluster-setslot)
    - [CLUSTER SLAVES](#cluster-slaves)
    - [CLUSTER SLOTS](#cluster-slots)
  - [READONLY](#readonly)
  - [READWRITE](#readwrite)

# 概述

参见 [Cluster](Advance/Cluster.md)

# 相关命令

## CLUSTER

### CLUSTER ADDSLOTS

```
CLUSTER ADDSLOTS slot [slot ...]
```
- 用于修改某个节点上的集群配置。具体的说它把一组哈希槽（hash slots）分配给接收命令的节点。
- 如果命令执行成功，节点将指定的哈希槽映射到自身，节点将获得指定的哈希槽，同时开始向集群广播新的配置。
- 需要注意：
    - 该命令只有当所有指定的 slots 在接收命令的节点上还没有分配得的情况下生效。节点将拒绝接纳已经分配到其他节点的 slots（包括它自己的）。
    - 同一个 slot 被指定多次的情况下命令会失败。
    - 执行这个命令有一个副作用，如果 slot 作为其中一个参数设置为 importing，一旦节点向自己分配该 slot（以前未绑定）这个状态将会被清除。

### CLUSTER BUMPEPOCH

```
CLUSTER BUMPEPOCH
```
- 推进集群配置历元（epoch）。
- 命令从连接节点触发集群配置历元的增量。如果节点的配置历元为 0，或者小于集群的最大历元，则历元将增加。
- 注意:配置历元管理由集群内部执行，依赖于获得节点的一致同意。
- `CLUSTER BUMPEPOCH` 试图增加配置历元，但没有得到一致同意，因此使用它可能违反“最后的故障转移胜利”（last failover wins）规则。小心使用。

### CLUSTER COUNT-FAILURE-REPORTS

```
CLUSTER COUNT-FAILURE-REPORTS node-id
```
- 返回指定节点的故障报告（failure report）个数，故障报告是 Redis Cluster 用来使节点的 PFAIL 状态（节点不可达）晋升到 FAIL 状态（集群中的大多数主机都同意在一段时间内无法访问节点）而的方式。
- 更多细节：
    - 一个节点会用 PFAIL 标记一个不可达时间超过配置中的超时时间的节点，这个超时时间是 Redis Cluster 配置中的基本选项。
    - 处于 PFAIL 状态的节点会将状态信息提供在心跳包的流言（gossip）部分。
    - 每当一个节点处理来自其他节点的流言（gossip）包时，该节点会建立故障报告（如果需要会刷新 TTL），并且会记住发送消息包的节点所认为处于 PFAIL 状态下的其他节点。
    - 每个故障报告的生存时间是节点超时时间的两倍。
    - 如果在一段给定的事件内，一个节点被另一个节点标记为 PFAIL 状态，并且在相同的时间内收到了其他大多数主节点关于该节点的故障报告（如果该节点是主节点包括它自己），那么该节点的故障状态会从 PFAIL 晋升为 FAIL，并且会广播一个消息，强制所有可达的节点将该节点标记为 FAIL。
- 该命令返回当前节点没有过期的故障报告个数（在两倍的节点超时时间收到的）。该计数值不包含当前节点，该节点是我们要求这个计数值是以我们作为参数所传递的 ID 的节点，这个计数值只包含该节点从其他节点接收到的故障报告。
- 当 Redis Cluster 的故障检测器不能正常工作时，这个命令主要用来调试。

### CLUSTER COUNTKEYSINSLOT

```
CLUSTER COUNTKEYSINSLOT slot
```
- 返回指定哈希槽中的 key 的数量。
- 该命令只查询本地数据集，所以连接到一个不服务指定哈希槽的节点总是返回 0。

### CLUSTER DELSLOTS

```
CLUSTER DELSLOTS slot [slot ...]
```
- 在 Redis Cluster 中，每个节点都会知道哪些主节点正在负责哪些特定的哈希槽。
- `DELSLOTS` 命令使一个特定的 Redis Cluster 节点去忘记（forget）一个主节点正在负责的哈希槽，这些哈希槽通过参数指定。
- 在已经接收到 `DELSLOTS` 命令的节点环境中，并且因此已经去除了指定哈希槽的关联，我们认为这些哈希槽是未绑定的。
- 请注意，当一个节点还没有被配置去负责他们（可以通过 `ADDSLOTS` 完成槽的分配）并且如果该节点没有收到关于谁拥有这些哈希槽的消息时（节点通过心跳包或者更新包获取消息），这些未绑定的哈希槽是自然而然本来就存在的。
- 如果一个节点认为一些哈希槽是未绑定的，但是从其他节点接收到一个心跳包，得知这些哈希槽已经被其他节点负责，那么会立即确立其关联关系。而且，如果接收到一个心跳包或更新包的配置纪元比当前节点的大，那么会重新建立关联。
- 但是，请注意：
    - 命令只在参数指定的哈希槽已经和某些节点关联时有效。
    - 如果同一个哈希槽被指定多次，该命令会失败。
    - 命令执行的副作用是，因为不在负责哈希槽，节点可能会进入下线状态。

### CLUSTER FAILOVER

```
CLUSTER FAILOVER [FORCE|TAKEOVER]
```
- 该命令只能在群集 slave 节点执行，让 slave 节点进行一次人工故障切换。
- 人工故障切换是预期的操作，而非发生了真正的故障，目的是以一种安全的方式 (数据无丢失) 将当前 master 节点和其中一个 slave 节点 (执行 `CLUSTER FAILOVER` 的节点) 交换角色。流程如下：
    1. 当前 slave 节点告知其 master 节点停止处理来自客户端的请求
    2. master 节点将当前 replication offset 回复给该 slave 节点
    3. 该 slave 节点在未应用至 replication offset 之前不做任何操作，以保证 master 传来的数据均被处理
    4. 该 slave 节点进行故障转移，从群集中大多数的 master 节点获取 epoch，然后广播自己的最新配置
    5. 原 master 节点收到配置更新：解除客户端的访问阻塞，回复重定向信息，以便客户端可以和新 master 通信
- 当该 slave 节点 (将切换为新 master 节点) 处理完来自 master 的所有复制，客户端的访问将会自动由原 master 节点切换至新 master 节点。
- master 节点 down 的情况下的人工故障转移
    - FORCE：slave 节点不和 master 协商 (master 也许已不可达)，从上如 4 步开始进行故障切换。当 master 已不可用，而我们想要做人工故障转移时，该选项很有用。
    - TAKEOVER：忽略群集一致验证的的人工故障切换。实现了 FORCE 的所有功能，同时为了能够进行故障切换放弃群集验证。
    - 详见 [CLUSTER FAILOVER [FORCE|TAKEOVER]](http://redis.cn/commands/cluster-failover.html)

### CLUSTER FLUSHSLOTS

```
CLUSTER FLUSHSLOTS
```
- 从节点中删除所有插槽。
- 从连接节点清除所有关于槽的信息。它只能在数据库为空时调用。

### CLUSTER FORGET

```
CLUSTER FORGET node-id
```
- 该命令可以从收到命令的 Redis 群集节点的节点信息列表中移除指定 ID 的节点。换句话说，从收到命令的 Redis 群集节点的节点表（nodes table）中删除指定节点。
- 该命令不是将待删除节点的信息简单从内部配置中简单删除，它同时实现了禁止列表功能：不允许已删除的节点再次被添加进来，否则已删除节点会因为处理其他节点心跳包中的 gossip section 时被再次添加。
- 命令执行详细：假设我们有四个节点：A,B,C,D。为了得到一个三节点群集 A,B,C，我们可以做如下操作：
    1. 将 D 上的哈希槽重分配到节点 A,B,C。
    2. 节点 D 现在已经空了，但是节点 A,B,C 的节点信息表中仍然有 D 的信息.
    3. 我们连接节点 A，发送命令 CLUSTER FORGET D。
    4. 节点 B 发送心跳包给节点 A，包含节点 D 的信息。
    5. 节点 A 无节点 D 信息，无法识别节点 D (参见步骤 3)，因此开始与节点 D 握手。
    6. 节点 D 最终再次添加进节点 A 的节点信息表中.
- 上述的移除方法很不稳定，因此我们需要尽快发送命令 `CLUSTER FORGET` 给所有节点，以期没有 gossip sections 在同时处理。因为这个原因，命令 `CLUSTER FORGET` 为每个节点实现了包含超时时间的禁止列表，因此我们命令实际的执行情况如下：
    1. 从收到命令节点的节点信息列表中删除待删除节点的节点信息。
    2. 已删除的节点的节点 ID 被加入禁止列表，保留 1 分钟
    3. 收到命令的节点，在处理从其他节点发送过来的 gossip sections 会跳过所有在禁止列表中的节点。
- 这样，我们就有 1 分钟的时间窗口来通知群集中的所有节点，我们想要删除某个节点。
- 在如下情况下，该命令无法成功执行并返回错误
    - 节点信息表中无法找到指定删除节点的节点信息
    - 收到命令的节点是 slave 节点，指定要删除的节点被识别出是它的当前 master 节点。
    - 收到命令的节点和待删除的节点是同一节点

### CLUSTER GETKEYSINSLOT

```
CLUSTER GETKEYSINSLOT slot count
```
- 本命令返回存储在连接节点的指定哈希槽里面的 key 的列表。key 的最大数量通过 count 参数指定，所以这个 API 可以用作 keys 的批处理。
- 这个命令的主要是用于 rehash 期间 slot 从一个节点移动到另外一个节点。集群 rehash 的具体做法在 Redis 集群规范文档，或者你可以查询 `CLUSTER SETSLOT` 命令文档的附录。

### CLUSTER INFO

```
CLUSTER INFO
```
- 使用 INFO 风格的形式展现了关于 Redis 集群的重要参数。
- 输出内容说明：

    | 内容                            | 说明                                                                                                                                                                                                               |
    | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | cluster_state                   | ok 状态表示集群可以正常接受查询请求；fail 状态表示至少有一个哈希槽没有被绑定（说明有哈希槽没有被绑定到任意一个节点），或者在错误的状态（节点可以提供服务但是带有 FAIL 标记），或者该节点无法联系到多数 master 节点 |
    | cluster_slots_assigned          | 已分配到集群节点的哈希槽数量（不是没有被绑定的数量）。哈希槽全部被分配到集群节点是集群正常运行的必要条件                                                                                                           |
    | cluster_slots_ok                | 哈希槽状态不是 FAIL 和 PFAIL 的数量                                                                                                                                                                                |
    | cluster_slots_pfail             | 哈希槽状态是 PFAIL 的数量。只要哈希槽状态没有被升级到 FAIL 状态，这些哈希槽仍然可以被正常处理。PFAIL 状态表示我们当前不能和节点进行交互，但这种状态只是临时的错误状态                                              |
    | cluster_slots_fail              | 哈希槽状态是 FAIL 的数量。如果值不是 0，那么集群节点将无法提供查询服务，除非 cluster-require-full-coverage 被设置为 no                                                                                             |
    | cluster_known_nodes             | 集群中节点数量，包括处于握手状态还没有成为集群正式成员的节点                                                                                                                                                       |
    | cluster_size                    | 至少包含一个哈希槽且能够提供服务的 master 节点数量                                                                                                                                                                 |
    | cluster_current_epoch           | 集群本地 Current Epoch 变量的值。这个值在节点故障转移过程时有用，它总是递增和唯一的                                                                                                                                |
    | cluster_my_epoch                | 当前正在使用的节点的 Config Epoch 值。这个是关联在本节点的版本值                                                                                                                                                   |
    | cluster_stats_messages_sent     | 通过 node-to-node 二进制总线发送的消息数量                                                                                                                                                                         |
    | cluster_stats_messages_received | 通过 node-to-node 二进制总线接收的消息数量                                                                                                                                                                         |
- 更多关于 Current Epoch 和 Config Epoch 变量的说明，请参考 Redis 集群规范文档。

### CLUSTER KEYSLOT

```
CLUSTER KEYSLOT key
```
- 返回一个整数，用于标识指定键所散列到的哈希槽。该命令主要用来调试和测试，因为它通过一个 API 来暴露 Redis 底层哈希算法的实现。
- 该命令的使用示例：
    - 客户端库可能会使用 Redis 来测试他们自己的哈希算法，生成随机的键并且使用他们自己实现的算法和 Redis 的 `CLUSTER KEYSLOT` 命令来散列这些键，然后检查结果是否相同。
    - 人们会使用这个命令去检查哈希槽是哪个，然后关联 Redis Cluster 的节点，并且负责一个给定的键。
- 例如
    ```
    > CLUSTER KEYSLOT somekey
    11058
    > CLUSTER KEYSLOT foo{hash_tag}
    (integer) 2515
    > CLUSTER KEYSLOT bar{hash_tag}
    (integer) 2515
    ```
    注意该命令实现了完整的哈希算法，包括支持 hash tags，这是 Redis Cluster 键一个特殊的哈希算法，如果键名中存在左右大括号的模式，只会散列在 '{' 和 '}' 之间的字符串，为了去强制将多个键由一个节点来处理。

### CLUSTER MEET

```
CLUSTER MEET ip port
```
- 用来连接不同的开启集群支持的 Redis 节点，以进入工作集群。
- 基本的思想是每个节点默认都是相互不信任的，并且被认为是未知的节点，以便万一因为系统管理错误或地址被修改，而不太可能将多个不同的集群节点混成一个集群。
- 因此，为了使给定的节点能将另一个节点接收到组成 Redis Cluster 的节点列表中，这里只有两种方法：
    - 系统管理员发送一个 `CLUSTER MEET` 命令强制一个节点去会面另一个节点。
    - 一个已知的节点发送一个保存在 gossip 部分的节点列表，包含着未知的节点。如果接收的节点已经将发送节点信任为已知节点，它会处理 gossip 部分并且发送一个握手消息给未知的节点。
- 请注意，Redis Cluster 需要形成一个完整的网络（每个节点都连接着其他每个节点），但是为了创建一个集群，不需要发送形成网络所需的所有 `CLUSTER MEET` 命令。发送 `CLUSTER MEET` 消息以便每个节点能够到达其他每个节点只需通过一条已知的节点链就足够了。由于在心跳包中会交换 gossip 信息，将会创建节点间缺失的链接。
- 所以，如果我们通过 `CLUSTER MEET` 链接节点 A 和节点 B，并且节点 B 和 C 有链接，那么节点 A 和节点 C 会发现他们握手和创建链接的方法。
- 实现细节：MEET 和 PING 包
    - 当一个给定的节点接收到一个 `CLUSTER MEET` 消息时，命令中指定的节点仍然不知道我们发送了命令，所以为了使节点强制将接收命令的节点将它作为信任的节点接受它，它会发送 MEET 包而不是 PING 包。
    - 两个消息包有相同的格式，但是 MEET 强制使接收消息包的节点确认发送消息包的节点为可信任的。

### CLUSTER MYID

```
CLUSTER MYID
```
- 返回节点的 id（与连接的集群节点相关联的唯一的、自动生成的标识符）。

### CLUSTER NODES

```
CLUSTER NODES
```
- 集群中的每个节点都有当前集群配置的一个视图（快照），视图的信息由该节点所有已知节点提供，包括与每个节点的连接状态，每个节点的标记位（flags)，属性和已经分配的哈希槽等等。
- `CLUSTER NODES` 提供了当前连接节点所属集群的配置信息，信息格式和 Redis 集群在磁盘上存储使用的序列化格式完全一样（在磁盘存储信息的结尾还存储了一些额外信息）。
- 通常，如果你想知道哈希槽与节点的关联关系，你应该使用 `CLUSTER SLOTS` 命令。`CLUSTER NODES` 主要是用于管理任务，调试和配置监控。redis-trib 也会使用该命令管理集群。
- 序列化格式
    - 输出是空格分割的 CSV 字符串，每行代表集群中的一个节点。
    - 每行的组成结构如下:
        ```
        <id> <ip:port> <flags> <master> <ping-sent> <pong-recv> <config-epoch> <link-state> <slot> <slot> ... <slot>
        ```
    - 详见 [CLUSTER NODES](http://redis.cn/commands/cluster-nodes.html)

### CLUSTER REPLICAS

```
CLUSTER REPLICAS node-id
```
- 提供从指定 master 节点复制的副本节点列表，输出格式同命令 `CLUSTER NODES`。
- 若特定节点状态未知，或在接收命令节点的节点信息表中，该节点不是主节点，则命令失败。
- 请注意，当已经添加，迁移或者删除一个副本时，在群集中未及时更新配置信息的节点上执行 `CLUSTER REPLICAS` 命令，仍返回原有配置信息。当然，所有节点最终将会同步群集中其他节点的信息 (网络正常情况下，需要几秒钟时间)。

### CLUSTER REPLICATE

```
CLUSTER REPLICATE node-id
```
- 重新配置一个节点成为指定 master 的 salve 节点。如果收到命令的节点是一个 empty master，那么该节点的角色将由 master 切换为 slave。
- 一旦一个节点变成另一个 master 节点的 slave，无需通知群集内其他节点这一变化：节点间交换信息的心跳包会自动将新的配置信息分发至所有节点。
- 基于如下假设，一个 slave 节点会接受该命令
    - 指定节点在它的节点信息表中存在
    - 指定节点无法识别接收我们命令的节点实例
    - 指定节点是一个 master
- 如果收到命令的节点不是 slave 而是 master，只要在如下情况下，命令才会执行成功，该节点才会切换为 slave：
    - 该节点不保存任何哈希槽
    - 该节点是空的，key 空间中不存储任何键
- 如果命令执行成功，新的 slave 会立即尝试连接它的 master 以便进行数据复制。

### CLUSTER RESET

```
CLUSTER RESET [HARD|SOFT]
```
- 重置一个 Redis Cluster 的节点，或多或少激烈的方式取决于重置类型，可以是 hard 的或 soft 的。请注意，如果主节点持有一个或多个键，则此命令对它们无效，在这种情况下，要完全重置主节点键，必须首先删除主节点键，比如先使用 `FLUSHALL`，然后 `CLUSTER RESET`。
- 节点上的效果如下：
    - 群集中的节点都被忽略
    - 所有已分派 / 打开的槽会被重置，以便 slots-to-nodes 对应关系被完全清除
    - 如果节点是 slave，它会被切换为 (空) master。它的数据集已被清空，因此最后也会变成一个空 master。
    - hard 重置时：生成新的节点 ID
    - hard 重置时：变量 currentEpoch 和 configEpoch 被设置为 0
    - 新配置被持久化到节点磁盘上的群集配置信息文件中
- 当需要为一个新的或不同的群集提供一个新的群集节点是可使用该命令，同时它也在 Redis Cluster 测试框架中被广泛使用，它用于在每个新的测试单元启动是初始化群集状态。
- 如果重置类型没有指定，使用省缺值 soft。

### CLUSTER SAVECONFIG

```
CLUSTER SAVECONFIG
```
- 强制保存配置 nodes.conf 至磁盘。
- 该命令主要用于 nodes.conf 节点状态文件丢失或被删除的情况下重新生成文件。当使用 `CLUSTER` 命令对群集做日常维护时，该命令可以用于保证新生成的配置信息会被持久化到磁盘。
- 当然，这类命令应该没定时调用将配置信息持久化到磁盘，保证系统重启之后状态信息还是正确的。

### CLUSTER SET-CONFIG-EPOCH

```
CLUSTER SET-CONFIG-EPOCH config-epoch
```
- 该命令为一个全新的节点设置指定的 config epoch, 仅在如下情况下有效：
    - 节点的节点信息表为空
    - 节点的当前 config epoch 为 0
- 这些先决条件是需要的，因为通常情况下，人工修改一个节点的配置 epoch 是不安全的，我们想保证一点：在获取哈希槽的所有权时，拥有更高配置 epoch 值的节点获胜。
- 但是该规则也有一个例外，在群集创建的时候，Redis 群集配置 epoch 冲突解决算法会解决群集启动时新的节点配置成相同配置 epoch 的问题，但是这个处理过程很慢，为了保证不管发生任何情况，都不会有两个节点拥有相同的配置 epoch。
- 因此，当一个新群集创建的时候，使用命令 `CONFIG SET-CONFIG-EPOCH` 为每个一个节点分派渐进的配置 epoch，然后再加入群集。

### CLUSTER SETSLOT

```
CLUSTER SETSLOT slot IMPORTING|MIGRATING|STABLE|NODE [node-id]
```
- `CLUSTER SETSLOT` 根据如下子命令选项，修改接受节点中哈希槽的状态：

    | 子命令    | 说明                                    |
    | --------- | --------------------------------------- |
    | MIGRATING | 将一个哈希槽设置为迁移（migrating）状态 |
    | IMPORTING | 将一个哈希槽设置为导入（importing）状态 |
    | STABLE    | 从哈希槽中清除导入和迁移状态            |
    | NODE      | 将一个哈希槽绑定到另一个不同的节点      |
- 该命令和子命令选项可以用来启动和结束一个群集重新哈希槽的操作，操作期间备操作哈希槽会在源节点被设置为迁移中状态，目标节点设置为导入中状态每个子命令详细如下：
    ```
    CLUSTER SETSLOT <slot> MIGRATING <destination-node-id>
    ```
    - 为了可以将一个哈希槽设置成这种状态，收到命令的节点必须是该哈希槽的所有者，否则报错。
    - 当一个哈希槽被设置为 migrating 状态，节点将会有如下操作：
        1. 如果处理的是存在的 key，命令正常执行
        2. 如果要处理的 key 不存在，接收命令的节点将发出一个重定向 ASK，让客户端紧在 destination-node 重试该查询。在这种情况下，客户端不应该将该哈希槽更新为节点映射。
        3. 如果命令包含多个 keys，如果都不存在，处理方式同第 2 项，如果都存在，处理方式同第 1 项，如果只是部分存在，针对即将完成迁移至目标节点的 keys 按序返回 TRYAGAIN 错误，以便批量 keys 命令可以执行。
    ```
    CLUSTER SETSLOT <slot> IMPORTING <source-node-id>
    ```
    - 该子命令是 MIGRATING 的反向操作，将 keys 从指定源节点导入目标节点。该命令仅能在目标节点不是指定槽的所有者时生效。
    - 当一个槽被设置为导入中状态时，该节点的变动情况如下
        1. 涉及该哈希槽的命令均被拒绝，并产生一个 MOVED 重定向，但是如果该命令跟着一个 ASKING 命令，在这种情况下，表示该命令已经执行。当迁移中的节点产生 ASK 重定向时，客户端会连接目标节点，在发送命令后紧接着发送 ASKING。按照这种策略，涉及源节点已不存在的 keys 或者已经迁移至目标节点的 keys 的命令，都在目标节点执行。说明如下：
        2. 新的 keys 总是在目标节点创建。在哈希槽的迁移中，我们只迁移旧 keys 不会创建新建的 keys
        3. 涉及已经迁移的 keys 的命令都会被目的节点处理，目的节点会是新的哈希槽的所有者，以保证一致性。
        4. 如果没有 ASKING，命令没什么特殊，ASKING 保证哈希槽映射关系错误的客户端不会再目的节点操作错误：在还没完成迁移的 key 创建一个新的版本。
    ```
    CLUSTER SETSLOT <slot> STABLE
    ```
    - 该子命令仅用于清理槽中迁移中 / 导入中的状态。它主要用于修复群集在使用 redis-trip fix 卡在一个错误状态的问题。一般情况下，使用下节介绍的命令 `SETSLOT... NODE...` 迁移完成时，这两种状态会被自动清理。
    ```
    CLUSTER SETSLOT <slot> NODE <node-id>
    ```
    - 子命令 NODE 使用方法最复杂，它后接指定节点的哈希槽，该命令仅在特定情况下有效，并且不同的槽状态会有不同的效果，前提条件和对应的效果如下：
        1. 如果接受命令的节点是当前操作哈希槽的所有者，但是该命令的操作结果是将操作的槽分配到另一个节点，因此，要操作的哈希槽中还有 keys，该命令会返回错误。
        2. 如果槽是 migrating 状态，当该槽被分配至其他节点时，状态被清除。
        3. 如果槽在接收命令的节点上是 importing 状态，该命令将槽分配给这个节点 (当进行重哈希时，最终结果哈希槽从一个节点迁移至目的节点) 并做如下操作： A) 状态 importing 被清除 B) 如果该节点的配置 epoch 不是群集中最大的，它将生成一个新的配置 epoch。这样，在经历过故障转移或者槽迁移的群集中，能够拿到新的哈希槽的所有权。
    - 特别注意：第 3 项是群集中一个节点不和其他节点协商独自产生一个新配置 epoch 的唯一情况，且仅在人工配置时发生。不过因为群集使用了配置 epoch 碰撞解决算法，该步骤不可能在两个节点的 epoch 相同的情况下完成群集安装。

### CLUSTER SLAVES

```
CLUSTER SLAVES node-id
```
- 该命令会列出指定 master 节点所有 slave 节点，格式同 `CLUSTER NODES`。
- 当指定节点未知或者根据接收命令的节点的节点信息表指定节点不是主节点，命令执行错误。
- 注意：当一个 slave 被添加，移动或者删除时，我们在一个配置信息没有更新的群集节点上执行命令 CLUSTER SLAVES 获取将是脏信息。不过最终 (无网络分区的情况下大概几秒钟) 所有节点都会同步指定 master 节点的 salve 节点信息。

### CLUSTER SLOTS

```
CLUSTER SLOTS
```
- 返回哈希槽和 Redis 实例映射关系。这个命令对客户端实现集群功能非常有用，使用这个命令可以获得哈希槽与节点（由 IP 和端口组成）的映射关系，这样，当客户端收到（用户的）调用命令时，可以根据（这个命令）返回的信息将命令发送到正确的 Redis 实例.
- （嵌套对象）结果数组：
    - 每一个（节点）信息:
        - 哈希槽起始编号
        - 哈希槽结束编号
        - 哈希槽对应 master 节点，节点使用 IP/Port 表示
        - master 节点的第一个副本
        - 第二个副本
        - … 直到所有的副本都打印出来
    - 每个结果包含该哈希槽范围的所有存活的副本，没有存活的副本不会返回.
    - （每个节点信息的）第三个（行）对象一定是 IP/Port 形式的 master 节点。之后的所有 IP/Port 都是该哈希槽范围的 Redis 副本。
    - 如果一个集群实例中的哈希槽不是连续的（例如 1-400,900,1800-6000），那么哈希槽对应的 master 和 replica 副本在这些不同的哈希槽范围会出现多次。

## READONLY

```
READONLY
```
- 开启与 Redis Cluster 从节点连接的读请求
- 通常，从节点将重定向客户端到认证过的主节点，以获取在指定命令中所涉及的哈希槽，然而客户端能通过 `READONLY` 命令将从节点设置为只读模式。
- `READONLY` 告诉 Redis Cluster 从节点客户端愿意读取可能过时的数据并且对写请求不感兴趣。
- 当连接处于只读模式，只有操作涉及到该从节点的主节点不服务的键时，集群将会发送一个重定向给客户端。这可能是因为：
    - 客户端发送一个有关这个从节点的主节点不服务哈希槽的命令。
    - 集群被重新配置（例如重新分片）并且从节点不在服务给定哈希槽的命令。

## READWRITE

```
READWRITE
```
- 禁止与 Redis Cluster 从节点连接的读请求（read query）。
- 默认情况下禁止 Redis Cluster 从节点的读请求，但是可以使用 `READONLY` 去在每个连接的基础上改变这个行为，`READWRITE` 命令将连接的只读模式重置为读写模式。