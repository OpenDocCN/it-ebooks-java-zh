# Java 基础 03 构造器与方法重载

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

在[方法与数据成员](http://www.cnblogs.com/vamei/archive/2013/03/25/2964430.html)中，我们提到，Java 中的对象在创建的时候会初始化(initialization)。初始化时，对象的数据成员被赋予初始值。我们可以显式初始化。如果我们没有给数据成员赋予初始值，数据成员会根据其类型采用默认初始值。

显式初始化要求我们在写程序时就确定初始值，这有时很不方便。我们可以使用构造器(constructor)来初始化对象。构造器可以初始化数据成员，还可以规定特定的操作。这些操作会在创建对象时自动执行。

### 定义构造器

构造器是一个方法。像普通方法一样，我们在类中定义构造器。构造器有如下基本特征:

1.  构造器的名字和类的名字相同
2.  构造器没有返回值

我们定义 Human 类的构造器:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human(160);
        System.out.println(aPerson.getHeight());
    }

}

class Human
{
    /**
     * constructor
     */
    Human(int h)
    {
        this.height = h;
        System.out.println("I'm born");
    }

    /**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    int height;
}
```

上面的程序会打印

I'm born
160

构造器可以像普通方法一样接收参数列表。这里，构造器 Human()接收一个整数作为参数。在方法的主体中，我们将该整数参数赋予给数据成员 height。构造器在对象创建时做了两件事:

*   为数据成员提供初始值 this.height = h;
*   执行特定的初始操作 System.out.println("I'm born");

这样，我们就可以在调用构造器时，灵活的设定初始值，不用像显示初始化那样拘束。

构造器是如何被调用的呢？我们在创建类的时候，采用的都是 new Human()的方式。实际上，我们就是在调用 Human 类的构造器。当我们没有定义该方法时，Java 会提供一个空白的构造器，以便使用 new 的时候调用。但当我们定义了构造器时，在创建对象时，Java 会调用定义了的构造器。在调用时，我们提供了一个参数 160。从最后的运行结果中也可以看到，对象的 height 确实被初始化为 160。

### 初始化方法的优先级

在[方法与数据成员](http://www.cnblogs.com/vamei/archive/2013/03/25/2964430.html)中，我们可以看到，如果我们提供显式初始值，那么数据成员就会采用显式初始值，而不是默认初始值。但如果我们既提供显式初始值，又在构造器初始化同一数据成员，最终的初始值将由构造器决定。比如下面的例子:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human(160);
        System.out.println(aPerson.getHeight());
    }

}

class Human
{
    /**
     * constructor
     */
    Human(int h)
    {
        this.height = h; 
    }

    /**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    int height=170; // explicit initialization
}
```

运行结果为:

160

对象最终的初始化值与构建方法中的值一致。因此:

构建方法 > 显式初始值 > 默认初始值

(事实上，所谓的优先级与初始化时的执行顺序有关，我将在以后深入这一点)

### 方法重载

一个类中可以定义不止一个构造器，比如:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human neZha   = new Human(150, "shit");
        System.out.println(neZha.getHeight()); 
    }

}

class Human
{
    /**
     * constructor 1
     */
    Human(int h)
    {
        this.height = h;
        System.out.println("I'm born");
    }

    /**
     * constructor 2
     */
    Human(int h, String s)
    {
        this.height = h;
        System.out.println("Ne Zha: I'm born, " + s);
    }

    /**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    int height;
}
```

运行结果:

Ne Zha: I'm born, shit
150

 ![](img/d5a897f75eeef8fd23f6699e27049066.jpg)

上面定义了两个构造器，名字都是 Human。两个构造器有不同的参数列表。

在使用 new 创建对象时，Java 会根据提供的参数来决定构建哪一个构造器。比如在构建 neZha 时，我们提供了两个参数: 整数 150 和字符串"shit"，这对应第二个构建方法的参数列表，所以 Java 会调用第二个构建方法。

在 Java 中，Java 会同时根据方法名和参数列表来决定所要调用的方法，这叫做方法重载(method overloading)。构建方法可以进行重载，普通方法也可以重载，比如下面的 breath()方法:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        aPerson.breath(10);
    }

}

class Human
{
    /**
       * breath() 1
       */
    void breath()
    {
        System.out.println("hu...hu...");
    }

   /**
    * breath() 2
    */
    void breath(int rep)
    {
        int i;
        for(i = 0; i < rep; i++) {
            System.out.println("lu...lu...");
        }
    }

    int height;
}
```

运行结果:

lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...
lu...lu...

可以看到，由于在调用的时候提供了一个参数: 整数 10，所以调用的是参数列表与之相符的第二个 breath()方法。

### 总结

constructor 特征: 与类同名，无返回值

constructor 目的: 初始化，初始操作

方法重载: 方法名 + 参数列表 -> 实际调用哪一个方法

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章