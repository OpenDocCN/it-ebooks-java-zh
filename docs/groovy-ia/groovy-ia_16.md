# 实战 Groovy: 用 Groovy 进行 Ant 脚本编程

*为更具表现力、更可控的构建而组合使用 Ant 和 Groovy*

Ant 和 Maven 两者在构建处理工具的世界中占统治地位。但是 XML 却凑巧是一种非常没有表现力的配置格式。在“实战 Groovy”这个新系列的第 2 期中，Andrew Glover 将介绍 Groovy 的生成器实用工具，这个工具能够极其容易地把 Groovy 与 Ant 和 Maven 结合在一起，形成更具表现力、更可控的构建。

Ant 作为 Java 项目构建工具的普遍性和实用性是无法超越的。即使是 Maven 这个构建领域的新锐工具，也要把自己的许多强大能力归功于从 Ant 学到的经验。但是，这两个工具有共同的不足之处：扩展性。即使 XML 的可移植性在促进 Ant 和 Maven 走向开发前端上扮演了主要角色，XML 作为构建配置格式的角色仍然或多或少地限制了构建过程的表现力。

例如，虽然在 Ant 和 Maven 中都有条件逻辑，但是用 XML 表示有些繁琐。而且，虽然可以定义定制任务来扩展 Ant 的构建过程，但是这么做通常会把应用程序的行为局限在有限的沙箱中。因此，在本文中，我将向您演示如何把 Groovy 和 Ant *在* Maven *内部* 结合起来，形成更强大的表现力，从而对构建过程进行更好的行为控制。

很快您自己就会看到，Groovy 给 Ant 和 Maven 都带来了令人瞩目的增强，Groovy 的优点在于它常常能恰到好处地弥补 XML 的遗漏！实际上，看起来 Groovy 的创建者肯定也曾经历过用 XML 实施 Ant 和 Maven 构建过程的痛苦，因为他们引入了 `AntBuilder`，这是一个强大的新工具，支持在 Groovy 脚本中利用 Ant。

在 *实战 Groovy* 的这一期中，我将向您展示不用 XML，转用 Groovy 作为您的构建配置格式，对构造过程进行增强是多么容易。闭包（closure）是 Groovy 的一个重要特性，而且是这门语言之所以与众不同的核心，所以我在转到下一节之前，先要对闭包做一个快速回顾。

## 快速回顾闭包

就像 Groovy 自己的一些非常著名的祖先一样，Groovy 也支持 *匿名函数* 或 *闭包（closure）* 的概念。如果您曾经用 Python 或 Jython 编写过代码，那么您可能对闭包比较熟悉，在这两种语言中，都是用 `lambda` 关键字引入闭包。在 Ruby 中，如果 *没有* 利用块或闭包，那么您就不得不实际地编写脚本。即使是 Java 语言，也要通过它的 *匿名内部类*，对匿名函数提供了有限形式的支持。

在 Groovy 中，闭包是能够封装行为的第一级匿名函数。当 Ruby 的缔造者 Yukihiro Matsumoto 注意到这些强大的第一类对象可以“传递给另一个函数，然后函数可以调用传入的 [闭包]”时，也开始染指闭包的应用（请参阅 参考资料，查看完整的采访）。当然，亲自操作 Groovy 是了解闭包对于这门美好的令人激动的语言是一笔多么大的财富的最好方法。

## 关于这一系列的教程

把任何一个工具集成进您的开发实践的关键是，知道什么时候使用它而什么时候把它留在箱子中。脚本语言可以是您的工具箱中极为强大的附件，但是只有在恰当地应用到适当地场景中才是这样。为了这个目标， *实战 Groovy* 是一系列文章，专门介绍 Groovy 的实际应用，并教给您什么时候、如何成功地应用它们。

* * *

## 闭包实例

