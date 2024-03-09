# 一、通往 lambda 之路：语法篇

> 来源：[Java 8 新特性探究（一）通往 lambda 之路 语法篇](http://my.oschina.net/benhaile/blog/175012)

现在开始要灌输一些概念性的东西了，这能帮助你理解 lambda 更加透彻一点，如果你之前听说过，也可当是温习，所谓温故而知新……

在开始之前，可以同步下载 jdk 8 和 IDE，IDE 根据个人习惯了，不过[Eclipse](http://res.importnew.com/eclipse "Eclipse ImportNew 主页")官方版本还没出来，所以目前看的话，netbean7.4 是首选的，毕竟前段子刚刚出的正式版本，以下是他们的下载地址。

*   jdk 8：[`jdk8.java.net/download.html`](https://jdk8.java.net/download.html) （毕竟是国外的网站，如果下载慢，可以到我的云盘下载[`pan.baidu.com/share/link?shareid=61064476&uk=4060588963`](http://pan.baidu.com/share/link?shareid=61064476&uk=4060588963)）
*   Netbeans 7.4 正式版：[`netbeans.org/downloads/`](https://netbeans.org/downloads/)(推荐，oracle 官方发布)
*   IDEA 12 EAP：[`confluence.jetbrains.net/display/IDEADEV/IDEA+12+EAP`](http://confluence.jetbrains.net/display/IDEADEV/IDEA+12+EAP)
*   Unofficial builds of Eclipse：[`downloads.efxclipse.org/eclipse-java8/`](http://my.oschina.net/benhaile/admin/:http:/downloads.efxclipse.org/eclipse-java8)

### **函数式接口**

函数式接口（functional interface 也叫功能性接口，其实是同一个东西）。简单来说，函数式接口是只包含一个方法的接口。比如 Java 标准库中的 java.lang.Runnable 和 java.util.Comparator 都是典型的函数式接口。java 8 提供 @FunctionalInterface 作为注解,这个注解是非必须的，只要接口符合函数式接口的标准（即只包含一个方法的接口），虚拟机会自动判断，但 最好在接口上使用注解@FunctionalInterface 进行声明，以免团队的其他人员错误地往接口中添加新的方法。

Java 中的 lambda 无法单独出现，它需要一个函数式接口来盛放，lambda 表达式方法体其实就是函数接口的实现，下面讲到语法会讲到

### **Lambda 语法**

包含三个部分

1.  一个括号内用逗号分隔的形式参数，参数是函数式接口里面方法的参数
2.  一个箭头符号：->
3.  方法体，可以是表达式和代码块，方法体函数式接口里面方法的实现，如果是代码块，则必须用{}来包裹起来，且需要一个 return 返回值，但有个例外，若函数式接口里面方法返回值是 void，则无需{}

总体看起来像这样

```java
(parameters) -> expression 或者 (parameters) -> { statements; } 
```

看一个完整的例子，方便理解

```java
/**
 * 测试 lambda 表达式
 *
 * @author benhail
 */
public class TestLambda {

    public static void runThreadUseLambda() {
        //Runnable 是一个函数接口，只包含了有个无参数的，返回 void 的 run 方法；
        //所以 lambda 表达式左边没有参数，右边也没有 return，只是单纯的打印一句话
        new Thread(() ->System.out.println("lambda 实现的线程")).start(); 
    }

    public static void runThreadUseInnerClass() {
        //这种方式就不多讲了，以前旧版本比较常见的做法
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("内部类实现的线程");
            }
        }).start();
    }

    public static void main(String[] args) {
        TestLambda.runThreadUseLambda();
        TestLambda.runThreadUseInnerClass();
    }
} 
```

可以看出，使用 lambda 表达式设计的代码会更加简洁，而且还可读。

### **方法引用**

其实是 lambda 表达式的一个简化写法，所引用的方法其实是 lambda 表达式的方法体实现，语法也很简单，左边是容器（可以是类名，实例名），中间是”::”，右边是相应的方法名。如下所示：

```java
ObjectReference::methodName 
```

一般方法的引用格式是

1.  如果是静态方法，则是 ClassName::methodName。如 Object ::equals
2.  如果是实例方法，则是 Instance::methodName。如 Object obj=new Object();obj::equals;
3.  构造函数.则是 ClassName::new

再来看一个完整的例子，方便理解

```java
import java.awt.FlowLayout;
import java.awt.event.ActionEvent;
import javax.swing.JButton;
import javax.swing.JFrame;

/**
 *
 * @author benhail
 */
public class TestMethodReference {

    public static void main(String[] args) {

        JFrame frame = new JFrame();
        frame.setLayout(new FlowLayout());
        frame.setVisible(true);

        JButton button1 = new JButton("点我!");
        JButton button2 = new JButton("也点我!");

        frame.getContentPane().add(button1);
        frame.getContentPane().add(button2);
        //这里 addActionListener 方法的参数是 ActionListener，是一个函数式接口
        //使用 lambda 表达式方式
        button1.addActionListener(e -> { System.out.println("这里是 Lambda 实现方式"); });
        //使用方法引用方式
        button2.addActionListener(TestMethodReference::doSomething);

    }
    /**
     * 这里是函数式接口 ActionListener 的实现方法
     * @param e 
     */
    public static void doSomething(ActionEvent e) {

        System.out.println("这里是方法引用实现方式");

    }
} 
```

可以看出，doSomething 方法就是 lambda 表达式的实现，这样的好处就是，如果你觉得 lambda 的方法体会很长，影响代码可读性，方法引用就是个解决办法

### **总结**

以上就是 lambda 表达式语法的全部内容了，相信大家对 lambda 表达式都有一定的理解了，但只是代码简洁了这个好处的话，并不能打动很多观众，java 8 也不会这么令人期待，其实 java 8 引入 lambda 迫切需求是因为 lambda 表达式能简化集合上数据的多线程或者多核的处理，提供更快的集合处理速度 ，这个后续会讲到，关于 JEP126 的这一特性，将分 3 部分，之所以分开，是因为这一特性可写的东西太多了，这部分让读者熟悉 lambda 表达式以及方法引用的语法和概念，第二部分则是虚拟扩展方法（default method）的内容，最后一部分则是大数据集合的处理，解开 lambda 表达式的最强作用的神秘面纱。敬请期待。。。。