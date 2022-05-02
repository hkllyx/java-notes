# Java IO

[java.io.xmind](../../../resource/xmind/java.io.xmind)

## 文件操作

### `File`

```java
File extends Object implements Serializable, Comparable<File>
```

文件和目录路径名的抽象表示。可以用于对文件的增删改查。

`File`类内部包含一个`FileSystem`类的对象，`File`类的方法基本都是对`FileSystem`方法的包装。

#### `FileSystem`

表示当前的文件系统的包私有抽象类，在`Windows`平台下子类为`WinNTFileSystem`。

为`File`的磁盘操作提供信息及工具，如文件名分隔符等等。

#### `FileFilter` & `FilenameFilter`

用于过滤文件的接口。二者均是函数式接口,不同的是内部`accept()`方法接受的参数不同。

`FileFilter`接受一个`File`对象。`FilenameFilter`接受一个`File`对象表示目录，然后接受一个`String`对象表示文件名。

### `FileDescriptor`

```java
FileDescriptor extends Object
```

文件描述符类的实例用作表示打开的文件、开放套接字或其他字节源或信宿的底层机器特定结构的不透明句柄。主要实际用途是创建一个`FileInputStream`或`FileOutputStream`。

`FileDescriptor`表示文件时，可以将`FileDescriptor`看成是该文件。但是，不能直接通过 `FileDescriptor`对该文件进行操作，需要新创建`FileDescriptor`对应的`FileInputStream`或`FileOutputStream`，再对文件进行操作。

应用程序不应创建自己的文件描述符。

`FileDescriptor`有三个静态常量

- `sdOut`（`System.out`的句柄）
- `sdIn`（`System.in`的句柄）
- `sdErr`（`System.err`的句柄）

### `RandomAccessFile`

```java
RandomAccessFile extends Object
    implements DataOutput, DataInput, Closeable
```

该类的实例支持读取和写入随机访问文件。

随机访问文件的行为类似于存储在文件系统中的大量字节，也可以看出一个**隐式数组**。有一个游标，或是隐式数组的索引，称为文件指针。文件指针可以通过`getFilePointer()`读取，和由`seek(long)`设置。

输入操作读取从文件指针开始的字节，并使文件指针超过读取的字节。如果在读/写模式下创建随机访问文件，则输出操作也可用。输出操作从文件指针开始写入字节，并使文件指针超过写入的字节。文件指针写入时超出隐式数组的范围会导致其扩展。

读写模式

| 字符串表示 | 读写     | 说明                                                               |
| :--------- | :------- | :----------------------------------------------------------------- |
| `r`        | 仅读     | 调用任何写入方法都将抛出`IOException`                              |
| `rw`       | 可读可写 | 如果该文件尚不存在，则将尝试创建该文件                             |
| `rws`      | 可读可写 | 与`rw`一样，并且还要求将文件内容或元数据的更新同步写入底层存储设备 |
| `rwd`      | 可读可写 | 与`rw`一样，并且还要求将文件内容的更新同步写入底层存储设备         |

在这个类中的所有读取都为`true`。但如果在读取所需的字节数之前到达文件结尾（EOF），则会抛出`EOFException`（一种`IOException`）。如果任何字节由于除达到文件末尾之外的任何原因而无法读取，则抛出`IOException`而非`EOFException`。例如，如果流已经被关闭，则会抛出`IOException`。

## 字节操作

### `InputStream` & `OutputStream`

```java
InputStream extends Object implements Closeable
OutputStream extends Object implements Closeable, Flushable
```

抽象类。是所以表示输入/输出字节流的类的超类。`InputStream`的子类必须实现的方法（抽象方法）是`read()`，`OutputStream`的子类必须实现的方法是`write(int)`。

### `ByteArrayInputStream` & `ByteArrayOutputStream`

```java
ByteArrayInputStream extends InputStream
ByteArrayOutputStream extends OutputStream
```

`ByteArrayInputStream`实际上没有开启输入流，只是在内部使用一个缓冲（`byte[]`），其中包含可以从Stream中读取的字节。缓冲可以是另一个输入流从文件读取的，也可以是直接内存获取（定义数组传入）。

`ByteArrayOutputStream`实现了将数据写入`byte[]`的输出流。当数据写入缓冲区时，缓冲区会自动增长（增长一倍）。数据可以使用`toByteArray()`和`toString()`检索。

`ByteArrayInputStream`和`ByteArrayOutputStream`是直接和内存交互的输入输出流，可以将他们看成特殊的数组。

关闭`ByteArrayInputStream`和`ByteArrayOutputStream`没有任何效果（空实现）。在关闭流之后，可以调用此类中的方法，而不抛出`IOException`。

