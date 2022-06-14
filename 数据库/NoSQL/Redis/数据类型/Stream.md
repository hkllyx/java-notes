# Stream

Redis Stream是Redis 5.0新增加的数据结构。

Redis Stream主要用于消息队列（MQ，Message Queue），Redis本身是有一个Redis发布订阅 （pub/sub）来实现消息队列的功能，但它有个缺点就是消息无法持久化，如果出现网络断开、Redis宕机等，消息就会被丢弃。简单来说Redis发布订阅可以分发消息，但无法记录历史消息。

而Redis Stream提供了消息的持久化和主备复制功能，可以让任何客户端访问任何时刻的数据，并且能记住每一个客户端的访问位置，还能保证消息不丢失。

Redis Stream的结构如下所示，它有一个消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的ID和对应的内容：

![Redis Stream](../../../../resource/img/Redis/Redis%20Stream.png)

每个Stream都有唯一的名称，它就是Redis的key，在我们首次使用`XADD`指令追加消息时自动创建。

- Consumer Group：消费者组，使用`XGROUP CREATE`命令创建，一个消费组有多个消费者（Consumer）
- Last_delivered_id：没个消费组会有个游标Last_delivered_id，组内任意一个消费者读取了消息都会使游标Last_delivered_id往前移动
- Consumer：消费者，消费Stream中的消息的客户端
- Pending_ids：消费者的状态变量，作用是维护消费者的未确认的id。Pending_ids记录了当前已经被读取但是还没有确认（Ack，Acknowledge character）的消息

## 相关命令

### `XADD`

```redis
XADD key [NOMKSTREAM] [MAXLEN | MINID [= | ~] threshold [LIMIT count]]
    id | * field value [ field value ...]
```

将指定项目（Entry）追加到指定key的Stream中。如果key不存在，会自动预先创建一个Stream然后追加，可以使用`NOMKSTREAM`选项关闭自动创建Stream的功能。

`XADD`是唯一可以向Stream添加数据的Redis命令，但是还有其他命令，例如`XDEL`和`XTRIM`能够从Stream中删除数据。

一个Stream项目是由一组键值对组成的，它基本上是一个小的字典。键值对以用户给定的顺序存储，并且读取Stream的命令（如`XRANGE`或者`XREAD`）可以保证按照通过`XADD`添加的顺序返回。

Stream项目的ID标识Stream内唯一的项目。如果指定的ID参数是`*`，`XADD`命令会自动生成一个唯一的ID。但是，也可以指定一个良好格式的ID，以便新的项目以指定的ID准确存储，虽然仅在极少数情况下有用。

良好的ID是由`-`隔开的两个数字组成的，例如，`1526919030474-55`。两个部分数字都是64位的，当自动生成ID时，第一部分是生成ID的Redis实例的毫秒格式的Unix时间。第二部分只是一个序列号，以及是用来区分同一毫秒内生成的ID的。

ID保证始终是递增的：如果比较刚插入的项目的ID，它将大于其他任何过去的ID，因此项目在Stream中始终是排序的。为了保证这个特性，如果Stream中当前最大的ID的时间大于实例的当前本地时间（例如，再本地始终回调了，或者在故障转移之后新主机具有不同的绝对时间，则可能发生这种情况），将会使用前者，并将ID的序列部分递增。

当用户为`XADD`命令指定显式ID时，最小有效的ID是`0-1`，并且用户必须指定一个比当前Stream中的任何ID都要大的ID，否则命令将失败。通常使用特定ID仅在您有另一个系统生成唯一ID（例如SQL表），并且您确实希望Redis Stream ID与该另一个系统的ID匹配时才有用。

`XADD`也包含`XTRIM`的功能，它可以保证每次向Stream添加项目后保持Stream的大小不变，从而有效得限流。虽然Redis Stream支持精准，默认也是精确的限制Stream大小，但由于内部的表现方式是的近似限制是更加有效的。

### `XLEN`

```redis
XLEN key
```

