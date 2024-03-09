# 实战 Groovy: 使用闭包、ExpandoMetaClass 和类别进行元编程

*随心所欲添加方法*

进入到 Groovy 风格的元编程世界。在运行时向类动态添加方法的能力 — 甚至 Java™ 类以及 `final` Java 类 — 强大到令人难以置信。不管是用于生产代码、单元测试或介于两者之间的任何内容，即使是最缺乏热情的 Java 开发人员也会对 Groovy 的元编程能力产生兴趣。

人们一直以来都认为 Groovy 是一种面向 JVM 的*动态* 编程语言。在这期 [*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 文章中，您将了解*元编程*— Groovy 在运行时向类动态添加新方法的能力。它的灵活性远远超出了标准 Java 语言。通过一系列代码示例（都可以通过 下载 获得），将认识到元编程是 Groovy 的最强大、最实用的特性之一。

## 建模

程序员的工作就是使用软件建模真实的世界。对于真实世界中存在的简单域 — 比如具有鳞片或羽毛的动物通过产卵繁育后代，而具有毛皮的动物则通过产仔繁殖 — 可以很容易地使用软件对行为进行归纳，如清单 1 所示：

##### 清单 1\. 使用 Groovy 对动物进行建模

```java
class ScalyOrFeatheryAnimal{
  ScalyOrFeatheryAnimal layEgg(){
    return new ScalyOrFeatheryAnimal()
  }
}

class FurryAnimal{
  FurryAnimal giveBirth(){
    return new FurryAnimal()
  }
} 
```

## 关于本系列

Groovy 是在 Java 平台上运行的一种现代编程语言。它能够与现有 Java 代码无缝集成，同时引入了各种生动的新特性，比如闭包和元编程。简单来讲，Groovy 是 Java 语言的 21 世纪版本。

将任何新工具整合到开发工具包中的关键是知道何时使用它以及何时将它留在工具包中。Groovy 的功能可以非常强大，但惟一的条件是正确应用于适当的场景。因此，[*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 系列将探究 Groovy 的实际应用，以便帮助您了解何时以及如何成功使用它们。

不幸的是，真实的世界总是充满了例外和极端情况 — 鸭嘴兽既有皮毛，又通过产卵繁殖后代。我们精心考虑的每一项软件抽象几乎都存在与之相反的方面。

如果用来建模域的软件语言由于太过死板而无法处理不可避免的例外情况，那么最终的情形就像是受雇于一个小官僚机构的固执的公务员 — “对不起，Platypus 先生，如果要想我们的系统可以跟踪到您的话，您必须会生孩子。”

另一方面，Groovy 之类的动态语言为您提供了灵活性，使您能够更加准确地使用软件建模现实世界，而不是预先作出假设（并且通常是无效的），让现实向您妥协。如果 `Platypus` 类需要一个 `layEgg()` 方法，Groovy 可以满足要求，如清单 2 所示：

##### 清单 2\. 动态添加 `layEgg()` 方法

```java
Platypus.metaClass.layEgg = {->
  return new FurryAnimal()
}

def baby = new Platypus().layEgg() 
```

如果觉得这里举的有关动物的例子有些浅显，那么考虑 Java 语言中最常用的一个类：`String`。

* * *

## Groovy 为 `java.lang.String` 提供的新方法

使用 Groovy 的乐趣之一就在于它添加到 `java.lang.String` 中的新方法。`padRight()` 和 `reverse()` 等方法提供了简单的 `String` 转换，如清单 3 所示。（有关 GDK 添加到 `String` 的所有新方法的列表的链接，见 参考资料。正如 GDK 在其首页中所说，“本文档描述了添加到 JDK 并更具 groovy 特征的方法。”）

##### 清单 3\. Groovy 添加到 `String` 的方法

```java
println "Introduction".padRight(15, ".")
println "Introduction".reverse()

//output
Introduction...
noitcudortnI 
```

但是添加到 `String` 的方法并不仅限于简单的功能。如果 `String` 是一个组织良好的 URL，那么只需一行代码，您就可以将 `String` 转换为 `java.net.URL` 并返回 HTTP GET 请求的结果，如清单 4 所示：

##### 清单 4\. 发出 HTTP GET 请求

```java
println "http://thirstyhead.com".toURL().text

//output
<html>
  <head>
    <title>ThirstyHead: Training done right.</title>
<!-- snip --> 
```

再举一个例子，运行一个本地 shell 就像发出远程网络调用那么简单。一般情况下我将在命令提示中输入 `ifconfig en0` 以检查网卡的 TCP/IP 设置。（如果您使用的是 Windows® 而不是 Mac OS X 或 Linux®，那么尝试使用 `ipconfig`）。在 Groovy 中，我可以通过编程的方式完成同样的事情，参见清单 5：

##### 清单 5\. 在 Groovy 中发出一个 shell 命令

```java
println "ifconfig en0".execute().text

//output
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
    ether 00:17:f2:cb:bc:6b
    media: autoselect status: inactive
  //snip 
```

我并没有说 Groovy 的优点在于您*不能* 使用 Java 语言做同样的事情。您当然可以。Groovy 的优点在于这些方法似乎可以直接添加到 `String` 类 — 这绝非易事，因为 `String` 是 `final` 类。（稍后将详细讨论这点）。清单 6 展示了 Java 中的相应内容 `String.execute().text`：

##### 清单 6\. 使用 Java 语言发出 shell 命令

```java
Process p = new ProcessBuilder("ifconfig", "en0").start();
BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
String line = br.readLine();
while(line != null){
  System.out.println(line);
  line = br.readLine();
} 
```

这看上去有点像在机动车辆管理局的各个窗口之间辗转，不是吗？“对不起，先生，要查看您请求的 `String`，首先需要去别处获得一个 `BufferedReader`。”

是的，您可以构建方便的方法和实用类来帮助将这个问题抽象出来，但是惟一的 `com.mycompany.StringUtil` 替代方法就是使用一个类来代替将方法直接添加到所属位置的行为：`String` 类。（当然就是 `Platypus.layEgg()`！）

那么 Groovy 究竟如何做 — 将新方法添加到无法扩展的类，或直接进行修改？要理解这一点，需要了解 *closures* 和 `ExpandoMetaClass`。

* * *

## 闭包和 `ExpandoMetaClass`

Groovy 提供了一种无害的但功能强大的语言特性 — 闭包 — 如果没有它的话，鸭嘴兽将永远无法下蛋。简单来说，闭包就是指定的一段可执行代码。它是一个未包含在类中的方法。清单 7 演示了一个简单闭包：

##### 清单 7\. 一个简单闭包

```java
def shout = {src->
  return src.toUpperCase()
}

println shout("Hello World")

//output
HELLO WORLD 
```

拥有一个独立的方法当然很棒，但是与将方法放入到现有类的能力相比，还是有些逊色。考虑清单 8 中的代码，其中并未创建接受 `String` 作为参数的方法，相反，我将方法直接添加到 `String` 类：

##### 清单 8\. 将 `shout` 方法添加到 `String`

```java
String.metaClass.shout = {->
  return delegate.toUpperCase()
}

println "Hello MetaProgramming".shout()

//output
HELLO METAPROGRAMMING 
```

未包含任何参数的 `shout()` 闭包被添加到 `String` 的 `ExpandoMetaClass` (EMC) 中。每个类 — 包括 Java 和 Groovy — 都包含在一个 EMC 中，EMC 将拦截对它的方法调用。这意味着即使 `String` 为 `final`，仍然可以将方法添加到其 EMC 中。因此，现在看上去仿佛 `String` 有一个 `shout()` 方法。

由于 Java 语言中不存在这种关系，因此 Groovy 必须引入一个新的概念：*委托（delegate）*。`delegate` 是 EMC 所围绕的类。

首先了解到方法调用包含在 EMC 中，然后了解了 `delegate`，您就可以执行任何有趣的操作。比如，注意清单 9 然后重新定义 `String` 的 `toUpperCase()` 方法：

##### 清单 9\. 重新定义 `toUpperCase()` 方法

```java
String.metaClass.shout = {->
  return delegate.toUpperCase()
}

String.metaClass.toUpperCase = {->
  return delegate.toLowerCase()
}

println "Hello MetaProgramming".shout()

//output
hello metaprogramming 
```

这个操作看上去仍然有些不严谨（甚至有些危险！）。尽管现实中很少需要修改 `toUpperCase()` 方法的行为，但是想象一下为代码单元测试带来的好处？元编程提供了快速、简单的方法，使潜在的随机行为具有了必然性。比如，清单 10 演示了 `Math` 类的静态 `random()` 方法被重写：

##### 清单 10\. 重写 `Math.random()` 方法

```java
println "Before metaprogramming"
3.times{
  println Math.random()
}

Math.metaClass.static.random = {->
  return 0.5
}

println "After metaprogramming"
3.times{
  println Math.random()
}

//output
Before metaprogramming
0.3452
0.9412
0.2932
After metaprogramming
0.5
0.5
0.5 
```

现在，想像一下对发出开销较高的 SOAP 调用的类进行单元测试。无需创建接口和去掉整个模拟对象的存根 — 您可以有选择地重写方法并返回一个简单的模拟响应。（您将在下一小节看到使用 Groovy 实现单元测试和模拟的例子）。

Groovy 元编程是一种运行时行为 — 这个行为从程序启动一直持续到程序运行。但是如果希望对元编程进行更多的显示该怎么做（对于编写单元测试尤其重要）？在下一小节，您将了解揭秘元编程的秘密。

* * *

## 解密元编程

清单 11 封装了我在 `GroovyTestCase` 中编写的演示代码，这样就可以更加严格地对输出进行测试。（参见 “[实战 Groovy: 用 Groovy 更迅速地对 Java 代码进行单元测试](http://www.ibm.com/developerWorks/cn/java/j-pg11094/)” 了解更多有关使用 `GroovyTestCase` 的信息）。

##### 清单 11\. 使用单元测试分析元编程

```java
class MetaTest extends GroovyTestCase{

  void testExpandoMetaClass(){
    String message = "Hello"
    shouldFail(groovy.lang.MissingMethodException){
      message.shout()
    }

    String.metaClass.shout = {->
      delegate.toUpperCase()
    }

    assertEquals "HELLO", message.shout()

    String.metaClass = null
    shouldFail{
      message.shout()
    }
  }
} 
```

在命令提示中输入 `groovy MetaTest` 以运行该测试。

注意，只需将 `String.metaClass` 设置为 `null`，就可以取消元编程。

但是，如果您不希望 `shout()` 方法出现在所有 `String` 中该怎么办呢？您可以仅调整单一实例的 EMC（而不是类），如清单 12 所示：

##### 清单 12\. 对单个实例进行元编程

```java
void testInstance(){
  String message = "Hola"
  message.metaClass.shout = {->
    delegate.toUpperCase()
  }

  assertEquals "HOLA", message.shout()
  shouldFail{
    "Adios".shout()
  }
} 
```

如果准备一次性添加或重写多个方法，清单 13 展示了如何以块的方式定义新方法：

##### 清单 13\. 一次性对多个方法进行元编程

```java
void testFile(){
  File f = new File("nonexistent.file")
  f.metaClass{
    exists{-> true}
    getAbsolutePath{-> "/opt/some/dir/${delegate.name}"}
    isFile{-> true}
    getText{-> "This is the text of my file."}
  }

  assertTrue f.exists()
  assertTrue f.isFile()
  assertEquals "/opt/some/dir/nonexistent.file", f.absolutePath
  assertTrue f.text.startsWith("This is")
} 
```

注意，我再也不关心文件是否存在于文件系统中。我可以将它发送给这个单元测试中的其他类，并且它表现得像一个真正的文件。当 `f` 变量在测试结束时超出范围之后，还会执行定制行为。

尽管 `ExpandoMetaClass` 十分强大，但是 Groovy 提供了另一种元编程方法，使用了它独有的一组功能：*类别（category）*。

* * *

## 类别和 `use` 块

解释 `Category` 的最佳方法就是了解它的实际运行。清单 14 演示了使用 `Category` 来将 `shout()` 方法添加到 `String`：

##### 清单 14\. 使用一个 `Category` 进行元编程

```java
class MetaTest extends GroovyTestCase{
  void testCategory(){
    String message = "Hello"
    use(StringHelper){
      assertEquals "HELLO", message.shout()
      assertEquals "GOODBYE", "goodbye".shout()
    }

    shouldFail{
      message.shout()
      "foo".shout()
    }
  }
}

class StringHelper{
  static String shout(String self){
    return self.toUpperCase()
  }
} 
```

如果曾经从事过 Objective-C 开发，那么应当对这个技巧感到熟悉。`StringHelper``Category` 是一个普通类 — 它不需要扩展特定的父类或实现特殊的接口。要向类型为 `T` 的特定类添加新方法，只需定义一个静态方法，它接受类型 `T` 作为第一个参数。由于 `shout()` 是一个接受 `String` 作为第一个参数的静态方法，因此所有封装到 `use` 块中的 `String` 都获得了一个 `shout()` 方法。

那么，什么时候应该选择 `Category` 而不是 EMC？EMC 允许您将方法添加到某个类的单一实例或所有实例中。可以看到，定义 `Category` 允许您将方法添加到*特定* 实例中 — 只限于 `use` 块内部的实例。

虽然 EMC 允许您动态定义新行为，然而 `Category` 允许您将行为保存到独立的类文件中。这意味着您可以在不同的情况下使用它：单元测试、生产代码，等等。定义单独类的开销在重用性方面获得了回报。

清单 15 演示了对同一个 `use` 块同时使用 `StringHelper` 和新创建的 `FileHelper`：

##### 清单 15\. 在 `use` 块中使用多个类别

```java
class MetaTest extends GroovyTestCase{
  void testFileWithCategory(){
    File f = new File("iDoNotExist.txt")
    use(FileHelper, StringHelper){
      assertTrue f.exists()
      assertTrue f.isFile()
      assertEquals "/opt/some/dir/iDoNotExist.txt", f.absolutePath
      assertTrue f.text.startsWith("This is")

      assertTrue f.text.shout().startsWith("THIS IS")
    }

    assertFalse f.exists()
    shouldFail(java.io.FileNotFoundException){
      f.text
    }
  }
}

class StringHelper{
  static String shout(String self){
    return self.toUpperCase()
  }
}

class FileHelper{
 static boolean exists(File f){
   return true
 }

 static String getAbsolutePath(File f){
   return "/opt/some/dir/${f.name}"
 }

 static boolean isFile(File f){
   return true
 }

 static String getText(File f){
   return "This is the text of my file."
 }
} 
```

但是有关类别的最有趣的一点是它们的实现方式。EMC 需要使用闭包，这意味着您只能在 Groovy 中实现它们。由于类别仅仅是包含静态方法的类，因此可以用 Java 代码进行定义。事实上，可以在 Groovy 中重用现有的 Java 类 — 对元编程来说总是含义不明的类。

清单 16 演示了使用来自 Jakarta Commons Lang 包（见 参考资料）的类进行元编程。`org.apache.commons.lang.StringUtils` 中的所有方法都一致地遵守 `Category` 模式 — 静态方法接受 `String` 作为第一个参数。这意味着可以使用现成的 `StringUtils` 类作为 `Category`。

##### 清单 16\. 使用 Java 类进行元编程

```java
import org.apache.commons.lang.StringUtils

class CommonsTest extends GroovyTestCase{
  void testStringUtils(){
    def word = "Introduction"

    word.metaClass.whisper = {->
      delegate.toLowerCase()
    }

    use(StringUtils, StringHelper){
      //from org.apache.commons.lang.StringUtils
      assertEquals "Intro...", word.abbreviate(8)

      //from the StringHelper Category
      assertEquals "INTRODUCTION", word.shout()

      //from the word.metaClass
      assertEquals "introduction", word.whisper()
    }
  }
}

class StringHelper{
  static String shout(String self){
    return self.toUpperCase()
  }
} 
```

输入 `groovy -cp /jars/commons-lang-2.4.jar:. CommonsTest.groovy` 以运行测试（当然，您需要修改在系统中保存 JAR 的路径）。

* * *

## 元编程和 REST

为了不让您产生元编程*只对* 单元测试有用的误解，下面给出了最后一个例子。回忆一下 “[实战 Groovy：构建和解析 XML](http://www.ibm.com/developerworks/cn/java/j-pg05199/)” 中可以预报当天天气情况的 RESTful Yahoo! Web 服务。通过将上述文章中的 `XmlSlurper` 技巧与本文的元编程技巧结合起来，您就可以通过 10 行代码查看任何 ZIP 码所代表的位置的天气信息，如清单 17 所示：

##### 清单 17\. 添加一个 `weather` 方法

```java
String.metaClass.weather={->
  if(!delegate.isInteger()){
    return "The weather() method only works with zip codes like '90201'"
  }
  def addr = "http://weather.yahooapis.com/forecastrss?p=${delegate}"
  def rss = new XmlSlurper().parse(addr)
  def results = rss.channel.item.title
  results << "\n" + rss.channel.item.condition.@text
  results << "\nTemp: " + rss.channel.item.condition.@temp
}

println "80020".weather()

//output
Conditions for Broomfield, CO at 1:57 pm MDT
Mostly Cloudy
Temp: 72 
```

可以看到，元编程提供了极好的灵活性。您可以使用本文介绍的任何（或所有）技巧来向任意数量的类添加方法。

* * *

## 结束语

要求世界因语言的限制而改变显然不切实际。使用软件建模真实世界意味着需要有足够灵活的工具来处理所有极端情况。幸运的是，使用 Groovy 提供的闭包、`ExpandoMetaClasses` 和类别，您就拥有了一组出色的工具，可以根据自己的意愿添加行为。

在下一期文章中，我将重新审视 Groovy 在单元测试方面的强大功能。使用 Groovy 编写测试会带来实际的效益，不管是 `GroovyTestCase` 或包含注释的 JUnit 4.x 测试用例。您将看到 GMock 的实际作用 — 一种使用 Groovy 编写的模拟框架。到那时，希望您能够发现 Groovy 的许多实际应用。

* * *

## 下载

| 描述 | 名字 | 大小 |
| --- | --- | --- |
| 本文示例的源代码 | [j-pg06239.zip](http://www.ibm.com/developerworks/apps/download/index.jsp?contentid=413546&filename=j-pg06239.zip&method=http&locale=zh_CN) | 7KB |