# 五、05\. Java NIO Scatter / Gather

> 原文链接：[`tutorials.jenkov.com/java-nio/scatter-gather.html`](http://tutorials.jenkov.com/java-nio/scatter-gather.html)

Java NIO 发布时内置了对 scatter / gather 的支持。scatter / gather 是通过通道读写数据的两个概念。

Scattering read 指的是从通道读取的操作能把数据写入多个 buffer，也就是 sctters 代表了数据从一个 channel 到多个 buffer 的过程。

gathering write 则正好相反，表示的是从多个 buffer 把数据写入到一个 channel 中。

Scatter/gather 在有些场景下会非常有用，比如需要处理多份分开传输的数据。举例来说，假设一个消息包含了 header 和 body，我们可能会把 header 和 body 保存在不同独立 buffer 中，这种分开处理 header 与 body 的做法会使开发更简明。

## Scattering Reads

"scattering read"是把数据从单个 Channel 写入到多个 buffer，下面是示意图：

![scatter.png](http://tutorials.jenkov.com/images/java-nio/scatter.png)

**图片 5.1** scatter.png

**Java NIO: Scattering Read**

用代码来表示的话如下：

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

ByteBuffer[] bufferArray = { header, body };

channel.read(bufferArray);
```

观察代码可以发现，我们把多个 buffer 写在了一个数组中，然后把数组传递给 channel.read()方法。read()方法内部会负责把数据按顺序写进传入的 buffer 数组内。一个 buffer 写满后，接着写到下一个 buffer 中。

实际上，scattering read 内部必须写满一个 buffer 后才会向后移动到下一个 buffer，因此这并不适合消息大小会动态改变的部分，也就是说，如果你有一个 header 和 body，并且 header 有一个固定的大小（比如 128 字节）,这种情形下可以正常工作。

## Gathering Writes

"gathering write"把多个 buffer 的数据写入到同一个 channel 中，下面是示意图：

![gather.png](http://tutorials.jenkov.com/images/java-nio/gather.png)

**图片 5.2** gather.png

**Java NIO: Gathering Write**

用代码表示的话如下：

```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);

//write data into buffers

ByteBuffer[] bufferArray = { header, body };

channel.write(bufferArray);
```

类似的传入一个 buffer 数组给 write，内部机会按顺序将数组内的内容写进 channel，这里需要注意，写入的时候针对的是 buffer 中 position 到 limit 之间的数据。也就是如果 buffer 的容量是 128 字节，但它只包含了 58 字节数据，那么写入的时候只有 58 字节会真正写入。因此 gathering write 是可以适用于可变大小的 message 的，这和 scattering reads 不同。