返回Stream中的项目数。如果指定的key不存在，则此命令返回0，就好像该流为空。但是请注意，与其他的Redis类型不同，0长度Stream是可能的，所以你应该调用`TYPE`或者`EXISTS`来检查该key是否存在。

一旦内部没有任何的项目（例如调用`XDEL`后），流不会被自动删除，因为可能还存在与其相关联的消费者组。

### `XRANGE`、`XREVRANGE`

```redis
XRANGE key start end [COUNT count]
```

返回Stream中ID介于start和end范围（闭区间）的项目。

`XRANGE` 命令有许多用途：

- 返回特定时间范围的项目。这是可能的，因为Stream项目的ID与时间相关
- 增量迭代流，每次迭代只返回几个项目。但它在语义上比`SCAN`函数族强大很多
- 从流中获取单个项目——start和end相同

特殊ID：`-`和`+`分别表示Stream中可能的最小ID（`0-0`）和最大ID（`18446744073709551615-18446744073709551615`），写起来更方便。

不完整的ID：Stream的ID由两部分组成——时间戳和同一时间内项目数序号，`XRANGE`中可以只写第一部分的时间戳，他会被自动补全，即start在后边补上`-0`，而end补上`-18446744073709551615`。这种使用释放可以非常方便获取指定时间的事件消息。

可以在start和end前加上`(`来使用开区间。

可以使用`COUNT`选项限制返回项目数量。可以用来分步遍历Stream的项目（start设为上一部返回的最大ID，end设为`+`，每次遍历count个）。

返回值包括两部分：

- Stream消息ID
- Stream消息的字段数字

```redis
XREVRANGE key end start [COUNT count]
```

此命令与`XRANGE`完全相同，但显著的区别是以相反的顺序获取和返回项目。

### `XDEL`

```redis
XDEL key id [id ...]
```

从key对应的Stream中移除指定的项目，并返回成功删除的项目的数量，在指定的id有不存在的情况下，返回的数量可能与传递的ID数量不同。

通常，可以Redis Stream想象为一个仅支持附加（append-only）的数据结构，但是Redis Steam是存在于内存中的，所以我们也可以删除项目。这也许会有用，例如，为了遵守特定的隐私策略。

理解删除项目的底层细节：

Redis Stream以一种内存高效的方式表示：使用基数树（radix tree）来索引包含线性数十个Stream项目的宏结点。通常，当你从Stream中删除一个项目的时候，项目并没有真正被驱逐，只是被标记为删除。

最终，如果宏结点中的所有项目都被标记为删除，则会销毁整个节点，并回收内存。这意味着如果你从Stream里删除大量的项目，比如超过50%的项目，则每一个项目的内存占用可能会增加，因为Stream将会开始变得碎片化。然而，Stream的性能将保持不变。

在Redis未来的版本中，当一个宏结点内删除项目达到一定数量的时候，我们有可能会触发节点垃圾回收机制。目前，根据我们对这种数据结构的预期用途，还不太适合增加这样的复杂度。

### `XTRIM`

```redis
XTRIM key MAXLEN | MINID [= | ~] threshold [LIMIT count]
```

如有需要，以将淘汰旧项目（ID较小的项目）的方式调整key对应的Stream长度（Stream中项目的数量）。

策略：

- `MAXLEN`：淘汰项目直至不超过threshold（必须是正整数，表示数量）
- `MINID`：淘汰ID小于threshold（项目ID格式，可以是只是第一部分，会自动补充`-0`）的项目

近视精确修建：因为精确得修建到指定长度需要服务器额外工作，所以提供`~`来近似精确修剪。`~`和`MAXLEN`选项搭配使用的时候，Stream的长度会被修剪到至少threshold，但可能会稍微更大些。这可以尽早停止修剪以获取性能。

另一个控制修剪数量的方式是使用`LINMIT`选项，它指定可以淘汰最大数目个项目。如果没有指定，省缺值 = 100 * 宏结点中项目数。设定为0则表示没有限制。