在清单 1 中，我又使用了一些本系列的第 1 篇文章曾经用过的代码；但是这一次的重点是闭包。如果您已经读过那篇文章（请参阅 参考资料），那么您可以回忆起我用来演示用 Groovy 进行单元测试的基于 Java 的包过滤对象。这次，我还用相同的示例开始，但是用闭包对它进行了极大的增强。下面是基于 Java 的 `Filter` 接口。

##### 清单 1\. 还记得这个简单的 Java Filter 接口吗？

```java
public interface Filter {
  void setFilter(String fltr);  
  boolean applyFilter(String value);
} 
```

上次，在定义了 `Filter` 类型之后，我接着定义了两个实现，这两个实现名为 `RegexPackageFilter` 和 `SimplePackageFilter`，它们分别使用了正则表达式和简单的 `String` 操作。

对于没用闭包写成的代码来说，到现在为止还算不错。在清单 2 中，您会开始看到只有一点语法上的变化，代码就不同了（更好了！）。我将通过定义通用的 `Filter` 类型（如下所示）开始，但是这次是用 Groovy 定义的。请注意与 `Filter` 类关联的 `strategy` 属性。这个属性是闭包的一个实例，在执行 `applyFilter` 方法的时候可以调用它。

##### 清单 2\. 更加 Groovy 的过滤器 —— 使用闭包

```java
class Filter{
   strategy
   boolean applyFilter(str){
      return strategy.call(str)
   }
} 
```

添加闭包意味着我不用像在原来的 `Filter` 接口中那样必须定义一个接口类型，或者依赖特定的实现才能得到期望的行为。相反，我可以定义一个通用的 `Filter` 类型，并给它提供一个闭包，让它在执行 `applyFilter` 方法期间应用。

下一步是定义两个闭包。第一个通过对指定参数应用简单的 `String` 操作来模拟 `SimplePackageFilter`（来自前一篇文章）。当创建新的 `Filter` 类型时，对应的名为 `simplefilter` 的闭包会被传递到构造器中。第二个闭包（在几个进行代码验证的 `assert` 之后出现）对指定的 `String` 应用正则表达式。这次我还是要创建一个新的 `Filter` 类型，向正则表达式传递一个名为 `rfilter` 的闭包，并执行一些 `assert`，以确保每件事都正常。所有细节如清单 3 所示:

##### 清单 3\. 使用 Groovy 闭包的简单魔术

```java
simplefilter = { str | 
   if(str.indexOf("java.") >= 0){
     return true
   }else{
     return false
   }
}

fltr = new Filter(strategy:simplefilter)
assert !fltr.apply("test")
assert fltr.apply("java.lang.String")

rfilter = { istr |
   if(istr =~ "com.vanward.*"){
     return true
   }else{
     return false
   }
}

rfltr = new Filter(strategy:rfilter)
assert !rfltr.apply("java.lang.String")
assert rfltr.apply("com.vanward.sedona.package") 
```

非常令人难忘，对吧？使用闭包，我能够把对期望行为的定义推迟到运行时 —— 所以不必像在以前的设计中要做的那样再定义新的 `Filter` 类型并编译它。虽然我用 Java 代码时能用匿名内部类做类似的事情，但是用 Groovy 更容易、神秘性更少一些。

闭包确实是非常强大的家伙。它们还代表处理行为的另外一种方式 —— 而 Groovy（以及它的远亲 Ruby）对它有非常严重的依赖。所以闭包会是我们下一个主题的要点，下一个主题是用生成器进行构建。

* * *

## 用生成器进行构建

使 Groovy 中的 Ant 更迷人的核心之处是 *生成器*。实际上，生成器允许您很方便地在 Groovy 中表示树形数据结构，例如 XML 文档。而且，女士们先生们请看，秘密在这：使用生成器，特别是 `AntBuilder`，您可以毫不费力地构造 Ant 的 XML 构建文件，不必处理 XML 就可以 *执行生成的行为*。而这并不是在 Groovy 中使用 Ant 的惟一优点。与 XML 不同，Groovy 是非常有表现力的开发环境，在这个环境中，您可以容易地编写循环结构、条件选择代码，甚至可以利用“重用”的威力，而不必像以前那样，费力地用剪切-粘贴操作来创建新 build.xml 文件。而且您做这些工作时，完全是在 Java 平台中！

