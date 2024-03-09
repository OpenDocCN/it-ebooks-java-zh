# alt.lang.jre: 感受 Groovy

*介绍 Java 平台的一种新标准语言*

虽然 Java 语言因其严密性和扩展性的承诺而在整整一代程序员中胜出，但是 Groovy 预示了 Java 平台上的一个编程新时代，这种语言是以方便性、适宜性和敏捷性为出发点定义的。在新的 *alt.lang.jre*专栏的第二期文章中，Andrew Glover 对提议添加到 Java 平台的标准编程语言作了非正式的介绍。

如果您在使用 Java 平台（block），不管时间长短，您都有可能听说过 Groovy。Groovy 是超级明星开发人员 James Strachan 和 Bob McWhirter 发明的，它是一种敏捷开发语言，完全以 Java 编程 API 为基础。Groovy 当前正处于 Java Specification Request 的开始阶段，它于 2004 年 3 月底获得批准。Groovy 还是一种脚本语言，有些人说它会永久性地改变您看待和使用 Java 平台的方式。

在其对 JSR 241 （请参阅 参考资料）的开放评论中，Groovy 的共同规范领导者 Richard Monson-Haefel 说他对 Groovy 的支持是建立在总有一天 Java 平台要包括一种敏捷开发语言这一信念上。与许多移植到 Java 平台的脚本语言不同，Groovy 是 _ 为 _JRE 而写的。在规范请求中（参阅 参考资料），Groovy 的制造者提出了“Java 不仅是一种编程语言，更是一个健壮的平台，可以有多种语言在上面运行和共存”（Monson-Haefel 语）的思想。

新 *alt.lang.jre*专栏的这第二期文章的目的是让读者了解 Groovy。我首先回答关于这种新语言的一些最显然的问题（为什么需要它？），然后以代码为基础概述 Groovy 最令人兴奋的功能。

## 为什么需要另一种语言？

正如在 上个月的专栏中介绍的，Groovy 不是与 JRE 兼容的惟一脚本语言。Python、Ruby 和 Smalltalk 就是成功地移植到 Java 平台上的三种脚本语言。对于一些开发人员，这带来了问题：为什么要另一种语言？毕竟，我们许多人已经将 Java 代码与 Jython 或者 JRuby 结合来快速开发应用程序，为什么还要学习另一种语言？回答是 *您不一定要学习一种新语言以用 Groovy 编码*。Groovy 与其他 JRE 兼容脚本语言的不同在于它的语法以及重用 Java 库。Jython 和 JRuby 共享它们前身（分别是 Python 和 Ruby）的外观，Groovy 让人觉得就像是 Java 语言，不过限制要少得多。

## 关于本系列