内部使用`Synchronized`关键字保证线程安全

### `FileInputStream` & `FileOutputStream`

```java
FileInputStream extends InputStream
FileOutputStream extends OutputStream
```

`FileInputStream`从文件系统中的文件获取输入字节，什么文件可读由底层主机环境觉得。可用于读取诸如图像数据的原始字节流。若是字符文件，则优先考虑使用`FileReader`。

`FileOutputStream`用于将数据写入到`File`或`FileDescriptor`的输出流，文件是否可用或可能被创建取决于底层平台（某些平台一次只能允许打开一个文件来写入一个`FileOutputStream`或其他文件写入对象，在这种情况下，如果所涉及的文件已经打开，则此类中的构造函数将失败）。可用于写入诸如图像数据的原始字节流。 对于写入字符文件，优先考虑使用`FileWriter`。

内部使用大量的`native`方法

### `FilterInputStream` & `FilterOutputStream`

```java
FilterInputStream extends InputStream
FilterOutputStream extends OutputStream
```

是所有过滤输入/输出流类的超类。

`FilterOutputStream`内部包含一个已经存在的输入流，用作其基本的数据源.`FilterOutputStream`内部则包含一个已经存在的输入流。在操作过程中可能会对数据进行转换或提供附加功能，用作其基本数据接收器。

本身不值得关注，只是简单地覆盖了父类所有的方法，方法内将所有请求传递给内部包含的输入流。其子类可以覆盖这些方法中的一些，并且还可以提供附加的方法和领域。

### `BufferedInputStream` & `FilterOutputStream`

```java
BufferedInputStream extends FilterInputStream
BufferedOutputStream extends FilterOutputStream
```

当创建`BufferedInputStream`时，会在内部创建一个缓冲区数组。当从流中读取或跳过字节时，内部缓冲区将根据需要从所包含的输入流中重新填充，一次填充多个字节。

`BufferedInputStream`为内部输入流添加了功能，即缓冲输入和支持`mark()`和`reset()`方法的功能。`mark()`操作会记住输入流中的一点，`reset()`操作会导致从最近的`mark()`操作记住的点之后到`reset()`操作时的点中间所有的数据被前重新读取。（即`reset()`会是读取位置返回到最近的`mark()`操作记住的点）。

`BufferedOutputStream`实现缓冲输出流。通过设置这样的输出流，应用程序可以向底层输出流一次写入多个字节，而不必为每个字节的写入都调用底层系统。

### `DataInputStream` & `DataOutputStream`

```java
DataOutputStream extends FilterOutputStream implements DataOutput
DataInputStream extends FilterInputStream implements DataInput
```

允许应用程序以独立于机器的方式便携得从底层输入流读取/写入Java基本数据类型。

数据输出流使应用程序以便携式方式将原始Java数据类型写入输出流。然后应用程序可以使用数据输入流来读取数据。

对于多线程访问来说不一定是安全的。线程安全是可选的。

### `PushbackInputStream`

```java
PushbackInputStream extends FilterInputStream
```

`PushbackInputStream`退回流读取到指定的一个值或一组值时，可以退回读取前的位置，给用户第二次读取的机会。

可以利用该机制跳过读取某些值，或改变其值。对于方便地读取代码片段中数量不定的由特定字节构成的数据非常有用。读取数据的终止字节后，可以使用`unread()`使数据未读，以便输入流上的下一个读操作将重新读取被推回的字节。

### `PrintStream`

```java
PrintStream extends FilterOutputStream
```

`PrintStream`自身处理了异常，所以永远不会抛出异常。

可以在初始化时设置是否自动冲洗缓冲（autoFlush）。

字符使用平台的默认字符编码字节转换成字节输出。在需要编写字符而不是字节的情况下，应使用 `PrintWriter`类。

`println()`方法可以自动在末尾输出`\n`。

`System.out`就是一个`PrintStream`对象。

### `PipedOutputStream` & `PipedOutputStream`

```java
PipedInputStream extends InputStream
PipedOutputStream extends OutputStream
```

`PipedInputStream`应连接到`PipedOutputStream`。`PipedInputStream`取`PipedOutputStream`提供的任意数据字节。

通常，数据被从由一个线程持有的读`PipedInputStream`，传输到到由一个其他线程持有的对应`PipedOutputStream`。不建议尝试从单个线程使用这两个流的对象，因为它可能会造成线程死锁。

如果正在提供的数据字节到管道中的`PipedInputStream`/`PipedOutputStream`的线程不再存活，管道被认为是破裂的（broken）。