生成器的优点，尤其是 Groovy 的 `AntBuilder`，在于它们的语法表示完全体现了它们所代表的 XML 文件的逻辑进程。被附加在 `AntBuilder` 实例上的方法与对应的 Ant 任务匹配；同样的，这些方法可以接收参数（以 `map` 的形式），参数对应着任务的属性。而且，嵌套标签（例如 `include` 和 `fileset`）也定义成闭包。

### 构建块：示例 1

我要用一个超级简单的示例向您介绍生成器：一个叫做 `echo` 的 Ant 任务。在清单 4 中，我创建了一个普通的、每天都会用到的 Ant 的 `echo` 标记的 XML 版本（用在这不要奇怪）：

##### 清单 4\. Ant 的 Echo 任务

```java
<echo message="This was set via the message attribute"/>
<echo>Hello World!</echo> 
```

事情在清单 5 中变得更有意思了：我用相同的 Ant 标签，并在 Groovy 中用 `AntBuilder` 类型重新定义了它。注意，我可以使用 `echo` 的属性 `message`，也可以只传递一个期望的 `String`。

##### 清单 5\. 用 Groovy 表示的 Ant 的 Echo 任务

```java
ant = new AntBuilder()
ant.echo(message:"mapping it via attribute!")         
ant.echo("Hello World!") 
```

生成器特别吸引人的地方是它可以让我把普通的 Groovy 特性与生成器语法混合，从而创建丰富的行为集。在清单 6 中，您应当开始看出可能性是无穷的：

##### 清单 6\. 用 Groovy 和 Ant 进行流控制（flow control）

```java
ant = new AntBuilder()
ant.mkdir(dir:"/dev/projects/ighr/binaries/")
try{
    ant.javac(srcdir:"/dev/projects/ighr/src", 
       destdir:"/dev/projects/ighr/binaries/" )
}catch(Throwable thr){
    ant.mail(mailhost:"mail.anywhere.com", subject:"build failure"){
       from(address:"buildmaster@anywhere.com", name:"buildmaster")
       to(address:"dev-team@anywhere.com", name:"Development Team")
       message("Unable to compile ighr's source.")
    }
} 
```

在这个示例中，我要捕获源代码编译时的错误条件。注意 `catch` 块中定义的 `mail` 对象如何接受定义了 `from`、 `to` 和 `message` 属性的闭包。

_ 哎呀！_Groovy 的功能真多！当然，知道什么时候应用这么聪明的特性是问题的关键，而我们都在不断地为之努力。幸运的是，实践出真知，一旦您开始使用 Groovy（或者为了这个原因使用任何脚本语言），您就会找到许多在实际工作中使用它的机会；从中了解它在哪里才真正适用。在下一节中，我将查看一个典型的、现实的示例，并在其中使用一些在这里介绍的特性。

* * *

## 应用 Groovy

对于这个示例，我们假设需要为我的代码定期建立校验和（checksum）报告。一旦实现了这个报告，就可以用它在我需要的时候检验文件的完整性。下面是校验和报告的高级技术用例：

1.  编译所有的源代码。
2.  对二进制类文件运行 md5 算法。
3.  创建简单的报告，列出每个类文件及其对应的校验和。

完全摈弃 Ant 或 Maven，整个构建过程都使用 Groovy，在这个例子中，这样做有点极端。但实际上，正如我前面解释的，Groovy 是这些工具的极大 *增强*，而 *不是* 替代。所以，用 Groovy 的表现力处理后两项任务，而把第一步信托给 Ant 或 Maven，这样做才有意义。