虽然本系列的大多数读者熟悉 Java 语言以及它是如何在跨平台虚拟机上运行的，但是只有少数人知道 Java Runtime Environment 可以承载 Java 语言之外的语言。本系列文章对 JRE 的多种替代语言进行了综述。这里讨论的大多数语言是开放源代码的，许多是免费使用的，有少数是商业产品，必须购买。在 [*alt.lang.jre*](http://www.ibm.com/developerworks/views/java/articles.jsp?S_TACT=105AGX52&expand&sort_order=desc&search_by=alt.lang.jre&S_CMP=cn-a-j&show_abstract=true&sort_by=Date&view_by=Search)系列中介绍的所有语言都得到了 JRE 支持，并且作者相信它们增强了 Java 平台的动态性和灵活性特征。

像 Jython 这样的语言是在它们的父语言库上建立的，而 Groovy 使用了 Java 开发人员最熟悉的功能和库 *—— 但是将它们放到了一个敏捷开发框架中*。敏捷开发的基本宗旨是代码应该很好地适合范围广泛的任务，并可以不同的方式应用。Groovy 通过以下方式落实了这些宗旨：

*   使开发人员不用编译。
*   允许动态类型。
*   使合成结构容易。
*   使其脚本可以在普通 Java 应用程序中使用。
*   提供一个 shell 解析器。

这些特性使 Groovy 成为一种特别容易学习和使用的语言，不管您是有经验的 Java 开发人员还是刚接触 Java 平台。在下面几节中，我将详细讨论上述特性。

* * *

## 看呀，没有 javac！

像许多脚本语言一样，Groovy 不用为运行时编译。这意味着 Groovy 脚本是 *在它们运行时*解释的，就像 JavaScript 是在观看 Web 页时由浏览器解释的一样。运行时判断会有执行速度的代价，这有可能使脚本语言不能用于对性能有要求的项目，但是无编译的代码在构建-运行周期中可以提供很多好处。运行时编译使 Groovy 成为快速原型设计、构建不同的实用程序和测试框架的理想平台。

## 脚本的能力

脚本语言很流行，因为它们容易学习并且为开发人员设置的限制较少。脚本语言通常使用简单的、相当简洁的语法，这使开发人员可以用比大多数编程语言所需要的更少的代码创建真实世界的应用程序。像 Perl、Python、Ruby 和现在的 Groovy，用其敏捷方式编写代码而使编程工作达到一个新的效率水平。这种提高的敏捷性通常会使开发人员的效率提高。脚本语言的成功表明脚本不是一种小范围内使用的技术或者黑客的娱乐工具，而是一种由像 Google、Yahoo 和 IBM 这样的世界级公司所使用的切实可行的技术。

例如，运行脚本 Emailer.groovyin Groovy 就是在命令行键入 `groovy Emailer.groovy` 这么容易。如果希望运行同样的 Java 文件（Emailer.java），显然必须键入额外的命令： `javac Emailer.java` ，然后是 `java Emailer` 。虽然这看起来可能有些微不足道，但是可以容易设想运行时编译在应用程序开发的更大上下文中的好处。

可以在稍后看到，Groovy 还允许脚本放弃 main 方法以静态地运行一个关联的应用程序。

* * *

## 动态 dynamo

像其他主流脚本语言一样，Groovy 不需要像 C++ 和 Java 语言这样的正式语言的显式类型。在 Groovy 中，一个对象的类型是在运行时动态发现的，这极大地减少了要编写的代码数量。首先可以通过分析清单 1 和 2 中的简单例子看到这一点。

清单 1 显示了在 Java 语言中如何将一个本地变量声明为 `String` 。注意必须声明类型、名和值。

##### 清单 1\. Java 静态类型

```java
String myStr = "Hello World"; 
```

在清单 2 中，您看到同样的声明，但是不需要声明变量类型。

##### 清单 2\. Groovy 动态类型

```java
myStr = "Hello World" 
```

您可能还注意到了，在清单 2 中我可以去掉声明中的分号。在定义方法及其相关的参数时动态类型有戏剧性的后果：多态具有了全新的意义！事实上，使用动态类型， *不使用*继承就可以得到多态的全部能力。在清单 3 中，可以真正开始看到动态类型在 Groovy 的灵活性方面所起的作用。

##### 清单 3\. 更多 Groovy 动态类型

```java
class Song{
  length
  name
}
class Book{
  name
  author
}
def doSomething(thing){
  println "going to do something with a thing named = " + thing.name
} 
```

这里，我定义了两个 Groovy 类， `Song` 和 `Book` ，我将在后面对它们进一步讨论。这两个类都包含一个 `name` 属性。我还定义了一个函数 `doSomething` ，它以一个 `thing` 为参数，并试图打印这个对象的 `name` 属性。

因为 `doSomething` 函数没有定义其输入参数的类型，只要对象包含一个 `name` 属性，那么它就可以工作。因此，在清单 4 中，可以看到在使用 `Song` 和 `Book` 的实例作为 `doSomething` 的输入时会有什么现象。

##### 清单 4\. 试验动态类型

```java
mySong = new Song(length:90, name:"Burning Down the House")
myBook = new Book(name:"One Duck Stuck", author:"Phyllis Root")
doSomething(mySong) //prints Burning Down the House
doSomething(myBook) //prints One Duck Stuck
anotherSomething = doSomething
anotherSomething(myBook) //prints One Duck Stuck 
```

除了展示 Groovy 中的动态类型，清单 4 的最后两行还揭示了创建对一个函数的引用有多容易。这是因为在 Groovy 中 *所有东西*都是对象，包括函数。

关于 Groovy 的动态类型声明最后要注意的是，它会导致更少的 `import` 语句。尽管 Groovy 需要 import 以显式使用类型，但是这些 import 可以使用别名以提供更短的名字。

### 动态类型综述

下面两个例子将到目前为止讨论过的 Groovy 中的动态类型的内容放到一起。下面的 Java 代码组和 Groovy 代码组利用了 Freemarker（参阅 参考资料），这是一个开放源代码模板引擎。这两组代码都只是简单地用一个目录和文件名创建一个 `Template` 对象，然后将相应对象的内容打印到标准输出，当然，不同之处是每一组代码处理这项任务所需要的代码量。

##### 清单 5\. 简单的 TemplateReader Java 类

```java
import java.io.File;
import java.io.IOException;
import freemarker.template.Configuration;
import freemarker.template.Template;
public class TemplateReader {
  public static void main(String[] args) {
    try{
      Configuration cfg = Configuration.getDefaultConfiguration();
      cfg.setDirectoryForTemplateLoading(
           new File("C:\\dev\\projects\\http-tester\\src\\conf"));

      Template temp = cfg.getTemplate("vendor-request.tmpl");

        System.out.println(temp.toString());
      }catch(IOException e){
        e.printStackTrace();
      }
  }
} 
```

初看之下，清单 5 中的 Java 代码相当简单 —— 特别是如果以前从来没见过脚本代码时。幸运的是，有清单 6 中的 Groovy 作为对比。现在这段代码很简单！

##### 清单 6\. 用 Groovy 编写的更简单的 TemplateReader

```java
import freemarker.template.Configuration as tconf
import java.io.File
cfg = tconf.getDefaultConfiguration()
cfg.setDirectoryForTemplateLoading(
  new File("C:\\dev\\projects\\http-tester\\src\\conf"))

temp = cfg.getTemplate("vendor-request.tmpl")
println temp.toString() 
```

Groovy 代码只有 Java 代码的一半那么长，下面是原因：

*   Groovy 代码只需要一半的 `import` 语句。还要注意， `freemarker.template.Configuration` 使用了别名 `tconf` ，使得语法更短。
*   Groovy 允许类型为 `Template` 的变量 `tmpl` 不声明其类型。
*   Groovy 不需要 `class` 声明或者 `main` 方法。
*   Groovy 不关心任何相应异常，使您可以不用导入 Java 代码中需要的 `IOException` 。

现在，在继续之前，想一下您所编写的最后一个 Java 类。您可能不得不编写很多 import 并声明类型，并在后面加上同样数量的分号。考虑用 Groovy 编写同样的代码会是什么情况。可以使用简练得多的语法，不需要遵守这么多的规则，并且得到完全相同的行为。

想一下，如果您正好是刚刚开始……

* * *

## 特别灵活的语法

谈到语法，灵活性是更有效地开发代码的主要因素。很像其有影响的对手（Python、Ruby 和 Smalltalk），Groovy 极大地简化了核心库的使用和它所模拟的语言（在这里是 Java 语言）的构造。为了让您对 Groovy 语法的灵活性有一个大体概念，我将展示它的一些主要结构，即类、函数（通过 `def` 关键词）、闭包、集合、范围、映射和迭代器。

### 类

在字节码水平，Groovy 类是真正的 Java 类。不同之处在于 Groovy 将类中定义的所有内容都默认为 `public` ，除非定义了特定的访问修饰符。而且，动态类型应用到字段和方法，不需要 `return` 语句。

在清单 7 中可以看到 Groovy 中类定义的例子，其中类 `Dog` 有一个 `getFullName` 方法，它实际上返回一个表示 `Dog` 的全名的 `String` 。并且所有方法都隐式地为 `public` 。

##### 清单 7\. 示例 Groovy 类：Dog

```java
class Dog{
  name
  bark(){
    println "RUFF! RUFF!"
  }

  getFullName(master){
    name + " " + master.lname
  }

  obeyMaster(){
    println "I hear you and will not obey."
  }
} 
```

在清单 8 中，推广到有两个属性 —— `fname` 和 `lname` —— 的类 `DogOwner` ，就是这么简单！

##### 清单 8\. 示例 Groovy 类：DogOwner

```java
class DogOwner{
  fname
  lname
  trainPet(pet){
    pet.obeyMaster()
  }

} 
```

在清单 9 中，用 Groovy 设置属性并对 `Dog` 和 `DogOwner` 实例调用方法。现在很明显，使用 Groovy 类比 Java 类要容易得多。虽然需要 `new` 关键词，但是类型是可选的，且设置属性（它隐式为 public）是相当轻松的。

##### 清单 9\. 使用 Groovy 类

```java
myDog = new Dog()
myDog.name = "Mollie"
myDog.bark()
myDog.obeyMaster() 
me = new DogOwner()
me.fname = "Ralf"
me.lname = "Waldo"
me.trainPet(myDog)
str = myDog.getFullName(me)
println str  // prints Mollie Waldo 
```

注意在 `Dog` 类中定义的 `getFullName` 方法返回一个 `String` 对象，在这里它是 “ `Mollie Waldo` ”。

## 第一类对象

*第一类对象* 是可以在运行时用数据创建并使用的对象。第一类对象还可以传递给函数和由函数输出、作为变量存储、由其他对象返回。Java 语言自带的基本数据类型，如 `int` 和 `boolean` ，不认为是第一类对象。许多面向对象纯粹论者哀叹这个小细节，一些人据此提出 Java 语言是否是真正的面向对象语言。Groovy 通过将所有内容声明为对象而解决了这一问题。

### Def

除了像许多脚本语言那样将所有对象指定为第一类对象（见侧栏），Groovy 还让您创建 *第一类函数*，它本身实质上就是对象。它们是用 `def` 关键词定义的并在类定义之外。您实际上在 清单 3 中已经看到了如何用 `def` 关键词定义第一类函数，并在 清单 4 中看到使用了一个函数。Groovy 的第一类函数定义简单脚本时特别有用。

### 闭包

Groovy 中最令人兴奋和最强大的功能是支持闭包。 *闭包（Closure）*是第一类对象，它类似于 Java 语言中的匿名内部类。闭包和匿名内部类都是可执行的一段代码，不过这两者之间有一些细微的不同。状态是自动传入传出闭包的。闭包可以有名字。它们可以重复使用。而且，最重要且对 Groovy 同样成立的是，闭包远比匿名内部类要灵活得多！

清单 10 展示了闭包的强大。清单中新的和改进的 `Dog` 类包括一个 `train` 方法，它实际上执行创建了 `Dog` 实例的闭包。

##### 清单 10\. 使用闭包

```java
class Dog{  
  action
  train(){
    action.call()
  }
}
sit = { println "Sit, Sit! Sit! Good dog"}
down = { println "Down! DOWN!" }
myDog = new Dog(action:sit)
myDog.train()  // prints Sit, Sit! Sit! Good dog
mollie = new Dog(action:down)
mollie.train() // prints Down! DOWN! 
```

而且，闭包还可以接收参数。如清单 11 所示， `postRequest` 闭包接收两个参数（ `location` 和 `xml` ），并使用 Jakarta Commons HttpClient 库（参阅 参考资料）将一个 XML 文档发送给指定位置。然后这个闭包返回一个表示响应的 `String` 。闭包定义下面是一个使用闭包的例子。事实上，调用闭包就像调用函数一样。

##### 清单 11\. 使用带参数的闭包

```java
import org.apache.commons.httpclient.HttpClient
import org.apache.commons.httpclient.methods.PostMethod
postRequest = { location, xml |
  clint = new HttpClient()
  mthd = new PostMethod(location)      
  mthd.setRequestBody(xml)
  mthd.setRequestContentLength(xml.length())
  mthd.setRequestHeader("Content-type", 
     "text/xml; charset=ISO-8859-1")

  statusCode = clint.executeMethod(mthd)
  responseBody = mthd.getResponseBody()
  mthd.releaseConnection()    
  return new String(responseBody)      
}
loc = "http://localhost:8080/simulator/AcceptServlet/"
vxml = "<test><data>blah blah blah</data></test>"
str = postRequest(loc, vxml)
println str 
```

## 自动装箱

自动装箱或者装箱转换是一个自动将像 `int` 、 `double` 和 `boolean` 这样的基本数据类型自动转换为可以在 `java.lang` 包中找到的它们的相应包装类型的过程。这一功能出现在 J2SE 1.5 中，使开发人员不必在源代码中手工编写转换代码。

### 集合

将对象组织到像列表和映射这样的数据结构中是一项基本的编码任务，是我们大多数人每天要做的工作。像大多数语言一样，Groovy 定义了一个丰富的库以管理这些类型的集合。如果曾经涉足 Python 或者 Ruby，那么应该熟悉 Groovy 的集合语法。如清单 12 所示，创建一个列表与在 Java 语言中创建一个数组很类似。（注意，列表的第二项自动装箱为一个 `Integer` 类型。)