### `XINFO`

用于Stream内省，即查看Stream内部信息。

#### `XINFO STREAM`

```redis
XINFO STREAM key [FULL [COUNT count]]
```

返回key对应的Stream的信息。默认包含：

- length：项目（Entry）数量，参见`XLEN`。
- radix-tree-keys：底层基数数据结构（radix data structure）中的key数量
- radix-tree-nodes：底层基数数据结构（radix data structure）中的node数量
- groups：Stream中定义的消费者组的数量
- last-generated-id：最近向Stream添加的项目的ID
- max-deleted-entry-id：从Stream中删除的最大项目ID
- entries-added：Stream生命周期添加的项目数量
- first-entry：向Stream中添加的第一个项目的ID
- last-entry：向Stream中添加的最后一个项目的ID

`FULL`选项返回更加详细的信息，会返回一个以升序排序项目数组。消费者组也会返回一个数组，每个组返回的信息同`XINFO GROUPS`、`XINFO CONSUMERS`。

`COUNT`选项用于限制返回的Stream和PEL项目的数量。省却值是10，如果设置为0则表示返回所有的项目，着可能比较耗时。

#### `XINFO GROUPS`

```redis
XINFO GROUPS key
```

返回key对应的Stream的消费者组数组。每个消费者组的信息默认包含：

- name：消费者组名称
- consumers：消费者组中的消费者
- pending：PEL中项目——被交付但是没有被确认的消息
- last-delivered-id：最后一个组内消费者交付的项目ID
- entries-read：组内消费者交付的最后一个项目的逻辑已读记数
- lag：延迟，等待组内消费者交付的项目数量

消费者组lag：消费者组lag是一个介于组已读项目数（entries-read）和Stream已加入项目数（entries-added）中间的数，换句话说就是未被消费者交付的项目数量。lag的值和趋势有助于针对消费者组做出缩放的决策。当lag比较大的时候需要向组添加更多的消费者，反正比较小的时候就需要移除消费者。

Stream计数（entries-added）由其生命周期内每个`XADD`加一来记录向Stream添加的项目数量。消费者组计数（entries-read）是一个消费者组已读项目的**逻辑**计数，逻辑这个词说得是这个计数是一个启发式（heuristic）的而非精确的计数。这个计数试图根据反应消费者组的**应该已读数量**来计算其当前的last-delivered-id。entries-added只有在从它开始于Stream的第一个项目并且处理了所有项目（处理前没有删除项目）的完美情况下才是一定精确的。

一下两种情况下不能确定lag，这时候认为组的entries-read无效，并且返回的lag为`NULL`：

- 消费者组创建时设定了一个随机last-delivered-id（`XGROUP CREATE`时指定或者`XGROUP SETID`修改），并且随机ID不是Stream的第一个ID、最后一个ID和零ID（`0-0`）
- 介于组last-delivered-id和Stream last-generated-i中的任意个项目被删除（通过`XDEL`）

当然，lag无效只是暂时的，在消费者持续处理消息的期间会自动恢复。一旦消费者组交付了Stream的最后一条消息，消费者组会被设置一个正确的逻辑已读计数，监控她的lag也就恢复了。

#### `XINFO CONSUMERS`

```redis
XINFO CONSUMERS key groupname
```

返回属于key对应的Stream的消费者组的消费者。包含以下信息：

- name：消费者名称
- pending：待处理数量，即被消费者交付但未确认的消息数量
- idle：消费者和消费者组交互的毫秒时长

### `XGROUP`

该命令用于管理Stream关联的消费者组。

#### `XGROUP CREATE`

```redis
XGROUP CREATE key group-name id | $ [MKSTREAM] [ENTRIESREAD entries-read]
```

使用group-name创建关联key对应的Stream的新消费者组。

同一个Stream中每一个消费者组都有唯一的group-name，如果group-name对应的消费者组已经存在，`XGROUP CREATE`会返回一个”-BUSYGROUP“错误

