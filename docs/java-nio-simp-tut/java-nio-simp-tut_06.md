# 六、06\. Java NIO Channel to Channel Transfers 通道传输接口

> 原文链接：[`tutorials.jenkov.com/java-nio/channel-to-channel-transfers.html`](http://tutorials.jenkov.com/java-nio/channel-to-channel-transfers.html)

在 Java NIO 中如果一个 channel 是 FileChannel 类型的，那么他可以直接把数据传输到另一个 channel。逐个特性得益于 FileChannel 包含的 transferTo 和 transferFrom 两个方法。

## transferFrom()

FileChannel.transferFrom 方法把数据从通道源传输到 FileChannel：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

toChannel.transferFrom(fromChannel, position, count);
```

transferFrom 的参数 position 和 count 表示目标文件的写入位置和最多写入的数据量。如果通道源的数据小于 count 那么就传实际有的数据量。 另外，有些 SocketChannel 的实现在传输时只会传输哪些处于就绪状态的数据，即使 SocketChannel 后续会有更多可用数据。因此，这个传输过程可能不会传输整个的数据。

## transferTo()

transferTo 方法把 FileChannel 数据传输到另一个 channel,下面是案例：

```java
RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
FileChannel      fromChannel = fromFile.getChannel();

RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
FileChannel      toChannel = toFile.getChannel();

long position = 0;
long count    = fromChannel.size();

fromChannel.transferTo(position, count, toChannel);
```

这段代码和之前介绍 transfer 时的代码非常相似，区别只在于调用方法的是哪个 FileChannel.

SocketChannel 的问题也存在与 transferTo.SocketChannel 的实现可能只在发送的 buffer 填充满后才发送，并结束。