##### 清单 12\. 使用集合

```java
collect = ['groovy', 29, 'here', 'groovy'] 
```

除了使列表更容易处理，Groovy 还为集合增加了几个新方法。这些方法使得，如 `统计` 值出现的次数、将整个列表 `结合` 到一起、对列表 `排序` 变得非常容易。可以在清单 13 中看到这些集合方法的使用。

##### 清单 13\. 使用 Groovy 集合

```java
aCollect = [5, 9, 2, 2, 4, 5, 6] 
println aCollect.join(' - ')  // prints 5 - 9 - 2 - 2 - 4 - 5 - 6
println aCollect.count(2)     // prints 2
println aCollect.sort()       // prints [2, 2, 4, 5, 5, 6, 9] 
```

**Maps**

像列表一样，映射也是一种在 Groovy 中非常容易处理的数据结构。清单 14 中的映射包含两个对象，键是 `name` 和 `date` 。注意可以用不同的方式取得值。

##### 清单 14\. 处理映射

```java
myMap = ["name" : "Groovy", "date" : new Date()]
println myMap["date"]
println myMap.date 
```

**范围**

在处理集合时，很可能会大量使用 `范围（Range）` 。 `范围` 实际上是一个很直观的概念，并且容易理解，利用它可以包含地或者排除地创建一组有序值。使用两个点 （ `..` ） 声明一个包含范围，用三个点 （ `...` ） 声明一个排除范围，如清单 15 所示。

