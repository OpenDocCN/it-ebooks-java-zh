# 十二、12\. Java NIO DatagramChannel 数据报通道

> 原文链接：[`tutorials.jenkov.com/java-nio/datagram-channel.html`](http://tutorials.jenkov.com/java-nio/datagram-channel.html)

一个 Java NIO DatagramChannel 死一个可以发送、接收 UDP 数据包的通道。由于 UDP 是面向无连接的网络协议，我们不可用像使用其他通道一样直接进行读写数据。正确的做法是发送、接收数据包。

## 打开一个 DatagramChannel（Opening a DatagramChannel）

打开一个 DatagramChannel 你这么操作：

```java
DatagramChannel channel = DatagramChannel.open();
channel.socket().bind(new InetSocketAddress(9999));
```

上述示例中，我们打开了一个 DatagramChannel，它可以在 9999 端口上收发 UDP 数据包。

## 接收数据（Receiving Data）

接收数据，直接调用 DatagramChannel 的 receive()方法：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();

**channel.receive(buf);**
```

receive()方法会把接收到的数据包中的数据拷贝至给定的 Buffer 中。如果数据包的内容超过了 Buffer 的大小，剩余的数据会被直接丢弃。

## 发送数据（Sending Data）

发送数据是通过 DatagramChannel 的 send()方法：

```java
String newData = "New String to wrte to file..."               +System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();

**int byteSent = channel.send(buf, new InetSocketAddress("jenkov.com", 80));**
```

上述示例会吧一个字符串发送到“jenkov.com”服务器的 UDP 端口 80.目前这个端口没有被任何程序监听，所以什么都不会发生。当发送了数据后，我们不会收到数据包是否被接收的的通知，这是由于 UDP 本身不保证任何数据的发送问题。

## 链接特定机器地址（Connecting to a Specific Address）

DatagramChannel 实际上是可以指定到网络中的特定地址的。由于 UDP 是面向无连接的，这种链接方式并不会创建实际的连接，这和 TCP 通道类似。确切的说，他会锁定 DatagramChannel,这样我们就只能通过特定的地址来收发数据包。

看一个例子先：

```java
channel.connect(new InetSocketAddress("jenkov.com"), 80));
```

当连接上后，可以向使用传统的通道那样调用 read()和 Writer()方法。区别是数据的读写情况得不到保证。下面是几个示例：

```java
int bytesRead = channel.read(buf);
```

```java
int bytesWritten = channel.write(buf);
```