# 注解（Annotations）

注解为程序提供元数据，但是，它不是程序的一部分。它们不会直接影响在注解的代码操作。

注解有如下的使用场景：

*   编译器信息— 编译器用注解检测到错误或抑制警告。
*   编译时和部署时的处理 — 软件工具可以处理注释的信息来生成代码，XML 文件，等等。
*   运行时处理 — 有些注解是在运行时进行检查.

## 注解的格式

格式如下：

```java
@Entity 
```

符号`@`告诉编译器这是个注解。

注解可以包含有名字或者没有名字的元素（elements），如：

```java
@Author(
   name = "Benjamin Franklin",
   date = "3/27/2003"
)
class MyClass() { ... } 
```

或者

```java
@SuppressWarnings(value = "unchecked")
void myMethod() { ... } 
```

当只有一个元素名字是 value 时，该名字可以省略，如：

```java
@SuppressWarnings("unchecked")
void myMethod() { ... } 
```

若注解没有元素，则连圆括号都可以省略。

同一个声明可以用多个注解：

```java
@Author(name = "Jane Doe")
@EBook
class MyClass { ... } 
```

若注解包含相同的类型，则被称为重复注解(repeating annotation)：

```java
@Author(name = "Jane Doe")
@Author(name = "John Smith")
class MyClass { ... } 
```

重复注解是 Java SE 8 里面支持的。

## 注解使用的地方

注解可以应用到程序声明的类，字段，方法，和其他程序元素。当在一个声明中使用，按照惯例，每个注解经常会出现在它自己的行。

Java SE8 开始，注解也可以应用于类型使用（type use），称为类型注解（type annotation）。 这里有些例子：

*   类实例创建表达式

```java
new @Interned MyObject() 
```

*   类型投射

```java
myString = (@NonNull String) str; 
```

*   实现条款

```java
class UnmodifiableList<T> implements
        @Readonly List<@Readonly T> { ... } 
```

*   抛出异常声明

```java
void monitorTemperature() throws
        @Critical TemperatureException { ... } 
```

## 声明一个注解类型

许多注解取代了本来已经在代码中的注释。

假设传统的软件组在每个类的类体的开始，使用注释提供了重要的信息：

```java
public class Generation3List extends Generation2List {

   // Author: John Doe
   // Date: 3/17/2002
   // Current revision: 6
   // Last modified: 4/12/2004
   // By: Jane Doe
   // Reviewers: Alice, Bill, Cindy

   // class code goes here

} 
```

使用注解提供一样的元数据，首先要声明一个注解类型，语法是：

```java
@interface ClassPreamble {
   String author();
   String date();
   int currentRevision() default 1;
   String lastModified() default "N/A";
   String lastModifiedBy() default "N/A";
   // Note use of array
   String[] reviewers();
} 
```

注解的声明，就像在 interface 声明前面添加一个`@`字符(`@`是 AT,即 Annotation Type)。注解类型，其实是接口的一种形式，后面会讲到。就目前而言，你不需要了解。

注解的声明的正文，包括注解元素的声明，看起来很像方法。注意，这里可以定义可选的默认值。

一旦注解定义好了，就可以在使用注解时，填充注解的值，就像这样：

```java
@ClassPreamble (
   author = "John Doe",
   date = "3/17/2002",
   currentRevision = 6,
   lastModified = "4/12/2004",
   lastModifiedBy = "Jane Doe",
   // Note array notation
   reviewers = {"Alice", "Bob", "Cindy"}
)
public class Generation3List extends Generation2List {

// class code goes here

} 
```

**注：**要让`@ClassPreamble`的信息出现在 Javadoc 生成的文档，必须使用`@Documented`注解定义`@ClassPreamble`

```java
// import this to use @Documented
import java.lang.annotation.*;

@Documented
@interface ClassPreamble {

   // Annotation element definitions

} 
```

## 预定义注解的类型

有这么几种注解类型预定义在 Java SE API 了。一些注解类型是供 Java 编译器使用，一些是供其他注解使用。

### Java 语言使用的注解

定义在 java.lang 中的是 `@Deprecated`, `@Override`, 和 `@SuppressWarnings`

`@Deprecated`注解指示，标识的元素是废弃的(deprecated)，不应该再使用。编译器会在任何使用到`@Deprecated`的类，方法，字段的程序时产生警告。当元素是废弃的，它也应该使用 Javadoc 的 `@deprecated` 标识文档化，如下面的例子。两个 Javadoc 注释和注解中的“@”符号的使用不是巧合 - 它们是相关的概念上。另外，请注意 Javadoc 标记开始用小写字母“d”和注解开始以大写字母“D”。