##### 清单 15\. 处理范围

```java
myRange = 29...32
myInclusiveRange = 2..5
println myRange.size() // prints 3
println myRange[0]   // prints 29
println myRange.contains(32) //prints false
println myInclusiveRange.contains(5) //prints true 
```

**用范围实现循环**

在循环结构中，范围可以实现相当巧妙的想法。在清单 16 中，将 `aRange` 定义为一个排除范围，循环打印 a、b、c 和 d。

##### 清单 16\. 用范围实现循环

```java
aRange = 'a'...'e'
for (i in aRange){
  println i
} 
```

**集合的其他功能**

如果不熟悉 Python 和其他脚本语言，那么您在 Groovy 集合中发现的一些其他功能会让您印象深刻。例如，创建了集合后，可以用负数在列表中反向计数，如清单 17 所示。

##### 清单 17\. 负索引

```java
aList = ['python', 'ruby', 'groovy']
println aList[-1] // prints groovy
println aList[-3] // prints python 
```

Groovy 还让您可以用范围分割列表。分割可获得列表的准确子集，如清单 18 所示。

##### 清单 18\. 用范围分割

```java
fullName = "Andrew James Glover"
mName = fullName[7...13]
print "middle name: " + mName // prints James 
```

**集合类似于 Ruby**

