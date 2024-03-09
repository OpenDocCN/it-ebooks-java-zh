# 实战 Groovy: 使用 Groovy 模板进行 MVC 编程

*使用 Groovy 模板引擎框架简化报表视图*

视图是 MVC 编程的一个重要部分，而 MVC 编程本身又是企业应用程序开发的一个重要组件。在这篇实战 Groovy 的文章中，Andrew Glover 向您介绍了 Groovy 的模板引擎框架是如何用来简化视图编程的，并如何使您的代码更加经久容易维护。

在最近的 *实战 Groovy*系列中，我们已经介绍过 Groovy 是构建报表统计程序的一个非常好的工具。我们使用了一个校验和报表统计应用程序的例子向您介绍了 使用 Groovy 编写 Ant 脚本和一个数据库报表统计来展示 使用 GroovySql 进行 JDBC 编程。这两个示例统计报表都是在 Groovy 脚本中生成的，而且都具有一个相关的“视图”。

在校验和统计报表的例子中，视图代码有些混乱。更糟糕的是，它可能会对维护造成一些困难：如果我曾经希望修改视图的某个特定方面，就必须修改脚本中的代码。因此在本月的文章中，我将向您展示如何使用 Groovy 模板引擎和上一篇文章中的统计报表应用程序来对统计报表进行简化。

模板引擎与 XSLT 很类似，可以产生模板定义的任何格式，包括 XML、HTML、SQL 和 Groovy 代码。与 JSP 类似，模板引擎会简化将视图关注的内容分隔成单独的实体，例如 JSP 或模板文件。例如，如果一个报表希望的输出是 XML，那么您就可以创建一个 XML 模板，它可以包含一些占位符，在运行时替换为实际的值。模板引擎然后可以通过读取该模板并在这些占位符和运行时的值之间建立映射来实现对模板的转换。这个过程的输出结果是一个 XML 格式的文档。

## 关于本系列文章

在开发实践中采用任何工具的关键是了解何时使用这些工具，何时将其弃而不用。脚本语言可能是对工具包的一个功能强大的扩充，但是只有在正确应用于适当的环境时才是如此。总之，*实战 Groovy*系列文章是专门用来展示对 Groovy 的实际使用，以及何时和如何成功应用它。

在开始介绍 Groovy 模板之前，我想首先来回顾一下 Groovy 中 `String`类型的概念。所有的脚本语言都试图使字符串的使用变得异常简单。再次声明，Groovy 并不是让我们的技能逐步退化。在开始之前，请点击本页面顶部或底部的 **Code**图标（或者参阅 下载一节的内容），下载本文的源代码。

## 不能获得充足的 Strings

不幸的是，Java™ 代码中的 Strings 非常有限，不过 Java 5.0 应用程序承诺将有一些引人注目的新特性。现在，Groovy 解决了两个最差的 Java `String`限制，简化了编写多行字符串和进行运行时替换的功能。有一些简单的例子可以对 Groovy `String`类型进行加速，在本文中我们将使用这些例子。

如果您在普通的 Java 代码中编写一个多行的 `String`类型，就会最终要使用很多讨厌的 `+`号，对吗？但是在清单 1 中您可以看到，Groovy 不用再使用这些 `+`号了，这使您可以编写更加清晰、简单的代码。

##### 清单 1\. Groovy 中的多行字符串

```java
 String example1 = "This is a multiline
  string which is going to
  cover a few lines then
  end with a period." 
```

Groovy 还支持 here-docs 的概念，如清单 2 所示。*here-doc*是创建格式化 `String`（例如 HTML 和 XML）的一种便利机制。注意 here-doc 语法与普通的 `String`声明并没有很大的不同，不过它需要类 Python 的三重引号。

##### 清单 2\. Groovy 中的 Here-docs

```java
 itext =
"""
 This is another multiline String
 that takes up a few lines. Doesn't
 do anything different from the previous one.
""" 
```