```java
// Javadoc comment follows
    /**
     * @deprecated
     * explanation of why it was deprecated
     */
    @Deprecated
    static void deprecatedMethod() { }
} 
```

`@Override`注解通知编译器，覆盖父类声明的元素。

```java
// mark method as a superclass method
// that has been overridden
@Override 
int overriddenMethod() { } 
```

虽然不要求在覆盖方法时，必须使用注解，但是它可以避免错误。如果一个方法标记为`@Override`，但是无法正确覆盖父类的任何方法，编译器会产生错误。

`@SuppressWarnings`告诉编译器，抑制正常情况下会产生的特定的警告。下面的例子，一个废弃的方法被使用，编译器正常会产生警告，而这个情况下，这个注解导致警告会被抑制。

```java
// use a deprecated method and tell 
// compiler not to generate a warning
@SuppressWarnings("deprecation")
void useDeprecatedMethod() {
    // deprecation warning
    // - suppressed
    objectOne.deprecatedMethod();
} 
```

每个编译器的警告属于一个类别。Java 语言规范有两个类别："deprecation" 和"unchecked"。"unchecked" 会在使用以前的写的泛型的遗留代码进行交互时，产生警告。抑制更多类别的警告，使用下面的语法：

```java
@SuppressWarnings({"unchecked", "deprecation"}) 
```

`@SafeVarargs`注解，当应用于方法或构造，断言代码不对其可变参数（varargs）的参数进行潜在的不安全操作。当使用这个注释类型时，与可变参数相关未检查警告被抑制。

`@FunctionalInterface`是在 Java SE 8 中引入，由 Java 语言规范定义的那样，表示该类型声明意在成为功能性的接口。

### 注解应用于其他注解

注解应用于其他注解称为元注解（ meta-annotations）。java.lang.annotation 中定义了多种元注解。

`@Retention` 注解指定了标记的注解如何存储：

*   RetentionPolicy.SOURCE - 该标记注解只保留在源码级，而由编译器忽略。
*   RetentionPolicy.CLASS - 该标记注释是由编译器在编译时保留，但由 Java 虚拟机（JVM）忽略。
*   RetentionPolicy.RUNTIME - 该标记注解由 JVM 保留，因此可以使用在运行时环境。

