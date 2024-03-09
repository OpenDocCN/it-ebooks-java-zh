# Java 基础 07 包

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！

我们已经写了一些 Java 程序。之前的每个 Java 程序都被保存为一个文件，比如 Test.java。随后，该程序被编译为 Test.class。我们最终使用$java Test 来运行程序。

然而，在一个正常的 Java 项目中，我们往往需要编写不止一个.java 程序，最终的 Java 产品包括了所有的 Java 程序。因此，Java 需要解决组织 Java 程序的问题。包(package)的目的就是为了更好的组织 Java 程序。

 ![](img/d961b8fa7f8d0342cd91536d37be5524.jpg)

### 包的建立

包的建立非常简单。我们只用在 Java 程序的开始加入 package 就可以了。我们以 Human 类为例，将它放入包中:

```java
package com.vamei.society;

public class Human
{
    /**
     * constructor
     */
    public Human(int h)
    {
        this.height = h;
        System.out.println("I'm born");
    }

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
```

上面的第一行语句

package com.vamei.society;

表示该程序在 com.vamei.society 包中。com.vamei(vamei.com 的反写)表示包作者的域名 (很可惜，这个域名已经被别人注册了，所以只起演示作用)。Java 要求包要有域名前缀，以便区分不同作者。society 为进一步的本地路径名。com.vamei.society 共同构成了包的名字。

包为 Java 程序提供了一个命名空间(name space)。一个 Java 类的完整路径由它的包和类名共同构成，比如 com.vamei.society.Human。相应的 Human.java 程序要放在 com/vamei/society/下。类是由完整的路径识别的，所以不同的包中可以有同名的类，Java 不会混淆。比如 com.vamei.society.Human 和 com.vamei.creature.Human 是两个不同的类。

再看一个细节。Human 类是 public 的，其构造方法也是 public 的，所以任意其他对象都可以调用该类。我们之前说过，一个 Java 文件中只能有一个 public 的类，该类要去.java 文件同名。一个类可以没有 public 关键字，它实际上也表示一种权限: 该类在它所在的包中可见。也就是说，包中的其他 Java 程序可以访问该类。这是 Java 中的默认访问权限。

同样，对象的成员也可以是默认权限(包中可见)。比如我们去掉 getHeight()方法前面的 public 关键字。

### 包的调用

我们只需要将 Human.java 编译的 Human.class 放入相应的文件夹就可以了。比如，我将 Human.class 放入 com/vamei/society/中。实际上，你也可以把.java 文件放入相应路径，Java 会在使用时自动编译。

如果整个包(也就是 com 文件夹)位于当前的工作路径中，那么不需要特别的设置，就可以使用包了，比如下面的 TestAgain.java:

```java
import com.vamei.society.*;

public class TestAgain
{
    public static void main(String[] args)
    {
        Human aPerson = new Human(180);
        System.out.println(aPerson.getHeight());
    }

}
```

import 用于识别路径。利用 import 语句，我们可以引入相应路径下的类。*表示引入 society 文件夹下的所有类。在 TestAgain 中，我们直接使用了 Human 类。

我们也可以提供类的完整的路径。这可以区分同名但不同路径的类，比如:

```java
public class TestAgain
{
    public static void main(String[] args)
    {
        com.vamei.society.Human aPerson = 
                  new com.vamei.society.Human(180);
        System.out.println(aPerson.getHeight());
    }

}
```

由于我们提供了完整的类路径，所以不需要使用 import 语句。

如果包没有放在当前工作路径下，我们在使用包时，需要通知 Java。比如，我们将包放在/home/vamei/javapackage 中，这样 Human.class 位于/home/vamei/javapackage/com/vamei/society/Human.class，而我们的工作路径为/home/vamei。这样，包就无法被找到。一个方法是在使用 javac 和 java 时，用-classpath 说明包所在的文件夹路径，比如:

$javac -classpath /home/vamei/javapackage:. TestAgain.java

$java -classpath /home/vamei/javapackage:. TestAgain

就是从/home/vamei/javapackage 和工作路径(.)中寻找包。Java 可以从/home/vamei/javapackage 中可以找到 Human 类，从.中可以找到 TestAgain 类。

另外也可以设置系统的 CLASSPATH 环境变量，将上述路径加入到该变量中，而不用每次都键入-classpath 选项。

类似于包的机制在其他语言中也很常见，比如 Python 中的 import 机制。它们都是为了更好的组织和使用已有的程序。利用包，我们可以比较容易的拓展 Java 程序，使用已有的 Java 程序库。注意到，包管理的是.class 文件。Java 号称"一次编译，处处运行" (Compile Once, run anywhere)。.class 文件可以在任意装有 Java 虚拟机(JVM, Java Virtual Machine)的平台上运行，这帮助我们克服了系统差异造成的程序移植困难。

系统之间的差异可以非常大。如果我们用 C 语言编写程序，需要将源程序在各个平台上重新编译，以适应不同的硬件条件。 Java 虚拟机衔接了平台和 Java 宇宙，它构成了硬件和编程逻辑的中间层。JVM 隐藏了硬件差异，提供给程序员一个“标准”的 Java 宇宙。而.class 文件可以看做这个 Java 宇宙中流通的通货。在 JVM 的基础设施下，加上包的管理辅助，Java 程序实现了良好的可移植性 (portability)。

### 总结

package, import

默认权限: 包中可见

-classpath, CLASSPATH

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章