# Java 基础 05 实施接口封装与接口封装与接口

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！ 

在[封装与接口](http://www.cnblogs.com/vamei/archive/2013/03/27/2982209.html)中，private 关键字封装了对象的内部成员。经过封装，产品隐藏了内部细节，只提供给用户接口(interface)。

接口是非常有用的概念，可以辅助我们的抽象思考。在现实生活中，当我们想起某个用具的时候，往往想到的是该用具的功能性接口。比如杯子，我们想到加水和喝水的可能性，高于想到杯子的材质和价格。也就是说，一定程度上，用具的接口等同于用具本身。内部细节则在思考过程中被摒弃。

![](img/dbed7859a9519c806557af0dd1841786.jpg)

a cup in mind

在 public 和 private 的封装机制，我们实际上同时定义了类和接口，类和接口混合在一起。Java 还提供了 interface 这一语法。这一语法将接口从类的具体定义中剥离出来，构成一个独立的主体。

### interface

以杯子为例，定义一个杯子的接口:

```java
interface Cup {
    void addWater(int w);
    void drinkWater(int w);
}
```

Cup 这个 interface 中定义了两个方法的原型(stereotype): addWater()和 drinkWater()。一个方法的原型规定了方法名，参数列表和返回类型。原型可以告诉外部如何使用这些方法。

在 interface 中，我们

*   不需要定义方法的主体
*   不需要说明方法的可见性

注意第二点，interface 中的方法默认为 public。正如我们在[封装与接口](http://www.cnblogs.com/vamei/archive/2013/03/27/2982209.html)中讲到的，一个类的 public 方法构成了接口。所以，所有出现在 interface 中的方法都默认为 public。

我们可以在一个类的定义中实施接口，比如下面的 MusicCup (可以播放音乐的杯子):

```java
class MusicCup implements Cup 
{
    public void addWater(int w) 
    {
        this.water = this.water + w;
    }

    public void drinkWater(int w)
    {
        this.water = this.water - w;
    }

    private int water = 0;
}
```

我们用 implements 关键字来实施 interface。一旦在类中实施了某个 interface，必须在该类中定义 interface 的所有方法(addWater()和 drinkWater())。类中的方法需要与 interface 中的方法原型相符。否则，Java 将报错。

在类中可以定义 interface 没有提及的其他 public 方法。也就是说，interface 规定一个必须要实施的最小接口。比如下面的 waterContent()方法就没有在 Cup 接口中规定原型:

```java
class MusicCup implements Cup 
{
    public void addWater(int w) 
    {
        this.water = this.water + w;
    }

    public void drinkWater(int w)
    {
        this.water = this.water - w;
    }

    public int waterContent()
    {
        return this.water;
    }

    private int water = 0;
}
```

### 分离接口的意义

我们使用了 interface，但这个 interface 并没有减少我们定义类时的工作量。我们依然要像之前一样，具体的编写类。我们甚至于要更加小心，不能违反了 interface 的规定。既然如此，我们为什么要使用 interface 呢？

事实上，interface 就像是行业标准。一个工厂(类)可以采纳行业标准 (implement interface)，也可以不采纳行业标准。但是，一个采纳了行业标准的产品将有下面的好处:

*   更高质量: 没有加水功能的杯子不符合标准。
*   更容易推广: 正如电脑上的 USB 接口一样，下游产品可以更容易衔接。

如果我们已经有一个 Java 程序，用于处理符合 Cup 接口的对象，比如领小朋友喝水。那么，只要我们确定，我们给小朋友的杯子(对象)实施了 Cup 接口，就可以确保小朋友可以执行喝水这个动作了。至于这个杯子(对象)是如何具体定义喝水这个动作的，我们就可以留给相应的类自行决定 (比如用吸管喝水，或者开一个小口喝水)。

在计算机科学中，接口是很重要的概念。比如任何提供 UNIX 接口的操作系统都可以称作 UNIX 系统。Linux，Mac OS，Solaris 都是 UNIX 系统，它们提供相似的接口。但是，各个系统的具体实施(源代码)互不相同。Linux 是开源的，你可以查看它的每一行代码，但你还是不知道如何去编写一个 Solaris 系统。

![](img/e7eb144b69ee78e93f765543a4ea5e05.jpg)

相同的 UNIX 接口

### 实施多个接口

一个类可以实施不止一个的 interface。比如我们有下面一个 interface:

```java
interface MusicPlayer {
    void play();
}
```

我们再来考虑 MusicCup 类。MusicCup 可以看做播放器和杯子的混合体。

![](img/5617a9ef55df1f007d22fa0bd7606953.jpg)

所以 MusicCup 应该具备两套接口，即同时实施 MusicPlayer 接口和 Cup 接口:

```java
class MusicCup implements MusicPlayer, Cup
{
    public void addWater(int w) 
    {
        this.water = this.water + w;
    }

    public void drinkWater(int w)
    {
        this.water = this.water - w;
    }

    public void play()
    {
        System.out.println("la...la...la");
    }

    private int water = 0;
}
```

最后，可以尝试将本文中的 interface 和类定义放在同一个文件中，并编写 Test 类，运行一下。

### 总结

interface, method stereotype, public

implements interface

implements interface1, interface2

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章