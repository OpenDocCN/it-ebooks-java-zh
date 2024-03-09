# 十、10\. Java NIO ServerSocketChannel 服务端套接字通道

> 原文链接：[`tutorials.jenkov.com/java-nio/server-socket-channel.html`](http://tutorials.jenkov.com/java-nio/server-socket-channel.html)

在 Java NIO 中，ServerSocketChannel 是用于监听 TCP 链接请求的通道，正如 Java 网络编程中的 ServerSocket 一样。

ServerSocketChannel 实现类位于 java.nio.channels 包下面。 下面是一个示例程序：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bin(new InetSocketAddress(9999));
while(true) {
  SocketChannel socketChannel = serverSocketChannel.accept();
  //do something with socketChannel...
}
```

## 打开 ServerSocketChannel

打开一个 ServerSocketChannel 我们需要调用他的 open()方法，例如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
```

## 关闭 ServerSocketChannel

关闭一个 ServerSocketChannel 我们需要调用他的 close()方法，例如：

```java
serverSocketChannel.close();
```

## 监听链接

通过调用 accept()方法，我们就开始监听端口上的请求连接。当 accept()返回时，他会返回一个 SocketChannel 连接实例，实际上 accept()是阻塞操作，他会阻塞带去线程知道返回一个连接； 很多时候我们是不满足于监听一个连接的，因此我们会把 accept()的调用放到循环中，就像这样：

```java
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    //do something with socketChannel...
}
```

当然我们可以在循环体内加上合适的中断逻辑，而不是单纯的在 while 循环中写 true，以此来结束循环监听；

## 非阻塞模式

实际上 ServerSocketChannel 是可以设置为非阻塞模式的。在非阻塞模式下，调用 accept()函数会立刻返回，如果当前没有请求的链接，那么返回值为空 null。因此我们需要手动检查返回的 SocketChannel 是否为空，例如：

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);

while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();

    if(socketChannel != null){
        //do something with socketChannel...
    }
}
```