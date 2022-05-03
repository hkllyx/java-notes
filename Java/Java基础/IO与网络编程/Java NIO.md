# Java NIO

[java.nio.xmind](../../../resource/xmind/java.nio.xmind)

Java NIO是Java 4之后新出的一套IO接口，这里的“N”是相对于原有标准的Java IO和Java网络编程接口，NIO提供了一种完全不同的操作方式。当然，NIO中的“N”还可以理解为“Non-blocking”，而不是单纯的“New”。

**标准的Java IO编程接口是面向“流”的**。而**NIO是面向缓冲区（“块”）和基于通道**的，数据总是从通道中读到缓冲区内，或者从缓冲区写入到通道中。

Java NIO使我们可以进行非阻塞IO操作。比如说，单线程中从通道读取数据到缓冲区，同时可以继续做别的事情，当数据读取到缓冲区中后，线程再继续处理数据。写数据时也是同理。

NIO中有一个“Slectors”的概念。Selector可以检测多个通道的事件状态（例如：链接打开，数据到达）这样单线程就可以操作多个通道的数据。

## 通道（Channel）

Java NIO的通道（Channel）和流非常相似，主要有以下几点区别：

- 通道可以读也可以写，流一般来说是单向的（只能读或者写）。
- 通道可以异步读写。
- 通道总是基于缓冲区来读写。

通道包括以下类型：

- `FileChannel`：从文件中读写数据；
- `DatagramChannel`：通过UDP读写网络中数据；
- `SocketChannel`：通过TCP读写网络中数据；
- `ServerSocketChannel`：可以监听新进来的TCP连接，对每一个新进来的连接都会创建一个`SocketChannel`。

### `FileChannel`

`FileChannel`是用于连接文件的通道。通过文件通道可以读、写文件的数据。`FileChannel`是相对标准Java IO API的可选接口。

`FileChannel`不可以设置为非阻塞模式，他只能在阻塞模式下运行。

```java
RandomAccessFile raf = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel fc = raf.getChannel();
```

### `ServerSocketChannel`

`ServerSocketChannel`是用于监听TCP链接请求的通道，正如Java网络编程中的`ServerSocket`一样。

```java
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.socket().bind(new InetSocketAddress(8888));
```

### `SocketChannel`

`SocketChannel`是用于TCP网络连接的套接字接口，相当于Java网络编程中的`Socket`接口。

```java
SocketChannel sc = SocketChannel.open();
sc.connect(new InetSocketAddress("127.0.0.1", 80));
```

### `DatagramChannel`

`DatagramChannel`是用于UDP网络连接的套接字接口，相当于Java网络编程中的`DatagramSocket`接口。

```java
DatagramChannel dc = DatagramChannel.open();
dc.connect(new InetSocketAddress("127.0.0.1", 80));
```

## 缓冲区（Buffer）

发送给一个通道的所有数据都必须首先放到缓冲区（Buffer）中，同样地，从通道中读取的任何数据都要先读到缓冲区中。也就是说，不会直接对通道进行读写数据，而是要先经过缓冲区。

缓冲区实质上是一个数组，但它不仅仅是一个数组。缓冲区提供了对数据的结构化访问，而且还可以跟踪系统的读/写进程。

缓冲区包括以下类型：

- `ByteBuffer`
- `CharBuffer`
- `ShortBuffer`
- `IntBuffer`
- `LongBuffer`
- `FloatBuffer`
- `DoubleBuffer`

所以的缓冲区都可读，但不一定可写，可以调用`isReadOnly()`检测。

多个并发线程使用缓冲区是不安全的。如果一个缓冲区要被多个线程使用，那么应该通过适当的同步来控制对缓冲区的访问。

### 新建缓冲区

缓冲区类的构造方法均是省缺访问修饰符，所以不可以使用`new`关键字创建缓冲区实例，而是通过其提供的静态方法来创建。

```java
ByteBuffer bf1 = ByteBuffer.allocate(1024);

ByteBuffer bf2 = ByteBuffer.wrap(new byte[10]);
ByteBuffer bf3 = ByteBuffer.wrap(new byte[10], 0, 10);
```

### 缓冲区状态变量

- `capacity`：缓冲区的容量。非负且不可变。
- `limit`：第一个不该读/写的的元素的位置。`limit`$\leq$`capacity`。
- `position`：表示当前位置，是当前或者是下一个读/写的起始位置。非负且`position`$\leq$`limit`。

### 数据传输

数据传输：每一个`Buffer`都定义了`get()`和`put()`操作：

