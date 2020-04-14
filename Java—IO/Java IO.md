# JAVA  IO

# 一、概览

Java 的 I/O 大概可以分成以下几类：

- 磁盘操作：File
- 字节操作：InputStream 和 OutputStream
- 字符操作：Reader 和 Writer
- 对象操作：Serializable
- 网络操作：Socket
- 新的输入/输出：NIO

# 二、流式IO

> I/O 流屏蔽了实际的 I/O 设备中处理数据的细节：
>
> 1. 字节流对应原生的二进制数据；
> 2. 字符流对应字符数据，它会自动处理与本地字符集之间的转换；
> 3. 缓冲流可以提高性能，通过减少底层 API 的调用次数来优化 I/O。

从 JDK 文档的类层次结构中可以看到，Java 类库中的 I/O 类分成了输入和输出两部分。在设计 Java 1.0 时，类库的设计者们就决定让所有与输入有关系的类都继承自 `InputStream`，所有与输出有关系的类都继承自 `OutputStream`。所有从 `InputStream` 或 `Reader` 派生而来的类都含有名为 `read()` 的基本方法，用于读取单个字节或者字节数组。同样，所有从 `OutputStream` 或 `Writer` 派生而来的类都含有名为 `write()` 的基本方法，用于写单个字节或者字节数组。但是，我们通常不会用到这些方法，它们之所以存在是因为别的类可以使用它们，以便提供更有用的接口。

我们很少使用单一的类来创建流对象，而是通过叠合多个对象来提供所期望的功能（这是**装饰器设计模式**）

## InputStream

`InputStream` 表示那些从不同数据源产生输入的类，如**[表 I/O-1]**所示，这些数据源包括：

1. 字节数组；
2. `String` 对象；
3. 文件；
4. “管道”，工作方式与实际生活中的管道类似：从一端输入，从另一端输出；
5. 一个由其它种类的流组成的序列，然后我们可以把它们汇聚成一个流；
6. 其它数据源，如 Internet 连接。

每种数据源都有相应的 `InputStream` 子类。另外，`FilterInputStream` 也属于一种 `InputStream`，它的作用是为“装饰器”类提供基类。其中，“装饰器”类可以把属性或有用的接口与输入流连接在一起。