Groovy 使用 `GString`来简化运行时替换。如果您不知道 GString 是什么，我可以确信您之前肯定见过它，而且还可能使用过它。简单来说，`GString`允许您使用 *与 bash 类似的*`${}`语法进行替换。`GString`的优势是您从来都不需要知道自己正在使用的是 `GString`类型；只需要在 Groovy 中简单地编写 `String`即可，就仿佛是在 Java 代码中一样。

## 关于模板引擎

模板引擎已经存在很长一段时间了，几乎在每个现代语言中都可以找到。普通的 Java 语言具有 Velocity 和 FreeMarker；Python 有 Cheetah 和 Ruby ERB；Groovy 也有自己的引擎。要学习更多有关模板引擎的内容，请参阅 参考资料。

##### 清单 3\. Groovy 中的 GString

```java
 lang = "Groovy"
 println "Uncle man, Uncle man, I dig ${lang}." 
```

在清单 3 中，我们创建了一个名为 `lang`的变量，并将其值设置为“Groovy”。我们打印了一个 `GString`类型的 `String`，并要求将单词 “dig” 后面的内容替换为 `${lang}`的值。如果实际运行一下，这段代码会打印“Uncle man, Uncle man, I dig Groovy.” 一切都不错，不是吗？

运行时替换实际上是动态语言的一个通用特性；与其他情况一样，Groovy 还会更进一步。Groovy 的 `GString`允许您对替换的值调用 *autocall*方法，当开始构建动态文本时，这会产生很多变化。例如，在清单 4 中，我可以对指定的变量按照 `String`对象类型调用一个方法（在本例中是 `length()`方法）。

##### 清单 4\. GString 自动调用

```java
 lang = "Groovy"
 println "I dig any language with ${lang.length()} characters in its name!" 
```

清单 4 中的代码会打印出“I dig any language with 6 characters in its name!”在下一节中，我将向您展示如何使用 Groovy 的自动调用特性在您的模板中启用一些复杂的特性。

* * *

## Groovy 模板

对模板的使用可以分解为两个主要的任务：首先，创建模板；其次，提供映射代码。使用 Groovy 模板框架创建模板与创建 JSP 非常类似，因为您可以重用 JSP 中见过的语法。创建这些模板的关键在于对那些运行时要替换的变量的定义。例如，在清单 5 中，我们为创建 `GroovyTestCase`定义了一个模板。

##### 清单 5\. 一个创建 GroovyTestCase 的模板

```java
 import groovy.util.GroovyTestCase
 class <%=test_suite %> extends GroovyTestCase {
  <% for(tc in test_cases) {
     println "\tvoid ${tc}() { } "
  }%>
 } 
```

清单 5 中的模板就类似于一个 JSP 文件，因为我们使用了 `&lt;%`和 `&lt;%=`语法。然而，由于 Groovy 的灵活性很好，因此您并不局限于使用 JSP 语法。您还可以自由使用 Groovy 中杰出的 `GString`，如清单 6 所示。

##### 清单 6\. GString 的使用

```java
 <person>
  <name first="${p.fname}" last="${p.lname}"/>
 </person> 
```

在清单 6 中，我创建了一个简单的模板，它表示一个定义 `person`元素集合的 XML 文档。您可以看到这个模板期望一个具有 `fname`和 `lname`属性、名为 `p`的对象。

在 Groovy 中定义模板相当简单，其实应该就是这样简单才对。Groovy 模板并不是火箭科学，它们只不过是一种简化从模型中分离视图过程的手段。下一个步骤是编写运行时的映射代码。

* * *

## 运行时映射

既然已经定义了一个模板，我们就可以通过在已定义的变量和运行时值之间建立映射来使用这个模板了。与 Groovy 中常见的一样，有些看来似乎很复杂的东西实际上是非常简单的。我所需要的是一个 `映射`，其关键字是模板中的变量名，键值是运行时的值。