id指定新建的消费者需要观测的最后交付项目（last delivered entry）。特殊id——`$`表示Stream中最后一项的ID，在这种情况下，从该消费者组获取数据的消费者只能看到后续加入Stream的新项目。

通常，创建消费者组的时候要求key对应的Stream已经存在，当然可以使用`MKSTREAM`选项自动创建一个Stream。

指定entries-read可以在指定id为除了Stream中第一个项目的ID、最后一个项目ID和零ID（`0-0`）外的任意ID来开启延迟（lag）跟踪有效。这在知道指定ID（不包含）到最后一个项目之间有多少个项目很有用时很有用。此时，可以指定entries-read = entries-added - 项目数量。

#### `XGROUP DESTROY`

```redis
XGROUP DESTROY key group-name
```

即使存在活存活的消费者和待处理消息，消费者组也将被销毁，因此请确保仅在真正需要时才调用此命令。

#### `XGROUP CREATECONSUMER`

```Redis
XGROUP CREATECONSUMER key group-name consumer-name
```

为和key对应Stream相关联的group-name对应的消费者组创建一个名为consumer-name的消费者。

消费者也会在涉及消费者操作时（例如，`XREADGROUP`）自动创建。

#### `XGROUP DELCONSUMER`

```redis
XGROUP DELCONSUMER key group-name consumer-name
```

从消费者组中移除指定的消费者。

注意：消费者删除后其待处理消息会变成无法认领的（unclaimable）。因此，在删除前最好先认领或通知待处理项目。

#### `XGROUP SETID`

```redis
XGROUP SETID key id
```

通常情况下，消费者组的last-delivered-id在创建时被设定（作为`XGROUP CREATE`的最后一个参数）。`XGROUP SETID`可以直接修改最后交付ID而无需删除和重新创建消费者。

例如，如果你希望消费者组中的消费者重新处理流中的所有消息，可以将最后交付ID设为0。也可以使用`$`字符表示最后一项的ID，即不消费已存在的待处理消息。

### `XREAD`、`XREADGROUP`

```redis
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]
```

从一个或多个Stream中读取数据，只返回ID大于命令指定的最后接受ID（即之前读取返回的最大项目ID）的项目。命令有一个`BLOCK`选项用于在没有可用项目时阻塞，类似`BRPOP`和`BZPOPMIN`等命令。

当`BLOCK`选项为启用，命令时同步的，可以将命令看作类似`XRANGE`：它会返回Streams中指定范围的项目。但是，即使考虑同步使用时它和`XRANGE`有两个基本差异：

- 命令可以同时读取多个Stream。这是`XREAD`的一个重要特型，特别是在使用`BLOCK`阻塞时可以通过单连接监听多个key。
- `XRANGE`返回ID范围内的项目，而`XREAD`更适合去从大于指定ID的第一个项目开始消费Stream，因此传给`XREAD`的ID参数是从对应Stream已接收的最后一个元素的ID

`STREAMS`选项强制放在最后，因为后续的参数必须按一定格式传递：先指定一系列的key，然后是对应的已从Stream中最后接受项目的ID。每次指定的ID是上次返回的最后一个项目的ID，长此以往，最终返回一个空数组（已经读取了当前所有的数据）。

`BLOCK`选项：在同步情形下，`XREAD`可以获取新数据直至Stream中没有更多项目。然后，有时候需要等待处理通过`XADD`添加的数据。为了避免固定或自适应的间隔轮询，`XREAD`支持在没有数据时阻塞，一旦指定的Stream中有数据就会解锁。

需要特别注意的是，`XREAD`面向一系列等待指定ID范围的数据的客户端，所以每个消费者都是拷贝一份数据而非类似阻塞列表的出栈操作。

`BLOCK`可以指定超时时间。通常，阻塞的超时时间以秒为单位，但是这个命令要求传入超时时间的单位为毫秒，即使服务器的超时分辨率通常接近0.1秒。这可能导致某些用例下阻塞事件更短，而且随着服务器的超时事件分辨率的改善而改善。

