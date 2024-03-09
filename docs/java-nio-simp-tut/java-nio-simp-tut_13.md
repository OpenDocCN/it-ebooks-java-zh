# 十三、13.Java NIO Pipe 管道

> 原文链接：[`tutorials.jenkov.com/java-nio/pipe.html`](http://tutorials.jenkov.com/java-nio/pipe.html)

一个 Java NIO 的管道是两个线程间单向传输数据的连接。一个管道（Pipe）有一个 source channel 和一个 sink channel(没想到合适的中文名)。我们把数据写到 sink channel 中，这些数据可以同过 source channel 再读取出来。

下面是一个管道的示意图：

![`tutorials.jenkov.com/images/java-nio/pipe-internals.png`](http://tutorials.jenkov.com/images/java-nio/pipe-internals.png)

**图片 13.1** http://tutorials.jenkov.com/images/java-nio/pipe-internals.png

## 创建管道(Creating a Pipe)

打开一个管道通过调用 Pipe.open()工厂方法，如下：

```java
Pipe pipe = Pipe.open();
```

## 向管道写入数据（Writing to a Pipe）

向管道写入数据需要访问他的 sink channel：

```java
Pipe.SinkChannel sinkChannel = pipe.sink();
```

接下来就是调用 write()方法写入数据了：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    sinkChannel.write(buf);
}
```

## 从管道读取数据（Reading from a Pipe）

类似的从管道中读取数据需要访问他的 source channel：

```java
Pipe.SourceChannel sourceChannel = pipe.source();
```

接下来调用 read()方法读取数据：

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf);
```

注意这里 read()的整形返回值代表实际读取到的字节数。