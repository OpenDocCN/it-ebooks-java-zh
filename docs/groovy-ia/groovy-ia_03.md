# 实战 Groovy: @Delegate 注释

*探索静态类型语言中的 duck 类型的极限*

Scott Davis 将继续有关 Groovy 元编程的讨论，这一次他将深入研究 `@Delegate` 注释，@Delegate 注释模糊了数据类型和行为以及静态和动态类型之间的区别。

在过去几期 [*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 文章中，您已经了解了闭包和元编程之类的 Groovy 语言特性如何将动态功能添加到 Java™ 开发中。本文提供了更多这方面的内容。您将看到 `@Delegate` 注释如何演变自 `ExpandoMetaClass` 使用的 `delegate`。您将再一次领略到 Groovy 的动态功能如何使它成为单元测试的理想语言。

在 “[使用闭包、ExpandoMetaClass 和类别进行元编程](http://www.ibm.com/developerworks/cn/java/j-pg06239.html)” 一文中，您了解了 `delegate` 的概念。当将一个 `shout()` 方法添加到 `java.lang.String` 的 `ExpandoMetaClass` 中时，您使用 `delegate` 来表示两个类之间的关系，如清单 1 所示：

##### 清单 1\. 使用 `delegate` 访问 `String.toUpperCase()`

```java
String.metaClass.shout = {->
  return delegate.toUpperCase()
}

println "Hello MetaProgramming".shout()

//output
HELLO METAPROGRAMMING 
```

您不能表示为 `this.toUpperCase()`，因为 `ExpandoMetaClass` 并未包含 `toUpperCase()` 方法。类似地，也不能表示为 `super.toUpperCase()`，因为 `ExpandoMetaClass` 没有扩展 `String`。（事实上，它不可能扩展 `String`，因为 `String` 是一个 `final` 类）。Java 语言并不具备用于表示这两个类之间的共生关系的词汇。这就是为什么 Groovy 要引入 `delegate` 概念。

## 关于本系列

Groovy 是在 Java 平台上运行的一种现代编程语言。它能够与现有 Java 代码无缝集成，同时引入了各种生动的新特性，比如闭包和元编程。简单来讲，Groovy 是 Java 语言的 21 世纪版本。

将任何新工具整合到开发工具包中的关键是知道何时使用它以及何时将它留在工具包中。Groovy 的功能可以非常强大，但唯一的条件是正确应用于适当的场景。因此，[*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 系列将探究 Groovy 的实际应用，以便帮助您了解何时以及如何成功使用它们。

在 Groovy 1.6 中，`@Delegate` 注释被添加到该语言中。（从 参考资料 部分可以获得添加到 Groovy 1.6 中的所有新注释的列表）。该注释允许您向*任意* 类添加一个或多个委托 — 而不仅仅是 `ExpandoMetaClass`。

要充分地认识到 `@Delegate` 注释的威力，考虑 Java 编程中一个常见但复杂的任务：在 `final` 类的基础上创建一个新类。

## 复合模式和 `final` 类

假设您希望创建一个 `AllCapsString` 类，它具有 `java.lang.String` 的所有行为，唯一的不同是 — 正如名称暗示的那样 — 值始终以大写的形式返回。`String` 是一个 `final` 类 — Java 演化到尽头的产物。清单 2 证明您无法直接扩展 `String`：

##### 清单 2\. 扩展 `final` 类是不可能的

```java
class AllCapsString extends String{
}

$ groovyc AllCapsString.groovy

org.codehaus.groovy.control.MultipleCompilationErrorsException:
startup failed, AllCapsString.groovy: 1: You are not allowed to
overwrite the final class 'java.lang.String'.
 @ line 1, column 1.
   class AllCapsString extends String{
   ^

1 error 
```

这段代码无效，因此您的下一个最佳选择就是使用符合模式，如清单 3 所示（有关复合模式的更多信息，请参见 参考资料）：

##### 清单 3\. 对 `String` 类的新类型使用复合模式

```java
class AllCapsString{
  final String body

  AllCapsString(String body){
    this.body = body.toUpperCase()
  }

  String toString(){
    body
  }

  //now implement all 72 String methods
  char charAt(int index){
    return body.charAt(index)
  }

  //snip...
  //one method down, 71 more to go...
} 
```

因此，`AllCapsString` 类*拥有* 一个 `String`，但是其*行为* 不同于 `String`，除非您映射了所有 72 个 `String` 方法。要查看需要添加的方法，可以参考 Javadocs 中有关 `String` 的内容，或者运行清单 4 中的代码：

##### 清单 4\. 输出 `String` 类的所有方法

```java
String.class.methods.eachWithIndex{method, i->
  println "${i} ${method}"
}

//output
0 public boolean java.lang.String.contentEquals(java.lang.CharSequence)
1 public boolean java.lang.String.contentEquals(java.lang.StringBuffer)
2 public boolean java.lang.String.contains(java.lang.CharSequence)
... 
```

将 72 个 `String` 方法手动添加到 `AllCapsString` 并不是一种明智的方法，而是在浪费开发人员的宝贵时间。这就是 `@Delegate` 注释发挥作用的时候了。

* * *

## 了解 `@Delegate`

`@Delegate` 是一个编译时注释，指导编译器将所有 delegate 的方法和接口推到外部类中。

在将 `@Delegate` 注释添加到 `body` 之前，编译 `AllCapsString` 并使用 `javap` 进行检验，看看大部分 `String` 方法是否缺失，如清单 5 所示：

##### 清单 5\. 在使用 `@Delegate` 前使用 `AllCapsString`

```java
$ groovyc AllCapsString.groovy
$ javap AllCapsString
Compiled from "AllCapsString.groovy"
public class AllCapsString extends java.lang.Object
       implements groovy.lang.GroovyObject{
    public AllCapsString(java.lang.String);
    public java.lang.String toString();
    public final java.lang.String getBody();
    //snip... 
```

现在，将 `@Delegate` 注释添加到 `body`，如清单 6 所示。重复 `groovyc` 和 `javap` 命令，将看到 `AllCapsString` 具有与 `java.lang.String` 相同的所有方法和接口。

##### 清单 6\. 使用 `@Delegate` 注释将 `String` 的所有方法推到周围的类中

```java
class AllCapsString{
  @Delegate final String body

  AllCapsString(String body){
    this.body = body.toUpperCase()
  }

  String toString(){
    body
  }
}

$ groovyc AllCapsString.groovy
$ javap AllCapsString
Compiled from "AllCapsString.groovy"
public class AllCapsString extends java.lang.Object
       implements java.lang.CharSequence, java.lang.Comparable,
       java.io.Serializable,groovy.lang.GroovyObject{

    //NOTE: AllCapsString methods:   
    public AllCapsString(java.lang.String);
    public java.lang.String toString();
    public final java.lang.String getBody();    

    //NOTE: java.lang.String methods:
    public boolean contains(java.lang.CharSequence);
    public int compareTo(java.lang.Object);
    public java.lang.String toUpperCase();
    //snip... 
```

然而，注意，您仍然可以调用 `getBody()`，从而绕过被推入到环绕的 `AllCapsString` 类中的所有方法。通过将 `private` 添加到字段声明中 — `@Delegate final private String body` — 可以禁止显示普通的 getter/setter 方法。这将完成转换：`AllCapsString` 提供了 `String` 的全部行为，允许您根据情况覆盖 `String` 方法。

* * *

## 在静态语言中使用 duck 类型的限制

尽管 `AllCapsString` 目前拥有 `String` 的所有行为，但是它仍然不是一个*真正的*`String`。在 Java 代码中，无法使用 `AllCapsString` 作为 `String` 的临时替代，因为它并不是一个真正的 duck — 它只不过是冒充的。（动态语言被认为是使用 *duck* 类型；Java 语言使用*静态* 类型。参见 参考资料 获得更多与此有关的差异）。换句话说，由于 `AllCapsString` 并未真正扩展 `String`（或实现并不存在的 `Stringable` 接口），因此无法在 Java 代码中与 `String` 互相替换。清单 7 展示了在 Java 语言中将 `AllCapsString` 转换为 `String` 的失败例子：

##### 清单 7\. Java 语言中的静态类型阻止 `AllCapsString` 与 `String` 之间互相替换

```java
public class JavaExample{
  public static void main(String[] args){
    String s = new AllCapsString("Hello");
  }
}

$ javac JavaExample.java
JavaExample.java:5: incompatible types
found   : AllCapsString
required: java.lang.String
    String s = new AllCapsString("Hello");
               ^
1 error 
```

因此，通过允许您扩展被最初的开发人员明确禁止扩展的类，Groovy 的 `@Delegate` 并没有真正破坏 Java 的 `final` 关键字，但是您仍然可以获得与在不越界的情况下相同程度的威力。

请记住，您的类可以拥有多个 `delegate`。假设您希望创建一个 `RemoteFile` 类，它将同时具有 `java.io.File` 和 `java.net.URL` 的特征。Java 语言并不支持多重继承，但是您可以非常接近一对 `@Delegate`，如清单 8 所示。`RemoteFile` 类不是 `File` 也不是 `URL`，但是它却具有两者的行为。

##### 清单 8\. 多个 `@Delegate` 提供了多重继承的行为

```java
class RemoteFile{
  @Delegate File file
  @Delegate URL url
} 
```

如果 `@Delegate` 只能修改类的行为 — 而不是类型 — 这是否意味着对 Java 开发人员毫无价值？未必，即使是 Java 之类的静态类型语言也为 duck 类型提供了一种有限的形式，称为*多态*。

### 具有多态性的 duck

多态 — 该词源于希腊，用于描述 “多种形状” — 意味着只要一组类通过实现相同接口显式地共享相同的行为，它们就可以互相替换着使用。换句话说，如果定义了一个 `Duck` 类型的变量（假设 `Duck` 是一个正式定义 `quack()` 和 `waddle()` 方法的接口），那么可以将 `new Mallard()`、`new GreenWingedTeal()` 或者（我最喜爱的）`new PekingWithHoisinSauce()` 分配给它。

通过将 delegate 类的方法和接口全部提升到其他类，`@Delegate` 注释为多态提供了完整的支持。这意味着如果 delegate 类实现了接口，您又回到了为它创建一个临时替代这件事上来。

* * *

## `@Delegate` 和 `List` 接口

假设您希望创建一个名为 `FixedList` 的新类。它的行为应该类似 `java.util.ArrayList`，但是有一个重要的区别：您应当能够为可以添加到其中的元素的数量定义一个上限。这允许您创建一个 `sportsCar` 变量，该变量可以容纳两个乘客，但是不能比这再多了，`restaurantTable` 可以容纳 4 个用餐者，但是同样不能超过这个数字，以此类推。

`ArrayList` 类实现 `List` 接口。它为您提供了两个选项。您也可以让您的 `FixedList` 类实现 `List` 接口，但是您需要面对一项烦人的工作：为所有 `List` 方法提供一个实现。由于 `ArrayList` 并不是 `final` 类，另一个选择就是让 `FixedList` 扩展 `ArrayList`。这是一个非常有效的做法，但是如果（假设）`ArrayList` 被声明为 `final`，`@Delegate` 注释将提供第三个选择：通过将 `ArrayList` 作为 `FixedList` 的委托，您可以获得 `ArrayList` 的所有行为，同时自动实现 `List` 接口。

首先，使用一个 `ArrayList` 委托创建 `FixedList` 类，如清单 9 所示。`groovyc` / `javap` 是否可以检验 `FixedList` 不仅提供了与 `ArrayList` 相同的方法，还提供了相同的接口。

##### 清单 9\. 第一步创建 `FixedList` 类

```java
class FixedList{
  @Delegate private List list = new ArrayList()
  final int sizeLimit

  /**
    * NOTE: This constructor limits the max size of the list,
    *  not just the initial capacity like an ArrayList.
    */
  FixedList(int sizeLimit){
    this.sizeLimit = sizeLimit
  }
}  

$ groovyc FixedList.groovy
$ javap FixedList
Compiled from "FixedList.groovy"
public class FixedList extends java.lang.Object
             implements java.util.List,java.lang.Iterable,
             java.util.Collection,groovy.lang.GroovyObject{
    public FixedList(int);
    public java.lang.Object[] toArray(java.lang.Object[]);
    //snip.. 
```

目前我们还没有对 `FixedList` 的大小做任何限制，但这是一个很好的开始。如何确定 `FixedList` 的大小此时并不是*固定的*？您可以编写一些用后即扔的样例代码，但是如果 `FixedList` 将投入到生产中，您最好立即为其编写一些测试用例。

* * *

## 使用 `GroovyTestCase` 测试 `@Delegate`

要开始测试 `@Delegate`，编写一个单元测试，验证您可以将比您实际可添加的更多元素添加到 `FixedList`。清单 10 展示了这样一个测试：

##### 清单 10\. 首先编写一个失败的测试

```java
class FixedListTest extends GroovyTestCase{

  void testAdd(){
    List threeStooges = new FixedList(3)
    threeStooges.add("Moe")    
    threeStooges.add("Larry")
    threeStooges.add("Curly")
    threeStooges.add("Shemp")      
    assertEquals threeStooges.sizeLimit, threeStooges.size()
  }
}

$ groovy FixedListTest.groovy

There was 1 failure:
1) testAdd(FixedListTest)junit.framework.AssertionFailedError:
   expected:<3> but was:<4> 
```

似乎 `add()` 方法应当在 `FixedList` 中被重写，如清单 11 所示。重新运行这些测试仍然失败，但是这一次是因为抛出了异常。

##### 清单 11\. 重写 `ArrayList` 的 `add()` 方法

```java
class FixedList{
  @Delegate private List list = new ArrayList()
  final int sizeLimit

  //snip...

  boolean add(Object element){
    if(list.size() < sizeLimit){
      return list.add(element)
    }else{
      throw new UnsupportedOperationException("Error adding ${element}:" +
                 " the size of this FixedList is limited to ${sizeLimit}.")
    }
  }
}

$ groovy FixedListTest.groovy

There was 1 error:
1) testAdd(FixedListTest)java.lang.UnsupportedOperationException:
   Error adding Shemp: the size of this FixedList is limited to 3. 
```

由于使用了 `GroovyTestCase` 的方便的 `shouldFail` 方法，您可以捕捉到这个预期的异常，如清单 12 所示，这一次您终于成功运行了测试：

##### 清单 12\. `shouldFail()` 方法捕捉到预期的异常

```java
class FixedListTest extends GroovyTestCase{
  void testAdd(){
    List threeStooges = new FixedList(3)
    threeStooges.add("Moe")    
    threeStooges.add("Larry")
    threeStooges.add("Curly")
    assertEquals threeStooges.sizeLimit, threeStooges.size()
    shouldFail(java.lang.UnsupportedOperationException){
      threeStooges.add("Shemp")      
    }
  }
} 
```

* * *

## 测试操作符重载

在 “[美妙的操作符](http://www.ibm.com/developerworks/cn/java/j-pg10255.html)” 中，您了解到 Groovy 支持*操作符重载*。对于 `List`，可以使用 `&lt;&lt;` 添加元素以及传统的 `add()` 方法。编写如清单 13 所示的快速单元测试，确定使用 `&lt;&lt;` 不会意外破坏 `FixedList`：

##### 清单 13\. 测试操作员重载

```java
class FixedListTest extends GroovyTestCase{

  void testOperatorOverloading(){
    List oneList = new FixedList(1)
    oneList << "one"
    shouldFail(java.lang.UnsupportedOperationException){
      oneList << "two"
    }    
  }
} 
```

这次测试的成功应该能够让您感到轻松一些。

您还可以测试出错的情况。比如，清单 14 测试了在创建包含一个负数元素的 `FixedList` 时出现的情况：

##### 清单 14\. 测试极端情况

```java
class FixedListTest extends GroovyTestCase{
  void testNegativeSize(){
    List badList = new FixedList(-1)
    shouldFail(java.lang.UnsupportedOperationException){
      badList << "will this work?"
    }    
  }
} 
```

* * *

## 测试将一个元素插入到列表中间的情况

现在，您已经确信这个简单的重写过的 `add()` 方法可以正常工作，下一步是实现重载的 `add()` 方法，可以获取索引以及元素，如清单 15 所示：

##### 清单 15\. 使用索引添加元素

```java
class FixedList{
  @Delegate private List list = new ArrayList()
  final int sizeLimit

  void add(int index, Object element){
    list.add(index, element)
    trimToSize()
  }

  private void trimToSize(){
    if(list.size() > sizeLimit){
      (sizeLimit..<list.size()).each{
        list.pop()
      }
    }
  }
} 
```

注意，您可以（也应该）在任何可能的情况下使用 delegate 自带的功能 — 毕竟，这正是您优先选择 delegate 的原因。在这种情况下，您将让 `ArrayList` 执行添加操作，并去掉任何超出 `FixedList` 的大小的元素。（这个 `add()` 方法是否应该像另一个 `add()` 方法那样抛出一个 `UnsupportedOperationException`，您可以自己做出这个设计决策）。

`trimToSize()` 方法包含了一些值得关注的语法糖。首先，`pop()` 方法是由 Groovy 元编程到所有 `List` 中的内容。它删除了 `List` 中的最后一个元素，使用后进先出（last-in first-out，LIFO）的方式。

接下来，注意 `each` 循环中使用了一个 Groovy `range`。使用实数替换变量可能有助于使这一行为更加清晰。假设 `FixedList` 的 `sizeLimit` 的值为 `3`，并且在添加了新元素后，它的 `size()` 的值为 `5`。那么这个范围看上去应当类似于 `(3..5).each{}`。但是 `List` 使用的是基于 0 的标记法，因此列表中的元素不会拥有值为 `5` 的索引。通过指定 `(3..&lt;5).each{}`，您将 5 排除到了这个范围之外。

编写两个测试，如清单 16 所示，检验新的重载后的 `add()` 方法是否如期望的那样运行：

##### 清单 16\. 测试将元素添加到 `FixedList` 中的情况

```java
class FixedListTest extends GroovyTestCase{
  void testAddWithIndex(){
    List threeStooges = new FixedList(3)
    threeStooges.add("Moe")    
    threeStooges.add("Larry")
    threeStooges.add("Curly")
    threeStooges.add(2,"Shemp")
    assertEquals 3, threeStooges.size()
    assertFalse threeStooges.contains("Curly")
  }

  void testAddWithIndexOnALessThanFullList(){
    List threeStooges = new FixedList(3)
    threeStooges.add("Curly")
    assertEquals 1, threeStooges.size()

    threeStooges.add(0, "Larry")
    assertEquals 2, threeStooges.size()
    assertEquals "Larry", threeStooges[0]

    threeStooges.add(0, "Moe")
    assertEquals 3, threeStooges.size()
    assertEquals "Moe", threeStooges[0]
    assertEquals "Larry", threeStooges[1]
    assertEquals "Curly", threeStooges[2]
  }
} 
```

您是否注意到编写的测试代码的数量要多于生产代码？很好！我想说的是，对于每一段生产代码，您应当编写至少两倍数量的测试代码。

* * *

## 实现 `addAll()` 方法

要实现 `FixedList` 类，重写 `ArrayList` 中的 `addAll()` 方法，如清单 17 所示：

##### 清单 17\. 实现 `addAll()` 方法

```java
class FixedList{
  @Delegate private List list = new ArrayList()
  final int sizeLimit

  boolean addAll(Collection collection){
    def returnValue = list.addAll(collection)
    trimToSize()
    return returnValue
  }

  boolean addAll(int index, Collection collection){
    def returnValue = list.addAll(index, collection)
    trimToSize()
    return returnValue
  }
} 
```

现在编写相应的单元测试，如清单 18 所示：

##### 清单 18\. 测试 `addAll()` 方法

```java
class FixedListTest extends GroovyTestCase{
  void testAddAll(){
    def quartet = ["John", "Paul", "George", "Ringo"]
    def trio = new FixedList(3)
    trio.addAll(quartet)
    assertEquals 3, trio.size()
    assertFalse trio.contains("Ringo")
  }

  void testAddAllWithIndex(){
    def quartet = new FixedList(4)
    quartet << "John"
    quartet << "Ringo"
    quartet.addAll(1, ["Paul", "George"])
    assertEquals "John", quartet[0]
    assertEquals "Paul", quartet[1]
    assertEquals "George", quartet[2]
    assertEquals "Ringo", quartet[3]
  }
} 
```

您现在完成了全部工作。感谢 `@Delegate` 注释的强大威力，我们只使用大约 50 代码就创建了 `FixedList` 类。感谢 `GroovyTestCase` 使我们能够测试代码，从而允许您将其放入到生产环境中，并且确信它可以按照期望的那样操作。清单 19 展示了完整的 `FixedList` 类：

##### 清单 19\. 完整的 `FixedList` 类

```java
class FixedList{
  @Delegate private List list = new ArrayList()
  final int sizeLimit

  /**
    * NOTE: This constructor limits the max size of the list,
    *  not just the initial capacity like an ArrayList.
    */
  FixedList(int sizeLimit){
    this.sizeLimit = sizeLimit
  }

  boolean add(Object element){
    if(list.size() < sizeLimit){
      return list.add(element)
    }else{
      throw new UnsupportedOperationException("Error adding ${element}:" +
        " the size of this FixedList is limited to ${sizeLimit}.")
    }
  }

  void add(int index, Object element){
    list.add(index, element)
    trimToSize()
  }

  private void trimToSize(){
    if(list.size() > sizeLimit){
      (sizeLimit..<list.size()).each{
        list.pop()
      }
    }
  }

  boolean addAll(Collection collection){
    def returnValue = list.addAll(collection)
    trimToSize()
    return returnValue
  }

  boolean addAll(int index, Collection collection){
    def returnValue = list.addAll(index, collection)
    trimToSize()
    return returnValue
  }

  String toString(){
    return "FixedList size: ${sizeLimit}\n" + "${list}"
  }
} 
```

* * *

## 结束语

通过将新的*行为* 添加到类中而不是转换其*类型*，Groovy 的元编程功能实现了一组全新的动态可能性，同时不会违背 Java 语言的静态类型系统的规则。通过使用 `ExpandoMetaClass`（让您能够通过执行映射将任何新方法添加到现有类）和 `@Delegate`（让您能够通过外部包装类公开复合内部类的功能），Groovy 让 JVM 焕发新光彩。

在下一期文章中，我将演示一个得益于 Groovy 的灵活语法 Swing 而重新焕发生机的旧有技术。是的，Swing 的复杂性因为 Groovy 的 `SwingBuilder` 而消失。这使得桌面开发变得更加有趣和简单。到那时，希望您能够发现大量有关 Groovy 的实际应用。

* * *

## 下载

| 描述 | 名字 | 大小 |
| --- | --- | --- |
| 本文示例的源代码 | [j-pg08259.zip](http://www.ibm.com/developerworks/apps/download/index.jsp?contentid=430289&filename=j-pg08259.zip&method=http&locale=zh_CN) | 5KB |