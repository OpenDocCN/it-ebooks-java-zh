# 四、04\. Java NIO Buffer 缓冲区

> 原文链接：[`tutorials.jenkov.com/java-nio/buffers.html`](http://tutorials.jenkov.com/java-nio/buffers.html)

Java NIO Buffers 用于和 NIO Channel 交互。正如你已经知道的，我们从 channel 中读取数据到 buffers 里，从 buffer 把数据写入到 channels.

buffer 本质上就是一块内存区，可以用来写入数据，并在稍后读取出来。这块内存被 NIO Buffer 包裹起来，对外提供一系列的读写方便开发的接口。

## Buffer 基本用法（Basic Buffer Usage）

利用 Buffer 读写数据，通常遵循四个步骤：

*   把数据写入 buffer；
*   调用 flip；
*   从 Buffer 中读取数据；
*   调用 buffer.clear()或者 buffer.compact()

当写入数据到 buffer 中时，buffer 会记录已经写入的数据大小。当需要读数据时，通过 flip()方法把 buffer 从写模式调整为读模式；在读模式下，可以读取所有已经写入的数据。

当读取完数据后，需要清空 buffer，以满足后续写入操作。清空 buffer 有两种方式：调用 clear()或 compact()方法。clear 会清空整个 buffer，compact 则只清空已读取的数据，未被读取的数据会被移动到 buffer 的开始位置，写入位置则近跟着未读数据之后。

这里有一个简单的 buffer 案例，包括了 write，flip 和 clear 操作：

```java
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();

//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);

int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {

  buf.flip();  //make buffer ready for read

  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }

  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

## Buffer 的容量，位置，上限（Buffer Capacity, Position and Limit）

buffer 缓冲区实质上就是一块内存，用于写入数据，也供后续再次读取数据。这块内存被 NIO Buffer 管理，并提供一系列的方法用于更简单的操作这块内存。

一个 Buffer 有三个属性是必须掌握的，分别是：

*   capacity 容量
*   position 位置
*   limit 限制

position 和 limit 的具体含义取决于当前 buffer 的模式。capacity 在两种模式下都表示容量。

下面有张示例图，描诉了不同模式下 position 和 limit 的含义：

![buffers-modes.png](http://tutorials.jenkov.com/images/java-nio/buffers-modes.png)

**图片 4.1** buffers-modes.png

**Buffer capacity, position and limit in write and read mode.**

### 容量（Capacity）

作为一块内存，buffer 有一个固定的大小，叫做 capacity 容量。也就是最多只能写入容量值得字节，整形等数据。一旦 buffer 写满了就需要清空已读数据以便下次继续写入新的数据。

### 位置（Position）

当写入数据到 Buffer 的时候需要中一个确定的位置开始，默认初始化时这个位置 position 为 0，一旦写入了数据比如一个字节，整形数据，那么 position 的值就会指向数据之后的一个单元，position 最大可以到 capacity-1.

当从 Buffer 读取数据时，也需要从一个确定的位置开始。buffer 从写入模式变为读取模式时，position 会归零，每次读取后，position 向后移动。

### 上限（Limit）

在写模式，limit 的含义是我们所能写入的最大数据量。它等同于 buffer 的容量。

一旦切换到读模式，limit 则代表我们所能读取的最大数据量，他的值等同于写模式下 position 的位置。

数据读取的上限时 buffer 中已有的数据，也就是 limit 的位置（原 position 所指的位置）。

## Buffer Types

Java NIO 有如下具体的 Buffer 类型：

*   ByteBuffer
*   MappedByteBuffer
*   CharBuffer
*   DoubleBuffer
*   FloatBuffer
*   IntBuffer
*   LongBuffer
*   ShortBuffer

正如你看到的，Buffer 的类型代表了不同数据类型，换句话说，Buffer 中的数据可以是上述的基本类型；

MappedByteBuffer 稍有不同，我们会单独介绍。

## 分配一个 Buffer（Allocating a Buffer）

为了获取一个 Buffer 对象，你必须先分配。每个 Buffer 实现类都有一个 allocate()方法用于分配内存。下面看一个实例,开辟一个 48 字节大小的 buffer：

```java
ByteBuffer buf = ByteBuffer.allocate(48);
```

开辟一个 1024 个字符的 CharBuffer：

```java
CharBuffer buf = CharBuffer.allocate(1024);
```

## 写入数据到 Buffer（Writing Data to a Buffer）

写数据到 Buffer 有两种方法：

*   从 Channel 中写数据到 Buffer
*   手动写数据到 Buffer，调用 put 方法

下面是一个实例，演示从 Channel 写数据到 Buffer：

```java
 int bytesRead = inChannel.read(buf); //read into buffer.
```

通过 put 写数据：

```java
buf.put(127);
```

put 方法有很多不同版本，对应不同的写数据方法。例如把数据写到特定的位置，或者把一个字节数据写入 buffer。看考 JavaDoc 文档可以查阅的更多数据。

## 翻转（flip()）

flip()方法可以吧 Buffer 从写模式切换到读模式。调用 flip 方法会把 position 归零，并设置 limit 为之前的 position 的值。 也就是说，现在 position 代表的是读取位置，limit 标示的是已写入的数据位置。

## 从 Buffer 读取数据（Reading Data from a Buffer）

冲 Buffer 读数据也有两种方式。

*   从 buffer 读数据到 channel
*   从 buffer 直接读取数据，调用 get 方法

读取数据到 channel 的例子：

```java
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```

调用 get 读取数据的例子：

```java
byte aByte = buf.get();
```

get 也有诸多版本，对应了不同的读取方式。

## rewind()

Buffer.rewind()方法将 position 置为 0，这样我们可以重复读取 buffer 中的数据。limit 保持不变。

## clear() and compact()

一旦我们从 buffer 中读取完数据，需要复用 buffer 为下次写数据做准备。只需要调用 clear 或 compact 方法。

clear 方法会重置 position 为 0，limit 为 capacity，也就是整个 Buffer 清空。实际上 Buffer 中数据并没有清空，我们只是把标记为修改了。

如果 Buffer 还有一些数据没有读取完，调用 clear 就会导致这部分数据被“遗忘”，因为我们没有标记这部分数据未读。

针对这种情况，如果需要保留未读数据，那么可以使用 compact。 因此 compact 和 clear 的区别就在于对未读数据的处理，是保留这部分数据还是一起清空。

## mark() and reset()

通过 mark 方法可以标记当前的 position，通过 reset 来恢复 mark 的位置，这个非常像 canva 的 save 和 restore：

```java
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.
```

## equals() and compareTo()

可以用 eqauls 和 compareTo 比较两个 buffer

### equals()

判断两个 buffer 相对，需满足：

*   类型相同
*   buffer 中剩余字节数相同
*   所有剩余字节相等

从上面的三个条件可以看出，equals 只比较 buffer 中的部分内容，并不会去比较每一个元素。

### compareTo()

compareTo 也是比较 buffer 中的剩余元素，只不过这个方法适用于比较排序的：