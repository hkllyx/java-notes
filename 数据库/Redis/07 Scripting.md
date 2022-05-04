- [概述](#概述)
- [相关命令](#相关命令)
  - [EVAL](#eval)
    - [参数说明及要求](#参数说明及要求)
    - [Lua和Redis类型转换](#lua-和-redis-类型转换)
    - [脚本的原子性](#脚本的原子性)
    - [错误处理](#错误处理)
    - [带宽和 `EVALSHA`](#带宽和-evalsha)
    - [脚本缓存](#脚本缓存)
    - [纯函数脚本](#纯函数脚本)
    - [全局变量保护](#全局变量保护)
    - [使用选择内部脚本](#使用选择内部脚本)
    - [可用库](#可用库)
    - [沙箱 (sandbox) 和最大执行时间](#沙箱-sandbox-和最大执行时间)
    - [流水线 (pipeline) 上下文 (context) 中的 `EVALSHA`](#流水线-pipeline-上下文-context-中的-evalsha)
  - [EVALSHA](#evalsha)
  - [SCRIPT](#script)
    - [SCRIPT DEBUG](#script-debug)
    - [SCRIPT EXISTS](#script-exists)
    - [SCRIPT FLUSH](#script-flush)
    - [SCRIPT KILL](#script-kill)
    - [SCRIPT LOAD](#script-load)

# 概述

参见 [Lua Script](Advance|Lua&#32;Script.md)。

# 相关命令

## EVAL

```
EVAL script numkeys key [key ...] arg [arg ...]
```
- `EVAL` 和`EVALSHA`命令是从Redis 2.6.0版本开始的，使用内置的Lua解释器，可以对Lua脚本进行求值。

### 参数说明及要求

- 第一个参数是一段Lua脚本程序。这段Lua脚本不需要（也不应该）定义函数。它运行在Redis服务器中。
- 第二个参数是参数的个数，后面的参数（从第三个参数），表示在脚本中所用到的那些Redis key，这些key参数可以在Lua中通过全局变量KEYS数组，用1为基址的形式访问 ( KEYS[1], KEYS[2], ...)。
- 在命令的最后，那些不是键名参数的附加参数arg [arg…]，可以在Lua中通过全局变量ARGV数组访问，访问的形式和KEYS变量类似 ( ARGV[1], ARGV[2], ...)。
- 例如：
    ```
    EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
    1) "key1"
    2) "key2"
    3) "first"
    4) "second"
    ```
    注：返回结果是Redis multi bulk replies的Lua数组，这是一个Redis的返回类型，您的客户端库可能会将他们转换成数组类型。
- 这是从一个Lua脚本中使用两个不同的Lua函数来调用Redis的命令的例子：
    ```
    redis.call()
    redis.pcall()
    ```
    - `redis.call()` 与`redis.pcall()`很类似，他们唯一的区别是当redis命令执行结果返回错误时，前者将返回给调用者一个错误，而后者会将捕获的错误以Lua表的形式返回。
    - `redis.call()` 和`redis.pcall()`两个函数的参数可以是任意的Redis命令：
        ```
        > EVAL "return redis.call('set','foo','bar')" 0
        OK
        ```
        - 需要注意的是，上面这段脚本的确实现了将键foo的值设为bar的目的，但是，它违反了`EVAL`命令的语义，因为脚本里使用的所有键都应该由KEYS数组来传递，就像这样：
            ```
            > eval "return redis.call('set',KEYS[1],'bar')" 1 foo
            OK
            ```
        - 要求使用正确的形式来传递key是有原因的，因为不仅仅是`EVAL`这个命令，所有的Redis命令，在执行之前都会被分析，籍此来确定命令会对哪些key进行操作。
        - 因此，对于`EVAL`命令来说，必须使用正确的形式来传递key，才能确保分析工作正确地执行。
        - 除此之外，使用正确的形式来传递key还有很多其他好处，它的一个特别重要的用途就是确保Redis集群可以将你的请求发送到正确的集群节点。(对Redis集群的工作还在进行当中，但是脚本功能被设计成可以与集群功能保持兼容。) 不过，这条规矩并不是强制性的，从而使得用户有机会滥用 (abuse) Redis单实例配置 (single instance configuration)，代价是这样写出的脚本不能被Redis集群所兼容。

### Lua和Redis类型转换

- Lua脚本能返回一个值，这个值能按照一组转换规则从Lua转换成redis的返回类型。
- 当Lua通过`call()`或`pcall()`函数执行Redis命令的时候，命令的返回值会被转换成Lua数据结构。同样地，当Lua脚本在Redis内置的解释器里运行时，Lua脚本的返回值也会被转换成Redis协议 (protocol)，然后由`EVAL`将值返回给客户端。
- 数据类型之间的转换遵循这样一个设计原则：如果将一个Redis值转换成Lua值，之后再将转换所得的Lua值转换回Redis值，那么这个转换所得的Redis值应该和最初时的Redis值一样。
- 换句话说，Lua类型和Redis类型之间存在着一一对应的转换关系。
- Redis和Lua的相互转换参见 [EVAL](https://redis.io/commands/eval)
- 还有下面两点需要重点注意：
    - lua中整数和浮点数之间没有什么区别。因此，我们始终Lua的数字转换成整数的回复，这样将舍去小数部分。如果你想从Lua返回一个浮点数，你应该将它作为一个字符串（比如`ZSCORE`命令）。
    - 没有简单的方法让nils储存在Lua数组中，这是Lua表语义的结果，所以当Redis将Lua数组转换成Redis协议时，如果遇到nil，转换就会停止。

### 脚本的原子性

- Redis使用单个Lua解释器去运行所有脚本，并且，Redis也保证脚本会以原子性 (atomic) 的方式执行： 当某个脚本正在运行的时候，不会有其他脚本或Redis命令被执行。
- 这和使用`MULTI` / `EXEC`包围的事务很类似。在其他别的客户端看来，脚本的效果 (effect) 要么是不可见的 (not visible)，要么就是已完成的 (already completed)。
- 另一方面，这也意味着，执行一个运行缓慢的脚本并不是一个好主意。写一个跑得很快很顺溜的脚本并不难，因为脚本的运行开销 (overhead) 非常少，但是当你不得不使用一些跑得比较慢的脚本时，请小心，因为当这些蜗牛脚本在慢吞吞地运行的时候，其他客户端会因为服务器正忙而无法执行命令。

### 错误处理

- 前面的命令介绍部分说过，`redis.call()` 和`redis.pcall()`的唯一区别在于它们对错误处理的不同。
- 当`redis.call()`在执行命令的过程中发生错误时，脚本会停止执行，并返回一个脚本错误，错误的输出信息会说明错误造成的原因：
    ```
    > DEL foo
    (integer) 1
    > LPUSH foo a
    (integer) 1
    > EVAL "return redis.call('get','foo')" 0
    (error) ERR Error running script (call to f_6b1bf486c81ceb7edf3c093f4c48582e38c0e791): ERR Operation against a key holding the wrong kind of value
    ```
- 和`redis.call()`不同，`redis.pcall()` 出错时并不引发 (raise) 错误，而是返回一个带err域的Lua表 (table)，用于表示错误：
    ```
    > EVAL "return redis.pcall('get', 'foo')" 0
    (error) ERR Operation against a key holding the wrong kind of value
    ```

### 带宽和 `EVALSHA`

- `EVAL` 命令要求你在每次执行脚本的时候都发送一次脚本主体 (script body)。Redis有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。
- 为了减少带宽的消耗，Redis实现了`EVALSHA`命令，它的作用和`EVAL`一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的SHA1校验和 (sum)。
- `EVALSHA` 命令的表现如下：
    - 如果服务器还记得给定的SHA1校验和所指定的脚本，那么执行这个脚本。
    - 如果服务器不记得给定的SHA1校验和所指定的脚本，那么它返回一个特殊的错误，提醒用户使用`EVAL`代替 `EVALSHA`。
    - 以下是示例：
        ```
        > SET foo bar
        OK
        > EVAL "return redis.call('get','foo')" 0
        "bar"
        > EVALSHA 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0
        "bar"
        > EVALSHA ffffffffffffffffffffffffffffffffffffffff 0
        (error) `NOSCRIPT`No matching script. Please use [EVAL](/commands/eval).
        ```
- 客户端库的底层实现可以一直乐观地使用`EVALSHA`来代替`EVAL`，并期望着要使用的脚本已经保存在服务器上了，只有当NOSCRIPT错误发生时，才使用 `EVAL`命令重新发送脚本，这样就可以最大限度地节省带宽。
- 这也说明了执行`EVAL`命令时，使用正确的格式来传递键名参数和附加参数的重要性：因为如果将参数硬写在脚本中，那么每次当参数改变的时候，都要重新发送脚本，即使脚本的主体并没有改变，相反，通过使用正确的格式来传递键名参数和附加参数，就可以在脚本主体不变的情况下，直接使用`EVALSHA`命令对脚本进行复用，免去了无谓的带宽消耗。

### 脚本缓存

- Redis保证所有被运行过的脚本都会被永久保存在脚本缓存当中，这意味着，当`EVAL`命令在一个Redis实例上成功执行某个脚本之后，随后针对这个脚本的所有`EVALSHA`命令都会成功执行。
- 刷新脚本缓存的唯一办法是显式地调用`SCRIPT FLUSH`命令，这个命令会清空运行过的所有脚本的缓存。通常只有在云计算环境中，Redis实例被改作其他客户或者别的应用程序的实例时，才会执行这个命令。
- 缓存可以长时间储存而不产生内存问题的原因是，它们的体积非常小，而且数量也非常少，即使脚本在概念上类似于实现一个新命令，即使在一个大规模的程序里有成百上千的脚本，即使这些脚本会经常修改，即便如此，储存这些脚本的内存仍然是微不足道的。
- 事实上，用户会发现Redis不移除缓存中的脚本实际上是一个好主意。比如说，对于一个和Redis保持持久化链接 (persistent connection) 的程序来说，它可以确信，执行过一次的脚本会一直保留在内存当中，因此它可以在流水线中使用`EVALSHA`命令而不必担心因为找不到所需的脚本而产生错误 (稍候我们会看到在流水线中执行脚本的相关问题)。

### 纯函数脚本

- 在编写脚本方面，一个重要的要求就是，脚本应该被写成纯函数 (pure function)。
- 也就是说，脚本应该具有以下属性：
    - 对于同样的数据集输入，给定相同的参数，脚本执行的Redis写命令总是相同的。
    - 脚本执行的操作不能依赖于任何隐藏 (非显式) 数据，不能依赖于脚本在执行过程中、或脚本在不同执行时期之间可能变更的状态，并且它也不能依赖于任何来自I/O设备的外部输入。
- 使用系统时间 (system time)，调用像`RANDOMKEY`那样的随机命令，或者使用Lua的随机数生成器，类似以上的这些操作，都会造成脚本的求值无法每次都得出同样的结果。
- 为了确保脚本符合上面所说的属性，Redis做了以下工作：
    - Lua没有访问系统时间或者其他内部状态的命令
    - Redis会返回一个错误，阻止这样的脚本运行： 这些脚本在执行随机命令之后 (比如`RANDOMKEY`、`SRANDMEMBER`或`TIME`等)，还会执行可以修改数据集的Redis命令。如果脚本只是执行只读操作，那么就没有这一限制。注意，随机命令并不一定就指那些带RAND字眼的命令，任何带有非确定性的命令都会被认为是随机命令，比如`TIME`命令就是这方面的一个很好的例子。
    - 每当从Lua脚本中调用那些返回无序元素的命令时，执行命令所得的数据在返回给Lua之前会先执行一个静默 (slient) 的字典序排序 (lexicographical sorting)。举个例子，因为Redis的Set保存的是无序的元素，所以在Redis命令行客户端中直接执行 `SMEMBERS`，返回的元素是无序的，但是，假如在脚本中执行 `redis.call(“smembers”, KEYS[1])`，那么返回的总是排过序的元素。
    - 对Lua的伪随机数生成函数math.random和math.randomseed进行修改，使得每次在运行新脚本的时候，总是拥有同样的seed值。这意味着，每次运行脚本时，只要不使用math.randomseed，那么math.random产生的随机数序列总是相同的。
- 尽管有那么多的限制，但用户还是可以用一个简单的技巧写出带随机行为的脚本 (如果他们需要的话)。

### 全局变量保护

- 为了防止不必要的数据泄漏进Lua环境，Redis脚本不允许创建全局变量。如果一个脚本需要在多次执行之间维持某种状态，它应该使用Redis key来进行状态保存。
- 企图在脚本中访问一个全局变量 (不论这个变量是否存在) 将引起脚本停止，`EVAL` 命令会返回一个错误：
    ```
    > EVAL 'a=10' 0
    (error) ERR Error running script (call to f_933044db579a2f8fd45d8065f04a8d0249383e57): user_script:1: Script attempted to create global variable 'a'
    ```
- Lua的debug工具，或者其他设施，比如打印（alter）用于实现全局保护的meta table，都可以用于实现全局变量保护。
- 实现全局变量保护并不难，不过有时候还是会不小心而为之。一旦用户在脚本中混入了Lua全局状态，那么AOF持久化和复制（replication）都会无法保证，所以，请不要使用全局变量。
- 避免引入全局变量的一个诀窍是：将脚本中用到的所有变量都使用local关键字定义为局部变量。

### 使用选择内部脚本

- 在正常的客户端连接里面可以调用`SELECT`选择内部的Lua脚本，但是Redis 2.8.11和Redis 2.8.12在行为上有一个微妙的变化。
- 在2.8.12之前，会将脚本传送到调用脚本的当前数据库。从2.8.12开始Lua脚本只影响脚本本身的执行，但不修改当前客户端调用脚本时选定的数据库。
- 从补丁级发布的语义变化是必要的，因为旧的行为与Redis复制层固有的不相容是错误的原因。

### 可用库

- Redis Lua解释器可用加载以下Lua库：
    - **base** lib.
    - **table** lib.
    - **string** lib.
    - **math** lib.
    - **struct** lib.
    - **cjson** lib.
    - **cmsgpack** lib.
    - **bitop** lib.
    - **redis.sha1hex** function.
    - **redis.breakpoint** and **redis.debug** function in the context of the Redis Lua debugger.
- 每一个Redis实例都拥有以上的所有类库，以确保您使用脚本的环境都是一样的。
- struct, CJSON和cmsgpack都是外部库，所有其他库都是标准Lua库。

### 沙箱 (sandbox) 和最大执行时间

- 脚本应该仅仅用于传递参数和对Redis数据进行处理，它不应该尝试去访问外部系统 (比如文件系统)，或者执行任何系统调用。
- 除此之外，脚本还有一个最大执行时间限制，它的默认值是5秒钟，一般正常运作的脚本通常可以在几分之几毫秒之内完成，花不了那么多时间，这个限制主要是为了防止因编程错误而造成的无限循环而设置的。
- 最大执行时间的长短由lua-time-limit选项来控制 (以毫秒为单位)，可以通过编辑redis.conf文件或者使用`CONFIG GET`和`CONFIG SET`命令来修改它。
- 当一个脚本达到最大执行时间的时候，它并不会自动被Redis结束，因为Redis必须保证脚本执行的原子性，而中途停止脚本的运行意味着可能会留下未处理完的数据在数据集 (data set) 里面。
- 因此，当脚本运行的时间超过最大执行时间后，以下动作会被执行：
    1. Redis记录一个脚本正在超时运行
    2. Redis开始重新接受其他客户端的命令请求，但是只有`SCRIPT KILL`和`SHUTDOWN NOSAVE`两个命令会被处理，对于其他命令请求，Redis服务器只是简单地返回BUSY错误。
    3. 可以使用`SCRIPT KILL`命令将一个仅执行只读命令的脚本杀死，因为只读命令并不修改数据，因此杀死这个脚本并不破坏数据的完整性
    4. 如果脚本已经执行过写命令，那么唯一允许执行的操作就是 `SHUTDOWN NOSAVE`，它通过停止服务器来阻止当前数据集写入磁盘

### 流水线 (pipeline) 上下文 (context) 中的 `EVALSHA`

- 在流水线请求的上下文中使用`EVALSHA`命令时，要特别小心，因为在流水线中，必须保证命令的执行顺序。
- 一旦在流水线中因为`EVALSHA`命令而发生NOSCRIPT错误，那么这个流水线就再也没有办法重新执行了，否则的话，命令的执行顺序就会被打乱。
- 为了防止出现以上所说的问题，客户端库实现应该实施以下的其中一项措施：
    - 总是在流水线中使用`EVAL`命令
    - 检查流水线中要用到的所有命令，找到其中的`EVAL`命令，并使用`SCRIPT EXISTS`命令检查要用到的脚本是不是全都已经保存在缓存里面了。如果所需的全部脚本都可以在缓存里找到，那么就可以放心地将所有`EVAL`命令改成`EVALSHA`命令，否则的话，就要在流水线的顶端 (top) 将缺少的脚本用`SCRIPT LOAD`命令加上去。

## EVALSHA

```
EVALSHA sha1 numkeys key [key ...] arg [arg ...]
```
- 根据给定的SHA1校验码，对缓存在服务器中的脚本进行求值。将脚本缓存到服务器的操作可以通过`SCRIPT LOAD`命令进行。这个命令的其他地方，比如参数的传入方式，都和`EVAL`命令一样。

## SCRIPT

### SCRIPT DEBUG

```
SCRIPT DEBUG YES|SYNC|NO
```
- 对后续使用`EVAL`执行脚本开启调试模式。Redis包含完整Lua Debugger和codename LDB，这大大降低了复杂脚本编写的难度。在调试模式下，Redis既做调试服务器又做客户端，像redis-cli可以单步执行，设置断点，观察变量等等，更多LDB信息参见Redis Lua debugger
- 注意：使用开发环境Redis服务器调试Lua脚本，避免在生产环境Redis服务器调试。
- LDB可以设置成两种模式：同步和异步。异步模式下，服务器会创建新的调试连接，不阻塞其他连接，同时在调试连接结束后会回滚所有的数据修改，这可以保证再次调试时初始状态不变。同步模式下，调试过程中，服务器其他连接会被阻塞，当调试结束后，所有的数据修改会被保存。

    | 模式 | 说明                                                 |
    | ---- | ---------------------------------------------------- |
    | YES  | 打开非阻塞异步调试模式，调试Lua脚本 (回退数据修改) |
    | SYNC | 打开阻塞同步调试模式，调试Lua脚本 (保留数据修改稿) |
    | NO   | 关闭脚本调试模式                                     |

### SCRIPT EXISTS

```
SCRIPT EXISTS script [script ...]
```
- 检查脚本是否存在脚本缓存里面。
- 这个命令可以接受一个或者多个脚本SHA1信息，返回一个1或者0的列表，如果脚本存在或不存在。
- 还可以使用管道技术（pipelining operation）确保脚本加载（也可以使用`SCRIPT LOAD`），管道技术可以单独使用 `EVALSHA`来代替 `EVAL`，从而节省带宽（bandwidth）。
- 更多细节信息请参考 [EVAL](#eval)。

### SCRIPT FLUSH

```
SCRIPT FLUSH
```
- 清空Lua脚本缓存。
- 更多细节信息请参考 [EVAL](#eval)。

### SCRIPT KILL

```
SCRIPT KILL
```
- 杀死当前正在运行的Lua脚本，当且仅当这个脚本没有执行过任何写操作时，这个命令才生效。
- 这个命令主要用于终止运行时间过长的脚本，比如一个因为BUG而发生无限loop的脚本，诸如此类。
- `SCRIPT KILL` 执行之后，当前正在运行的脚本会被杀死，执行这个脚本的客户端会从`EVAL`命令的阻塞当中退出，并收到一个错误作为返回值。
- 另一方面，假如当前正在运行的脚本已经执行过写操作，那么即使执行`SCRIPT KILL`，也无法将它杀死，因为这是违反Lua脚本的原子性执行原则的。在这种情况下，唯一可行的办法是使用 `SHUTDOWN NOSAVE`命令，通过停止整个Redis进程来停止脚本的运行，并防止不完整 (half-written) 的信息被写入数据库中。
- 关于使用Redis对Lua脚本进行求值的更多信息，请参见 [EVAL](#eval)。

### SCRIPT LOAD

```
SCRIPTLOAD script
```
- 将脚本script添加到脚本缓存中，但并不立即执行该脚本。在脚本被加入到缓存之后，通过`EVALSHA`命令，可以使用脚本的SHA1校验和来调用这个脚本。`EVAL` 命令也会将脚本添加到脚本缓存中，但是它会立即对输入的脚本进行求值。
- 脚本可以在缓存中保留无限长的时间 (直到执行`SCRIPT FLUSH`为止) 如果给定的脚本已经在缓存里面了，那么不做动作。- 关于使用Redis对Lua脚本进行求值的更多信息，请参见 [EVAL](#eval)。
