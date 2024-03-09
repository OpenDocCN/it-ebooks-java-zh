# 库

## 库

很多的类库提供了 Clojure Proper 所没有提供的一些功能， 我们在前面的例子里面已经讨论过一些，下面列举一下没有提到的一些。并且这里有已知的类库的一个列表 [`clojure.org/libraries`](http://clojure.org/libraries) 。

*   clojure.tools.cli - 操作命令行参数并且输出帮助信息
*   clojure.data.xml - 以 lazy 的方式解析 XML
*   clojure.algo.monads - 有关 [monads](http://en.wikipedia.org/wiki/Monad_(functional_programming)) 的一些方法
*   clojure.java.shell - 提供一些函数和宏来创建子进程并且控制它们的输入/输出
*   clojure.stacktrace - 提供函数来简化 stacktrace 的输出 --- 只输出跟 Clojure 有关的东西
*   clojure.string - 提供操作字符串以及正则表达式的一些方法
*   clojure.tools.trace - 提供跟踪所有对某个方法的调用的输出以及返回值的跟踪

下面是个简要的例子要使用 clojure.java.shell 获取当前的工作目录。

```java
(use 'clojure.java.shell)
(def directory (sh "pwd")) 
```