- 相对操作：利用`position`状态不变量。从当前位置开始读取或写入一个或多个元素，然后根据传输的元素数量增加该位置。如果请求的传输超过限制，则相对`get()`操作抛出`BufferUnderflowException`，而`put()`操作抛出`BufferOverflowException`。在这两种情况下，都不传输任何数据。如：`get()`、`put(byte)`。
- 绝对操作：采用内部数组的索引，不影响`position`。如果索引参数超过限制，绝对`get()`和`put()`操作将抛出`IndexOutOfBoundsException`。如：`get(int)`、`put(int, byte)`。
- 数据也可以通过适当通道（`Channel`）的IO操作（始终是相对操作）在缓冲区中或缓冲区外传输。

### 标记和重置

标记：缓冲区的标记是在调用`reset()`时将重置其位置的索引。标记并不总是被定义的，但当它被定义时，它永远不会是负的，也永远不会大于`position`。

如果标记被定义，那么当`position`或`limit`调整到比标记小的值时，它将被丢弃。如果没有定义标记，则调用`reset()`将引发`InvalidMarkException`。

**`0`$\leq$`mark`$\leq$`position`$\leq$`limit`$\leq$`capacity`**

### 快速操作方法及调用链

除了用于访问位置、限制和容量值以及标记和重置的方法之外，该类还定义了以下对缓冲区的操作:

- `clear()`
    - 清除，将`position`、`mark`恢复初始状态，`limit`设为`capacity`。使缓冲区为新的读取操作做好准备。
    - 实际上缓冲区中数据并没有清空，我们只是把标记为修改了。
    - 如果缓冲区还有一些数据没有读取完，调用`clear`就会导致这部分数据被“遗忘”，因为并没有设置什么机制标记这部分数据未读。针对这种情况，如果需要保留未读数据，那么可以使用`compact()`。因此`compact()`和`clear()`的区别就在于对未读数据的处理，是保留这部分数据还是一起清空。但是要注意`compact()`没有声明在`Buffer`类中，而是在`ByteBuffer`等子类中。

    ```java
    position = 0;
    limit = capacity;
    mark = -1;
    ```

- `flip()`：翻转，可以将缓冲区从写模式转换成读模式。使缓冲区为新的写入操作做好准备。

    ```java
    limit = position;
    position = 0;
    mark = -1;
    ```

- `rewind()`：重置，将`position`恢复成`0`，可以重复读取缓冲区内容。使缓冲区准备好重新读取它已经缓冲的数据。

    ```java
    position = 0;
    mark = -1;
    ```

链式调用：原本不需要返回值的方法，都将其返回值设调用他们的`Buffer`。如`clear()`/`flip()`/`limit()`/`mark()`/`reset()`。例：`buffer.flip().position(23).limit(42)`;

### 内存映射文件IO

内存映射文件IO是一种读和写文件数据的方法，它可以比常规的基于流或者基于通道的IO快得多。

向内存映射文件写入可能是危险的，只是改变数组的单个元素这样的简单操作，就可能会直接修改磁盘上的文件。修改数据与将数据保存到磁盘是没有分开的。

下面代码行将文件的前1024个字节映射到内存中，`map()`方法返回一个`MappedByteBuffer`，它是`ByteBuffer`的子类。因此，可以像使用其他任何`ByteBuffer`一样使用新映射的缓冲区，操作系统会在需要时负责执行映射。

```java
MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
```

## 选择器（Selector）

NIO常常被叫做非阻塞IO，主要是因为NIO在网络通信中的非阻塞特性被广泛使用。

NIO实现了IO多路复用中的Reactor模型，一个线程`Thread`使用一个选择器`Selector`通过轮询的方式去监听多个通道`Channel`上的事件，从而让一个线程就可以处理多个事件。

通过配置监听的通道`Channel`为非阻塞，那么当`Channel`上的IO事件还未到达时，就不会进入阻塞状态一直等待，而是继续轮询其它`Channel`，找到IO事件已经到达的`Channel`执行。

因为创建和切换线程的开销很大，因此使用一个线程来处理多个事件而不是一个线程处理一个事件，对于IO密集型的应用具有很好地性能。

应该注意的是，只有socket channel才能配置为非阻塞，而`FileChannel`不能，为`FileChannel`配置非阻塞也没有意义。

### 创建选择器

```java
Selector s = Selector.open();
```

### 注册`Channel`到`Selector`上

为了使`Selector`能够轮询一个`Channel`，必须先把`Channel`注册到`Selector`上

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

`Channel`必须是非阻塞的。所以`FileChannel`不适用`Selector`，因为`FileChannel`不能切换为非阻塞模式。socket channel则可以正常使用。

