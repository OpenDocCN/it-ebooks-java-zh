# 一、01\. Java NIO 教程

> 原文链接：[`tutorials.jenkov.com/java-nio/index.html`](http://tutorials.jenkov.com/java-nio/index.html)

Java NIO 是 java 1.4 之后新出的一套 IO 接口，这里的的新是相对于原有标准的 Java IO 和 Java Networking 接口。NIO 提供了一种完全不同的操作方式。

> NIO 中的 N 可以理解为 Non-blocking，不单纯是 New

## Java NIO: Channels and Buffers

标准的 IO 编程接口是面向字节流和字符流的。而 NIO 是面向通道和缓冲区的，数据总是从通道中读到 buffer 缓冲区内，或者从 buffer 写入到通道中。

## Java NIO: Non-blocking IO

Java NIO 使我们可以进行非阻塞 IO 操作。比如说，单线程中从通道读取数据到 buffer，同时可以继续做别的事情，当数据读取到 buffer 中后，线程再继续处理数据。写数据也是一样的。

## Java NIO: Selectors

NIO 中有一个“slectors”的概念。selector 可以检测多个通道的事件状态（例如：链接打开，数据到达）这样单线程就可以操作多个通道的数据。 所有这些都会在后续章节中更详细的介绍。