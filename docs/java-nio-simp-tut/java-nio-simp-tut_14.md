# 十四、14\. Java NIO vs. IO

> 原文链接：[`tutorials.jenkov.com/java-nio/nio-vs-io.html`](http://tutorials.jenkov.com/java-nio/nio-vs-io.html)

当学习 Java 的 NIO 和 IO 时，有个问题会跳入脑海当中：什么时候该用 IO，什么时候用 NIO？

下面的章节中笔者会试着分享一些线索，包括两者之间的区别，使用场景以及他们是如何影响代码设计的。

## NIO 和 IO 之间的主要差异（Mian Differences Between Java NIO and IO）

下面这个表格概括了 NIO 和 IO 的主要差异。我们会针对每个差异进行解释。

| IO | NIO |
| --- | --- |
| Stream oriented | Buffer oriented |
| Blocking IO | No blocking IO |
|  | Selectors |

## 面向流和面向缓冲区比较(Stream Oriented vs. Buffer Oriented)

第一个重大差异是 IO 是面向流的，而 NIO 是面向缓存区的。这句话是什么意思呢？

Java IO 面向流意思是我们每次从流当中读取一个或多个字节。怎么处理读取到的字节是我们自己的事情。他们不会再任何地方缓存。再有就是我们不能在流数据中向前后移动。如果需要向前后移动读取位置，那么我们需要首先为它创建一个缓存区。

Java NIO 是面向缓冲区的，这有些细微差异。数据是被读取到缓存当中以便后续加工。我们可以在缓存中向向后移动。这个特性给我们处理数据提供了更大的弹性空间。当然我们任然需要在使用数据前检查缓存中是否包含我们需要的所有数据。另外需要确保在往缓存中写入数据时避免覆盖了已经写入但是还未被处理的数据。

## 阻塞和非阻塞 IO 比较（Blocking vs. No-blocking IO）

Java IO 的各种流都是阻塞的。这意味着一个线程一旦调用了 read(),write()方法，那么该线程就被阻塞住了，知道读取到数据或者数据完整写入了。在此期间线程不能做其他任何事情。

Java NIO 的非阻塞模式使得线程可以通过 channel 来读数据，并且是返回当前已有的数据，或者什么都不返回如果但钱没有数据可读的话。这样一来线程不会被阻塞住，它可以继续向下执行。

通常线程在调用非阻塞操作后，会通知处理其他 channel 上的 IO 操作。因此一个线程可以管理多个 channel 的输入输出。

## Selectors

Java NIO 的 selector 允许一个单一线程监听多个 channel 输入。我们可以注册多个 channel 到 selector 上，然后然后用一个线程来挑出一个处于可读或者可写状态的 channel。selector 机制使得单线程管理过个 channel 变得容易。

## NIO 和 IO 是如何影响程序设计的（How NIO and IO Influences Application Design）

开发中选择 NIO 或者 IO 会在多方面影响程序设计：

1.  使用 NIO、IO 的 API 调用类
2.  数据处理
3.  处理数据需要的线程数

### API 调用(The API Calls)

显而易见使用 NIO 的 API 接口和使用 IO 时是不同的。不同于直接冲 InputStream 读取字节，我们的数据需要先写入到 buffer 中，然后再从 buffer 中处理它们。

### 数据处理（The Processing of Data）

数据的处理方式也随着是 NIO 或 IO 而异。 在 IO 设计中，我们从 InputStream 或者 Reader 中读取字节。假设我们现在需要处理一个按行排列的文本数据，如下：

```java
Name: Anna
Age: 25
Email: anna@mailserver.com
Phone: 1234567890
```

这个处理文本行的过程大概是这样的：

```java
InputStream input = ... ; // get the InputStream from the client socket

BufferedReader reader = new BufferedReader(new InputStreamReader(input));

String nameLine   = reader.readLine();
String ageLine    = reader.readLine();
String emailLine  = reader.readLine();
String phoneLine  = reader.readLine();
```

## 小结

NIO 允许我们只用一条线程来管理多个通道（网络连接或文件），随之而来的代价是解析数据相对于阻塞流来说可能会变得更加的复杂。

如果你需要同时管理成千上万的链接，这些链接只发送少量数据，例如聊天服务器，用 NIO 来实现这个服务器是有优势的。类似的，如果你需要维持大量的链接，例如 P2P 网络，用单线程来管理这些 链接也是有优势的。这种单线程多连接的设计可以用下图描述：

![nio-vs-io-3.png](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-3.png)

**图片 14.1** nio-vs-io-3.png

**Java NIO: A single thread managing multiple connections**

如果链接数不是很多，但是每个链接的占用较大带宽，每次都要发送大量数据，那么使用传统的 IO 设计服务器可能是最好的选择。下面是经典 IO 服务设计图：

![nio-vs-io-4.png](http://tutorials.jenkov.com/images/java-nio/nio-vs-io-4.png)

**图片 14.2** nio-vs-io-4.png

**Java IO: A classic IO server design - one connection handled by one thread.**