如果愿意的话，还可以将 Groovy 集合作为 Ruby 集合。可以用类似 Ruby 的语法，以 `&lt;&lt;` 语法附加元素、用 `+` 串接和用 `-` 对集合取差，甚至还可以用 `*` 语法处理集合的重复，如清单 19 所示。 注意，还可以用 `==` 比较集合。

##### 清单 19\. Ruby 风格的集合

```java
collec = [1, 2, 3, 4, 5]
collec << 6 //appended 6 to collec
acol = ['a','b','c'] * 3 //acol now has 9 elements
coll =  [10, 11]
coll2 = [12, 13]
coll3 = coll + coll2 //10,11,12,13
difCol = [1,2,3] - [1,2] //difCol is 3
assert [1, 2, 3] == [1, 2, 3] //true 
```

### 迭代器

在 Groovy 中，迭代任何序列都相当容易。迭代字符序列所需要的就是一个简单的 `for` 循环，如清单 20 所示。（正如您现在可能注意到的，Groovy 提供了比 Java 1.5 以前的传统语法更自然的 `for` 循环语法。）

##### 清单 20\. 迭代器示例

```java
str = "uncle man, uncle man"
for (ch in str){
  println ch
} 
```

Groovy 中的大多数对象具有像 `each` 和 `find` 这样的以闭包为参数的方法。用闭包来迭代对象会产生几种令人兴奋的可能性，如清单 21 所示。