`PipedInputStream`内部包含一个缓冲区，将读取和写入操作相分离。

### `SequenceInputStream`

```java
SequenceInputStream extends InputStream
```

表示其他输入流的逻辑级联。在内部使用一个`Enumeration`保存一系列的`InputStream`。

从第一个读取到文件的结尾，然后从第二个文件读取，依此类推，直到最后一个输入流达到文件的结尾。

注意，只是逻辑上的简单关联，所以`available()`只能返回正在读取的输入流的可读或可跳过字节数；`read(byte[], int, int)`也只能读取当前正在读的输入流。

## 字符操作

### `Reader` & `Writer`

```java
Reader extends Object
    implements Closeable, AutoCloseable, Readable

Writer extends Object
    implements Closeable, Flushable, Appendable, AutoCloseable
```

抽象类。是所有字符输入/输出流的父类

`Reader`的子类必须实现方法是`read(char[]，int，int)`和`close()`。

`Writer`的子类必须实现的方法是`write(char[]，int，int)`，`flush()`和`close()`。

然而，大多数子类将覆盖这里定义的一些其他方法，以便提供更高的效率，附加的功能。

内部使用`synchronized`关键字（锁实例而非类）保证线程安全。

### `CharArrayReader` & `CharArrayWriter`

```java
CharArrayReader extends Reader
CharArrayWriter extends Writer
```

类似`ByteArrayInputStream`和`ByteArrayOutputStream`。

类内部使用`char[]`实现了一个字符缓冲区。

`CharArrayReader`当数据写入缓冲区时，缓冲区会自动增长（增长一倍）。数据可以使用 `toCharArray()`和`toString()`检索。

关闭没有任何效果。

### `InputStreamReader` & `OutputStreamReader`

```java
InputStreamReader extends Reader
OutputStreamWriter extends Writer
```

是从字节流到字符流的桥梁：它读取字节后使用指定的字符集将字节解码为字符。使用的字符集可以由字符集的名称指定，也可以被明确指定某个字符集（`java.nio.charset.Charset`），或者是使用平台的默认字符集。

`InputStreamReader`调用`read()`方法可能会从底层的字节输入流读取一个或多个字节。为了使字节更高效地转换为字符，可以从底层流读取比满足当前读取操作的更多字节。为了最大的效率，应考虑在使用`BufferedReader`包装`InputStreamReader`。

`OutputStreamWriter`调用`write()`方法都会使编码转换器在给定字符上被调用。所得到的字节在写入底层输出流之前累积在缓冲区中。可以指定此缓冲区的大小，但一般它的大小都是足够的。请注意，传递给`write()`方法的字符不会缓冲，缓冲的是编码后的字节。为了最大的效率，应考虑在使用`BufferedWriter`包装`OutputStreamWriter`，以避免频繁的转换器调用。

surrogate pair是由两个`char`值组成。高surrogate为`\uD800`-`\uDBFF`，后面跟着低surrogate，`\uDC00`-`\uDFFF`。一个错误的surrogate元素是一个高surrogate后面没有跟着低surrogate，或低surrogate前面没有高surrogate。

`OutputStreamWriter`用字符集的默认替换序列（substitution sequence）替换格式不正确的surrogate元素和不可映射的字符序列。当需要对编码过程进行更多控制时，应使用`java.nio.charset.CharsetEncoder`类。

#### 字符集

编码就是把字符转换为字节，而解码是把字节重新组合成字符。如果编码和解码过程使用不同的编码方式那么就出现了乱码。

字符集就是编码/转码的规则。

常见字符集：

- gbk：中文字符占2个字节，英文字符占1个字节；
- utf-8：中文字符占3个字节，英文字符占1个字节；
- utf-16：中文字符和英文字符都占2字节。
    - utf-16be：be指Big Endian（大头）。例如：`a`（ascii为`0x61`），使用utf-16be就是`[0x00,0x61]`。
    - utf-16le，le指Little Endian（小头）。例如：`a`（ascii为`0x61`），使用utf-16le就是`[0x61,0x00]`。

Java的内存编码使用双字节编码utf-16be，这不是指Java只支持这一种编码方式，而是说`char`这种类型使用utf-16be进行编码。`char`类型占16位，也就是两个字节，Java使用这种双字节编码是为了让一个中文或者一个英文都能使用一个`char`来存储。

### `FileReader` & `FileWriter`

```java
FileReader extends InputStreamReader
FileWriter extend OutputStreamWriter
```

读取和输出字符文件的便利类。类的构造函数指定默认字符编码和合适的字节缓冲区大小。

