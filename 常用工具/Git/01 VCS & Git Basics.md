- [版本控制](#版本控制)
  - [本地版本控制系统](#本地版本控制系统)
  - [集中化的版本控制系统](#集中化的版本控制系统)
  - [分布式版本控制系统](#分布式版本控制系统)
- [Git基础](#git-基础)
  - [直接记录快照，而非差异比较](#直接记录快照而非差异比较)
  - [近乎所有操作都是本地执行](#近乎所有操作都是本地执行)
  - [Git保证完整性](#git-保证完整性)
  - [Git一般只添加数据](#git-一般只添加数据)
  - [三种状态及三个工作区域](#三种状态及三个工作区域)
  - [Git工作流程](#git-工作流程)
  - [Git配置](#git-配置)
  - [Git基础命令](#git-基础命令)
    - [git help & --help](#git-help----help)
    - [git init](#git-init)
    - [git clone](#git-clone)

# 版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。可以对任何类型的文件进行版本控制，一般情况下，我们对保存着软件源代码的文件进行版本控制。

有了版本控制，你就可以将某个文件回溯到之前的状态，甚至将整个项目都回退到过去某个时间点的状态，你可以比较文件的变化细节，查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等。

使用版本控制系统通常还意味着，就算你乱来一气把整个项目中的文件改的改删的删，你也照样可以轻松恢复到原先的样子。但额外增加的工作量却微乎其微。

## 本地版本控制系统

许多人习惯用复制整个项目目录的方式来保存不同的版本，或许还会改名加上备份时间以示区别。这么做唯一的好处就是简单，但是特别容易犯错。有时候会混淆所在的工作目录，一不小心会写错文件或者覆盖意想外的文件。

为了解决这个问题，人们很久以前就开发了许多种本地版本控制系统，大多都是采用某种简单的数据库来记录文件的历次更新差异。

![本地版本控制系统](img/local.png)

## 集中化的版本控制系统

接下来人们又遇到一个问题，如何让在不同系统上的开发者协同工作？于是，集中化的版本控制系统（Centralized Version Control Systems，CVCS）应运而生。这类系统，诸如CVS、Subversion以及Perforce等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。多年以来，这已成为版本控制系统的标准做法。

![集中化的版本控制系统](img/centralized.png)

这种做法带来了许多好处，特别是相较于老式的本地VCS来说。现在，每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个CVCS要远比在各个客户端上维护本地数据库来得轻松容易。

事分两面，有好有坏。这么做最显而易见的缺点是中央服务器的单点故障。如果宕机一小时，那么在这一小时内，谁都无法提交更新，也就无法协同工作。如果中心数据库所在的磁盘发生损坏，又没有做恰当备份，毫无疑问你将丢失所有数据 —— 包括项目的整个变更历史，只剩下人们在各自机器上保留的单独快照。本地版本控制系统也存在类似问题，只要整个项目的历史记录被保存在单一位置，就有丢失所有历史更新记录的风险。

## 分布式版本控制系统

于是分布式版本控制系统（Distributed Version Control System，DVCS）面世了。在这类系统中，像Git、Mercurial、Bazaar以及Darcs等，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

![分布式版本控制系统](img/distributed.png)

更进一步，许多这类系统都可以指定和若干不同的远端代码仓库进行交互。籍此，你就可以在同一个项目中，分别和不同工作小组的人相互协作。你可以根据需要设定不同的协作流程，比如层次模型式的工作流，而这在以前的集中式系统中是无法实现的。

# Git基础

## 直接记录快照，而非差异比较

Git和其它版本控制系统（包括Subversion和近似工具）的主要差别在于Git对待数据的方法。概念上来区分，其它大部分系统以文件变更列表的方式存储信息。这类系统（CVS、Subversion、Perforce、Bazaar等等）将它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。

![存储每个文件与初始版本的差异](img/deltas.png)

Git不按照以上方式对待或保存数据。反之，Git更像是把数据看作是对小型文件系统的一组快照。每次你提交更新，或在Git中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效，如果文件没有修改，Git不再重新存储该文件，而是只保留一个链接指向之前存储的文件。Git对待数据更像是一个**快照流**。

![存储项目随时间改变的快照](img/snapshots.png)

这是Git与几乎所有其它版本控制系统的重要区别。因此Git重新考虑了以前每一代版本控制系统延续下来的诸多方面。Git更像是一个小型的文件系统，提供了许多以此为基础构建的超强工具，而不只是一个简单的VCS。稍后我们在Git分支讨论Git分支管理时，将探究这种方式对待数据所能获得的益处。

## 近乎所有操作都是本地执行

在Git中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。如果你习惯于所有操作都有网络延时开销的集中式版本控制系统，Git在这方面会让你感到速度之神赐给了Git超凡的能量。因为你在本地磁盘上就有项目的完整历史，所以大部分操作看起来瞬间完成。

举个例子，要浏览项目的历史，Git不需外连到服务器去获取历史，然后再显示出来 —— 它只需直接从本地数据库中读取。你能立即看到项目历史。如果你想查看当前版本与一个月前的版本之间引入的修改，Git会查找到一个月前的文件做一次本地的差异计算，而不是由远程服务器处理或从远程服务器拉回旧版本文件再来本地处理。

这也意味着你离线或者没有VPN时，几乎可以进行任何操作。如你在飞机或火车上想做些工作，你能愉快地提交，直到有网络连接时再上传。如你回家后VPN客户端不正常，你仍能工作。使用其它系统，做到如此是不可能或很费力的。比如，用Perforce，你没有连接服务器时几乎不能做什么事；用Subversion和CVS，你能修改文件，但不能向数据库提交修改（因为你的本地数据库离线了）。这看起来不是大问题，但是你可能会惊喜地发现它带来的巨大的不同。

## Git保证完整性

Git中所有数据在存储前都计算校验和，然后以校验和来引用。这意味着不可能在Git不知情时更改任何文件内容或目录内容。这个功能建构在Git底层，是构成Git哲学不可或缺的部分。若你在传送过程中丢失信息或损坏文件，Git就能发现。

Git用以计算校验和的机制叫做SHA-1散列（hash，哈希）。这是一个由40个十六进制字符（0-9和a-f）组成的字符串，基于Git中文件的内容或目录结构计算出来。SHA-1哈希看起来是这样：

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git中使用这种哈希值的情况很多，你将经常看到这种哈希值。实际上，Git数据库中保存的信息都是以文件内容的哈希值来索引，而不是文件名。

## Git一般只添加数据

你执行的Git操作，几乎只往Git数据库中增加数据。很难让Git执行任何不可逆操作，或者让它以任何方式清除数据。同别的VCS一样，未提交更新时有可能丢失或弄乱修改的内容；但是一旦你提交快照到Git中，就难以再丢失数据，特别是如果你定期的推送数据库到其它仓库的话。

这使得我们使用Git成为一个安心愉悦的过程，因为我们深知可以尽情做各种尝试，而没有把事情弄糟的危险。更深度探讨Git如何保存数据及恢复丢失数据的话题，请参考撤消操作。

## 三种状态及三个工作区域

Git有三种状态，你的文件可能处于其中之一：

- 已提交（committed）：已提交表示数据已经安全的保存在本地数据库中。
- 已修改（modified）：已修改表示修改了文件，但还没保存到数据库中。
- 已暂存（staged）：已暂存表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入Git项目的三个工作区域的概念：

- Git仓库（git repository）：是Git用来保存项目的元数据和对象数据库的地方。这是Git中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。
- 工作目录（work directory, work tree）：是对项目的某个版本独立提取出来的内容。这些从Git仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
- 暂存区域（stage, index）：是一个文件，保存了下次将提交的文件列表信息，一般在Git仓库目录中。有时候也被称作 “索引”，不过一般说法还是叫暂存区域。

![工作目录、暂存区域以及Git仓库](img/areas.png)

## Git工作流程

基本的Git工作流程如下：

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到Git仓库目录。

相应的状态：

- 如果Git目录中保存着特定版本的文件，就属于已提交状态。
- 如果作了修改并已放入暂存区域，就属于已暂存状态。
- 如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

## Git配置

使用Git配置的第一步，就是配置用户的姓名及邮箱。如果值中包含空格，必须使用引号引用。

```
$ git config --global user.name hkllyx
$ git config --global user.email hkllyx@gmail.com
```

命令

```
git config [file-option] [type] [-z|--null] [<action>]
```

常见file-option：配置文件选项，文件生效范围越大，优先级越低。

| file-option | 说明                                             |
| ----------- | ------------------------------------------------ |
| --system    | 使用系统配置文件，对系统所以用户及其所有仓库生效 |
| --global    | 使用全局配置文件，对当前用户及其所有仓库生效     |
| --local     | 使用本地配置文件，对当前仓库生效                 |

type：变量值的数据类型。用于add/get/get-all/get-regex/--replace-all操作。

| 类型          | 说明                       |
| ------------- | -------------------------- |
| --bool        | 布尔型，值为true或false |
| --int         | 值时十进制数字             |
| --bool-or-int | 值时bool或int           |
| --path        | 值时路径，文件或目录       |

action：常见操作

| 操作                                   | 说明                                           |
| -------------------------------------- | ---------------------------------------------- |
| [--add] name value                     | 添加一个新的变量。若配置已存在，则直接替换旧值 |
| --get name [value_regex]               | 获取单值变量的值，value_regex对值进行筛选     |
| --get-all name [value_regex]           | 获取多值变量的值（也适用与单值变量）           |
| --get-regexp name [value_regex]        | 获取变量名符合指定正则表达式的变量的值         |
| --replace-all name value [value_regex] | 替换变量的所有值                               |
| --unset name [value_regex]             | 移除单值变量                                   |
| --unset-all name [value_regex]         | 移除多值变量                                   |
| -l, --list                             | 列出所有配置                                   |
| -e, --edit                             | 打开编辑窗口，编辑指定配置文件                 |

-z, --null：使用NULL字节终止值。用于不带 --add的添加替换操作 /get/get-all/get-regex/list操作。

常用配置及示例：

```
git config --global color.ui auto           # diff/log/status等命令输出着色
git config --global core.eol lf             # 换行符
git config --global core.autocrlf false     # 禁止签出自动将LF转换为CRLF
git config --global core.safecrlf true      # 确保CRLF转换可逆
git config --global credential.helper store # 存储Github身份验证信息
```

## Git基础命令

### git help & --help

可以使用`--help`或`help`子命令查看命令帮助信息及文档。

```
git --help|help
```

查看git命令帮助信息

```
git --help|help -a|--all
```

列出所有子命令

```
git --help|help -g|--guide
```

查看引导信息

```
git --help|help <subcommands>
git <subcommands> --help（此处只能使用 --help选项而不能使用help子命令）
```

查看某个子命令帮助文档

### git init

新建空白仓库或重新初始化现有仓库到指定目录

```
git init [option] [<dir>]
```

例如：

```
$ mkdir ~/test # 创建test目录
$ cd ~/test    # 进入test目录
$ git init     # 初始化空Git仓库，会在目录中生成 .git目录及文件
Initialized empty Git repository in /home/hkllyx/test/.git/
```

### git clone

克隆现有仓库到指定目录

```
git clone [options] [--] <repo> [<dir>]
```

例如：

```
$ git clone https://github.com/hkllyx/test.git # 从Github远程仓库中克隆
```