即使使用了`BLOCK`选项，如果`XREAD`返回了数据，也会以同步的方式处理，就像`BLOCK`选项不存在。

ID可以指定为`$`。某些情况，对之前已经加入的项目不关心，而是想关注从阻塞开始后加入的项目，需要获取阻塞前最大的项目ID，然后传入。这可能不大方便，所以可以传入`$`来代替，`$`当前最大的项目ID。

`$`只适用于第一次调用`XREAD`传入，而后续调用应该是传入前一次调用的返回值，不然可能会丢失两次调用之间加入的项目。

多客户端在单个Stream上阻塞的原理：阻塞列表操作在列表和有序集合中有一个出栈（pop）行为，元素会被从列表和有序集合中删除然后返回给客户端。这种情况下可以根据客户端阻塞结束的时间来公平地消费数据。通常Redis使用FIFO语义实现。然后在Stream上这不是个问题，因为服务器响应客户端的时候不会将项目从Stream删除，因此每一个客户端等待都可以在`XADD`加入新项目的时候尽快被响应。

```redis
XREADGROUP GROUP group consumer
    [COUNT count] [BLOCK milliseconds] [NOACK]
    STREAMS key [key ...] id [id ...]
```

`XREAD`的基础上支持消费者组。

使用`XREAD`时，客户端在所有项目到达Stream时被响应，而使用`XREADGROUP`，它可能会创建一个只消费部分到达Stream的项目的客户端分组。比如，一个Stream有三个新的项目A、B、C，一个消费者组以及它的两个消费者，然后一个消费者获得了A和C，而另一个获得了B。

在一个消费者组中，一个消费者（就是消费从Stream中消费的客户端）有一个唯一字符串名称。

消费者组的一个保证是一个指定的消费者只能看到交付给它的消息记录，这保证了每个消息只有一个所有者（消费者）。当然，消息认领（message claiming）特性可以让其他消费者认领消息防止某些消费者出现不可恢复的错误是消息不能被正确的处理。为了实现这些语义，消费者组需要消息成功处理后通过`XACK`命令显式确认。因为Stream会跟踪每个消费者组处理了哪些信息，因此这是必须的。

是否需要使用到消费者组：如果Stream有多个客户端，并且每个客户端都需要获取所有信息，则不需要使用消费者组；反之，如果想要切分Stream以及客户端内共享Stream，因此每个客户端都只能读取Stream内部分的信息，这是就需要使用到消费者组。

`XREAD`和`XREADGROUP`的区别：`XREADGROUP`需要指定消费者组以及它的消费者，消费者组由`XGROUP`命令创建，而消费者则是在第一次使用（先前不存在）的时候自动创建。不同的客户端应该使用不同的消费者。

当使用`XREADGROUP`读取消息时，服务器会记住消息被交付给那个客户端——消息会被存储在消费者组的待处理消息列表（PEL，Pending Entries List）中，这个列表存储那些被交付但是还没有确认的消息的ID。要想从PEL中移除消息，客户端需要使用`XACK`命令来确认被处理它。可以通过`XPENDING`命令查看PEL状态。

如果不要求可靠性，偶尔的消息丢失是可以接受的，那么可以使用`XREADGROUP`的`NOACK`选项从而不把消息加入到PEL中，这相当于读取的时候立马确认了消息。

`XREADGROUP`命令可以指定读取两类ID：

- 特殊ID`>`，意味着消费者只接收哪些从来没有被其他消费者接受的消息。即，只处理新消息
- 其他的合法ID，包括不完整ID，返回消费将处理的大于命令指定ID的消息。所以只要不是`>`，客户端就会获取其待处理的消息——消息被交付给客户端，但是没有被确认没有使用。当然，是没有使用`BLOCK`、`NOACK`的情况下。

