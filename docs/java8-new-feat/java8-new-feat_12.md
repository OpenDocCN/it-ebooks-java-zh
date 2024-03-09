# 十二、Nashorn：新犀牛

# 十二、Nashorn ：新犀牛

> 来源：[Java 8 新特性探究（十二）Nashorn ：新犀牛](http://my.oschina.net/benhaile/blog/290276)

### Nashorn 是什么

Nashorn，发音“nass-horn”,是德国二战时一个坦克的命名，同时也是 java8 新一代的 javascript 引擎–替代老旧，缓慢的[Rhino](http://www.oschina.net/p/rhino)，符合 [ECMAScript-262 5.1](http://www.oschina.net/p/ecmascript) 版语言规范。你可能想 javascript 是运行在 web 浏览器，提供对 html 各种 dom 操作，但是 Nashorn 不支持浏览器 DOM 的对象。这个需要注意的一个点。

### 关于 Nashorn 的入门

主要是两个方面，jjs 工具以及 javax.script 包下面的 API：

jjs 是在 java_home/bin 下面自带的，作为例子，让我们创建一个 func.js， 内容如下：

```java
function f(){return 1;};
print(f() + 1); 
```

运行这个文件，把这个文件作为参数传给 jjs

```java
jjs func.js 
```

输出结果：2

另一个方面是 javax.script，也是以前 Rhino 余留下来的 API

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName( "JavaScript" );
System.out.println( engine.getClass().getName() );
System.out.println( "Result:" + engine.eval( "function f() { return 1; }; f() + 1;" ) ); 
```

输出如下：

jdk.nashorn.api.scripting.NashornScriptEngine

Result: 2

基本用法也可以去[`my.oschina.net/jsmagic/blog/212455`](http://my.oschina.net/jsmagic/blog/212455) 这篇博文参考一下；

### Nashorn VS Rhino

javascript 运行在 jvm 已经不是新鲜事了，Rhino 早在 jdk6 的时候已经存在，但现在为何要替代 Rhino，官方的解释是 Rhino 相比其他 javascript 引擎（比如 google 的 V8）实在太慢了，要改造 Rhino 还不如重写。既然性能是 Nashorn 的一个亮点，下面就测试下性能对比，为了对比两者之间的性能，需要用到[Esprima](http://www.oschina.net/search?scope=project&q=esprima)，一个 ECMAScript 解析框架，用它来解析未压缩版的 jquery（大约 268kb），测试核心代码如下：

```java
static void rhino(String parser, String code) {
        String source = "speedtest";
        int line = 1;
        Context context = Context.enter();
        context.setOptimizationLevel(9);
        try {
            Scriptable scope = context.initStandardObjects();
            context.evaluateString(scope, parser, source, line, null);
            ScriptableObject.putProperty(scope, "$code", Context.javaToJS(code, scope));
            Object tree = new Object();
            Object tokens = new Object();
            for (int i = 0; i < RUNS; ++i) {
                long start = System.nanoTime();
                tree = context.evaluateString(scope, "esprima.parse($code)", source, line, null);
                tokens = context.evaluateString(scope, "esprima.tokenize($code)", source, line, null);
                long stop = System.nanoTime();
                System.out.println("Run #" + (i + 1) + ": " + Math.round((stop - start) / 1e6) + " ms");
            }
        } finally {
            Context.exit();
            System.gc();
        }
    }
    static void nashorn(String parser, String code) throws ScriptException,NoSuchMethodException {
        ScriptEngineManager factory = new ScriptEngineManager();
        ScriptEngine engine = factory.getEngineByName("nashorn");
        engine.eval(parser);
        Invocable inv = (Invocable) engine;
        Object esprima = engine.get("esprima");
        Object tree = new Object();
        Object tokens = new Object();
        for (int i = 0; i < RUNS; ++i) {
            long start = System.nanoTime();
            tree = inv.invokeMethod(esprima, "parse", code);
            tokens = inv.invokeMethod(esprima, "tokenize", code);
            long stop = System.nanoTime();
            System.out.println("Run #" + (i + 1) + ": " + Math.round((stop - start) / 1e6) + " ms");
        }
        // System.out.println("Data is " + tokens.toString() + " and " + tree.toString());
    } 
```

从代码可以看出，测试程序将执行 Esprima 的 parse 和 tokenize 来运行测试文件的内容，Rhino 和 Nashorn 分别执行 30 次，在开始时候，Rhino 需要 1726 ms 并且慢慢加速，最终稳定在 950ms 左右，Nashorn 却有另一个特色，第一次运行耗时 3682ms，但热身后很快加速，最终每次运行稳定在 175ms，如下图所示

![](img/525bb8ff2913a05ba77289d3df0ce8d4.png)

nashorn 首先编译 javascript 代码为 java 字节码，然后运行在 jvm 上，底层也是使用[invokedynamic](https://jcp.org/en/jsr/detail?id=292)命令来执行，所以运行速度很给力。

### **为何要用 java 实现 javascript**

这也是大部分同学关注的点，我认同的观点是：

1.  成熟的 GC
2.  成熟的 JIT 编译器
3.  多线程支持
4.  丰富的标准库和第三方库

总得来说，充分利用了 java 平台的已有资源。

### **总结**

新犀牛可以说是犀牛式战车，比 Rhino 速度快了许多，作为高性能的 javascript 运行环境，Nashorn 有很多可能。

举例， Avatar.js 是依赖于 Nashorn 用以支持在 JVM 上实现 Node.js 编程模型，另外还增加了其他新的功能，如使用一个内建的负载平衡器实现多事件循环，以及使用多线程实现轻量消息传递机制；Avatar 还提供了一个 Model-Store, 基于 JPA 的纯粹的 JavaScript ORM 框架。

在企业中另外一种借力 Nashorn 方式是脚本，相比通常我们使用 Linux 等 shell 脚本，现在我们也可以使用 Javascript 脚本和 Java 交互了，甚至使用 Nashorn 通过 REST 接口来监视服务器运行状况。

本文代码地址是：[`git.oschina.net/benhail/javase8-sample`](http://git.oschina.net/benhail/javase8-sample)