`register()`方法的第二个参数是一个“关注集”，代表我们关注的`Channel`状态，有四种基础状态可供监听：

- Connect：一个channel触发了一个事件也可视作该事件处于就绪状态。因此当channel与server连接成功后，那么就是“连接就绪”状态，对应`SelectionKey.OP_CONNECT`
- Accept：server channel接收请求连接时处于“可连接就绪”状态，对应`SelectionKey.OP_ACCEPT`
- Read：channel有数据可读时处于“读就绪”状态，对应`SelectionKey.OP_READ`
- Write：channel可以进行数据写入时处于“写就绪”状态，对应`SelectionKey.OP_WRITE`

如果对多个事件感兴趣可利用位的或运算结合多个常量，比如：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

### 常用方法

- `select()`：监听。选择一组其相应的通道已准备好进行IO操作的key。该方法执行阻塞选择操作。只有在至少选择了一个通道，或对此选择器调用`wakeup()`，或当前线程被中断（以先到者为准）后，它才会返回。
- `selectt(long)`：相比`select()`方法多加了一个阻塞时间，当到了指定时间仍未获取通道或是被中断，则会强行返回。
- `selectNow()`：类似`select()`，但该方法执行不阻塞选中操作。如果自上次选择操作后仍没有可选择（就绪）的通道，则此方法立即返回`0`。调用此方法可以清除以前调用`wakeup()`的效果。
- `wakeup()`：由于调用`select()`而被阻塞的线程，可以通过调用`wakeup()`来唤醒，即便此时已然没有channel处于就绪状态。具体操作是，在另外一个线程调用`wakeup()`，被`select()`的其他线程就会立刻返回。
- `close()`：关闭此选择器。如果一个线程当前在这个选择器的一个选择方法中被阻塞，那么它就像通过调用选择器的`wakeup()`方法一样被中断。仍然与此选择器关联的任何未取消的`SelectionKey`无效，其通道被注销，并且释放与此选择器关联的任何其他资源。如果此选择器已关闭，则调用此方法无效。关闭选择器之后，除了调用此方法或`wakeup()`方法之外，任何进一步尝试使用此选择器的方法都将抛出`ClosedSelectorException`。

### `SelectionKey`集

通道在选择器中注册时指定的“关注集”由`SelectionKey`对象表示。选择器内部维护三个`SelectionKey`集:

- key set：用此selector的注册的channel在注册时使用的“关注集”。这个集合由`keys()`获取。
- selected-key set：key set中对应的channel注册的“关注集”至少有一个就绪后，就会加入selected-key set，可以通过该set来获取就绪的channel。这个集合由`selectedKeys()`获取。selected-key set始终是key set的子集。
- cancelled-key set：是已取消关注但对应的通道尚未从selector注销的key的集合。不能直接访问（获取）。始终是key Set的子集。

在新创建的选择器中，这三个set都是空的。

### `SelectionKey`

`SelectionKey`的对象绑定了一些比较有价值的属性：

- interest set：这个“关注集合”实际上就是我们希望处理的事件的集合，它的值就是注册时传入的参数，我们可以用按为与运算把每个事件取出来。

    ```java
        boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
    ```

    同时，也直接提供了`intrestOps()`、`isAcceptable()`、`isConnectable()`、`isReadable()`、`isWritable()`等方法直接获取。后四个方法就是使用与运算的包装方法。
- ready set："就绪集合"中的值是当前channel处于就绪的值，一般来说在调用了`select()`方法后都会需要用到就绪状态。可以通过`readyOps()`、`isAcceptable()`、`isConnectable()`、`isReadable()`、`isWritable()`获取。
- channel：使用这个`SelectionKey`注册的通道。通过`channel()`方法获取。注意返回类型为`Channel`。
- selector：此`SelectionKey`注册的目标`Selector`。通过`selector()`获取。
- ttached object (optional)：可以给一个`SelectionKey`附加一个`Object`，这样做一方面可以方便我们识别某个特定的channel，同时也增加了channel关的附加信息。在`channel`注册时可以在`SelectionKey`上附加。

    ```java
    SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
    ```

    也可以直接在SelectionKey上附加。

    ```java
    selectionKey.attach(theObject);
    Object attachedObj = selectionKey.attachment();
    ```

## NIO与网络编程

### TCP

![NIO TCP通信模型](../../../resource/img/Java基础/NIO%20TCP通信模型.png)

### UDP

![NIO UDP通信模型](../../../resource/img/Java基础/NIO%20UDP通信模型.png)