##### 清单 21\. 带有迭代器的闭包

```java
[1, 2, 3].each {  
  val = it 
  val += val
  println val
}
[2, 4, 6, 8, 3].find { x |
     if (x == 3){
       println x
     }
} 
```

在清单 21 中，方法 `each` 作为迭代器。在这里，闭包添加元素的值，完成时 `val` 的值为 6。 `find` 方法也是相当简单的。每一次迭代传递进元素。在这里，只是测试值是否为 3。

* * *

## Groovy 的高级内容

到目前为止，我着重讲述的都是使用 Groovy 的基本方面，但是这种语言有比基本内容多得多的内容！我将以分析 Groovy 提供的一些高级开发功能作为结束，包括 Groovy 样式的 JavaBeans 组件、文件 IO、正则表达式和用 `groovyc` 编译。

### GroovyBean！

永远不变的是，应用程序最后要使用类似 struct 的对象表示真实世界的实体。在 Java 平台上，称这些对象为 JavaBean 组件，它们通常用于表示订单、客户、资源等的业务对象。Groovy 由于其方便的简写语法，以及在定义了所需 bean 的属性后自动提供构造函数，而简化了 JavaBean 组件的编写。结果自然就是极大地减少了代码，正如您可以自己从清单 22 和 23 中看到的。

在清单 22 中，可看到一个简单的 JavaBean 组件，它是用 Java 语言定义的。

##### 清单 22\. 一个简单的 JavaBean 组件

```java
public class LavaLamp {
  private Long model;
  private String baseColor;
  private String liquidColor;
  private String lavaColor;
  public String getBaseColor() {
    return baseColor;
  }
  public void setBaseColor(String baseColor) {
    this.baseColor = baseColor;
  }
  public String getLavaColor() {
    return lavaColor;
  }
  public void setLavaColor(String lavaColor) {
    this.lavaColor = lavaColor;
  }
  public String getLiquidColor() {
    return liquidColor;
  }
  public void setLiquidColor(String liquidColor) {
    this.liquidColor = liquidColor;
  }
  public Long getModel() {
    return model;
  }
  public void setModel(Long model) {
    this.model = model;
  }
} 
```

在清单 23 中，可以看到用 Groovy 写这个 bean 时所发生的事。所要做的就是定义属性，Groovy 会自动给您一个很好的构造函数以供使用。Groovy 还使您在操纵 `LavaLamp` 的实例时有相当大的灵活性。例如，我们可以使用 Groovy 的简写语法 *或者*传统的冗长的 Java 语言语法操纵 bean 的属性。

##### 清单 23\. 用 Groovy 编写的 JavaBeans 组件

```java
class LavaLamp{
  model
  baseColor
  liquidColor
  lavaColor
}
llamp = new LavaLamp(model:1341, baseColor:"Black", 
  liquidColor:"Clear", lavaColor:"Green")
println llamp.baseColor
println "Lava Lamp model ${llamp.model}"
myLamp = new LavaLamp()
myLamp.baseColor = "Silver"
myLamp.setLavaColor("Red")
println "My Lamp has a ${myLamp.baseColor} base"
println "My Lava is " + myLamp.getLavaColor() 
```

### 轻松的 IO

