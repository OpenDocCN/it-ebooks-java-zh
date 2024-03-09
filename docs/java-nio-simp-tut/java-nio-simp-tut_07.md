# 七、07\. Java NIO Selector 选择器

> 原文链接：[`tutorials.jenkov.com/java-nio/selectors.html`](http://tutorials.jenkov.com/java-nio/selectors.html)

Selector 是 Java NIO 中的一个组件，用于检查一个或多个 NIO Channel 的状态是否处于可读、可写。如此可以实现单线程管理多个 channels,也就是可以管理多个网络链接。

## 为什么使用 Selector（Why Use a Selector?）

用单线程处理多个 channels 的好处是我需要更少的线程来处理 channel。实际上，你甚至可以用一个线程来处理所有的 channels。从操作系统的角度来看，切换线程开销是比较昂贵的，并且每个线程都需要占用系统资源，因此暂用线程越少越好。

需要留意的是，现代操作系统和 CPU 在多任务处理上已经变得越来越好，所以多线程带来的影响也越来越小。如果一个 CPU 是多核的，如果不执行多任务反而是浪费了机器的性能。不过这些设计讨论是另外的话题了。简而言之，通过 Selector 我们可以实现单线程操作多个 channel。

这有一幅示意图，描述了单线程处理三个 channel 的情况：

![overview-selectors.png](http://tutorials.jenkov.com/images/java-nio/overview-selectors.png)

**图片 7.1** overview-selectors.png

**Java NIO: A Thread uses a Selector to handle 3 Channel's**

## 创建 Selector(Creating a Selector)

创建一个 Selector 可以通过 Selector.open()方法：

```java
Selector selector = Selector.open();
```

## 注册 Channel 到 Selector 上(Registering Channels with the Selector)

为了同 Selector 挂了 Channel，我们必须先把 Channel 注册到 Selector 上，这个操作使用 SelectableChannel。register()：

```java
channel.configureBlocking(false);
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

Channel 必须是非阻塞的。所以 FileChannel 不适用 Selector，因为 FileChannel 不能切换为非阻塞模式。Socket channel 可以正常使用。

注意 register 的第二个参数，这个参数是一个“关注集合”，代表我们关注的 channel 状态，有四种基础类型可供监听：

1.  Connect
2.  Accept
3.  Read
4.  Write

一个 channel 触发了一个事件也可视作该事件处于就绪状态。因此当 channel 与 server 连接成功后，那么就是“连接就绪”状态。server channel 接收请求连接时处于“可连接就绪”状态。channel 有数据可读时处于“读就绪”状态。channel 可以进行数据写入时处于“写就绪”状态。

上述的四种就绪状态用 SelectionKey 中的常量表示如下：

1.  SelectionKey.OP_CONNECT
2.  SelectionKey.OP_ACCEPT
3.  SelectionKey.OP_READ
4.  SelectionKey.OP_WRITE

如果对多个事件感兴趣可利用位的或运算结合多个常量，比如：

```java
int interestSet = SelectionKey.OP_READ | SelectionKey.OP_WRITE;
```

## SelectionKey's

在上一小节中，我们利用 register 方法把 Channel 注册到了 Selectors 上，这个方法的返回值是 SelectionKeys，这个返回的对象包含了一些比较有价值的属性：

*   The interest set
*   The ready set
*   The Channel
*   The Selector
*   An attached object (optional)

这 5 个属性都代表什么含义呢？下面会一一介绍。

### Interest Set

这个“关注集合”实际上就是我们希望处理的事件的集合，它的值就是注册时传入的参数，我们可以用按为与运算把每个事件取出来：

```java
int interestSet = selectionKey.interestOps();

boolean isInterestedInAccept  = interestSet & SelectionKey.OP_ACCEPT;
boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
```

### Ready Set

"就绪集合"中的值是当前 channel 处于就绪的值，一般来说在调用了 select 方法后都会需要用到就绪状态，select 的介绍在胡须文章中继续展开。

```java
int readySet = selectionKey.readyOps();
```

从“就绪集合”中取值的操作类似月“关注集合”的操作，当然还有更简单的方法，SelectionKey 提供了一系列返回值为 boolean 的的方法：

```java
selectionKey.isAcceptable();
selectionKey.isConnectable();
selectionKey.isReadable();
selectionKey.isWritable();
```

### Channel + Selector

从 SelectionKey 操作 Channel 和 Selector 非常简单：

```java
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();
```

### Attaching Objects

我们可以给一个 SelectionKey 附加一个 Object，这样做一方面可以方便我们识别某个特定的 channel，同时也增加了 channel 相关的附加信息。例如，可以把用于 channel 的 buffer 附加到 SelectionKey 上：

```java
selectionKey.attach(theObject);

Object attachedObj = selectionKey.attachment();
```

附加对象的操作也可以在 register 的时候就执行：

```java
SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject);
```

## 从 Selector 中选择 channel(Selecting Channels via a Selector)

一旦我们向 Selector 注册了一个或多个 channel 后，就可以调用 select 来获取 channel。select 方法会返回所有处于就绪状态的 channel。 select 方法具体如下：

*   int select()
*   int select(long timeout)
*   int selectNow()

select()方法在返回 channel 之前处于阻塞状态。 select(long timeout)和 select 做的事一样，不过他的阻塞有一个超时限制。

selectNow()不会阻塞，根据当前状态立刻返回合适的 channel。

select()方法的返回值是一个 int 整形，代表有多少 channel 处于就绪了。也就是自上一次 select 后有多少 channel 进入就绪。举例来说，假设第一次调用 select 时正好有一个 channel 就绪，那么返回值是 1，并且对这个 channel 做任何处理，接着再次调用 select，此时恰好又有一个新的 channel 就绪，那么返回值还是 1，现在我们一共有两个 channel 处于就绪，但是在每次调用 select 时只有一个 channel 是就绪的。

### selectedKeys()

在调用 select 并返回了有 channel 就绪之后，可以通过选中的 key 集合来获取 channel，这个操作通过调用 selectedKeys()方法：

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

还记得在 register 时的操作吧，我们 register 后的返回值就是 SelectionKey 实例，也就是我们现在通过 selectedKeys()方法所返回的 SelectionKey。

遍历这些 SelectionKey 可以通过如下方法：

```java
Set<SelectionKey> selectedKeys = selector.selectedKeys();

Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
}
```

上述循环会迭代 key 集合，针对每个 key 我们单独判断他是处于何种就绪状态。

注意 keyIterater.remove()方法的调用，Selector 本身并不会移除 SelectionKey 对象，这个操作需要我们收到执行。当下次 channel 处于就绪是，Selector 任然会吧这些 key 再次加入进来。

SelectionKey.channel 返回的 channel 实例需要强转为我们实际使用的具体的 channel 类型，例如 ServerSocketChannel 或 SocketChannel.

## wakeUp()

y 由于调用 select 而被阻塞的线程，可以通过调用 Selector.wakeup()来唤醒即便此时已然没有 channel 处于就绪状态。具体操作是，在另外一个线程调用 wakeup，被阻塞与 select 方法的线程就会立刻返回。

## close()

当操作 Selector 完毕后，需要调用 close 方法。close 的调用会关闭 Selector 并使相关的 SelectionKey 都无效。channel 本身不管被关闭。

## 完整的 Selector 案例(Full Selector Example)

这有一个完整的案例，首先打开一个 Selector,然后注册 channel，最后锦亭 Selector 的状态：

```java
Selector selector = Selector.open();

channel.configureBlocking(false);

SelectionKey key = channel.register(selector, SelectionKey.OP_READ);

while(true) {

  int readyChannels = selector.select();

  if(readyChannels == 0) continue;

  Set<SelectionKey> selectedKeys = selector.selectedKeys();

  Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

  while(keyIterator.hasNext()) {

    SelectionKey key = keyIterator.next();

    if(key.isAcceptable()) {
        // a connection was accepted by a ServerSocketChannel.

    } else if (key.isConnectable()) {
        // a connection was established with a remote server.

    } else if (key.isReadable()) {
        // a channel is ready for reading

    } else if (key.isWritable()) {
        // a channel is ready for writing
    }

    keyIterator.remove();
  }
}
```