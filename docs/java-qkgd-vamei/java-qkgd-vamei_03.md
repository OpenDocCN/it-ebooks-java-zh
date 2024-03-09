# Java 基础 02 方法与数据成员

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

在[Java 基础 01 从 HelloWorld 到面向对象](http://www.cnblogs.com/vamei/archive/2013/03/14/2958654.html)，我们初步了解了对象(object)。对象中的数据成员表示对象的状态。对象可以执行方法，表示特定的动作。

此外，我们还了解了类(class)。同一类的对象属于相同的类型(type)。我们可以定义类，并使用该定义来产生对象。

我们进一步深入到对象。了解 Java 中方法与数据成员的一些细节。

### 调用同一对象的数据成员

方法可以调用该对象的数据成员。比如下面我们给 Human 类增加一个 getHeight()的方法。该方法返回 height 数据成员的值:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        System.out.println(aPerson.getHeight());
    }

}

class Human
{/**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    int height;
}
```

我们新增了 getHeight 方法。这个方法有一个 int 类型的返回值。Java 中使用 return 来返回值。

注意 this，它用来指代对象自身。当我们创建一个 aPerson 实例时，this 就代表了 aPerson 这个对象。this.height 指 aPerson 的 height。

this 是隐性参数(implicit argument)。方法调用的时候，尽管方法的参数列表并没有 this，Java 都会“默默”的将 this 参数传递给方法。

(有一些特殊的方法不会隐性传递 this，我们会在以后见到)

this 并不是必需的，上述方法可以写成:

```java
    /**
     * accessor
     */
    int getHeight()
    {
        return height;
    }
```

Java 会自己去判断 height 是类中的数据成员。但使用 this 会更加清晰。

我们还看到了/** comments  */的添加注释的方法。

### 方法的参数列表

Java 中的方法定义与 C 语言中的函数类似。Java 的方法也可以接收参数列表(argument list)，放在方法名后面的括号中。下面我们定义一个 growHeight()的方法，该方法的功能是让人的 height 增高:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        System.out.println(aPerson.getHeight());
        aPerson.growHeight(10);
        System.out.println(aPerson.getHeight());
    }

}

class Human
{
/**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    /**
     * pass argument
     */
    void growHeight(int h)
    {
        this.height = this.height + h;
    }

    int height;
}
```

在 growHeight()中，h 为传递的参数。在类定义中，说明了参数的类型(int)。在具体的方法内部，我们可以使用该参数。该参数只在该方法范围，即 growHeight()内有效。

在调用的时候，我们将 10 传递给 growHeight()。aPerson 的高度增加了 10。

### 调用同一对象的其他方法

在方法内部，可以调用同一对象的其他方法。在调用的时候，使用 this.method()的形式。我们还记得，this 指代的是该对象。所以 this.method()指代了该对象自身的 method()方法。

![](img/3eb2131b68f115bdd8a03087cfd22513.jpg)

比如下面的 repeatBreath()函数:

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        aPerson.repeatBreath(10);
    }

}

class Human
{
    void breath()
    {
        System.out.println("hu...hu...");
    }

   /**
    * call breath()
    */
    void repeatBreath(int rep)
    {
        int i;
        for(i = 0; i < rep; i++) {
            this.breath();
        }
    }
    int height;
}
```

为了便于循环，在 repeatBreath()方法中，我们声明了一个 int 类型的对象 i。i 的作用域限定在 repeatBreath()方法范围内部。

(这与 C 语言函数中的自动变量类似)

### 数据成员初始化

在 Java 中，数据成员有多种初始化(initialize)的方式。比如上面的 getHeight()的例子中，尽管我们从来没有提供 height 的值，但 Java 为我们挑选了一个默认初始值 0。

基本类型的数据成员的默认初始值:

*   数值型: 0
*   布尔值: false
*   其他类型: null

我们可以在声明数据成员同时，提供数据成员的初始值。这叫做显式初始化(explicit initialization)。显示初始化的数值要硬性的写在程序中：

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        System.out.println(aPerson.getHeight());
    }

}

class Human
{/**
     * accessor
     */
    int getHeight()
    {
        return this.height;
    }

    int height = 175;
}
```

这里，数据成员 height 的初始值为 175，而不是默认的 0 了。

Java 中还有其它初始化对象的方式，我将在以后介绍。

### 总结

return

this, this.field, this.method()

默认初始值，显式初始化

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章