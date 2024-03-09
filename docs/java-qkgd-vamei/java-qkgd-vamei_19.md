# Java 进阶 05 多线程 Python 多线程与同步 Linux 多线程与同步

作者：Vamei 出处：http://www.cnblogs.com/vamei 欢迎转载，也请保留这段声明。谢谢！ 

### 多线程

多线程(multiple thread)是计算机实现多任务并行处理的一种方式。

在单线程情况下，计算机中存在一个控制权，并按照顺序依次执行指令。单线程好像是一个只有一个队长指挥的小队，整个小队同一个时间只能执行一个任务。

![](img/22b56acb9607e73ce3ad67e60f91462c.jpg)

单线程

在多线程情境下，计算机中有多个控制权。多个控制权可以同时进行，每个控制权依次执行一系列的指令。多线程好像是一个小队中的成员同时执行不同的任务。

可参考[Linux 多线程与同步](http://www.cnblogs.com/vamei/archive/2012/10/09/2715393.html)，并对比[Python 多线程与同步](http://www.cnblogs.com/vamei/archive/2012/10/11/2720042.html)

![](img/d26249476af7ea9263d9a598bfb8e5ac.jpg)

多线程

传统意义上，多线程是由操作系统提供的功能。对于单核的 CPU，硬件中只存在一个线程。在操作系统的控制下，CPU 会在不同的任务间(线程间)切换，从而造成多任务齐头并进的效果。这是单 CPU 分时复用机制下的多线程。现在，随着新的硬件技术的发展，硬件本身开始提供多线程支持，比如多核和超线程技术。然而，硬件的多线程还是要接受操作系统的统一管理。在操作系统之上的多线程程序依然通用。

多个线程可以并存于同一个进程空间。在 JVM 的一个进程空间中，一个栈(stack)代表了方法调用的次序。对于多线程来说，进程空间中需要有多个栈，以记录不同线程的调用次序。多个栈互不影响，但所有的线程将共享堆(heap)中的对象。

### 创建线程

Java 中“一切皆对象”，线程也被封装成一个对象。我们可以通过继承 Thread 类来创建线程。线程类中的的 run()方法包含了该线程应该执行的指令。我们在衍生类中覆盖该方法，以便向线程说明要做的任务:

```java
public class Test
{
    public static void main(String[] args)
    {
        NewThread thread1 = new NewThread();
        NewThread thread2 = new NewThread();
        thread1.start(); // start thread1
        thread2.start(); // start thread2
    }
}

/**
 * create new thread by inheriting Thread
 */
class NewThread extends Thread {

    private static int threadID = 0; // shared by all

    /**
     * constructor
     */
    public NewThread() {
        super("ID:" + (++threadID));
    }

    /**
     * convert object to string
     */
    public String toString() {
        return super.getName();
    }

    /**
     * what does the thread do?
     */
    public void run() {
        System.out.println(this);
    }
}
```

(++是 Java 中的累加运算符，即让变量加 1。这里++出现在 threadID 之前，说明先将 threadID 加 1，再对周边的表达式求值

toString 是 Object 根类的方法，我们通过覆盖该方法，来将对象转换成字符串。当我们打印该对象时，Java 将自动调用该方法。)

可以看到，Thread 基类的构建方法(super())可以接收一个字符串作为参数。该字符串是该线程的名字，并使用 getName()返回。

定义类之后，我们在 main()方法中创建线程对象。每个线程对象为一个线程。创建线程对象后，线程还没有开始执行。

我们调用线程对象的 start()方法来启动线程。start()方法可以在构造方法中调用。这样，我们一旦使用 new 创建线程对象，就立即执行。

Thread 类还提供了下面常用方法:

join(Thread tr)   等待线程 tr 完成

setDaemon()       设置当前线程为后台 daemon (进程结束不受 daemon 线程的影响)

Thread 类官方文档: [`docs.oracle.com/javase/6/docs/api/java/lang/Thread.html`](http://docs.oracle.com/javase/6/docs/api/java/lang/Thread.html)

### Runnable

实现多线程的另一个方式是实施 Runnable 接口，并提供 run()方法。实施接口的好处是容易实现多重继承(multiple inheritance)。然而，由于内部类语法，继承 Thread 创建线程可以实现类似的功能。我们在下面给出一个简单的例子，而不深入:

```java
public class Test
{
    public static void main(String[] args)
    {
        Thread thread1 = new Thread(new NewThread(), "first");
        Thread thread2 = new Thread(new NewThread(), "second");
        thread1.start(); // start thread1
        thread2.start(); // start thread2
    }
}

/**
 * create new thread by implementing Runnable
 */
class NewThread implements Runnable {
    /**
     * convert object to string
     */
    public String toString() {
        return Thread.currentThread().getName();
    }

    /**
     * what does the thread do?
     */
    public void run() {
        System.out.println(this);
    }
}
```

### synchronized

多任务编程的难点在于多任务共享资源。对于同一个进程空间中的多个线程来说，它们都共享堆中的对象。某个线程对对象的操作，将影响到其它的线程。

在多线程编程中，要尽力避免竞争条件(racing condition)，即运行结果依赖于不同线程执行的先后。线程是并发执行的，无法确定线程的先后，所以我们的程序中不应该出现竞争条件。

然而，当多任务共享资源时，就很容易造成竞争条件。我们需要将共享资源，并造成竞争条件的多个线程线性化执行，即同一时间只允许一个线程执行。

(可更多参考[Linux 多线程与同步](http://www.cnblogs.com/vamei/archive/2012/10/09/2715393.html))

下面是一个售票程序。3 个售票亭(Booth)共同售卖 100 张票(Reservoir)。每个售票亭要先判断是否有余票，然后再卖出一张票。如果只剩下一张票，在一个售票亭的判断和售出两个动作之间，另一个售票亭卖出该票，那么第一个售票亭(由于已经执行过判断)依然会齿形卖出，造成票的超卖。为了解决该问题，判断和售出两个动作之间不能有“空隙”。也就是说，在一个线程完成了这两个动作之后，才能有另一个线程执行。

在 Java 中，我们将共享的资源置于一个对象中，比如下面 r(Reservoir)对象。它包含了总共的票数；将可能造成竞争条件的，针对共享资源的操作，放在 synchronized(同步)方法中，比如下面的 sellTicket()。synchronized 是方法的修饰符。在 Java 中，同一对象的 synchronized 方法只能同时被一个线程调用。其他线程必须等待该线程调用结束，(余下的线程之一)才能运行。这样，我们就排除了竞争条件的可能。

在 main()方法中，我们将共享的资源(r 对象)传递给多个线程:

```java
public class Test
{
    public static void main(String[] args)
    {
        Reservoir r = new Reservoir(100);
        Booth b1 = new Booth(r);
        Booth b2 = new Booth(r);
        Booth b3 = new Booth(r);
    }
}

/**
 * contain shared resource
 */
class Reservoir {
    private int total;

    public Reservoir(int t) 
    {
        this.total = t;
    }

    /**
     * Thread safe method
     * serialized access to Booth.total
     */
    public synchronized boolean sellTicket() 
    {
        if(this.total > 0) {
            this.total = this.total - 1;
            return true; // successfully sell one
        }
        else {
            return false; // no more tickets
        }
    }
}

/**
 * create new thread by inheriting Thread
 */
class Booth extends Thread {
    private static int threadID = 0; // owned by Class object

    private Reservoir release;      // sell this reservoir 
    private int count = 0;          // owned by this thread object
    /**
     * constructor
     */
    public Booth(Reservoir r) {
        super("ID:" + (++threadID));
        this.release = r;          // all threads share the same reservoir
        this.start();
    }

    /**
     * convert object to string
     */
    public String toString() {
        return super.getName();
    }

    /**
     * what does the thread do?
     */
    public void run() {
        while(true) {
            if(this.release.sellTicket()) {
                this.count = this.count + 1;
                System.out.println(this.getName() + ": sell 1");
                try {
                    sleep((int) Math.random()*100);   // random intervals
                }
                catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            else {
                break;
            }
        }
        System.out.println(this.getName() + " I sold:" + count);
    }
}
```

(Math.random()用于产生随机数)

Java 的每个对象都自动包含有一个用于支持同步的计数器，记录 synchronized 方法的调用次数。线程获得该计数器，计数器加 1，并执行 synchronized 方法。如果方法内部进一步调用了该对象的其他 synchronized 方法，计数器加 1。当 synchronized 方法调用结束并退出时，计数器减 1。其他线程如果也调用了同一对象的 synchronized 方法，必须等待该计数器变为 0，才能锁定该计数器，开始执行。Java 中的类同样也是对象([Class 类对象](http://www.cnblogs.com/vamei/archive/2013/04/14/3013985.html))。Class 类对象也包含有计数器，用于同步。

### 关键代码

上面，我们利用 synchronized 修饰符同步了整个方法。我们可以同步部分代码，而不是整个方法。这样的代码被称为关键代码(critical section)。我们使用下面的语法:

```java
synchronized (syncObj) {

  ...;

}
```

花括号中包含的是想要同步的代码，syncObj 是任意对象。我们将使用 syncObj 对象中的计数器，来同步花括号中的代码。

欢迎继续阅读“[Java 快速教程](http://www.cnblogs.com/vamei/archive/2013/03/31/2991531.html)”系列文章