# 实战 Groovy: Groovy：Java 程序员的 DSL

*用 Groovy 编写更少的代码，完成更多的工作*

Groovy 专家 Scott Davis 将重新开始撰写 [*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 系列文章，该系列文章于 2006 年停止编写。作为开篇文章，本文将介绍 Groovy 最近的发展以及 Groovy 当前的状态。然后了解*大约* 从 2009 年开始，使用 Groovy 是多么轻松。

Andrew Glover 于 2004 年开始为 developerWorks 撰写关于 Groovy 的文章，他先撰写了 [*alt.lang.jre*](http://www.ibm.com/developerworks/cn/views/java/libraryview.jsp?search_by=alt.lang.jre) 系列中的介绍性文章 “[alt.lang.jre: 感受 Groovy](http://www.ibm.com/developerWorks/cn/java/j-alj08034/)”，又继续撰写了长期刊发的 [*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 系列。发表这些文章时市场上还没有出现关于 Groovy 的书籍（现在这样的书籍超过十几本），而且 Groovy 1.0 在几年后才于 2007 年 1 月发布。自 2006 年末发布 *实战 Groovy* 的最后一期后，Groovy 发生了很大的变化。

现在，Groovy 每个月的平均下载数量大约为 35,000。Mutual of Omaha 等保守的公司拥有超过 70,000 行 Groovy 生产代码。Groovy 在 Codehaus.org 中有一个最忙碌的邮件列表，这是托管该项目的位置（请参阅 参考资料）。Grails 是惟一一个拥有较多下载数量以及繁忙邮件列表的项目，它是在 Groovy 中实现的流行 Web 框架（请参阅 参考资料）。

在 JVM 中运行非 Java™ 语言不仅常见，而且也是 Sun 的 JVM 策略的核心部分。Groovy 加入进了 Sun 支持的备选语言（如 JavaScript, JavaFX, JRuby 和 Jython）行列中。2004 年所做的实验现在成为了最前沿的技术。

2009 年撰写的关于 Groovy 的文章在许多方面与 Andy 开始撰写的文章相同。2005 年确立的语法仍然保留至今。每个发行版都添加了引人注目的新功能，但是对于项目主管来说，保留向后兼容性是极为重要的。这项可靠的基础使得 Java 开发组织在其应用程序进入生产环境并开始依赖各项技术时，毫不犹豫地选择了 Groovy。

本文的目标是使经验丰富的 Java 开发人员可以像 Groovy 开发人员一样进行快速开发。不要被它的表面所蒙骗。本系列如名称所示全部都是实际使用的 Groovy 实践知识。在最开始编写完 “Hello, World” 之后，请准备好尽管掌握实际的应用。

## 关于本系列

Groovy 是运行在 Java 平台上的现代编程语言。它将提供与现有 Java 代码的无缝集成，同时引入闭包和元编程等出色的新功能。简言之，Groovy 是 21 世纪根据 Java 语言的需要编写的。

把任意一个新工具集成到开发工具包中的关键是：知道何时使用以及何时不应使用该工具。Groovy 可以提供强大的功能，但是必须正确地应用到适当的场景中。为此，[*实战 Groovy*](http://www.ibm.com/developerworks/cn/java/j-pg/) 系列将探究 Groovy 的实际应用，帮助您了解何时及如何成功应用它们。

## 安装 Groovy

如果您以前从未使用过 Groovy，则首先需要安装 Groovy。安装步骤非常简单，这些步骤与安装 Ant 和 Tomcat 等常见 Java 应用程序甚至安装 Java 平台本身的步骤相同：

1.  [下载](http://groovy.codehaus.org/Download) 最新的 Groovy ZIP 文件或 tarball。
2.  将存档解压缩到所选目录中（您应当避免在目录名称中使用空格）。
3.  创建 `GROOVY_HOME` 环境变量。
4.  把 GROOVY_HOME/bin 添加到 `PATH` 中。

Groovy 运行在 Java 5 或 6 上的效果最佳。在命令提示中输入 `java -version` 以确认您使用的是最新版本。然后键入 `groovy -version` 以确保 Groovy 已正确安装。

所有主要 IDE（Eclipse、IntelliJ 和 NetBeans）都有支持自动完成和分步调试等功能的 Groovy 插件。虽然拥有一个优秀的 IDE 对于目前编写 Java 代码来说几乎成为了一项必要要求，但是对于 Groovy 来说并非绝对。得益于 Groovy 语言的简明性，许多人都选择使用简单的文本编辑器编写。vi 和 Emacs 等常见开源编辑器都提供 Groovy 支持，Textpad（面向 Windows®）和 TextMate（面向 Mac OS X）等便宜的商业文本编辑器也提供 Groovy 支持（有关更多信息，请参阅 参考资料）。

您稍后将在本文中看到，将 Groovy 与现有 Java 项目集成起来十分简单。您只需将一个 Groovy JAR 从 GROOVY_HOME/embeddable 添加到类路径中并把现有的 `javac` Ant 任务封装到 `groovyc` 任务中（Maven 提供类似的支持）。

但是在掌握这些知识之前，我将从必修的 “Hello World” 示例开始。

* * *

## 进入 Groovy 世界

您知道 “Hello World” 示例应当会演示哪些内容 — 它是用给定语言可以编写的最简单的程序。清单 1 中所示的 “Hello World” Java 代码的有趣之处在于，需要了解中间语言的知识才能完全了解代码含义：

##### 清单 1\. 用 Java 代码编写的 “Hello World” 示例

```java
public class HelloJavaWorld{
  public static void main(String[] args){
    System.out.println("Hello Java World");
  }
} 
```

首先创建名为 HelloJavaWorld.java 的文件并输入 `public class HelloJavaWorld`。许多刚开始使用 Java 的开发人员学到的第一课是如果类名与文件名不完全匹配（包括大小写），则类无法编译。另外，好奇的学生将在此时开始询问关于 `public` 和 `private` 之类的访问修饰符。

下一行 — `public static void main(String[] args)`— 通常将引发关于实现细节的一连串问题：什么是 `static`？什么是 `void`？为什么需要将方法命名为 `main`？什么是 `String` 数组？而最后，尝试向刚开始使用 Java 的开发人员说明 `out` 是 `System` 类中的 `PrintStream` 对象的 `public`、`static`、`final` 实例。我永远也忘不了学生说 “天哪！其实我只是想输出 ‘Hello’”。

将此示例与用 Groovy 编写的 “Hello World” 进行对照。创建名为 HelloGroovyWorld.groovy 的文件并输入清单 2 中所示的代码行：

##### 清单 2\. 用 Groovy 代码编写的 “Hello World” 示例

```java
println "Hello Groovy World" 
```

是的，这段代码是与清单 1 中所示的 Java 示例等效的 Groovy 代码。在本例中，所有实现细节 — 并不立即解决手头问题的 “知识” — 都隐藏在后台，只显示简单输出 “Hello” 的代码。输入 `groovy HelloGroovyWorld` 以确认它可以工作。

这个小示例将演示 Groovy 的双重价值：它将显著地减少需要编写的代码行数，同时保留 Java 等效代码的语义。在下一节中，您将进一步探究这种理念。

* * *

## 深入研究 Hello World

有经验的 Java 开发人员都知道在 JVM 中运行代码之前必须先编译这些代码。但是，Groovy 脚本在任何位置都不显示为类文件。这是否意味着可以直接执行 Groovy 源代码？答案是 “不一定，但是它看上去是这样，对不对？”

Groovy 解释器将先在内存中编译源代码，然后再将其转到 JVM 中。您可以通过输入 `groovyc HelloGroovyWorld.groovy` 手动执行此步骤。但是，如果尝试使用 `java` 运行得到的类，将显示清单 3 中所示的异常：

##### 清单 3\. 尝试在 `CLASSPATH` 中没有 Groovy JAR 的情况下运行经过编译的 Groovy 类

```java
$ java HelloGroovyWorld
Exception in thread "main" java.lang.NoClassDefFoundError: groovy/lang/Script 
```

如前述，Groovy JAR 必须包含在 `CLASSPATH` 中。再次尝试执行代码，这一次把 `-classpath` 实参传递到 `java` 中，如清单 4 所示：

##### 清单 4\. 用 `java` 命令成功运行经过编译的 Groovy 类

```java
//For UNIX, Linux, and Mac OS X
$ java -classpath $GROOVY_HOME/embeddable/groovy-all-x.y.z.jar:. HelloGroovyWorld
Hello Groovy World

//For Windows
$ java -classpath %GROOVY_HOME%/embeddable/groovy-all-x.y.z.jar;. HelloGroovyWorld
Hello Groovy World 
```

现在您已经取得了一些进展。但是为了证明 Groovy 脚本真正地保留 Java 示例的语义，您需要深入钻研字节码。首先，输入 `javap HelloJavaWorld`，如清单 5 所示：

##### 清单 5\. 解释 Java 字节码

```java
$ javap HelloJavaWorld
Compiled from "HelloJavaWorld.java"
public class HelloJavaWorld extends java.lang.Object{
  public HelloJavaWorld();
  public static void main(java.lang.String[]);
} 
```

除了为您添加了 `javac` 编译器之外，这段代码中不应当有过多令人惊讶之处。您无需显式输入 `extends java.lang.Object` 或提供类的默认构造函数。

现在，输入 `javap HelloGroovyWorld`，如清单 6 所示：

##### 清单 6\. 解释 Groovy 字节码

```java
$ javap HelloGroovyWorld
Compiled from "HelloGroovyWorld.groovy"
public class HelloGroovyWorld extends groovy.lang.Script{
  ...
  public static void main(java.lang.String[]);
  ...
} 
```

## 什么是 DSL？

Martin Fowler 普及了特定于领域语言的理念（请参阅 参考资料）。他把 DSL 定义为 “侧重特定领域的表达有限的计算机编程语言”。“有限的表达” 并不是指语言的用途有限，只是表示这种语言提供了足够用于适当表达 “特定领域” 的词汇表。DSL 是一种很小的专用语言，这与 Java 语言等大型通用语言形成对比。

SQL 就是一种优秀的 DSL。您无法使用 SQL 编写操作系统，但它是处理关系数据库这一有限领域的理想选择。在同样意义上，Groovy 是 Java 平台的 DSL，因为它是有限领域的 Java 开发的理想选择。我在这里使用 *DSL* 是为了启发读者，并不是特别的精确。如果我把 Groovy 称为 *常用 Java 语言的内部 DSL*，可能更容易被 DSL 纯粹主义者接受。

Dave Thomas 进一步阐明 DSL 的概念（请参阅 参考资料）。他写道，“无论领域专家在何时交流……他们都在说行业术语，这是他们为与同行进行有效交流而创造出的更简略的专用语言”。可能将 Groovy 视为 “简略的 Java 语言” 更能说明 Groovy 与 Java 语言之间的关系。本文的下一节将提供另一个示例。

在这里，您可以看到 `groovyc` 编译器将获取源文件的名称并创建了一个同名的类（该类扩展 `groovy.lang.Script` 而非 `java.lang.Object`，这应当能帮助您理解尝试在 `CLASSPATH` 中没有 Groovy JAR 的情况下运行文件抛出 `NoClassDefFoundError` 异常的原因）。在所有其他编译器提供的方法之中，您应当能够找到一种优秀的旧 `public static void main(String[] args)` 方法。`groovyc` 编译器把脚本行封装到此方法中以保留 Java 语义。这意味着在使用 Groovy 时可以利用所有的现有 Java 知识。

例如，下面是在 Groovy 脚本中接受命令行输入的方法。创建名为 Hello.groovy 的新文件并添加清单 7 中的代码行：

##### 清单 7\. 接受命令行输入的 Groovy 脚本

```java
println "Hello, " + args[0] 
```

现在在命令行中输入 `groovy Hello Jane`。`args``String` 数组就在这里，就像任何一位 Java 开发人员期望的那样。在这里使用 `args` 对于新手可能没意义，但是它对于经验丰富的 Java 开发人员意义非凡。

Groovy 将把 Java 代码缩减为基本要素。您刚刚编写的 Groovy 脚本几乎和可执行的伪代码一样。表面上，该脚本简单得足以让新手能够理解，但是对于经验丰富的开发人员，它没有去掉 Java 语言的底层强大功能。这就是我将 Groovy 视为 Java 平台的特定于领域语言（DSL）的原因（请参阅 什么是 DSL？ 侧栏）。

* * *

## 普通的旧 Groovy 对象

JavaBean — 或更通俗的名称，普通的旧 Java 对象（Plain Old Java Object，POJO）— 是 Java 开发的主要支柱。在创建 POJO 以表示域对象时，您应当遵循定义好的一组期望。类应当为 `public`，并且字段应当为带有一组对应的 `public` getter 和 setter 方法的 `private`。清单 8 显示了一个典型的 Java POJO：

##### 清单 8\. Java POJO

```java
public class JavaPerson{
  private String firstName;
  private String lastName;

  public String getFirstName(){ return firstName; }
  public void setFirstName(String firstName){ this.firstName = firstName; }

  public String getLastName(){ return lastName; }
  public void setLastName(String lastName){ this.lastName = lastName; }
} 
```

普通的旧 Groovy 对象（Plain Old Groovy Object，POGO）是 POJO 的简化的替代者。它们完全保留了 POJO 的语义，同时显著减少了需要编写的代码量。清单 9 显示了用 Groovy 编写的 “简易” person 类：

##### 清单 9\. Groovy POGO

```java
class GroovyPerson{
  String firstName
  String lastName
} 
```

除非您另外指定，否则 Groovy 中的所有类都是 `public` 的。所有属性都是 `private` 的，而所有方法都是 `public` 的。编译器将为每个属性自动提供一组 `public` getter 和 setter 方法。用 `javac` 编译 `JavaPerson` 并用 `groovyc` 编译 `GroovyPerson`。现在通过 `javap` 运行它们以确认该 Groovy 示例拥有 Java 示例所拥有的所有内容，甚至可以扩展 `java.lang.Object`（在先前的 `HelloGroovyWorld` 示例中未指定类，因此 Groovy 转而创建了扩展 `groovy.lang.Script` 的类）。

所有这些意味着您可以立即开始使用 POGO 作为 POJO 的替代选择。Groovy 类是只剩下基本元素的 Java 类。在编译了 Groovy 类之后，其他 Java 类可以轻松地使用它，就好像它是用 Java 代码编写的。为了证明这一点，请创建一个名为 JavaTest.java 的文件并添加清单 10 中的代码：

##### 清单 10\. 从 Java 代码中调用 Groovy 类

```java
public class JavaTest{
  public static void main(String[] args){
    JavaPerson jp = new JavaPerson();
    jp.setFirstName("John");
    jp.setLastName("Doe");
    System.out.println("Hello " + jp.getFirstName());

    GroovyPerson gp = new GroovyPerson();
    gp.setFirstName("Jane");
    gp.setLastName("Smith");
    System.out.println("Hello " + gp.getFirstName());
  }
} 
```

即使 getter 和 setter 不显示在 Groovy 源代码中，这项测试也证明了它们是存在于经过编译的 Groovy 类中并且完全可以正常运行。但是如果我不向您展示 Groovy 中的相应测试，则这个示例不算完整。创建一个名为 TestGroovy.groovy 的文件并添加清单 11 中的代码：

##### 清单 11\. 从 Groovy 中调用 Java 类

```java
JavaPerson jp = new JavaPerson(firstName:"John", lastName:"Doe")
println "Greetings, " + jp.getFirstName() + ". 
   It is a pleasure to make your acquaintance."

GroovyPerson gp = new GroovyPerson(lastName:"Smith", firstName:"Jane")
println "Howdy, ${gp.firstName}. How the heck are you?" 
```

您首先可能会注意到，Groovy 提供的新构造函数允许您命名字段并按照所需顺序指定这些字段。甚至更有趣的是，可以在 Java 类或 Groovy 类中使用此构造函数。这怎么可能？事实上，Groovy 先调用默认的无实参构造函数，然后调用每个字段的相应 setter。您可以模拟 Java 语言中的类似行为，但是由于 Java 语言缺少命名实参并且两个字段都是 `Strings`，因此您无法按照任何顺序传入名字和姓氏字段。

接下来，注意 Groovy 支持传统的 Java 方法来执行 `String` 串联，以及在 `String` 中直接嵌入用 `${}` 圈起的代码的 Groovy 方法（这些称为 `GString`，它是 *Groovy Strings* 的简写）。

最后，在类中调用 getter 时，您将看到使用 Groovy 语法的更多优点。您无需使用更冗长的 `gp.getFirstName()`，只需调用 `gp.firstName`。看上去您是在直接访问字段，但是事实上您在调用后台的相应的 getter 方法。Setter 将按照相同的方法工作：`gp.setLastName("Jones")` 和 `gp.lastName = "Jones"` 是等效的，后者将在后台调用前者。

我希望您也能够承认，在任何情况下，Groovy 都类似于简易版的 Java 语言 — “领域专家” 可能使用 Groovy “与同行进行有效地交流”，或者类似于老朋友之间随意的谈笑。

* * *

## 归根结底，Groovy 就是 Java 代码

Groovy 最被低估的一个方面是它完全支持 Java 语法的事实。如前述，在使用 Groovy 时，您不必放弃一部分 Java 知识。在开始使用 Groovy 时，大部分代码最终看上去都像是传统的 Java 代码。但是随着您越来越熟悉新语法，代码将逐渐发展为包含更简明、更有表现力的 Groovy 风格。

要证明 Groovy 代码可以看上去与 Java 代码完全相同，请将 JavaTest.java 复制到 JavaTestInGroovy.groovy 中，然后输入 `groovy JavaTestInGroovy`。您应当会看到同样的输出，但是请注意，您无需在运行前先编译 Groovy 类。

此次演示应当使经验丰富的 Java 开发人员能够不假思索地选择 Groovy。由于 Java 语法也是有效的 Groovy 语法，因此最开始的学习曲线实际上是不存在的。您可以将现有的 Java 版本与 Groovy、现有的 IDE 以及现有的生产环境结合使用。这意味着对您日常工作的干扰非常少。您只需确保 Groovy JAR 位于 `CLASSPATH` 中并调整构建脚本，以便 Groovy 类与 Java 类同时编译。下一节将向您展示如何向 Ant build.xml 文件中添加 `groovyc` 任务。

* * *

## 用 Ant 编译 Groovy 代码

如果 `javac` 是可插拔的编译器，则可以指示它同时编译 Groovy 和 Java 文件。但是它不是，因此您只需用 `groovyc` 任务在 Ant 中封装 `javac` 任务。这将允许 `groovyc` 编译 Groovy 源代码，并允许 `javac` 编译 Java 源代码。

当然，`groovyc` 既可以编译 Java 文件，又可以编译 Groovy 文件，但是还记得 `groovyc` 添加到 `HelloGroovyWorld` 和 `GroovyPerson` 中的额外的方便方法么？这些额外的方法也将被添加到 Java 类中。最好的方法可能就是让 `groovyc` 编译 Groovy 文件，而让 `javac` 编译 Java 文件。

要从 Ant 中调用 `groovyc`，请使用 `taskdef` 定义任务，然后像平时使用 `javac` 任务一样使用 `groovyc` 任务（有关更多信息，请参阅 参考资料）。清单 12 显示了 Ant 构建脚本：

##### 清单 12\. 用 Ant 编译 Groovy 和 Java 代码

```java
<taskdef name="groovyc"
         classname="org.codehaus.groovy.ant.Groovyc"
         classpathref="my.classpath"/>

<groovyc srcdir="${testSourceDirectory}" destdir="${testClassesDirectory}">
 <classpath>
   <pathelement path="${mainClassesDirectory}"/>
   <pathelement path="${testClassesDirectory}"/>
   <path refid="testPath"/>
 </classpath>
 <javac debug="on" />
</groovyc> 
```

顺便说一下，包含 `${}` 的 `String` 看上去疑似 `GString`，是不是？Groovy 是一种优秀的语言，它从各种其他语言和库中借鉴了语法和功能。您经常会觉得似乎在其他语言中见过类似的特性。

* * *

## 结束语

这是一次 Groovy 旋风之旅。您了解了一些关于 Groovy 曾经的地位及其目前现状的信息。您在系统中安装了 Groovy，并且通过一些简单的示例，了解了 Groovy 为 Java 开发人员提供的强大功能。

Groovy 不是运行在 JVM 上的惟一的备选语言。JRuby 是了解 Ruby 的 Java 开发人员的优秀解决方案。Jython 是了解 Python 的 Java 开发人员的优秀解决方案。但是正如您所见，Groovy 是了解 Java 语言的 Java 开发人员的优秀解决方案。Groovy 提供了类似 Java 的简明语法，同时保留了 Java 语义，这一点非常引人注目。而且，采用新语言的路径不包含 `del *.*` 或 `rm -Rf *` 是一次很好的变革，您不这样认为吗？

下期文章中，您将了解 Groovy 中的迭代。代码通常需要逐项遍历内容，不管它是列表、文件，还是 XML 文档。您将看到非常普遍的 `each` 闭包。到那时，我希望您可以发现大量 Groovy 的实际应用。