消息交付给消费者时，如果消息从来没有被交付过，则说明是一个新信息，添加到消费者的PEL（PEL没有则创建）中。如果消息已经添加到这个消费者的PEL中了，则会更新最后交付计数器（last delivery counter）为当前时间，并且交付数+1。可以使用`XPENDING`命令查看相关信息。

### `XPENDING`

```redis
XPENDING key group [[IDLE min-idle-time] start end count [consumer]]
```

通过消费者组从Stream中获取数据，而不是确认这些数据，可能会创建PEL。

一旦消息被成功处理，`XACK`命令会立即从PEL中移除它，因为消费者组不再需要跟踪它并其所有者。

`XPENDING`命令是检查PEL的接口，是一个非常重要的命令，可用于观察和了解消费者组正在发生的事情：哪些客户端是活跃的，哪些消息在等待消费，或者查看是否有空闲的消息。此外，该命令与`XCLAIM`一起使用实现长时间故障的消费者的恢复。

`XPENDING`可以选择指定消费者查看其PEL。但返回信息不会有区别，因为一个消息只能交付给一个消费者。而且指定消费者也不影响效率，即使消费者组可能有多个消费者，因为消费者组有一个全局PEL的同时每个消费者都有单独的PEL。

`XPENDING`也可以使用选项过滤：

- 使用`IDLE`选项来过滤出哪些闲置时间超过指定时间的消息
- start和end的语法功能类似`XRANGE`

调用`XPENDING`命令会返回Stream中指定消费者组的待处理消息摘要。第一个返回值是待处理消息总数量，然后是所有待处理消息信息，信息内容包括：

- 消息ID
- 消息的所有者——获取了消息但没有确定消息的消费者
- 从消息最后一次交付给消费者到当前时间的毫秒数
- 消息被交付的次数

### `XCLAIM`

```redis
XCLAIM key group consumer min-idle-time ID [ID ...]
    [IDLE ms] [TIME ms-unix-time]
    [RETRYCOUNT count] [FORCE] [JUSTID]
```

在Stream的消费者组上下文中，此命令改变待处理消息的所有权，新的所有者是在命令参数中指定的消费者。通常发生在这种情形下：

1. 有一个具有关联消费者组的Stream
2. 消费者A在消费者组的上下文中通过`XREADGROUP`从流中读取一条消息
3. 读取消息时随之在消费者组的PEL中创建了一个待处理消息项目，这意味着这条消息已传递给给定的消费者，但是尚未通过`XACK`确认
4. 突然这个消费者出现故障，且永远无法恢复
5. 其他消费者可以使用`XPENDING`检查已经过时很长时间的待处理消息列表，为了继续处理这些消息，他们使用`XCLAIM`来获得消息的所有权，并继续处理。

消息只有在其空闲时间大于指定的空闲时间时才会被认领（claim）。因为`XCLAIM`会重置消息的空闲时间（因为这是处理消息的一次新尝试）。

两个消费者试图同时认领消息不可能成功，即只可能有一个消费者能成功认领消息。这避免了多次处理消息（虽然一般情况下无法完全避免多次处理）。

此外，`XCLAIM`会增加消息的尝试交付次数。通过这种方式，由于某些原因而无法处理的消息（例如因为消费者在尝试处理期间崩溃），将开始具有更大的计数器，并可以在系统内部被检测到。

该命令有多个选项，但是大部分主要用于内部使用，以便将`XCLAIM`或其他命令的结果传递到AOF文件，以及传递相同的结果到从节点，并且不太可能对普通用户有用：