内部没有任何新增方法或覆盖父类的方法，只是将一个`FileInputStream`/`FileOutputStream`包装传递给父类的构造器。

用于读取字符流。要读取/输出原始字节流，应使用`FileInputStream`/`FileOutputStream`。

### `FilterReader` & `FilterWriter`

```java
abstract FilterReader extends Reader
abstract FilterWriter extends Writer
```

抽象类。是所以过滤字符输入/输出流的父类

抽象类本身提供了将所有请求传递给包含流的默认方法。子类应覆盖其中一些方法，还可能提供其他方法和字段。

类似`FilterInputStream`和`FilterOutputStream`，但子类更少，因为`Reader`和`Writer`本身就相当于对字节流过滤和保证，在`java.io`包中只有一个子类：`PushbackReader`。

### `PushbackReader`

```java
PushbackReader extends FilterReader
```

`PushBackInputStream`字符输出流形式。

### `BufferedReader` & `BufferedWriter`

```java
BufferedReader extends Reader
BufferedWriter extends Writer
```

`BufferedInputStream`和`FilterOutputStream`的字符流形式。

### `PrintWriter`

```java
PrintWriter extends Writer
```

`PrintStream`的字符流形式。

### `PipedReader` & `PipedWriter`

```java
PipedReader extends Reader
PipedWriter extends Writer
```

`PipedOutputStream`和`PipedOutputStream`的字符流形式

### `StringReader` & `StringWriter`

```java
StringReader extends Reader
StringWriter extends Writer
```

`StringReader`一个字符流，其数据源是一个字符串。

`StringWriter`在字符串缓冲区中收集其输出的字符流，然后可以用于构造字符串。内部是一个`StringBuffer`。

关闭`StringWriter`没有任何效果。在流已关闭后，可以调用此类中的方法，而不抛出`IOException`。

## 对象操作

### `ObjectInputStream` & `ObjectOutputStream`

```java
ObjectOutputStream extends OutputStream
    implements ObjectOutput, ObjectStreamConstants

ObjectInputStream extends InputStream
    implements ObjectInput, ObjectStreamConstants
```

`ObjectOutputStream`将基本类型类型和Java对象图写入`OutputStream`。可以使用 `ObjectInputStream`读取（重新构造）对象。

对象的持久存储可以通过使用文件输入字节流来实现。

如果流是网络套接字流（Socket），则可以在另一台主机或另一个进程上重新构造对象。

只有实现了`java.io.Serializable`接口的类实例才能被写入`ObjectOutputStream`。

### `java.io.Serializable`

类的序列化由实现`java.io.Serializable`接口启用。不实现此接口的类不能序列化或反序列化。可序列化类的所有子类都是可序列化的。序列化接口没有方法或字段，仅用于标识可串行化的语义。

为了保证不可序列化类的子类可以序列化，子类需承担保存和恢复父类的`public`，`protected`和包（如果子类和父类在同一个包，可访问）字段的状态的责任。只有当**父类具有可访问的无参数构造函数**来初始化类的状态子类才能承担这些责任。否则，声明这个子类`Serializable`会造成错误。错误将在运行时检测到。

在反序列化期间，不可序列化的父类的字段将使用该类的`public`或`protected`无参构造函数进行初始化。对于可序列化的子类，必须可以访问无参构造函数。可序列化子类的字段将从流中恢复。

当遍历对象图时，可能会遇到不支持`Serializable`接口的对象。在这种情况下，将抛出`NotSerializableException`，并将标识不可序列化对象的类。

在序列化和反序列化过程中需要特殊处理的类必须在类中采用这些方法：

```java
private void writeObject(java.io.ObjectOutputStream out) throws IOException

private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;

private void readObjectNoData() throws ObjectStreamException;
```

- `writeObject()`方法负责为其特定的类编写对象的状态，以便相应的`readObject()`方法可以恢复它。可以通过调用`out.defaultWriteObject()`来调用保存对象字段的默认机制。该方法不需要关注属于其超类或子类的状态。通过使用`writeObject()`方法或通过使用`DataOutput`接口的实现类将基本数据类型字段写入来保存状态。
- `readObject()`方法负责从流中读取并恢复类字段。可以通过调用`in.defaultReadObject()`来调用恢复对象的非静态和非瞬态字段的默认机制。`defaultReadObject()`方法使用流中的信息将保存在流中的对象的字段分配给当前对象中相应命名的字段。当处理类进化到添加新字段时，这将处理这种情况。该方法不需要关注属于其超类或子类的状态。通过使用`writeObject()`方法或通过使用`DataOutput()`接口的实现类将基本数据类型写入恢复状态。
- `readObjectNoData()`方法负责初始化父类不可反序列的特定类的对象的状态。这可能发生在接收方使用与发送方不同的反序列化实例的类的版本的情况下，并且接收者的版本扩展了不被发送者版本扩展的类。 如果序列化流已被篡改，也可能发生这种情况; 因此，尽管存在“敌对”或不完整的源流，`readObjectNoData()`可用于正确初始化反序列化对象。

