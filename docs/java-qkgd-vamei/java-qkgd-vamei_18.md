# Java 进阶 04 RTTI

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

运行时类型识别(RTTI, Run-Time Type Identification)是 Java 中非常有用的机制，在 Java 运行时，RTTI 维护类的相关信息。

多态(polymorphism)是基于 RTTI 实现的。RTTI 的功能主要是由 Class 类实现的。

### Class 类

Class 类是"类的类"(class of classes)。如果说类是对象的抽象和集合的话，那么 Class 类就是对类的抽象和集合。

每一个 Class 类的对象代表一个其他的类。比如下面的程序中，Class 类的对象 c1 代表了 Human 类，c2 代表了 Woman 类。

```java
public class Test
{
    public static void main(String[] args)
    {
        Human aPerson = new Human();
        Class c1      = aPerson.getClass();
        System.out.println(c1.getName());
```

```java

```
        Human anotherPerson = new Woman();
        Class c2      = anotherPerson.getClass();
        System.out.println(c2.getName());  
    }
}

class Human
{    
    /**
     * accessor
     */
    public int getHeight()
    {
       return this.height;
    }

    /**
     * mutator
     */
    public void growHeight(int h)
    {
        this.height = this.height + h;
    }
private int height; 
}

class Woman extends Human
{
    /**
     * new method
     */
    public Human giveBirth()
    {
        System.out.println("Give birth");
        return (new Human());
    }

}
```java

当我们调用对象的 getClass()方法时，就得到对应 Class 对象的引用。
在 c2 中，即使我们将 Women 对象的引用向上转换为 Human 对象的引用，对象所指向的 Class 类对象依然是 Woman。
Java 中每个对象都有相应的 Class 类对象，因此，我们随时能通过 Class 对象知道某个对象“真正”所属的类。无论我们对引用进行怎样的类型转换，对象本身所对应的 Class 对象都是同一个。当我们通过某个引用调用方法时，Java 总能找到正确的 Class 类中所定义的方法，并执行该 Class 类中的代码。由于 Class 对象的存在，Java 不会因为类型的向上转换而迷失。这就是多态的原理。

![](img/a28ddc2c281132e9bd9087ed4b294341.jpg)
getClass: 我是谁?

除了 getClass()方法外，我们还有其他方式调用 Class 类的对象。

```
public class Test
{
    public static void main(String[] args)
    {
        Class c3      = Class.forName("Human");
        System.out.println(c1.getName());

        Class c4      = Woman.class
        System.out.println(c2.getName());  
    }
}
```java

上面显示了两种方式:

*   forName()方法接收一个字符串作为参数，该字符串是类的名字。这将返回相应的 Class 类对象。
*   Woman.class 方法是直接调用类的 class 成员。这将返回相应的 Class 类对象。

Class 类的方法
Class 对象记录了相应类的信息，比如类的名字，类所在的包等等。我们可以调用相应的方法，比如:
getName()         返回类的名字
getPackage()      返回类所在的包

可以利用 Class 对象的 newInstance()方法来创建相应类的对象，比如:

```
Human newPerson = c1.newInstance();  
```java

newInstance()调用默认的不含参数的构建方法。

我们可以获得类定义的成员:
getFields()       返回所有的 public 数据成员
getMethods()      返回所有的 public 方法
可以进一步使用 Reflection 分析类。这里不再深入。

Class 类更多的方法可查询官方文档:
[`docs.oracle.com/javase/1.5.0/docs/api/java/lang/Class.html`](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/Class.html)

Class 类的加载
当 Java 创建某个类的对象，比如 Human 类对象时，Java 会检查内存中是否有相应的 Class 对象。
如果内存中没有相应的 Class 对象，那么 Java 会在.class 文件中寻找 Human 类的定义，并加载 Human 类的 Class 对象。
在 Class 对象加载成功后，其他 Human 对象的创建和相关操作都将参照该 Class 对象。

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章

```