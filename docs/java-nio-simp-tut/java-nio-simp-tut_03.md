# 三、03\. Java NIO Channel 通道

> 原文链接：[`tutorials.jenkov.com/java-nio/channels.html`](http://tutorials.jenkov.com/java-nio/channels.html)

Java NIO Channel 通道和流非常相似，主要有以下几点区别：

*   通道可以度也可以写，流一般来说是单向的（只能读或者写）。
*   通达可以异步读写。
*   通道总是基于缓冲区 Buffer 来读写。

正如上面提到的，我们可以从通道中读取数据，写入到 buffer；也可以中 buffer 内读数据，写入到通道中。下面有个示意图：

![overview-channels-buffers.png](http://tutorials.jenkov.com/images/java-nio/overview-channels-buffers.png)

**图片 3.1** overview-channels-buffers.png

**Java NIO: Channels read data into Buffers, and Buffers write data into Channels**

## Channel 的实现（Channel Implementations）

下面列出 Java NIO 中最重要的集中 Channel 的实现：

*   FileChannel
*   DatagramChannel
*   SocketChannel
*   ServerSocketChannel

FileChannel 用于文件的数据读写。 DatagramChannel 用于 UDP 的数据读写。 SocketChannel 用于 TCP 的数据读写。 ServerSocketChannel 允许我们监听 TCP 链接请求，每个请求会创建会一个 SocketChannel.

## Channel 的基础示例（Basic Channel Example）

这有一个利用 FileChannel 读取数据到 Buffer 的例子：

```java
    RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
    FileChannel inChannel = aFile.getChannel();

    ByteBuffer buf = ByteBuffer.allocate(48);

    int bytesRead = inChannel.read(buf);
    while (bytesRead != -1) {

      System.out.println("Read " + bytesRead);
      buf.flip();

      while(buf.hasRemaining()){
          System.out.print((char) buf.get());
      }

      buf.clear();
      bytesRead = inChannel.read(buf);
    }
    aFile.close();
```

注意 buf.flip()的调用。首先把数据读取到 Buffer 中，然后调用 flip()方法。接着再把数据读取出来。在后续的章节中我们还会讲解先关知识。