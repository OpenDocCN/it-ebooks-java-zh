# Java 8 Nashorn 教程

# Java 8 Nashorn 教程

> 原文：[Java 8 Nashorn Tutorial](http://winterbe.com/posts/2014/04/05/java8-nashorn-tutorial/)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)

这个教程中，你会通过简单易懂的代码示例，来了解 Nashorn JavaScript 引擎。Nashorn JavaScript 引擎是 Java SE 8 的一部分，并且和其它独立的引擎例如[Google V8](https://code.google.com/p/v8/)（用于 Google Chrome 和[Node.js](http://nodejs.org/)的引擎）互相竞争。Nashorn 通过在 JVM 上，以原生方式运行动态的 JavaScript 代码来扩展 Java 的功能。

在接下来的 15 分钟内，你会学到如何在 JVM 上在运行时动态执行 JavaScript。我会使用小段代码示例来演示最新的 Nashron 语言特性。你会学到如何在 Java 代码中调用 JavaScript 函数，或者相反。最后你会准备好将动态脚本集成到你的 Java 日常业务中。

![](img/6340742c.jpg)

更新 - 我现在正在编写用于浏览器的 Java8 数据流 API 的 JavaScript 实现。如果你对此感兴趣，请在 Github 上访问[Stream.js](https://github.com/winterbe/streamjs)。非常期待你的反馈。

## 使用 Nashron

Nashorn JavaScript 引擎可以在 Java 代码中编程调用，也可以通过命令行工具`jjs`使用，它在`$JAVA_HOME/bin`中。如果打算使用`jjs`，你可能希望设置符号链接来简化访问：

```
$ cd /usr/bin
$ ln -s $JAVA_HOME/bin/jjs jjs
$ jjs
jjs> print('Hello World'); 
```

这个教程专注于在 Java 代码中调用 Nashron，所以让我们先跳过`jjs`。Java 代码中简单的 HelloWorld 如下所示：

```
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval("print('Hello World!');"); 
```

为了在 Java 中执行 JavaScript，你首先要通过`javax.script`包创建脚本引擎。这个包已经在[Rhino](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino)（来源于 Mozilla、Java 中的遗留 JS 引擎）中使用了。

JavaScript 代码既可以通过传递 JavaScript 代码字符串，也可以传递指向你的 JS 脚本文件的`FileReader`来执行：

```
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("script.js")); 
```

Nashorn JavaScript 基于[ECMAScript 5.1](http://es5.github.io/)，但是它的后续版本会对 ES6 提供支持：

> Nashorn 的当前策略遵循 ECMAScript 规范。当我们在 JDK8 中发布它时，它将基于 ECMAScript 5.1。Nashorn 未来的主要发布基于[ECMAScript 6](http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts)。

Nashorn 定义了大量对 ECMAScript 标准的语言和 API 扩展。但是首先让我们看一看 Java 和 JavaScript 代码如何交互。

## 在 Java 中调用 JavaScript 函数

Nashorn 支持从 Java 代码中直接调用定义在脚本文件中的 JavaScript 函数。你可以将 Java 对象传递为函数参数，并且从函数返回数据来调用 Java 方法。

下面的 JavaScript 函数稍后会在 Java 端调用：

```
var fun1 = function(name) {
    print('Hi there from Javascript, ' + name);
    return "greetings from javascript";
};

var fun2 = function (object) {
    print("JS Class Definition: " + Object.prototype.toString.call(object));
}; 
```

为了调用函数，你首先需要将脚本引擎转换为`Invocable`。`Invocable`接口由`NashornScriptEngine`实现，并且定义了`invokeFunction`方法来调用指定名称的 JavaScript 函数。

```
ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");
engine.eval(new FileReader("script.js"));

Invocable invocable = (Invocable) engine;

Object result = invocable.invokeFunction("fun1", "Peter Parker");
System.out.println(result);
System.out.println(result.getClass());

// Hi there from Javascript, Peter Parker
// greetings from javascript
// class java.lang.String 
```

执行这段代码会在控制台产生三行结果。调用函数`print`将结果输出到`System.out`，所以我们会首先看到 JavaScript 输出。

现在让我们通过传入任意 Java 对象来调用第二个函数：

```
invocable.invokeFunction("fun2", new Date());
// [object java.util.Date]

invocable.invokeFunction("fun2", LocalDateTime.now());
// [object java.time.LocalDateTime]

invocable.invokeFunction("fun2", new Person());
// [object com.winterbe.java8.Person] 
```

Java 对象在传入时不会在 JavaScript 端损失任何类型信息。由于脚本在 JVM 上原生运行，我们可以在 Nashron 上使用 Java API 或外部库的全部功能。

## 在 JavaScript 中调用 Java 方法

在 JavaScript 中调用 Java 方法十分容易。我们首先需要定义一个 Java 静态方法。

```
static String fun1(String name) {
    System.out.format("Hi there from Java, %s", name);
    return "greetings from java";
} 
```

Java 类可以通过`Java.type`API 扩展在 JavaScript 中引用。它就和 Java 代码中的`import`类似。只要定义了 Java 类型，我们就可以自然地调用静态方法`fun1()`，然后像`sout`打印信息。由于方法是静态的，我们不需要首先创建实例。

```
var MyJavaClass = Java.type('my.package.MyJavaClass');

var result = MyJavaClass.fun1('John Doe');
print(result);

// Hi there from Java, John Doe
// greetings from java 
```

在使用 JavaScript 原生类型调用 Java 方法时，Nashorn 如何处理类型转换？让我们通过简单的例子来弄清楚。

下面的 Java 方法简单打印了方法参数的实际类型：

```
static void fun2(Object object) {
    System.out.println(object.getClass());
} 
```

为了理解背后如何处理类型转换，我们使用不同的 JavaScript 类型来调用这个方法：

```
MyJavaClass.fun2(123);
// class java.lang.Integer

MyJavaClass.fun2(49.99);
// class java.lang.Double

MyJavaClass.fun2(true);
// class java.lang.Boolean

MyJavaClass.fun2("hi there")
// class java.lang.String

MyJavaClass.fun2(new Number(23));
// class jdk.nashorn.internal.objects.NativeNumber

MyJavaClass.fun2(new Date());
// class jdk.nashorn.internal.objects.NativeDate

MyJavaClass.fun2(new RegExp());
// class jdk.nashorn.internal.objects.NativeRegExp

MyJavaClass.fun2({foo: 'bar'});
// class jdk.nashorn.internal.scripts.JO4 
```

JavaScript 原始类型转换为合适的 Java 包装类，而 JavaScript 原生对象会使用内部的适配器类来表示。要记住`jdk.nashorn.internal`中的类可能会有所变化，所以不应该在客户端面向这些类来编程。

> 任何标记为“内部”的东西都可能会从你那里发生改变。

## `ScriptObjectMirror`

在向 Java 传递原生 JavaScript 对象时，你可以使用`ScriptObjectMirror`类，它实际上是底层 JavaScript 对象的 Java 表示。`ScriptObjectMirror`实现了`Map`接口，位于`jdk.nashorn.api`中。这个包中的类可以用于客户端代码。

下面的例子将参数类型从`Object`改为`ScriptObjectMirror`，所以我们可以从传入的 JavaScript 对象中获得一些信息。

```
static void fun3(ScriptObjectMirror mirror) {
    System.out.println(mirror.getClassName() + ": " +
        Arrays.toString(mirror.getOwnKeys(true)));
} 
```

当向这个方法传递对象（哈希表）时，在 Java 端可以访问其属性：

```
MyJavaClass.fun3({
    foo: 'bar',
    bar: 'foo'
});

// Object: [foo, bar] 
```

我们也可以在 Java 中调用 JavaScript 的成员函数。让我们首先定义 JavaScript `Person`类型，带有属性`firstName` 和 `lastName`，以及方法`getFullName`。

```
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.getFullName = function() {
        return this.firstName + " " + this.lastName;
    }
} 
```

JavaScript 方法`getFullName`可以通过`callMember()`在`ScriptObjectMirror`上调用。

```
static void fun4(ScriptObjectMirror person) {
    System.out.println("Full Name is: " + person.callMember("getFullName"));
} 
```

当向 Java 方法传递新的`Person`时，我们会在控制台看到预期的结果：

```
var person1 = new Person("Peter", "Parker");
MyJavaClass.fun4(person1);

// Full Name is: Peter Parker 
```

## 语言扩展

Nashorn 定义了多种对 ECMAScript 标准的语言和 API 扩展。让我们看一看最新的特性：

### 类型数组

JavaScript 的原生数组是无类型的。Nashron 允许你在 JavaScript 中使用 Java 的类型数组：

```
var IntArray = Java.type("int[]");

var array = new IntArray(5);
array[0] = 5;
array[1] = 4;
array[2] = 3;
array[3] = 2;
array[4] = 1;

try {
    array[5] = 23;
} catch (e) {
    print(e.message);  // Array index out of range: 5
}

array[0] = "17";
print(array[0]);  // 17

array[0] = "wrong type";
print(array[0]);  // 0

array[0] = "17.3";
print(array[0]);  // 17 
```

`int[]`数组就像真实的 Java 整数数组那样。但是此外，在我们试图向数组添加非整数时，Nashron 在背后执行了一些隐式的转换。字符串会自动转换为整数，这十分便利。

### 集合和范围遍历

我们可以使用任何 Java 集合，而避免使用数组瞎折腾。首先需要通过`Java.type`定义 Java 类型，之后创建新的实例。

```
var ArrayList = Java.type('java.util.ArrayList');
var list = new ArrayList();
list.add('a');
list.add('b');
list.add('c');

for each (var el in list) print(el);  // a, b, c 
```

为了迭代集合和数组，Nashron 引入了`for each`语句。它就像 Java 的范围遍历那样工作。

下面是另一个集合的范围遍历示例，使用`HashMap`：

```
var map = new java.util.HashMap();
map.put('foo', 'val1');
map.put('bar', 'val2');

for each (var e in map.keySet()) print(e);  // foo, bar

for each (var e in map.values()) print(e);  // val1, val2 
```

### Lambda 表达式和数据流

每个人都热爱 lambda 和数据流 -- Nashron 也一样！虽然 ECMAScript 5.1 没有 Java8 lmbda 表达式的简化箭头语法，我们可以在任何接受 lambda 表达式的地方使用函数字面值。

```
var list2 = new java.util.ArrayList();
list2.add("ddd2");
list2.add("aaa2");
list2.add("bbb1");
list2.add("aaa1");
list2.add("bbb3");
list2.add("ccc");
list2.add("bbb2");
list2.add("ddd1");

list2
    .stream()
    .filter(function(el) {
        return el.startsWith("aaa");
    })
    .sorted()
    .forEach(function(el) {
        print(el);
    });
    // aaa1, aaa2 
```

### 类的继承

Java 类型可以由`Java.extend`轻易扩展。就像你在下面的例子中看到的那样，你甚至可以在你的脚本中创建多线程的代码：

```
var Runnable = Java.type('java.lang.Runnable');
var Printer = Java.extend(Runnable, {
    run: function() {
        print('printed from a separate thread');
    }
});

var Thread = Java.type('java.lang.Thread');
new Thread(new Printer()).start();

new Thread(function() {
    print('printed from another thread');
}).start();

// printed from a separate thread
// printed from another thread 
```

### 参数重载

方法和函数可以通过点运算符或方括号运算符来调用：

```
var System = Java.type('java.lang.System');
System.out.println(10);              // 10
System.out"println";         // 11.0
System.out"println(double)";   // 12.0 
```

当使用重载参数调用方法时，传递可选参数类型`println(double)`会指定所调用的具体方法。

### Java Beans

你可以简单地使用属性名称来向 Java Beans 获取或设置值，不需要显式调用读写器：

```
var Date = Java.type('java.util.Date');
var date = new Date();
date.year += 1900;
print(date.year);  // 2014 
```

### 函数字面值

对于简单的单行函数，我们可以去掉花括号：

```
function sqr(x) x * x;
print(sqr(3));    // 9 
```

### 属性绑定

两个不同对象的属性可以绑定到一起：

```
var o1 = {};
var o2 = { foo: 'bar'};

Object.bindProperties(o1, o2);

print(o1.foo);    // bar
o1.foo = 'BAM';
print(o2.foo);    // BAM 
```

### 字符串去空白

我喜欢去掉空白的字符串：

```
print("   hehe".trimLeft());            // hehe
print("hehe    ".trimRight() + "he");   // hehehe 
```

### 位置

以防你忘了自己在哪里：

```
print(__FILE__, __LINE__, __DIR__); 
```

### 导入作用域

有时一次导入多个 Java 包会很方便。我们可以使用`JavaImporter`类，和`with`语句一起使用。所有被导入包的类文件都可以在`with`语句的局部域中访问到。

```
var imports = new JavaImporter(java.io, java.lang);
with (imports) {
    var file = new File(__FILE__);
    System.out.println(file.getAbsolutePath());
    // /path/to/my/script.js
} 
```

### 数组转换

一些类似`java.util`的包可以不使用`java.type`或`JavaImporter`直接访问：

```
var list = new java.util.ArrayList();
list.add("s1");
list.add("s2");
list.add("s3"); 
```

下面的代码将 Java 列表转换为 JavaScript 原生数组：

```
var jsArray = Java.from(list);
print(jsArray);                                  // s1,s2,s3
print(Object.prototype.toString.call(jsArray));  // [object Array] 
```

下面的代码执行相反操作：

```
var javaArray = Java.to([3, 5, 7, 11], "int[]"); 
```

### 访问超类

在 JavaScript 中访问被覆盖的成员通常比较困难，因为 Java 的`super`关键字在 ECMAScript 中并不存在。幸运的是，Nashron 有一套补救措施。

首先我们需要在 Java 代码中定义超类：

```
class SuperRunner implements Runnable {
    @Override
    public void run() {
        System.out.println("super run");
    }
} 
```

下面我在 JavaScript 中覆盖了`SuperRunner`。要注意创建新的`Runner`实例时的 Nashron 语法：覆盖成员的语法取自 Java 的匿名对象。

```
var SuperRunner = Java.type('com.winterbe.java8.SuperRunner');
var Runner = Java.extend(SuperRunner);

var runner = new Runner() {
    run: function() {
        Java.super(runner).run();
        print('on my run');
    }
}
runner.run();

// super run
// on my run 
```

我们通过`Java.super()`扩展调用了被覆盖的`SuperRunner.run()`方法。

### 加载脚本

在 JavaScript 中加载额外的脚本文件非常方便。我们可以使用`load`函数加载本地或远程脚本。

我在我的 Web 前端中大量使用[Underscore.js](http://underscorejs.org/)，所以让我们在 Nashron 中复用它：

```
load('http://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.6.0/underscore-min.js');

var odds = _.filter([1, 2, 3, 4, 5, 6], function (num) {
    return num % 2 == 1;
});

print(odds);  // 1, 3, 5 
```

外部脚本会在相同 JavaScript 上下文中被执行，所以我们可以直接访问 underscore 的对象。要记住当变量名称互相冲突时，脚本的加载可能会使你的代码崩溃。

这一问题可以通过把脚本文件加载到新的全局上下文来绕过：

```
loadWithNewGlobal('script.js'); 
```

## 命令行脚本

如果你对编写命令行（shell）脚本感兴趣，来试一试[Nake](https://github.com/winterbe/nake)吧。Nake 是一个 Java 8 Nashron 的简化构建工具。你只需要在项目特定的`Nakefile`中定义任务，之后通过在命令行键入`nake -- myTask`来执行这些任务。任务编写为 JavaScript，并且在 Nashron 的脚本模式下运行，所以你可以使用你的终端、JDK8 API 和任意 Java 库的全部功能。

对 Java 开发者来说，编写命令行脚本是前所未有的简单...

## 到此为止

我希望这个教程对你有所帮助，并且你能够享受 Nashron JavaScript 引擎之旅。有关 Nashron 的更多信息，请见[这里](http://docs.oracle.com/javase/8/docs/technotes/guides/scripting/nashorn/)、[这里](http://www.oracle.com/technetwork/articles/java/jf14-nashorn-2126515.html)和[这里](https://wiki.openjdk.java.net/display/Nashorn/Nashorn+extensions)。使用 Nashron 编写 shell 脚本的教程请见[这里](http://docs.oracle.com/javase/8/docs/technotes/guides/scripting/nashorn/shell.html#sthref24)。

我最近发布了一篇后续文章，关于如何在 Nashron 中使用 Backbone.js 模型。如果你想要进一步学习 Java8，请阅读我的 Java8 教程，和我的 Java8 数据流教程。

这篇 Nashron 教程中的可运行的源代码托管在[Github](https://github.com/winterbe/java8-tutorial)上。请随意[fork 我的仓库](https://github.com/winterbe/java8-tutorial/fork)，或者在[Twitter](https://twitter.com/winterbe_)上向我反馈。

请坚持编程！