| 选项                | 说明                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `IDLE ms`           | 设置消息的闲置时间（自最后一次交付到目前的时间）。省缺值为0，即时间计数被重置，因为消息现在有新的所有者来尝试处理它                                                                                                      |
| `TIME ms-unix-time` | 与`IDLE`相同，但它不是设置相对的毫秒数，而是将空闲时间设置为一个指定的Unix时间（以毫秒为单位）。这对于重写生成`XCLAIM`命令的AOF文件很有用                                                                                |
| `RETRYCOUNT count`  | 将重试计数器设置为指定的值。这个计数器在每一次消息被交付的时候递增。通常，`XCLAIM`不会更改这个计数器，它只在调用`XPENDING`命令时提供给客户端：这样客户端可以检测到异常，例如在大量传递尝试后由于某种原因从未处理过的消息 |
| `FORCE`             | 在PEL中创建待处理消息项目，即使某些指定的ID尚未在分配给不同客户端的PEL中。但是消息必须存在于流中，否则不存在的消息ID将会被忽略                                                                                           |
| `JUSTID`            | 只返回成功认领的消息ID数组，不返回实际的消息                                                                                                                                                                             |

### `XAUTOCLAIM`

```redis
XAUTOCLAIM key group consumer min-idle-time start-id [COUNT count] [JUSTID]
```

更改符合条件的待处理Stream项目的所有者。这个功能等同先调用`XPENDING`然后调用`XCLAIM`，但提供了更直接的方式处理交付失败的信息。

类似`XCLAIM`命令，`XAUTOCLAIM`命令将key所在的Stream的将指定消费者组中闲置时间大于等于min-idle-time，ID大于等于start-id的消息的所有者改为指定的消费者。

`COUNT`选项默认count = 100，限制命令一次认领的最多消息数。内部处理时，命令从start-id开始扫描消费者组的PEL中闲置时间大于等于min-idle-time的项目。命令最大扫描的待处理项目的数量是10 * count。扫描这么多次是可能的，因为PEL中可能有足够的项目不满足条件。

命令返回被认领了的消息数组，如果使用了`JUSTID`选项则只返回消息ID，并且意味着重试计数器（retry counter）不是递增的。当PEL中没有消息，则返回`0-0`标识已完成。当然，可以使用`0-0`作为下一次`XAUTOCLAIM`的start-id，因为过了一定时间可能有旧待处理项目可能满足了筛选条件。

只有闲置时间超过min-idle-time的消息才会被处理，而且处理之后闲置时间会被重置。这可以确保只有一个消费者可以在特定的时间成功地认领待处理消息，并且微不足道地降低了多次处理同一消息的可能性。

当遍历PEL时，如果`XAUTOCLAIM`碰到一个不再存在于Stream中的消息（被`XDEL`删除）则不会处理它，并且从PEL中删除它。这个特性从Redis 7.0开始支持。这些消息的ID也会作为返回消息的一部分返回。

最后，`XAUTOCLAIM`认领消息会增加消息的尝试交付次数（attempted deliveries count），除非使用了`JUSTID`选项，它只传递ID而不是消息本身。可以通过尝试交付次数来检测处理过程中发生的一些错误，例如消费者在处理消息时系统崩溃。

返回值包括三部分：

- 作为下次`XAUTOCLAIM`的start-id的Stream ID
- 认领成功的消息数组，格式同`XRANGE`
- 存在于PEL但不再存于在Stream的ID数组

### `XACK`

```redis
XACK key group id [id ...]
```

用于从Stream的消费者组的PEL（待处理项目列表）中删除一条或多条消息。

当一条消息交付到某个消费者时，它将被存储在PEL中等待处理。通常发生在调用`XREADGROUP`命令，或者通过调用`XCLAIM`命令让一个消费者接管消息的时候发生。

待处理消息被交付到某些消费者，但是服务器尚不确定它是否至少被处理了一次。因此对新调用`XREADGROUP`来获取消费者的消息历史记录（比如用0作为ID）将返回此类消息。类似地，待处理的消息将由检查PEL的`XPENDING`命令列出。

一旦消费者成功地处理完一条消息，应该调用`XACK`确认它，这样这个消息就不会被再次处理，且关于此消息的PEL项目也随之被清除，从Redis服务器释放内存。

该命令返回成功确认的消息数。某些消息ID可能不再是PEL的一部分（例如，它们已经被确认），而且`XACK`不会把他们算到成功确认的数量中。
