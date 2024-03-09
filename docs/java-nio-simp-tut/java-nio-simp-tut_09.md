# 九、09\. Java NIO SocketChannel 套接字通道

> 原文链接：[`tutorials.jenkov.com/java-nio/socketchannel.html`](http://tutorials.jenkov.com/java-nio/socketchannel.html)

在 Java NIO 体系中，SocketChannel 是用于 TCP 网络连接的套接字接口，相当于 Java 网络编程中的 Socket 套接字接口。创建 SocketChannel 主要有两种方式，如下：

1.  打开一个 SocketChannel 并连接网络上的一台服务器。
2.  当 ServerSocketChannel 接收到一个连接请求时，会创建一个 SocketChannel。

## 建立一个 SocketChannel 连接

打开一个 SocketChannel 可以这样操作：

```java
SocketChannel socketChannel = SocketChannel.open();
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));
```

## 关闭一个 SocketChannel 连接

关闭一个 SocketChannel 只需要调用他的 close 方法，如下：

```java
socketChannel.close();
```

## 从 SocketChannel 中读数据

从一个 SocketChannel 连接中读取数据，可以通过 read()方法，如下：

```java
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = socketChannel.read(buf);
```

首先需要开辟一个 Buffer。从 SocketChannel 中读取的数据将放到 Buffer 中。

接下来就是调用 SocketChannel 的 read()方法.这个 read()会把通道中的数据读到 Buffer 中。read()方法的返回值是一个 int 数据，代表此次有多少字节的数据被写入了 Buffer 中。如果返回的是-1,那么意味着通道内的数据已经读取完毕，到底了（链接关闭）。

## 向 SocketChannel 写数据

向 SocketChannel 中写入数据是通过 write()方法，write 也需要一个 Buffer 作为参数。下面看一下具体的示例：

```java
String newData = "New String to write to file..." + System.currentTimeMillis();

ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());

buf.flip();

while(buf.hasRemaining()) {
    channel.write(buf);
}
```

仔细观察代码，这里我们把 write()的调用放在了 while 循环中。这是因为我们无法保证在 write 的时候实际写入了多少字节的数据，因此我们通过一个循环操作，不断把 Buffer 中数据写入到 SocketChannel 中知道 Buffer 中的数据全部写入为止。

## 非阻塞模式

我们可以吧 SocketChannel 设置为 non-blocking（非阻塞）模式。这样的话在调用 connect(), read(), write()时都是异步的。

### connect()

如果我们设置了一个 SocketChannel 是非阻塞的，那么调用 connect()后，方法会在链接建立前就直接返回。为了检查当前链接是否建立成功，我们可以调用 finishConnect(),如下：

```java
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress("http://jenkov.com", 80));

while(! socketChannel.finishConnect() ){
    //wait, or do something else...    
}
```

### write()

在非阻塞模式下，调用 write()方法不能确保方法返回后写入操作一定得到了执行。因此我们需要把 write()调用放到循环内。这和前面在讲 write()时是一样的，此处就不在代码演示。

### read()

在非阻塞模式下，调用 read()方法也不能确保方法返回后，确实读到了数据。因此我们需要自己检查的整型返回值，这个返回值会告诉我们实际读取了多少字节的数据。

### Selector 结合非阻塞模式

SocketChannel 的非阻塞模式可以和 Selector 很好的协同工作。把一个活多个 SocketChannel 注册到一个 Selector 后，我们可以通过 Selector 指导哪些 channels 通道是处于可读，可写等等状态的。后续我们会再详细阐述如果联合使用 Selector 与 SocketChannel。