例如，如果一个简单的模板有一个名为 `favlang`的变量，我就需要与 `favlang`键值建立一个 `映射`。这个键值可以是根据我自己的喜好选择的任何脚本语言（在本例中，当然是 Groovy）。

在清单 7 中，我们定义了这样一个简单的模板，在清单 8 中，我将向您展示对应的映射代码。

##### 清单 7\. 用来展示映射的简单代码

```java
 My favorite dynamic language is ${favlang} 
```

清单 8 显示了一个简单的类，它一共做了 5 件事，其中有 2 件是很重要的。您可以说出它们是什么吗？

##### 清单 8\. 为一个简单的模板映射值

```java
 package com.vanward.groovy.tmpl
 import groovy.text.Template
 import groovy.text.SimpleTemplateEngine
 import java.io.File
 class SimpleTemplate{
  static void main(args) {
    fle = new File("simple-txt.tmpl")
    binding = ["favlang": "Groovy"]
    engine = new SimpleTemplateEngine()
    template = engine.createTemplate(fle).make(binding)
    println template.toString()
  }
 } 
```

在清单 8 中为这个简单的模板映射值简单得令人吃惊。

首先，我创建了一个 `File`实例，它指向模板 `simple-txt.tmpl`。

然后创建了一个 `binding`对象；实际上，这就是一个 `映射`。我将在模板中找到的值 `favlang`映射到 `String`Groovy 上。这种映射是在 Groovy 中使用模板最重要的步骤，或者说在任何具有模板引擎的语言中都是如此。

接下来，我创建了一个 `SimpleTemplateEngine`实例，在 Groovy 中，它恰巧就是模板引擎框架的一个具体实现。然后我将模板（`simple-txt.tmpl`）和 `binding`对象传递给这个引擎实例。在清单 8 中，第二个重要的步骤是将模板及其 `binding`对象绑定在一起，这也是使用模板引擎的关键所在。从内部来说，框架将在从 `binding`对象中找到的值与对应模板中的名字之间建立映射。

清单 8 中的最后一个步骤是打印进程的输出信息。正如您可以看到的一样，创建一个 `binding`对象并提供正确的映射是件轻而易举的小事，至少在我们这个简单的例子中是如此。在下一节中，我们将会使用一个更加复杂的例子对 Groovy 模板引擎进行测试。

* * *

## 更复杂的模板

在清单 9 中，我已经创建了一个 `Person`类来表示在 清单 6 中定义的 `person`元素。

##### 清单 9\. Groovy 中的 Person 类

```java
 class Person{
 age
 fname
 lname
 String toString(){
  return "Age: " + age + " First Name: " + fname + " Last Name: " + lname
 }
 } 
```

在清单 10 中，您可以看到对上面定义的 `Person`类的实例进行映射的代码。

##### 清单 10\. 在 Person 类与模板之间建立映射

```java
 import java.io.File
 import groovy.text.Template
 import groovy.text.SimpleTemplateEngine
 class TemplatePerson{
  static void main(args) {
    pers1 = new Person(age:12, fname:"Sam", lname:"Covery")
    fle = new File("person_report.tmpl")
    binding = ["p":pers1]
    engine = new SimpleTemplateEngine()
    template = engine.createTemplate(fle).make(binding)
    println template.toString()
  }
 } 
```

上面的代码看起来很熟悉，不是吗？实际上，它与 清单 8 非常类似，不过增加了一行创建 `pers1`实例的代码。现在，再次快速查看一下 清单 6 中的代码。您看到模板是如何引用属性 `fname`和 `lname`的了吗？我所做的操作是创建一个 `Person`实例，其 `fname`属性设置为“Sam”，属性 `lname`设置为“Covery”。

在运行清单 10 中的代码时，输出结果是 XML 文件，用来定义 `person`元素，如清单 11 所示。

##### 清单 11\. Person 模板的输出结果

```java
 <person>
  <name first="Sam" last="Covery"/>
 </person> 
```

### 映射一个列表

