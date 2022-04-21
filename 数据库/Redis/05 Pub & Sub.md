- [概述](#概述)
- [相关命令](#相关命令)
  - [SUBSCRIBE & PSUBSCRIBE](#subscribe--psubscribe)
  - [PUBLISH](#publish)
  - [UNSUBCRIBE & PUNSUBSCRIBE](#unsubcribe--punsubscribe)
  - [PUBSUB](#pubsub)
    - [PUBSUB CHANNELS](#pubsub-channels)
    - [PUBSUB NUMSUB](#pubsub-numsub)
    - [PUBSUB NUMPAT](#pubsub-numpat)

# 概述

- `SUBSCRIBE`, `UNSUBSCRIBE`, `PUBLISH` 命令实现了发布 / 订阅消息范式
- 发布者（发送者）不是被设计成将消息发布给特定的订阅者（接受者），而是发布在特定的信道（channels），不需要了解该信道的订阅者
- 订阅者订阅一个或多个信道，也不需要了解该信道的发布者
- 这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑

# 相关命令

## SUBSCRIBE & PSUBSCRIBE

```
SUBSCRIBE channel [channel ...]
```
- 订阅给指定信道的信息。
- 一旦客户端进入订阅状态，客户端就只可接受订阅相关的命令，其他命令一律失效。

```
PSUBSCRIBE pattern [pattern ...]
```
- 订阅给定的模式的信道
- 支持的模式

    | 模式 | 说明                     |
    | ---- | ------------------------ |
    | `?`  | 匹配任意字符一次         |
    | `*`  | 匹配任意字符任意此       |
    | `[]` | 匹配中括号中任意字符一次 |
    | `\`  | 对特殊字符进行转义       |

## PUBLISH

```
PUBLISH channel message
```
- 将信息发送到指定的信道
- 返回值为收到消息的客户端数量

## UNSUBCRIBE & PUNSUBSCRIBE

```
SUBSCRIBE channel [channel ...]
```
- 指示客户端退订给定的信道，若没有指定信道，则退订所有信道
- 如果没有信道被指定，即，一个无参数的 `UNSUBSCRIBE` 调用被执行，那么客户端使用 `SUBSCRIBE` 命令订阅的所有信道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的信道

```
PUNSUBSCRIBE [pattern [pattern ...]]
```
- 指示客户端退订指定模式，若果没有提供模式则退出所有模式。

## PUBSUB

```
PUBSUB subcommand [argument [argument ...]]
```
- 是自省命令，能够检测 PUB/SUB 子系统的状态。它由分别详细描述的子命令组成

### PUBSUB CHANNELS

```
PUBSUB CHANNELS [pattern]
```
- 列出当前活跃的信道，活跃是指信道含有一个或多个订阅者 (不包括从模式接收订阅的客户端)
- 如果 pattern 未提供，所有的信道都被列出，否则只列出匹配上指定全局 - 类型模式的信道被列出.

### PUBSUB NUMSUB

```
PUBSUB NUMSUB [channel-1 ... channel-N]
```
- 列出指定信道的订阅者个数 (不包括订阅模式的客户端订阅者)
- 注意，不给定任何信道而直接调用这个命令也是可以的，在这种情况下，命令只返回一个空列表

### PUBSUB NUMPAT

```
PUBSUB NUMPAT
```
- 返回订阅模式的数量 (使用命令 PSUBSCRIBE 实现)
- 注意，这个命令返回的不是订阅模式的客户端的数量，而是客户端订阅的所有模式的数量总和