Groovy IO 操作很轻松，特别是与迭代器和闭包结合时。Groovy 使用标准 Java 对象如 `File` 、 `Reader` 和 `Writer` ，并用接收闭包作参数的额外方法增强了它们。如在清单 24 中，可以看到传统的 `java.io.File` ，但是带有额外的、方便的 `eachLine` 方法。

##### 清单 24\. Groovy IO

```java
import java.io.File
new File("File-IO-Example.txt").eachLine{ line |
 println "read the following line -> " + line
} 
```

因为文件实质上是一系列行、字符等，所以可以相当简单地迭代它们。 `eachLine` 方法接收一个闭包并迭代文件的每一行，在这里是 `File-IO-Example.txt` 。 以这种方式使用闭包是相当强大的，因为 Groovy 保证所有文件资源都是关闭的，不考虑任何异常。这意味着无需大量 `try` / `catch` / `finally` 子句就可以进行文件 IO！

### 高级编译

Groovy 脚本实际上是字节码级别的 Java 类。因此，可以容易地用 `groovyc` 编译 Groovy 脚本。可以通过命令行或者 `Ant` 使用 `groovyc` 以生成脚本的类文件。这些类可以用普通 `java` 命令运行，只要 classpath 包括 `groovy.jar` 和 `asm.jar` ，这是 ObjectWeb 的字节码操纵框架。要了解更多编译 Groovy 的内容，请参阅 参考资料。

### 最大 RegEx

如果一种语言没有正则表达式处理，则它是没价值的。Groovy 使用 Java 平台的 `java.util.regex` 库 —— 但是做了少量基本的改变。例如，Groovy 使您可以用 `~` 表达式创建 `Pattern` 对象，用 `=~` 表达式创建 `Matcher` 对象，如清单 25 所示。

##### 清单 25\. Groovy RegEx

```java
str =  "Water, water, every where,
        And all the boards did shrink;
        Water, water, every where,
        Nor any drop to drink."
if (str =~ 'water'){
  println 'found a match'
}
ptrn = ~"every where"
nStr = (str =~ 'every where').replaceAll('nowhere')
println nStr 
```

您可能已经注意到了，可以在上述清单中定义 `String` 、 `str` ，而无需为每一新行添加结束引号和 `+` 。这是因为 Groovy 放松了要求字符串串接的普通 Java 约束。运行这段 Groovy 脚本会对匹配 `water` 的情况打印出 `true` ，然后打印出一节诗，其中所有出现 “ `every where` ”的地方都替换为 “ `nowhere` ”。

## 关于 （band）shell

Groovy 提供了两种不同的解释器，使所有有效的 Groovy 表达式可以交互地执行。这些 shell 是特别强大的机制，可以用它们迅速学习 Groovy。

* * *

## 结束语

像所有婴儿期的项目一样，Groovy 是正在发展的语言。习惯于使用 Ruby 和 Python （或者 Jython）的开发人员可能会怀念 mixins、脚本导入（尽管可以将所需要的可导入脚本编译为相应的 Java 类）和方法调用的命名参数等这些功能的方便性。 但是 Groovy 绝对是一种发展中的语言。随着其开发人员数量的增加，它很有可能结合这些功能及更多功能。

同时，Groovy 有很多优点。它很好地融合了 Ruby、Python 和 Smalltalk 的一些最有用的功能，同时保留了基于 Java 语言的核心语法。对于熟悉 Java 平台的开发人员，Groovy 提供了更简单的替代语言，且几乎不需要学习时间。对于刚接触 Java 平台的开发人员，它可以作为有更复杂语法和要求的 Java 语言的一个容易的入口点。

像在本系统讨论的其他语言一样，Groovy 不是要替代 Java 语言，而是作为它的另一种选择。与这里讨论的其他语言不一样，Groovy 遵循 Java 规范，这意味着它有可能与 Java 平台上的 Java 语言具有同等重要的作用。

在本月这一期 *alt.lang.jre*文章中，介绍了 Groovy 的基本框架和语法，以及它的一些高级编程功能。下个月将介绍在 Java 开发人员中最受欢迎的脚本语言： JRuby。