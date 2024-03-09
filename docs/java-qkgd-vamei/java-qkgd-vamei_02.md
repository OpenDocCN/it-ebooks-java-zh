# Java 基础 01 从 HelloWorld 到面向对象

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

Java 是完全面向对象的语言。Java 通过虚拟机的运行机制，实现“跨平台”的理念。我在这里想要呈现一个适合初学者的教程，希望对大家有用。

### "Hello World!"

先来看一个 HelloWorld.java 程序。这个程序在屏幕上打印出一串字符"Hello World!":

```java
public class HelloWorld
{
    public static void main(String[] args)
    {
        System.out.println("Hello World!");
    }
}
```

程序中包括 Java 的一些基本特征：

*   类(class)：上面程序定义了一个类 HelloWorld，该类的名字与.java 文件的名字相同。
*   方法(method)：类的内部定义了该类的一个方法 main。
*   语句(statement)：真正的“打印”功能由一个语句实现，即: System.out.println("Hello World!");

下面两点有关 Java 的书写方式：

*   Java 中的语句要以;结尾 (与 C/C++相同)。
*   用花括号{}来整合语句，形成程序块。通过程序块，我们可以知道程序的不同部分的范围，比如类从哪里开始，到哪里结束。

### 编译与运行

Java 程序要经过编译器编译才能执行。在 Linux 或 Mac 下，可以下载安装[Java JDK](http://www.oracle.com/technetwork/java/javase/downloads/index-jsp-138363.html)。

使用 javac 来编译。在命令行中输入下面语句编译:

```java
$javac HelloWorld.java
```

当前路径下，将有一个名为 HelloWorld.class 的文件生成。

使用 java 命令来运行。Java 会搜寻该类中的 main 方法，并执行。

```java
$java HelloWorld
```

### 变量

计算机语言通常需要在内存中存放数据，比如 C 语言中的变量，Java 也有类似的变量。Java 和 C 语言都是静态类型的语言。在使用变量之前，要声明变量的类型。

变量(variable)占据一定的内存空间。不同类型的变量占据不同的大小。Java 中的变量类型如下：

          存储大小     例值     注释

byte      1byte        3      字节

int       4bytes       3      整数

short     2bytes       3      短整数

long      8bytes       3      长整数

float     4bytes     1.2      单精度浮点数

double    8bytes     1.2      双精度浮点数

char      2bytes     'a'      字符

boolean   1bit      true      布尔值

在 Java 中，变量需要先声明(declare)才能使用。在声明中，我说明变量的类型，赋予变量以特别名字，以便在后面的程序中调用它。你可以在程序中的任意位置声明变量。 比如:

```java
public class Test
{
    public static void main(String[] args)
    {
        System.out.println("Declare in the middle:");
        int a;
        a = 5;
        System.out.println(a);  // print an integer
    }
}
```

上面 a 是变量名。可以在声明变量的同时，给变量赋值，比如 int a = 5;

*** “变量”的概念实际上来自于面向过程的编程语言。在 Java 中，所谓的变量实际上是“基本类型” (premitive type)。我们将在类的讲解中更多深入。

上面的程序还可以看到，Java 中，可用//引领注释。

### 数组

Java 中有数组(array)。数组包含相同类型的多个数据。我用下面方法来声明一个整数数组:

int[] a;

在声明数组时，数组所需的空间并没有真正分配给数组。我可以在声明的同时，用 new 来创建数组所需空间:

int[] a = new int[100];

这里创建了可以容纳 100 个整数的数组。相应的内存分配也完成了。

我还可以在声明的同时，给数组赋值。数组的大小也同时确定。

int[] a = new int[] {1, 3, 5, 7, 9};

使用 int[i]来调用数组的 i 下标元素。i 从 0 开始。

其他类型的数组与整数数组相似。

### 表达式

表达式是变量、常量和运算符的组合，它表示一个数据。1 + 1 是常见的表达式。再比如:

```java
public class Test
{
    public static void main(String[] args)
    {
        System.out.println("Declare in the middle:");
        int a;
        a = 5 + 1;
        System.out.println(a);  // print an integer
    }
}
```

上面的 5 + 1 也是一个表达式，等于 6。 

**数学表达式**

数学运算，结果为一个数值

1 + 2                  加法

4 - 3.4                减法

7 * 1.5                乘法

3.5 / 7                除法

7 % 2                  求余数

**关系表达式**

判断表达式是否成立。即一个 boolean 值，真假

a > 4.2                大于

3.4 >= b               大于等于

1.5 < 9                小于

6 <= 1                 小于等于

2 == 2                 等于

2 != 2                 不等于

**布林表达式**

两个 boolean 值的与、或、非的逻辑关系

true && false          and

(3 > 1) || (2 == 1)    or

!true                  not

**位运算**

对整数的二进制形式逐位进行逻辑运算，得到一个整数

&                      and

|                      or

^                      xor

~                      not

5 << 3                 0b101 left shift 3 bits

6 >> 1                 0b110 right shift 1 bit

还有下列在 C 中常见的运算符，我会在用到的时候进一步解释:

m++                    变量 m 加 1

n--                    变量 n 减 1

condition ? x1 : x2   condition 为一个 boolean 值。根据 condition，取 x1 或 x2 的值

### 控制结构

Java 中控制结构(control flow)的语法与 C 类似。它们都使用{}来表达隶属关系。

**选择 (if)**

```java
if (conditon1) {
    statements;
    ...
}
else if (condition2) {
    statements;
    ...
}
else {
    statements;
    ...
}
```

上面的 condition 是一个表示真假值的表达式。statements;是语句。

练习 写一个 Java 程序，判断 2013 年是否是闰年。

**循环 (while)**

*while (condition) {*

*    statements;*

*}*

**循环 (do... while)**

*do {*

*    statements;*

*} while(condition);* // 注意结尾的;

**循环 (for)**

*for (initial; condition; update) {*

*    statements;*

*}*

**跳过或跳出循环**

在循环中，可以使用

*break;* // 跳出循环

*continue;* // 直接进入下一环

练习 写一个 Java 程序，计算从 1 加 2，加 3…… 一直加到 999 的总和

**选择 (switch)**

*switch(expression) {*

*    case 1:*

*        statements;*

*        break;*

*    case 2:*

*        statements;*

*        break;*

*    ...*

*    default:*

*        statements;*

*        break;*

*}*

### 面向对象

“对象”是计算机抽象世界的一种方式。“面向对象”可以用很多方式表达。下面是一种并不精确，但比较直观的理解方式：

1.  世界上的每一个事物都可以称为一个对象(object)，比如张三。对象有身份(Identity)，状态(State)和行为(Behavior)。
2.  对象的状态由数据成员(data member)表示。数据成员又被称作域(field)。我们用其他对象作为该对象的数据成员。比如一个表示身高的整数，比如一个鼻子。
3.  对象的行为由成员方法(member method)表示。我们简称为方法(method)。一个对象可以有多个方法，比如呼吸，睡觉。
4.  对象可以归类(class)，或者说归为同一类型(type)。同一类型的对象有相同的方法，有同类型的数据成员。某个类型的一个对象被称为该类型的一个实例(instance)。

![](img/5c144fdaa8e5394ee4bae50988c6412d.jpg)

类与对象

定义类的语法:

*class ClassName* 

*{*

*member1;* 

*    member2;*

*    ...*

*}*

我们定义一个 human 类:

```java
class Human 
{
    void breath()
    {
        System.out.println("hu...hu...");
    }

    int height;
}
```

在{}范围内，Human 类有两个成员： 一个数据成员 height，一个方法 breath()。

*   数据成员 height 是整数类型，可以用于存储一个整数。
*   方法代表了对象所能进行的动作，也就是计算机所能进行的操作。方法可以接受参数，并能返回值。在 breath()的定义中，breath 后面的()中为参数列表。由于参数列表为空，所以 breath()不接受参数。在 breath()之前的 void 为返回值的类型，说明 breath 不返回值。

(方法与面向过程语言中的函数类似) 

现在，我们创建对象 aPerson，并调用对象的方法 breath:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        aPerson.breath();
        System.out.println(aPerson.height);
    }

}

class Human
{
    void breath()
    {
       System.out.println("hu...hu...");
    }

    int height;
}
```

在 main 方法中，使用 new 关键字创建对象。即使是来自同一个类的对象，各个对象占据的内存也不相同，即对象的身份也不同。

Human aPerson 声明了 aPerson 对象属于 Human 类，即说明了对象的类型。

对象建立后，我们可以用 对象.数据成员 来引用数据成员，使用 对象.方法() 的方式来调用方法。正如我们在后面打印 aPerson.height。

### 总结

Java 的许多语法形式与 C/C++类似，但在细节和具体实现上又有差别，需要小心。

对象，类

对象: 方法，域(数据成员)

Java 是完全面向对象的语言。

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章