在 清单 5 中，我为 `GroovyTestCase`定义了一个模板。现在如果您看一下这个模板，就会注意到这个定义有一些逻辑用于在一个集合上迭代。在清单 12 中，您将看到一些非常类似的代码，不过这些代码的逻辑是用来映射一个测试用例 `列表`的。

##### 清单 12\. 映射测试用例列表

```java
 fle = new File("unit_test.tmpl")
 coll = ["testBinding", "testToString", "testAdd"]
 binding = ["test_suite":"TemplateTest", "test_cases":coll]
 engine = new SimpleTemplateEngine()
 template = engine.createTemplate(fle).make(binding)
 println template.toString() 
```

查看一下 清单 5，它显示了模板期望一个名为“test_cases”的 `列表`—— 在清单 12 中它定义为 `coll`，包含 3 个元素。我简单地将 `coll`设置为“test_cases”绑定对象中的键值，现在代码就准备好运行了。

现在应该十分清楚了，Groovy 模板非常容易使用。它还可以促进无所不在的 MVC 模式的使用；更重要的是，它们可以通过表示视图来支持转换为 MVC 代码。在下一节中，我将向您展示如何对上一篇文章中的一个例子应用本文中所学到的知识。

* * *

## 使用模板重构之前的例子

在使用 Groovy 编写 Ant 脚本的专栏中，我曾经编写了一个简单的工具，它对类文件产生校验和报告。如果您还记得，我当时笨拙地使用 `println`语句来产生 XML 文件。尽管我只能接受这段代码，但它是如此地晦涩，这您只需要看一下清单 13 就会一目了然。

##### 清单 13\. 糟糕的代码

```java
 nfile.withPrintWriter{ pwriter |
  pwriter.println("<md5report>")
    for(f in scanner){
       f.eachLine{ line |
         pwriter.println("<md5 class='" + f.path + "' value='" + line + "'/>")
       }
    }
  pwriter.println("</md5report>")
 } 
```

为了帮您回顾一下有关这段内容的记忆，清单 13 中的代码使用了一些数据，并使用 `PrintWriter`将其写入一个文件中（`nfile`实例）。注意我是如何将报告的视图组件（XML）硬编码在 `println`中的。这种方法的问题是它不够灵活。之后，如果我需要进行一些修改，就只能进入到 Groovy 脚本的逻辑中进行修改。在更糟糕的情况下，设想一下一个非程序员想要进行一些修改会是什么样子。Groovy 代码会受到很大的威胁。

将该脚本中的视图部分移动到模板中可以使维护更加方便，因为修改模板是任何人都会涉及的一个过程，因此我在此处也会这样做。

### 定义模板

现在我将开始定义模板了 —— 它看起来更加类似于想要的输出结果，采用了一些逻辑来循环遍历一组类。

##### 清单 14\. 为原来的代码应用模板

```java
 <md5report>
 <% for(clzz in clazzes) {
  println "<md5 class=\"${clzz.name}\" value=\"${clzz.value}\"/>"
 }%>
 </md5report> 
```

清单 14 中定义的模板与为 `GroovyTestCase`定义的模板类似，其中包括循环遍历一个集合的逻辑。还要注意我在此处混合使用了 JSP 和 `GString`的语法。

### 编写映射代码

定义好模板之后，下一个步骤是编写运行时的映射代码。我需要将原来的写入文件的逻辑替换为下面的代码：构建一个 `ChecksumClass`对象集合，然后将这些对象放到 `binding`对象中。

这个模型然后就会变成清单 15 中定义的 `ChecksumClass`。

##### 清单 15\. 在 Groovy 中定义的 CheckSumClass

```java
 class CheckSumClass{
  name
  value
  String toString(){
   return "name " + name + " value " + value
  }
 } 
```

Groovy 中类的定义非常简单，不是吗？

### 创建集合

接下来，我需要重构刚才写入文件的那段代码 —— 这一次采用一定的逻辑使用 `ChecksumClass`构造一个列表，如清单 16 所示。

