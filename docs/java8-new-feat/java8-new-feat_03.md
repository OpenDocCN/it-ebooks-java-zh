# 三、解开 lambda 最强作用的神秘面纱

> 来源：[Java 8 新特性探究（三）解开 lambda 最强作用的神秘面纱](http://my.oschina.net/benhaile/blog/177148)

我们期待了很久 lambda 为 java 带来闭包的概念，但是如果我们不在集合中使用它的话，就损失了很大价值。现有接口迁移成为 lambda 风格的问题已经通过 default methods 解决了，在这篇文章将深入解析 Java 集合里面的批量数据操作（bulk operation），解开 lambda 最强作用的神秘面纱。

### **1.关于 JSR335**

JSR 是 Java Specification Requests 的缩写，意思是 Java 规范请求,Java 8 版本的主要改进是 Lambda 项目（JSR 335），其目的是使 Java 更易于为多核处理器编写代码。JSR 335=[lambda 表达式](http://my.oschina.net/benhaile/blog/175012)+接口改进（[默认方法](http://my.oschina.net/benhaile/blog/176007)）+批量数据操作。加上前面两篇，我们已是完整的学习了 JSR335 的相关内容了。

### **2.外部 VS 内部迭代**

以前 Java 集合是不能够表达内部迭代的，而只提供了一种外部迭代的方式，也就是 for 或者 while 循环。

```java
List persons = asList(new Person("Joe"), new Person("Jim"), new Person("John"));
for (Person p :  persons) {
   p.setLastName("Doe");
} 
```

上面的例子是我们以前的做法，也就是所谓的外部迭代，循环是固定的顺序循环。在现在多核的时代，如果我们想并行循环，不得不修改以上代码。效率能有多大提升还说定，且会带来一定的风险（线程安全问题等等）。

要描述内部迭代，我们需要用到 Lambda 这样的类库,下面利用 lambda 和 Collection.forEach 重写上面的循环

```java
persons.forEach(p->p.setLastName("Doe")); 
```

现在是由 jdk 库来控制循环了，我们不需要关心 last name 是怎么被设置到每一个 person 对象里面去的，库可以根据运行环境来决定怎么做，并行，乱序或者懒加载方式。这就是内部迭代，客户端将行为 p.setLastName 当做数据传入 api 里面。

内部迭代其实和集合的批量操作并没有密切的联系，借助它我们感受到语法表达上的变化。真正有意思的和批量操作相关的是新的流（stream）API。新的 java.util.stream 包已经添加进 JDK 8 了。

### **3.Stream API**

流（Stream）仅仅代表着数据流，并没有数据结构，所以他遍历完一次之后便再也无法遍历（这点在编程时候需要注意，不像 Collection，遍历多少次里面都还有数据），它的来源可以是 Collection、array、io 等等。

**3.1 中间与终点方法**

流作用是提供了一种操作大数据接口，让数据操作更容易和更快。它具有过滤、映射以及减少遍历数等方法，这些方法分两种：中间方法和终端方法，“流”抽象天生就该是持续的，中间方法永远返回的是 Stream，因此如果我们要获取最终结果的话，必须使用终点操作才能收集流产生的最终结果。区分这两个方法是看他的返回值，如果是 Stream 则是中间方法，否则是终点方法。具体请参照[Stream 的 api](http://download.java.net/jdk8/docs/api/java/util/stream/Stream.html)。

简单介绍下几个中间方法（filter、map）以及终点方法（collect、sum）

**3.1.1Filter**

在数据流中实现过滤功能是首先我们可以想到的最自然的操作了。Stream 接口暴露了一个 filter 方法，它可以接受表示操作的[Predicate](http://javadocs.techempower.com/jdk18/api/java/util/function/Predicate.html)实现来使用定义了过滤条件的 lambda 表达式。

```java
List persons = …
Stream personsOver18 = persons.stream().filter(p -> p.getAge() > 18);//过滤 18 岁以上的人 
```

**3.1.2Map**

假使我们现在过滤了一些数据，比如转换对象的时候。Map 操作允许我们执行一个[Function](http://javadocs.techempower.com/jdk18/api/java/util/function/Function.html)的实现（Function<tu0002cr class="calibre23">的泛型 T,R 分别表示执行输入和执行结果），它接受入参并返回。首先，让我们来看看怎样以匿名内部类的方式来描述它：</tu0002cr>

```java
Stream adult= persons
              .stream()
              .filter(p -> p.getAge() > 18)
              .map(new Function() {
                  @Override
                  public Adult apply(Person person) {
                     return new Adult(person);//将大于 18 岁的人转为成年人
                  }
              }); 
```

现在，把上述例子转换成使用 lambda 表达式的写法：

```java
Stream map = persons.stream()
                    .filter(p -> p.getAge() > 18)
                    .map(person -> new Adult(person)); 
```

**3.1.3Count**

count 方法是一个流的终点方法，可使流的结果最终统计，返回 int，比如我们计算一下满足 18 岁的总人数

```java
int countOfAdult=persons.stream()
                       .filter(p -> p.getAge() > 18)
                       .map(person -> new Adult(person))
                       .count(); 
```

**3.1.4Collect**

collect 方法也是一个流的终点方法，可收集最终的结果

```java
List adultList= persons.stream()
                       .filter(p -> p.getAge() > 18)
                       .map(person -> new Adult(person))
                       .collect(Collectors.toList()); 
```

或者，如果我们想使用特定的实现类来收集结果：

```java
List adultList = persons
                 .stream()
                 .filter(p -> p.getAge() > 18)
                 .map(person -> new Adult(person))
                 .collect(Collectors.toCollection(ArrayList::new)); 
```

篇幅有限，其他的中间方法和终点方法就不一一介绍了，看了上面几个例子，大家明白这两种方法的区别即可，后面可根据需求来决定使用。

**3.2 顺序流与并行流**

每个 Stream 都有两种模式：顺序执行和并行执行。

顺序流：

```java
List <Person> people = list.getStream.collect(Collectors.toList()); 
```

并行流：

```java
List <Person> people = list.getStream.parallel().collect(Collectors.toList()); 
```

顾名思义，当使用顺序方式去遍历时，每个 item 读完后再读下一个 item。而使用并行去遍历时，数组会被分成多个段，其中每一个都在不同的线程中处理，然后将结果一起输出。

**3.2.1 并行流原理：**

```java
List originalList = someData;
split1 = originalList(0, mid);//将数据分小部分
split2 = originalList(mid,end);
new Runnable(split1.process());//小部分执行操作
new Runnable(split2.process());
List revisedList = split1 + split2;//将结果合并 
```

大家对 hadoop 有稍微了解就知道，里面的 MapReduce 本身就是用于并行处理大数据集的软件框架，其 处理大数据的核心思想就是大而化小，分配到不同机器去运行 map，最终通过 reduce 将所有机器的结果结合起来得到一个最终结果，与 MapReduce 不同，Stream 则是利用多核技术可将大数据通过多核并行处理，而 MapReduce 则可以分布式的。

**3.2.2 顺序与并行性能测试对比**

如果是多核机器，理论上并行流则会比顺序流快上一倍，下面是测试代码

```java
long t0 = System.nanoTime();

//初始化一个范围 100 万整数流,求能被 2 整除的数字，toArray（）是终点方法

int a[]=IntStream.range(0, 1_000_000).filter(p -> p % 2==0).toArray();

long t1 = System.nanoTime();

//和上面功能一样，这里是用并行流来计算

int b[]=IntStream.range(0, 1_000_000).parallel().filter(p -> p % 2==0).toArray();

long t2 = System.nanoTime();

//我本机的结果是 serial: 0.06s, parallel 0.02s，证明并行流确实比顺序流快

System.out.printf("serial: %.2fs, parallel %.2fs%n", (t1 - t0) * 1e-9, (t2 - t1) * 1e-9); 
```

**3.3 关于 Folk/Join 框架**

应用硬件的并行性在 java 7 就有了，那就是 java.util.concurrent 包的新增功能之一是一个 fork-join 风格的并行分解框架，同样也很强大高效，有兴趣的同学去研究，这里不详谈了，相比 Stream.parallel()这种方式，我更倾向于后者。

### **4.总结**

如果没有 lambda，Stream 用起来相当别扭，他会产生大量的匿名内部类，比如上面的 3.1.2map 例子，如果没有 default method，集合框架更改势必会引起大量的改动，所以 lambda+default method 使得 jdk 库更加强大，以及灵活，Stream 以及集合框架的改进便是最好的证明。

java 8 特性探究系列写了 3 篇了，作为大餐，将 java 8 的重量级特性 lambda 与 default method 写在前面，下篇上个小菜，荤素搭配，也是语言相关的，JEP104 Java 类型的注解的探究，同时谢谢大家的支持，欢迎提出建议。如果你想了解哪些特性，欢迎给我发留言。