`@Documented`注释表明，只要指定哪些元素应该使用 Javadoc 工具。 （默认情况下，注解不包括在 Javadoc 中。）有关详细信息，请参阅的 [Javadoc 工具页面](https://docs.oracle.com/javase/8/docs/technotes/guides/javadoc/index.html)。

`@Target` 用于标记其他注解，限制什么样的 Java 元素的注解可以应用到。`@Target` 注解指定以下元素类型作为其值之一：

*   ElementType.ANNOTATION_TYPE 可以应用于注释类型。
*   ElementType.CONSTRUCTOR 可以应用于构造体。
*   ElementType.FIELD 可以应用于一个字段或属性。
*   ElementType.LOCAL_VARIABLE 可以应用到局部变量。
*   ElementType.METHOD 可以应用于一方法级注释。
*   ElementType.PACKAGE 可以应用到一个包声明。
*   ElementType.PARAMETER 可以应用于方法的参数。
*   ElementType.TYPE 可以应用于类的任意元素。

`@Inherited` 指示注释类型可以从超类继承。（默认不是 true）。当用户查询注释类型,类没有这种类型注释，此时从这个类的父类中查询注释类型。这个注释只适用于类的声明。

`@Repeatable`注解，在 Java SE 8 中引入的，表示该标记的注解可以多次应用到同一声明或类型使用。欲了解更多信息，请参阅重复注解。

## 类型注解以及可拔插的类型系统

Java SE8 之前，注解只能用于声明，从 Java SE8 开始，注解也可以应用于类型使用（type use），称为类型注解（type annotation）。意味着，注解可以使用在任何使用的类型。

类型注解为 Java 程序提供了更强的类型检查分析。Java SE 8 版本不提供类型检查的框架，但它可以让你自己写（或下载）类型检查框架，该框架实现了与 Java 编译器一起使用的一个或多个可插拔模块。

例如，要确保在你的程序中一个特定变量从未被分配到 null ,从而避免引发 NullPointerException 异常。您可以编写自定义插件来检查这一点。然后，您可以修改代码以注明这个特定变量，以表明它是永远不会分配给 null。变量声明可能是这样的：

```java
@NonNull String str; 
```

当您编译代码，包括在命令行中的 NonNull 模块，如果它检测到潜在的问题，编译器输出警告，让您可以修改代码以避免错误。在更正代码后，消除所有警告，当程序运行时不会发生此特定错误。

您可以使用多个类型检查的模块，每个模块检查不同类型的错误。通过这种方式，你可以建立在 Java 类型系统之上，随时随地添加您想要的特定检查。

通过明智地使用类型注解和可插拔的类型检查器，你写的代码，将更强大，更不易出错。

在很多情况下，你不必写自己的类型检查模块。第三方组织已经在做这个工作了。例如，华盛顿大学(the University of Washington)创建的 Checker Framework 。该框架包括一个 NonNull 模块，以及一个正则表达式模块和互斥锁模块。欲了解更多信息，请参见 [Checker Framework](http://types.cs.washington.edu/checker-framework/)。

## 重复注解

若注解包含相同的类型，则被称为重复注解(repeating annotation)，这个是 Java SE 8 之后所支持的。

比如，你正在编写的代码使用计时器服务，使您能够在特定的时间或在某个计划，类似于 UNIX cron 服务运行的方法。现在，你要设置一个计时器，在下午 11:00 运行的方法，doPeriodicCleanup，在每月和每周五的最后一天要设置定时运行，创建一个`@Schedule`注释，并两次将其应用到了 doPeriodicCleanup 方法。在第一次使用指定月的最后一天和第二指定星期五在下午 11 点，使用如下：

```java
@Schedule(dayOfMonth="last")
@Schedule(dayOfWeek="Fri", hour="23")
public void doPeriodicCleanup() { ... } 
```

上面的示例是将注解应用在方法上。你可以在任何使用标准的注解地方使用重复注解。例如，你有一个类来处理未授权的访问异常。有一个`@Alert`注解的类标注为管理人员和另一个用于管理员：

```java
@Alert(role="Manager")
@Alert(role="Administrator")
public class UnauthorizedAccessException extends SecurityException { ... } 
```

由于兼容性的原因，重复的注释被存储在一个由 Java 编译器自动产生的容器注解（container annotation）里。为了使编译器要做到这一点，你的代码里两个声明都需要。

### 第一步：声明一个重复注解

重复注解用 @Repeatable 元注解标记。下面例子定义一个自定义的 @Schedule 重复注解：

```java
import java.lang.annotation.Repeatable;

@Repeatable(Schedules.class)
public @interface Schedule {
  String dayOfMonth() default "first";
  String dayOfWeek() default "Mon";
  int hour() default 12;
} 
```

`@Repeatable`元注解的值是由 Java 编译器生成存储重复注解的容器注解的类型。在本例中，容器注解的类型是 Schedules，所以重复注解 `@Schedule` 被存储在`@Schedules` 注解中。

应用相同注解到声明但没有首先声明它是可重复的，则在编译时会出错。

### 步骤 2：声明容器注解类型

容器注解类型必须有数组类型的元素 value,而数组类型的组件类型必须是重复注解类型，示例如下：

```java
public @interface Schedules {
    Schedule[] value();
} 
```

### 检索注解

反射 API 有几种方法可用于检索注解。返回单个注解的方法的行为，如[AnnotatedElement.getAnnotationByType(Class<t class="hljs-class">)</t>](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html#getAnnotationByType-java.lang.Class-)，如果所请求的类型的注解类型只存在一个则仅返回一个注解，则该行为是没有改变的。如果有多个请求类型的注解类型存在，则可以通过先得到他们的容器注解从而获取它们。这种方式下，传统代码继续工作。其他方法是在 Java SE 8 中，通过容器注释扫描到一次返回多个注解，如 [AnnotatedElement.getAnnotations(Class<t class="hljs-class">)</t>](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html#getAnnotations-java.lang.Class-)。见[AnnotatedElement](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/AnnotatedElement.html) 类的规范，查看所有的可用方法的信息。

### 设计考虑

当设计一个注解类型，你必须考虑到该类型的注解的基数(cardinality)。现在可以使用一个注释零次，一次，或者，如果注解的类型被标以`@Repeatable`，则不止一次。另外，也可以通过使用`@Target`元注解来限制注解类型在哪里使用。例如，您可以创建一个只能在方法和字段使用可重复的注解类型。精心设计的注解类型是非常重要的，要确保使用注解的程序员感觉越灵活和强大越好。