实际上，我们只要假设我在第一步中用的是 Maven，因为坦白地说，它是我个人偏爱的构建平台。在 Maven 中可以很容易地用 `java:compile` 和 `test:compile` 目标编译源文件；所以我保持这部分内容原封不动，让我的新目标引用前面的目标，把前面的目标作为前提条件。实际就是这样 —— 使用编译好的源文件，我准备继续运行校验和工具（checksum utility）；但是首先还需要做一些简单的设置。

* * *

## 设置 Md5ReportBuilder

为了通过 Ant 运行这个漂亮的校验和工具，我需要两条信息：哪个目录包含要进行校验和处理的文件，报告要写到哪个目录。对于工具的第一个参数，我希望它是一个用逗号分隔的目录列表。对后一个参数，我希望是报告目录。

我决定调用工作类 `Md5ReportBuilder`。清单 7 中定义了它的 `main` 方法：

##### 清单 7\. Md5ReportBuilder 的 Main 方法

```java
static void main(args) {         

  assert args[0] && args[1] != null

  dirs = args[0].split(",")
  todir = args[1]

  report = new Md5ReportBuilder()
  report.runCheckSum(dirs)  
  report.buildReport(todir)             
} 
```

上述步骤中的第一步是检查这两个参数是不是传递到工具中。然后我把第一个参数用逗号进行分割，创建了一个数组，保存要在上面运行 Ant 的 `checksum` 任务的目录。最后，我创建 `Md5ReportBuilder` 类的新实例，调用两个方法处理所需要的功能。

* * *

## 添加校验和

Ant 包含一个 `checksum` 任务，调用起来非常容易，但需要传递一个 `fileset` 给它，其中包含目标文件的集合。在这个例子中，目标文件是包含编译后的源文件的目录，以及对应的单元测试文件。我可以通过使用 `for` 循环中的迭代得到这些文件，在这个例子中，是在目录集合中进行迭代。对于每个目录，调用 `checksum` 任务；而且 `checksum` 任务只在 .class 文件上运行，如清单 8 所示：

##### 清单 8\. runCheckSum 方法

```java
/**
 * runs checksum task for each dir in collection passed in
 */
runCheckSum(dirs){
  ant = new AntBuilder()       
  for(idir in dirs){       
    ant.checksum(fileext:".md5.txt" ){
      fileset(dir:idir) {
        include(name:"**/*.class")        
      }
   }
 }
} 
```

* * *

## 构建报告

从这一点起，构建报告就变成了循环练习。读取每个新生成的校验和文件，把该文件对应的信息送到 `PrintWriter` 方法，然后将 XML 写入文件 —— 虽然用的是最丑陋的形式，如清单 9 所示：

##### 清单 9\. 构建报告

```java
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
  nfile.withPrintWriter{ pwriter |
     pwriter.println("<md5report>")
     for(f in scanner){
       f.eachLine{ line |
         pwriter.println("<md5 class='" + f.path + "' value='" + line + "'/>")
       }
     }
     pwriter.println("</md5report>")
  }        
} 
```

那么，如何处理这个报告呢？首先，我用 `FileScanner` 找到清单 8 中的 `checksum` 方法创建的每个校验和文件。然后，我创建一个新目录，在这个目录中再创建一个新文件。（如果我能通过一个简单的 `if` 语言检查目录是否已经存在，会不会更好一些？）然后我打开对应的文件，并用一个漂亮的闭包从 `scanner` 集合读取每个对应的 `File`。示例最后用写入 XML 元素的报告内容进行总结。

## Goal 就是您的目标！

不熟悉 Maven 的读者可能想知道 `goal` 谈的都是什么。把 Maven 中的 `goal` 当成 Ant 中的 `target` 就可以了。 *goal* 就是组织行为的一种方式。Maven 的 `goal` 被赋予名称，可以通过 `maven` 命令调用它们。在调用时，在指定 `goal` 找到的任务都会被执行。