> 例如实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```java
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

**表 I/O-1 `InputStream` 类型**

|            类             | 功能                                                         | 构造器参数                                                   | 如何使用                                                     |
| :-----------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
|  `ByteArrayInputStream`   | 允许将内存的缓冲区当做 `InputStream` 使用                    | 缓冲区，字节将从中取出                                       | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
| `StringBufferInputStream` | 将 `String` 转换成 `InputStream`                             | 字符串。底层实现实际使用 `StringBuffer`                      | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
|     `FileInputStream`     | 用于从文件中读取信息                                         | 字符串，表示文件名、文件或 `FileDescriptor` 对象             | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
|    `PipedInputStream`     | 产生用于写入相关 `PipedOutputStream` 的数据。实现“管道化”概念 | `PipedOutputSteam`                                           | 作为多线程中的数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
|   `SequenceInputStream`   | 将两个或多个 `InputStream` 对象转换成一个 `InputStream`      | 两个 `InputStream` 对象或一个容纳 `InputStream`对象的容器 `Enumeration` | 作为一种数据源：将其与 `FilterInputStream` 对象相连以提供有用接口 |
|    `FilterInputStream`    | 抽象类，作为“装饰器”的接口。其中，“装饰器”为其它的 `InputStream` 类提供有用的功能。见[表 I/O-3] | 见**[表 I/O-3]**                                             | 见**[表 I/O-3]**                                             |

**表 I/O-3：`FilterInputStream` 类型**

|           类            | 功能                                                         | 构造器参数                                | 如何使用                                                 |
| :---------------------: | :----------------------------------------------------------- | :---------------------------------------- | :------------------------------------------------------- |
|    `DataInputStream`    | 与 `DataOutputStream` 搭配使用，按照移植方式从流读取基本数据类型（`int`、`char`、`long` 等） | `InputStream`                             | 包含用于读取基本数据类型的全部接口                       |
|  `BufferedInputStream`  | 使用它可以防止每次读取时都得进行实际写操作。代表“使用缓冲区” | `InputStream`，可以指定缓冲区大小（可选） | 本质上不提供接口，只是向进程添加缓冲功能。与接口对象搭配 |
| `LineNumberInputStream` | 跟踪输入流中的行号，可调用 `getLineNumber()` 和 `setLineNumber(int)` | `InputStream`                             | 仅增加了行号，因此可能要与接口对象搭配使用               |
|  `PushbackInputStream`  | 具有能弹出一个字节的缓冲区，因此可以将读到的最后一个字符回退 | `InputStream`                             | 通常作为编译器的扫描器，我们可能永远也不会用到           |

## OutputStream

如**[表 I/O-2]**所示，该类别的类决定了输出所要去往的目标：字节数组（但不是 `String`，当然，你也可以用字节数组自己创建）、文件或管道。

另外，`FilterOutputStream` 为“装饰器”类提供了一个基类，“装饰器”类把属性或者有用的接口与输出流连接了起来。

**表 I/O-2：`OutputStream` 类型**

|           类            | 功能                                                         | 构造器参数                                      | 如何使用                                                     |
| :---------------------: | :----------------------------------------------------------- | :---------------------------------------------- | :----------------------------------------------------------- |
| `ByteArrayOutputStream` | 在内存中创建缓冲区。所有送往“流”的数据都要放置在此缓冲区     | 缓冲区初始大小（可选）                          | 用于指定数据的目的地：将其与 `FilterOutputStream`对象相连以提供有用接口 |
|   `FileOutputStream`    | 用于将信息写入文件                                           | 字符串，表示文件名、文件或 `FileDescriptor`对象 | 用于指定数据的目的地：将其与 `FilterOutputStream`对象相连以提供有用接口 |
|   `PipedOutputStream`   | 任何写入其中的信息都会自动作为相关 `PipedInputStream`的输出。实现“管道化”概念 | `PipedInputStream`                              | 指定用于多线程的数据的目的地：将其与 `FilterOutputStream`对象相连以提供有用接口 |
|  `FilterOutputStream`   | 抽象类，作为“装饰器”的接口。其中，“装饰器”为其它 `OutputStream` 提供有用功能。见[表 I/O-4] | 见**[表 I/O-4]**                                | 见**[表 I/O-4]**                                             |



**表 I/O-4：`FilterOutputStream` 类型**

|           类           | 功能                                                         | 构造器参数                                                   | 如何使用                                                    |
| :--------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- |
|   `DataOutputStream`   | 与 `DataInputStream`搭配使用，因此可以按照移植方式向流中写入基本数据类型（`int`、`char`、`long`等） | `OutputStream`                                               | 包含用于写入基本数据类型的全部接口                          |
|     `PrintStream`      | 用于产生格式化输出。其中 `DataOutputStream`处理数据的存储，`PrintStream` 处理显示 | `OutputStream`，可以用 `boolean`值指示是否每次换行时清空缓冲区（可选） | 应该是对 `OutputStream`对象的 `final`封装。可能会经常用到它 |
| `BufferedOutputStream` | 使用它以避免每次发送数据时都进行实际的写操作。代表“使用缓冲区”。可以调用 `flush()` 清空缓冲区 | `OutputStream`，可以指定缓冲区大小（可选）                   | 本质上并不提供接口，只是向进程添加缓冲功能。与接口对象搭配  |

## Reader与Writer

>设计 `Reader` 和 `Writer` 继承体系主要是为了国际化。老的 I/O 流继承体系仅支持 8 比特的字节流，并且不能很好地处理 16 比特的 Unicode 字符。由于 Unicode 用于字符国际化（Java 本身的 `char` 也是 16 比特的 Unicode），所以添加 `Reader` 和 `Writer` 继承体系就是为了让所有的 I/O 操作都支持 Unicode。另外，新类库的设计使得它的操作比旧类库要快。

不管是磁盘还是网络传输，最小的存储单元都是字节，而不是字符。但是在程序中操作的通常是字符形式的数据，因此需要提供对字符进行操作的方法。

几乎所有原始的 Java I/O 流类都有相应的 `Reader` 和 `Writer` 类来提供原生的 Unicode 操作。但是在某些场合，面向字节的 `InputStream` 和 `OutputStream` 才是正确的解决方案。特别是 `java.util.zip` 类库就是面向字节而不是面向字符的。因此，最明智的做法是尽量**尝试**使用 `Reader` 和 `Writer`，一旦代码没法成功编译，你就会发现此时应该使用面向字节的类库了。

下表展示了在两个继承体系中，信息的来源和去处（即数据物理上来自哪里又去向哪里）之间的对应关系：

|       来源与去处：Java 1.0 类       |           相应的 Java 1.1 类           |
| :---------------------------------: | :------------------------------------: |
|            `InputStream`            | `Reader`  适配器：`InputStreamReader`  |
|           `OutputStream`            | `Writer`  适配器：`OutputStreamWriter` |
|          `FileInputStream`          |              `FileReader`              |
|         `FileOutputStream`          |              `FileWriter`              |
| `StringBufferInputStream`（已弃用） |             `StringReader`             |
|           （无相应的类）            |             `StringWriter`             |
|       `ByteArrayInputStream`        |           `CharArrayReader`            |
|       `ByteArrayOutputStream`       |           `CharArrayWriter`            |
|         `PipedInputStream`          |             `PipedReader`              |
|         `PipedOutputStream`         |             `PipedWriter`              |

| 过滤器：Java 1.0 类     | 相应 Java 1.1 类                                             |
| :---------------------- | :----------------------------------------------------------- |
| `FilterInputStream`     | `FilterReader`                                               |
| `FilterOutputStream`    | `FilterWriter` (抽象类，没有子类)                            |
| `BufferedInputStream`   | `BufferedReader`（也有 `readLine()`)                         |
| `BufferedOutputStream`  | `BufferedWriter`                                             |
| `DataInputStream`       | 使用 `DataInputStream`（ 如果必须用到 `readLine()`，那你就得使用 `BufferedReader`。否则，一般情况下就用 `DataInputStream` |
| `PrintStream`           | `PrintWriter`                                                |
| `LineNumberInputStream` | `LineNumberReader`                                           |
| `StreamTokenizer`       | `StreamTokenizer`（使用具有 `Reader` 参数的构造器）          |
| `PushbackInputStream`   | `PushbackReader`                                             |

##  小结

Java 的 I/O 流类库的确能够满足我们的基本需求：我们可以通过控制台、文件、内存块，甚至因特网进行读写。通过继承，我们可以创建新类型的输入和输出对象。并且我们甚至可以通过重新定义“流”所接受对象类型的 `toString()` 方法，进行简单的扩展。当我们向一个期望收到字符串的方法传送一个非字符串对象时，会自动调用对象的 `toString()` 方法（这是 Java 中有限的“自动类型转换”功能之一）。

在 I/O 流类库的文档和设计中，仍留有一些没有解决的问题。例如，我们打开一个文件用于输出，如果在我们试图覆盖这个文件时能抛出一个异常，这样会比较好（有的编程系统只有当该文件不存在时，才允许你将其作为输出文件打开）。在 Java 中，我们应该使用一个 `File` 对象来判断文件是否存在，因为如果我们用 `FileOutputStream` 或者 `FileWriter` 打开，那么这个文件肯定会被覆盖。

I/O 流类库让我们喜忧参半。它确实挺有用的，而且还具有可移植性。但是如果我们没有理解“装饰器”模式，那么这种设计就会显得不是很直观。所以，它的学习成本相对较高。而且它并不完善，比如说在过去，我不得不编写相当数量的代码去实现一个读取文本文件的工具——所幸的是，Java 7 中的 nio 消除了此类需求。

# 三、NIO（同步非阻塞）