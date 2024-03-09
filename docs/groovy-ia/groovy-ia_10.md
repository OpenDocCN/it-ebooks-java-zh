# 实战 Groovy: Groovy 的腾飞

*熟悉 Groovy 新的、遵循 JSR 的语法*

随着 Groovy JSR-1（及其后续发行版本）的发布，Groovy 语法的变化已经规范化 —— 这意味着如果以前没有对此加以注意，那么现在是开始注意它的时候了。这个月，Groovy 的常驻实践者 Andrew Glover 将介绍 Groovy 语法最重要的变化，以及在经典 Groovy 中找不到的一个方便特性。

从我在 *alt.lang.jre*的系列文章“[Feeling Groovy](http://www.ibm.com/developerworks/java/library/j-alj08034.html?S_TACT=105AGX52&S_CMP=cn-a-j)”中介绍 Groovy 开始，差不多有一年时间了。从那时起，通过发行许多版本，逐步解决了语言实现中的问题，并满足开发人员社区的一些特性请求，Groovy 已经成熟了许多。最后，在今年四月，Groovy 有了一个重大飞跃，它正式发布了新的解析器，该解析器的目标就是将这门语言标准化为 JSR 进程的一部分。

在本月“*实战 Groovy*”这一期的文章中，我将祝贺 Groovy 的成长，介绍通过 Groovy 非常好用的新解析器规范化的一些最重要的变化：即变量声明和闭包。因为我要将一些新 Groovy 语法与我在关于 Groovy 的第一篇文章中介绍的经典语法进行比较，所以您可以在另一个浏览器窗口中打开“alt.lang.jre: 感受 Groovy”这篇文章。

## 为什么会发生这些变化？

如果您一直在跟踪 Groovy，不管是阅读文章和 blog，还是自己编写代码，您都可能已经遭遇过这门语言的一、两个微妙的问题。在进行一些灵敏的操作，例如对象导航，特别是使用闭包的时候，Groovy 偶尔会遇到歧义性问题和语法受限的问题。几个月之前，作为 JSR 进程的一部分，Groovy 团队开始着手解决这些问题。四月份，随 groovy-1.0-jsr-01 发行版本提供的解决方案是一个更新过的语法以及一个用来对语法进行标准化的新语法内容解析器。

## 关于本系列

将任何一个工具集成到开发实践中的关键是：知道什么时候使用它，什么时候把它留在箱子中。脚本语言可以是工具箱中极为强大的附件，但是只有在将它恰当应用到适当的地方时才这样。为了实现这个目标，*实战 Groovy*是一个文章系列，专门介绍 Groovy 的实际应用，并教导您什么时候使用它们，以及如何成功地应用它们。

好消息是：新语法是对语言的完全增强。另一个好消息是：它和以前的语法没有很大不同。像所有 Groovy 一样，语法的设计目标是较短的学习曲线和较大的回报。

当然，符合 JSR 的解析器也给新 Groovy 带来一些与目前“经典”语法不兼容的地方。如果用新的解析器运行本系列以前文章中的代码示例，那么您自己就可以看，代码示例可能无法工作！现在，这一点看起来可能有点苛刻 —— 特别是对 Groovy 这样自由的语言来说 —— 但是解析器的目标是确保作为 Java 平台的 *标准化*语言的 Groovy 的持续成长。可以把它当作新 Groovy 的一个有帮助的指南。

* * *

## Groovy 依然是 Groovy ！

在深入研究变化的内容之前，我要花几秒钟谈谈什么 *没有改变*。首先，动态类型化的基本性质没有改变。变量的显式类型化（即将变量声明为 `String`或 `Collection`） 依然是可选的。稍后，我会讨论对这一规则的一点新增内容。

知道分号依然是可选的时候，许多人都会感到轻松。对于放松对这个语法的使用，存在许多争论，但是最终少数派赢得了胜利。底线是：如果愿意，也可以使用分号。

集合（Collection）的使用大部分还保持不变。仍然可以用 *array*语法和 `map`，像以前那样（即最初从文章“alt.lang.jre: 感受 Groovy”中学到的方式）声明类似 `list`的集合。但范围上略有变化，我将在后面部分展示这一点。

最后，Groovy 对标准 JDK 类的增加没有发生多少变化。语法糖衣和漂亮的 API 也没变， 就像普通的 Java `File`类型的情况一样，我稍后将展示这一点。

* * *

## 容易变的变量

Groovy 的变量规则对新的符合 JSR 的语法的打击可能最大。经典的 Groovy 在变量声明上相当灵活（而且实际上很简洁）。而使用新的 JSR Groovy 时，所有的变量前都必须加上 `def`关键字或者 `private`、`protected`或 `public`这样的修饰符。当然，也可以声明变量类型。另外，如果正在定义类，希望声明属性（使用 JavaBeans 样式的 getter 和 setter 公开），那么也可以用 `@Property`关键字声明成员变量。请注意 —— *Property*中的 *P*是大写的！

例如，在“alt.lang.jre: 感受 Groovy”中介绍 *GroovyBeans*时，我在文章的清单 22 中定义了一个叫做 `LavaLamp`的类型。这个类不再符合 JSR 规范，如果要运行它，则会造成解析器错误。幸运的是，迁移这个类不是很困难：我要做的全部工作就是给所有需要的成员变量添加 `@Property`属性，如清单 1 所示：

##### 清单 1\. LavaLamp 的返回结果

```java
 package com.vanward.groovy 
 class LavaLamp{ 
  @Property model 
  @Property baseColor 
  @Property liquidColor 
  @Property lavaColor 
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

不是太坏，不是吗？

正如上面描述的，对于 *任何*变量，如果没有修饰符、`@Property`关键字或者类型，则需要具有 `def`关键字。例如，清单 2 的代码在 `toString`方法中包含一个中间变量 `numstr`，如果用 JSR 解析器运行此代码，则会造成一个错误：

##### 清单 2\. 不要忘记 def 关键字！

```java
 class Person { 
  @Property fname 
  @Property lname 
  @Property age 
  @Property address 
  @Property contactNumbers 
  String toString(){ 

   numstr = new StringBuffer() 

   if (contactNumbers != null){ 
       contactNumbers.each{ 
         numstr.append(it) 
         numstr.append(" ") 
       } 
   } 

   "first name: " + fname + " last name: " + lname + 
    " age: " + age + " address: " + address + 
    " contact numbers: " + numstr.toString() 
 } 
 } 
```

认识这个代码吗？它借用来自“在 Java 应用程序中加一些 Groovy 进来”一文的清单 1 中的代码。在清单 3 中，可以看到运行代码时弹出的错误消息：

##### 清单 3\. 错误消息

```java
 c:\dev\projects>groovy BusinessObjects.groovy 

 BusinessObjects.groovy: 13: The variable numstr is undefined in the current scope 
 @ line 13, column 4\. 
      numstr = new StringBuffer() 
      ^ 
 1 Error 
```

当然，解决方案也是在 `toString`方法中将 `def`关键字添加到 `numstr`。清单 4 显示了这个更合适的 *def*解决方案。

##### 清单 4\. 用 def 重新处理

```java
 String toString(){ 

   def numstr = new StringBuffer() 

   if (contactNumbers != null){ 
       contactNumbers.each{ 
         numstr.append(it) 
         numstr.append(" ") 
       } 
   } 

   "first name: " + fname + " last name: " + lname + 
    " age: " + age + " address: " + address + 
    " contact numbers: " + numstr.toString() 
 } 
```

另外，我还可以为 `numstr`提供一个像 `private`这样的修饰符，或者将它声明为 `StringBuffer`。不论哪种方法，重要的一点是：在 JSR Groovy 中，必须在变量前加上 *某些东西*。

* * *

## 封闭闭包（closure）

闭包的语法发生了变化，但是大多数变化只与参数有关。在经典的 Groovy 中，如果为闭包声明参数，就必须用 `|`字符作为分隔符。您可能知道，`|`也是普通 Java 语言中的位操作符；结果，在 Groovy 中，只有在某个闭包的参数声明的上下文中，才能使用 `|`字符。

在“alt.lang.jre: 感受 Groovy”的清单 21 中，我演示了迭代，查看了用于闭包的经典 Groovy 参数语法。可以回想一下，我在集合上运用了 `find`方法，该方法试图找到值 *3*。然后我传入了参数 `x`，它代表 `iterator`的下一个值 （有经验的 *Groovy*开发人员会注意到，`x`完全是可选的，我可以引用隐式变量 `it`）。使用 JSR Groovy 时，必须删除 `|`，并用 Nice *样式的*`-&gt;`分隔符代替它，如清单 5 所示：

##### 清单 5\. 新的 Groovy 闭包语法

```java
 [2, 4, 6, 8, 3].find { x -> 
  if (x == 3){ 
    println "found ${x}"
  } 
 } 
```

新的闭包语法有没有让您想起 Nice 语言的块语法呢？如果不熟悉 Nice 语言，请参阅 alt.lang.jre: Nice 的双倍功能，这是我在 *alt.lang.jre*系列上贡献的另一篇文章。

正如我在前面提到过的，Groovy 的 JDK 没有变。但是就像刚才所学到的，闭包却发生了变化；所以，使用 Groovy 的 JDK 中那些漂亮的 *API*的方式也发生了变化，但仅仅是轻微的变化。在清单 6 中，可以看到这些变化对 Groovy IO 的影响根本是微不足道的：

##### 清单 6\. Groovy 的 JDK 依旧功能强大！

```java
 import java.io.File 
 new File("maven.xml").eachLine{ line -> 
  println "read the following line -> " + line 
 } 
```

### 改编过滤器

现在，不得不让您跳过很大一部分，但您还记得在“用 Groovy 进行 Ant 脚本编程”一文中，我是如何介绍闭包的威力和工具的吗？谢天谢地，我在这个专栏的示例中所做的多数工作都很容易针对新语法重新进行改编。在清单 7 中，我只是将 `@Property`属性添加到 `Filter`的成员 `strategy`（最初在那篇文章的清单 2 和清单 3 中显示）。然后在闭包中添加 `-&gt;`分隔符，*万岁*—— 它可以工作了！

##### 清单 7\. 过滤改编！

```java
 package com.vanward.groovy 
 class Filter{ 
 @Property strategy 
 boolean apply(str){ 
  return strategy.call(str) 
 } 
 } 
 simplefilter = { str -> 
  if(str.indexOf("java.") >= 0){ 
    return true 
  }else{ 
    return false 
  } 
 } 

 fltr = new Filter(strategy:simplefilter) 
 assert !fltr.apply("test") 
 assert fltr.apply("java.lang.String") 

 rfilter = { istr -> 
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

目前为止还不坏，您觉得呢？新的 Groovy 语法很容易掌握！

* * *

## 对范围（range）的更改

Groovy 的范围语法的变化非常小。在经典的 Groovy 中，您可以通过使用 `...`语法指明排他性（即上界）来避开这些变化。在 JSR Groovy 中，只要去掉最后一个点（`.`），并用直观的 `&lt;`标识替代它即可。

请注意观察我在下面的清单 8 中对来自“Feeling Groovy”一文中的范围示例进行的改编：

##### 清单 8\. 新的范围语法

```java
 myRange = 29..<32 
 myInclusiveRange = 2..5 
 println myRange.size() // still prints 3 
 println myRange[0]   // still prints 29 
 println myRange.contains(32) //  still prints false 
 println myInclusiveRange.contains(5) // still prints true 
```

* * *

## 您是说存在歧义？

您可能注意到，在使用 Groovy 时，有一项微妙的功能可以让您获得方法引用，并随意调用这个引用。可以将方法指针当作调用对象方法的方便机制。关于方法指针，有意思的事情是：它们的使用可能就表明代码违反了迪米特法则。

您可能会问“什么是迪米特法则”呢？迪米特法则使用 *只与直接朋友对话*这句格言指出：我们应当避免调用由另一个对象方法返回的对象上的方法。例如，如果 `Foo`对象公开了一个 `Bar`对象类型，那么客户应当 *通过 `Foo`*访问 `Bar`的行为。结果可能是一些脆弱的代码，因为对某个对象的更改会传播到整个范围。

一位受尊敬的学者写了一篇优秀的文章，叫做“The Paperboy, the Wallet, and the Law of Demeter”（请参阅 参考资料）。这篇文章中的示例是用 Java 语言编写的；但是，我在下面用 Groovy 重新定义了这些示例。在清单 9 中，可以看到这些代码演示了迪米特法则 —— 如何用它洗劫人们的钱包！

##### 清单 9\. 迪米特在行动（同情啊，同情！）

```java
 package com.vanward.groovy 
 import java.math.BigDecimal 
 class Customer { 
 @Property firstName 
 @Property lastName 
 @Property wallet 
 } 
 class Wallet { 
 @Property value; 
 def getTotalMoney() { 
  return value; 
 } 

 def setTotalMoney(newValue) { 
  value = newValue; 
 } 
 def addMoney(deposit) { 
  value = value.add(deposit) 
 } 
 def subtractMoney(debit) { 
  value = value.subtract(debit) 
 } 
 } 
```

在清单 9 中，有两个定义的类型 —— 一个 `Customer`和一个 `Wallet`。请注意 `Customer`类型公开自己的 `wallet`实例的方式。正如前面所说的，代码简单的公开方式存在问题。例如，如果我（像原文章的作者所做的那样）添加了一个坏报童，去抢劫那些不知情客户的钱包，又会怎么样？在清单 10 中，我使用了 Groovy 的方法指针来做这件坏事。请注意我是如何使用 Groovy 新的 `&`方法指针语法，通过 `Customer`实例夺取对 `subtractMoney`方法的引用。

##### 清单 10\. 添加坏报童 ...

```java
iwallet = new Wallet(value:new BigDecimal(32)) 
victim = new Customer(firstName:"Lane", lastName:"Meyer", wallet:iwallet) 
//Didn't *ask* for a dime. Two Dollars.
victim.getWallet().subtractMoney(new BigDecimal("0.10")) 
//paperboy turns evil by snatching a reference to the subtractMoney method
mymoney = victim.wallet.&subtractMoney 
mymoney(new BigDecimal(2)) // "I want my 2 dollars!"
mymoney(new BigDecimal(25)) // "late fees!" 
```

现在，不要误会我：方法指针不是为了侵入代码或者获得对人们现金的引用！方法指针只是一种方便的机制。方法指针也很适合于重新连接上您喜爱的 80 年代的老电影！但是，如果您把这些可爱的、漂亮的东西弄湿了，那么它们可就帮不上您的忙了。严格地说，可以把 Groovy 的 `println`快捷方式当作 `System.out.println`的隐式方法指针。

如果一直都很留心，那么您可能已经注意到，JSR Groovy 要求我使用新的 `&`语法来创建 `subtractMoney`方法的指针。您可能已经猜到，这个添加消除了经典 Groovy 中的歧义性。

* * *

## 一些新东西！

如果在 Groovy 的 JSR 发行版中没有什么 *新东西*，那就没有意思了，不是吗？谢天谢地，JSR Groovy 引入了 `as`关键字，它是一个方便的类型转换机制。这个特性与新的对象创建语法关系密切，新的语法可以用类似数组的语法很容易地在 Groovy 中创建 *非定制*类。所谓非定制，指的是在 JDK 中可以找到的类，例如 `Color`、`Point`、`File`，等等。

在清单 11 中，我用这个新语法创建了一些简单类型：

##### 清单 11\. Groovy 中的新语法

```java
 def nfile = ["c:/dev", "newfile.txt"] as File 
 def val = ["http", "www.vanwardtechnologies.com", "/"] as URL 
 def ival = ["89.90"] as BigDecimal 
 println ival as Float 
```

注意，我用便捷语法创建了一个新 `File`和 `URL`，还有 `BigDecimal`，还要注意的是，我可以用 `as`把 `BigDecimal`类型转换成 `Float`类型。

* * *

## 接下来是什么呢？

JSR 对 Groovy 的规范化过程并没有结束，特别是在有些东西在 Groovy 的当前版本中（在本文发布时是 JSR-2）仍然不起作用的情况下。例如，在新的 Groovy 中，不能用 `do`/`while`循环。此外，新的 Groovy 还无法完全支持 Java 5.0 的 `for`循环概念。结果是，可以使用 `in`语法，但是不能使用新推出的 `:`语法。

这些都是重要的特性，不能没有，但是不用担心 —— Groovy 小组正在努力工作，争取在未来几个月内实现它们。请参阅 参考资料，下载最新的发行版本，并学习更多关于 JSR Groovy 进程的内容；还请继续关注下个月的“*实战 Groovy*”，下个月我（和两个客座专栏作家）将深入讨论 Groovy 闭包的更精彩的细节。