我敢打赌您立刻就注意到在 `File` 的那些实例上的 `withPrintWriter` 方法是多么强大。我不需要考虑异常或关闭文件，因为它替我处理了每件事。我只是把我希望的行为通过闭包传递给它，然后就准备就绪了！

* * *

## 运行工具

这个 Groovy 巡演中的下一站是把它装配进构建过程，特别是放在我的 maven.xml 文件中。非常幸运的是，这是这个相对容易的练习中最容易的部分，如清单 10 所示：

##### 清单 10\. 在 Maven 中运行 Md5ReportBuilder

```java
<goal name="gmd5:run" prereqs="java:compile,test:compile">
  <path id="groovy.classpath">                        
    <ant:pathelement path="${plugin.getDependencyClasspath()}"/>
    <ant:pathelement location="${plugin.dir}"/>
    <ant:pathelement location="${plugin.resources}"/>            
  </path>
  <java classname="groovy.lang.GroovyShell" fork="yes">
    <classpath refid="groovy.classpath"/>
    <arg value="${plugin.dir}/src/groovy/com/vanward/md5builder/Md5ReportBuilder.groovy"/>
    <arg value="${maven.test.dest},${maven.build.dest}"/>
    <arg value="${maven.build.dir}/md5-report"/>
  </java>
</goal> 
```

正如前面所解释过的，我已经规定必须在完整的编译之前运行校验和工具，所以，我的目标有两个前提条件： `java:compile` 和 `test:compile`。类路径 Classpath 总是很重要，所以我专门为了让 Groovy 运行特别创建了一个正确的 classpath。最后清单 10 所示的目标调用 Groovy 的外壳，把要运行的脚本和脚本对应的两个参数传给外壳 —— 第一个是要在其上运行校验和（用逗号分隔的）目录，第二个是报告要写入的目录。

* * *

## 最后一步

完成对 maven.xml 文件的编码之后，就差不多完成了所有要做的事，但是还有最后一步：我需要用所需的依赖关系来更新 `project.xml` 文件。没什么好奇怪的，Maven 需要具有一些必要的 Groovy 依赖项才能工作。这些依赖项是汇编代码、字节码操纵库、公共命令行（commons-cli-处理命令行解析）Ant、以及对应的 ant 启动器。这个示例的依赖项如清单 11 所示：

##### 清单 11\. Groovy 必需的依赖项

```java
<dependencies>
  <dependency>
    <groupId>groovy</groupId>
    <id>groovy</id>
    <version>1.0-beta-6</version>
  </dependency>
  <dependency>
    <groupId>asm</groupId>
    <id>asm</id>
    <version>1.4.1</version>
  </dependency>
  <dependency>
    <id>commons-cli</id>
    <version>1.0</version>
  </dependency>
  <dependency>
    <groupId>ant</groupId>
    <artifactId>ant</artifactId>
    <version>1.6.1</version>
  </dependency>
  <dependency>
    <groupId>ant</groupId>
    <artifactId>ant-launcher</artifactId>
    <version>1.6.1</version>
  </dependency>
</dependencies> 
```

* * *

## 结束语

在 *实战 Groovy* 的第 2 期中，您看到了 Groovy 的表现力和灵活性与 Ant 和 Maven 无与伦比的应用结合在一起时发生了什么。对于任何一个工具，Groovy 都提供了有吸引力的替代 XML 的构建格式。它让您用循环构造和条件逻辑控制程序流，极大地增强了构建过程。

在向您展示 Groovy 的实用一面的同时，这个系列还专门介绍了它最适当的应用。应用任何技术的要点之一是认真思考它要应用的环境。在这个案例中，我向您展示了如何用 Groovy *增强* 而不是 *替换* 已经非常强大的工具：Ant。

下个月，我要介绍 Groovy 的另外一项依赖闭包的特性。GroovySql 是个超级方便的小工具，可以使数据库查询、更新、插入以及所有对应的逻辑管理起来特别容易！下期再见！