若要在将对象写入到流中时替换一些内容需要实现：

```java
ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
```

如果该方法存在并且可以通过在被序列化的对象的类中定义的方法来访问，则该`writeReplace()`方法通过序列化来调用。因此，该方法可以具有私有，受保护和包私有访问。子类访问此方法遵循Java可访问性规则。

相应地，当从流中读取实例时需要指定替换的类应实现。

```java
ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
```

这个`readResolve()`方法遵循与`writeReplace()`相同的调用规则和可访问性规则。

序列化运行时将每个可序列化的类与称为`serialVersionUID`的版本号相关联，该序列号在反序列化期间用于验证序列化对象的发送者和接收者是否已加载与该序列化兼容的对象的类。

如果接收方加载了一个具有不同于相应发件人类的`serialVersionUID`的对象的类，则反序列化将导致`InvalidClassException`。一个可序列化的类可以通过声明一个名为`serialVersionUID`的字段来显式地声明它自己的`serialVersionUID`，该字段必须是`static final long`：

```java
ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
```

如果可序列化类没有显式声明`serialVersionUID`，则序列化运行时将根据Java（TM）对象序列化规范中所述的类的各个方面计算该类的默认`serialVersionUID`值。

但是，强烈建议所有可序列化的类都明确声明`serialVersionUID`值，因为默认的`serialVersionUID`计算对类详细信息非常敏感，这可能会因编译器实现而异，因此可能会在反序化期间导致`InvalidClassException`。

因此，为了保证不同Java编译器实现之间的一致的`serialVersionUID`值，一个可序列化的类必须声明一个显式的`serialVersionUID`值。还强烈建议，显式的`serialVersionUID`声明在可能的情况下使用`private`修饰符，因为这种声明仅适用于立即声明的类`serialVersionUID`字段在子类失效。

数组类不能声明一个显式的`serialVersionUID`，所以它们总是具有默认的计算值，但是对于数组类，放弃了匹配`serialVersionUID`值的要求。

### Externalizable

```java
public interface Externalizable extends java.io.Serializable {

    void writeExternal(ObjectOutput out) throws IOException;

    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

只有实现`Externalizable`的类的实例的身份才能写入序列化流中，并且该类负责保存和恢复其实例的内容。

`Externalizable`接口的`writeExternal()`和`readExternal()`方法由一个类来实现，可以使类完全控制对象及其超类型的流的格式和内容。这些方法必须明确地与超类型协调来保存其状态。这些方法取代了`writeObject()`和`readObject()`方法的定制实现。

对象序列化使用`Serializable`和`Externalizable`接口。对象持久化机制也可以使用它们。要存储的每个对象都被测试为`Externalizable`接口。 如果对象支持`Externalizable`，则调用`writeExternal()`方法。 如果对象不支持`Externalizable`，并且实现`Serializable`，则使用`ObjectOutputStream`保存对象。

当一个`Externalizable`对象被重建时，使用`public`无参构造函数创建一个实例，然后调用`readExterna() `方法。可从`ObjectInputStream`读取可序列化对象。

`Externalizable`实例可以通过`Serializable`接口中记录的`writeReplace()`和`readResolve()`方法来指定一个替换对象。

## 装饰器模式

Java的流函数的整体架构是"装饰器设计模式"，也就是说，所有的流函数都可以按照所需的功能进行任意组合、互相嵌套、包裹。而我们的字符型流处理函数本质上也是对字节型流处理函数的一次包装、或者说加载了字节型流处理函数的功能。

以`InputStream`为例：

- `InputStream`是抽象组件；
- `FileInputStream`是`InputStream`的子类，属于具体组件，提供了字节流的输入操作；
- `FilterInputStream`属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如`BufferedInputStream`为`FileInputStream`提供缓存的功能。

实例化一个具有缓存功能的字节流对象时，只需要在`FileInputStream`对象上再套一层`BufferedInputStream`对象即可。

```java
FileInputStream fis = new FileInputStream(filePath);
BufferedInputStream bis = new BufferedInputStream(fis);
```

`DataInputStream`装饰者提供了对更多数据类型进行输入的操作，比如`int`、`double`等基本类型。