##### 清单 16\. 重构代码创建一个 ChecksumClass 的集合

```java
 clssez = []
 for(f in scanner){
  f.eachLine{ line |
   iname = formatClassName(bsedir, f.path)
   clssez << new CheckSumClass(name:iname, value:line)
  }
 } 
```

清单 16 显示了使用类 Ruby 的语法将对象添加到 `列表`中是多么简单 —— 这就是 _ 奇妙的 _groovy。我首先使用 `[]`语法创建 `清单`。然后使用简短的 `for`循环，后面是一个带有闭包的迭代器。这个闭包接受每一个 `line`（在本例中是一个校验和值），并创建一个新定义的 `CheckSumClass`实例（使用 Groovy 的自动生成的构造函数），并将二者添加到集合中。还不错 —— 这样编写起来也很有趣。

### 添加模板映射

我需要做的最后一件事情是添加模板引擎特定的代码。这段代码将执行运行时映射，并将对应的格式化后的模板写入原始的文件中，如清单 17 所示。

##### 清单 17\. 使用模板映射重构原来的代码

```java
 fle = new File("report.tmpl")
 binding = ["clazzes": clzzez]
 engine = new SimpleTemplateEngine()
 template = engine.createTemplate(fle).make(binding)
 nfile.withPrintWriter{ pwriter |
  pwriter.println template.toString()
 } 
```

现在，清单 17 中的代码对您来说太陈旧了。我利用了清单 16 中的 `列表`，并将其放入 `binding`对象。然后读取 `nfile`对象，并将相应的输出内容从 清单 14 中的映射模板写入文件中。

在将这些内容都放入清单 18 之前，您可能希望返回 清单 13 最后看一眼开始时使用的那段蹩脚的代码。下面是新的代码，您可以进行比较一下：

##### 清单 18\. 看，新的代码！

```java
 /**
 *
 */
 buildReport(bsedir){
 ant = new AntBuilder()
 scanner = ant.fileScanner {
   fileset(dir:bsedir) {
     include(name:"**/*class.md5.txt")
   }
 }
 rdir = bsedir + File.separator + "xml" + File.separator
 file = new File(rdir)
 if(!file.exists()){
   ant.mkdir(dir:rdir)
 }
 nfile = new File(rdir + File.separator + "checksum.xml")
 clssez = []
 for(f in scanner){
   f.eachLine{ line |
    iname = formatClassName(bsedir, f.path)
    clssez << new CheckSumClass(name:iname, value:line)
   }
 }
 fle = new File("report.tmpl")
 binding = ["clazzes": clzzez]
 engine = new SimpleTemplateEngine()
 template = engine.createTemplate(fle).make(binding)
 nfile.withPrintWriter{ pwriter |
   pwriter.println template.toString()
 }
 } 
```

现在，虽然我没有声明要编写非常漂亮的代码，但是这些代码当然不会像原来的代码一样 *糟糕*了。回顾一下，我所干的事情不过是将一些蹩脚的 `println`替换成 Groovy 的更精巧的模板代码。（一些熟悉重构的人可能会说我应该使用 *Extract Method*进一步对代码进行优化。）

* * *

## 结束语

在本月的教程中，我希望已经向您展示了 Groovy 的 *视图*。更 *明确地*说，当您需要快速开发一些需要视图的简单程序时，Groovy 的模板框架是普通 Java 编码的一种很好的替代品。模板是很好的一种抽象，如果使用正确，它可以极大地降低应用程序的维护成本。

下一个月，我将向您展示如何使用 Groovy 来构建使用 Groovlets 的 Web 应用程序。现在，享受使用 Groovy 的模板开发吧！

* * *

## 下载

| 描述 | 名字 | 大小 |
| --- | --- | --- |
|  |

[j-pg02155-source.zip](http://www.ibm.com/developerworks/apps/download/index.jsp?contentid=58276&filename=j-pg02155-source.zip&method=http&locale=zh_CN) | 2.18 KB |