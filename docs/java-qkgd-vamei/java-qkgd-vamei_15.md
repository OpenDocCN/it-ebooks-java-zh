# Java 进阶 01 String 类

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

之前的[Java 基础](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)系列中讨论了 Java 最核心的概念，特别是面向对象的基础。在 Java 进阶中，我将对 Java 基础进行补充，并转向应用层面。

大部分编程语言都能够处理字符串(String)。字符串是有序的字符集合，比如"Hello World!"。在 Java 中，字符串被存储为 String 类对象。调用字符串对象的方法，可以实现字符串相关的操作。

String 类包含在 java.lang 包中。这个包会在 Java 启动的时候自动 import，所以可以当做一个内置类(built-in class)。我们不需要显式的使用 import 引入 String 类。

### 创建字符串

我们之前使用类来创建对象。需要注意的时候，创建 String 类对象不需要 new 关键字。比如:

```java
public class Test
{
    public static void main(String[] args)
    {
        String s = "Hello World!";
        System.out.println(s);                     
    }
}
```

实际上，当你写出一个"Hello World"表达式时，内存中就已经创建了该对象。如果使用 new String("Hello World!")，会重复创建出一个字符串对象。

![](img/56166a503086d15c5599fff1f9fe514c.jpg)

An Object

String 类是唯一一个不需要 new 关键字来创建对象的类。使用的时候需要注意。

### 字符串操作

可以用+实现字符串的连接(concatenate)，比如:

"abc" + s

字符串的操作大都通过字符串的相应方法实现，比如下面的方法:

方法                               效果

s.length()                        返回 s 字符串长度

s.charAt(2)                       返回 s 字符串中下标为 2 的字符

s.substring(0, 4)                 返回 s 字符串中下标 0 到 4 的子字符串

s.indexOf("Hello")                返回子字符串"Hello"的下标

s.startsWith(" ")                 判断 s 是否以空格开始

s.endsWith("oo")                  判断 s 是否以"oo"结束

s.equals("Good World!")           判断 s 是否等于"Good World!"

                                  ==只能判断字符串是否保存在同一位置。需要使用 equals()判断字符串的内容是否相同。

s.compareTo("Hello Nerd!")        比较 s 字符串与"Hello Nerd!"在词典中的顺序，

                                  返回一个整数，如果<0，说明 s 在"Hello Nerd!"之前；

                                              如果>0，说明 s 在"Hello Nerd!"之后；

                                              如果==0，说明 s 与"Hello Nerd!"相等。

s.trim()                          去掉 s 前后的空格字符串，并返回新的字符串

s.toUpperCase()                   将 s 转换为大写字母，并返回新的字符串

s.toLowerCase()                   将 s 转换为小写，并返回新的字符串

s.replace("World", "Universe")    将"World"替换为"Universe"，并返回新的字符串

### 不可变对象

String 类对象是不可变对象(immutable object)。程序员不能对已有的不可变对象进行修改。我们自己也可以创建不可变对象，只要在接口中不提供修改数据的方法就可以。

然而，String 类对象确实有编辑字符串的功能，比如 replace()。这些编辑功能是通过创建一个新的对象来实现的，而不是对原有对象进行修改。比如:

s = s.replace("World", "Universe");

右边对 s.replace()的调用将创建一个新的字符串"Hello Universe!"，并返回该对象(的引用)。通过赋值，引用 s 将指向该新的字符串。如果没有其他引用指向原有字符串"Hello World!"，原字符串对象将被垃圾回收。

![](img/678cb2bb29b24235d5de864f1e5a94bc.jpg)

不可变对象

### Java API

Java 提供了许多功能强大的包。Java 学习的一个重要方面是了解这些包以及其中包含的 API(Application Programming Interface)。String 类定义在 java.lang.String。你可以查询下面的 Oracle 网址，来找到该类的官方文档:

[`docs.oracle.com/javase/6/docs/api/java/lang/String.html`](http://docs.oracle.com/javase/6/docs/api/java/lang/String.html)

该文档中包含了 String 类最全面的介绍。

事实上，API 文档中有丰富的内容，你通过下面链接概览:

[`docs.oracle.com/javase/6/docs/api/`](http://docs.oracle.com/javase/6/docs/api/)

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章