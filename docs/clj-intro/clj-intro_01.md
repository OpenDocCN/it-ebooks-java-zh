# 简介

## 简介

这篇文章的目的是以通俗易懂的方式引导大家进入 Clojure 的世界。文章涵盖了 cojure 的大量的特性， 对每一个特性的介绍我力求简介。你不用一条一条往下看，尽管跳到你感兴趣的条目。

请把你的意见，建议发送到 mark@ociweb.com(如果是对文章翻译的建议，请直接在文章下面留言: [`xumingming.sinaapp.com/302/clojure-tutorial/`](http://xumingming.sinaapp.com/302/clojure-tutorial/) )。我对下面这样的建议特别感兴趣:

*   你说是 X, 其实是 Y
*   你说是 X, 但其实说 Y 会更贴切
*   你没有提到 X, 但是我认为 X 是一个非常重要的话题

对这篇文章的更新可以在 [`www.ociweb.com/mark/clojure/`](http://www.ociweb.com/mark/clojure/) 找到， 同时你也可以在 [`www.ociweb.com/mark/stm/`](http://www.ociweb.com/mark/stm/) 找到有关 Software Transactional Memory 的介绍， 以及 Clojure 对 STM 的实现。

这篇文章里面的代码示例里面通常会以注释的形式说明每行代码的结果/输出，看下面的例子：

```java
(+ 1 2) ; showing return value: 3
(println "Hello") ; return nil, showing output:Hello 
```