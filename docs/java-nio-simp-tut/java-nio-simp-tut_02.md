# 二、02\. Java NIO 概览

> 原文链接：[`tutorials.jenkov.com/java-nio/overview.html`](http://tutorials.jenkov.com/java-nio/overview.html)

NIO 包含下面几个核心的组件：

*   Channels
*   Buffers
*   Selectors

整个 NIO 体系包含的类远远不止这几个，但是在笔者看来 Channel,Buffer 和 Selector 组成了这个核心的 API。其他的一些组件，比如 Pipe 和 FileLock 仅仅只作为上述三个的负责类。因此在概览这一节中，会重点关注这三个概念。其他的组件会在各自的部分单独介绍。

## 通道和缓冲区（Channels and Buffers）

通常来说 NIO 中的所有 IO 都是从 Channel 开始的。Channel 和流有点类似。通过 Channel，我们即可以从 Channel 把数据写到 Buffer 中，也可以吧数据冲 Buffer 写入到 Channel，下图是一个示意图：

![`tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png`](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

**图片 2.1** http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png

**Java NIO: Channels read data into Buffers, and Buffers write data into Channels**

有很多的 Channel，Buffer 类型。下面列举了主要的几种：

*   FileChannel
*   DatagramChannel
*   SocketChannel
*   ServerSocketChannel

正如你看到的，这些 channel 基于于 UPD 和 TCP 的网络 IO，以及文件 IO。 和这些类一起的还有其他一些比较有趣的接口，在本节中暂时不多介绍。为了简介起见，我们会在必要的时候引入这些概念。 下面是核心的 Buffer 实现类的列表：

*   ByteBuffer
*   CharBuffer
*   DoubleBuffer
*   FloatBuffer
*   IntBuffer
*   LongBuffer
*   ShortBuffer

这些 Buffer 涵盖了可以通过 IO 操作的基础类型：byte,short,int,long,float,double 以及 characters. NIO 实际上还包含一种 MappedBytesBuffer,一般用于和内存映射的文件。

## 选择器（Selectors）

选择器允许单线程操作多个通道。如果你的程序中有大量的链接，同时每个链接的 IO 带宽不高的话，这个特性将会非常有帮助。比如聊天服务器。 下面是一个单线程中 Slector 维护 3 个 Channel 的示意图：

![`tutorials.jenkov.com/images/java-nio/overview-selectors.png`](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

**图片 2.2** http://tutorials.jenkov.com/images/java-nio/overview-selectors.png

**Java NIO: A Thread uses a Selector to handle 3 Channel's**

要使用 Selector 的话，我们必须把 Channel 注册到 Selector 上，然后就可以调用 Selector 的 selectr()方法。这个方法会进入阻塞，直到有一个 channel 的状态符合条件。当方法范湖后，线程可以处理这些时间。