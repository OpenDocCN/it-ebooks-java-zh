# 在 Nashron 中使用 Backbone.js

# 在 Nashron 中使用 Backbone.js

> 原文：[Using Backbone.js with Nashorn](http://winterbe.com/posts/2014/04/07/using-backbonejs-with-nashorn/)
> 
> 译者：[飞龙](https://github.com/wizardforcel)
> 
> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)

这个例子展示了如何在 Java8 的 Nashron JavaScript 引擎中使用 Backbone.js 模型。Nashron 在 2014 年三月首次作为 Java SE 8 的一部分发布，并通过以原生方式在 JVM 上运行脚本扩展了 Java 的功能。对于 Java Web 开发者，Nashron 尤其实用，因为它可以在 Java 服务器上复用现有的客户端代码。传统的[Node.js](http://nodejs.org/)具有明显优势，但是 Nashorn 也能够缩短 JVM 的差距。

当你在 HTML5 前端使用现代的 JavaScript MVC 框架，例如[Backbone.js](http://backbonejs.org/)时，越来越多的代码从服务器后端移动到 Web 前端。这个方法可以极大提升用户体验，因为在使用视图的业务逻辑时节省了服务器的很多往返通信。

Backbone 允许你定义模型类，它们可以用于绑定视图（例如 HTML 表单）。当用户和 UI 交互时 Backbone 会跟踪模型的升级，反之亦然。它也能通过和服务器同步模型来帮助你，例如调用服务端 REST 处理器的适当方法。所以你最终会在前端实现业务逻辑，将你的服务器模型用于处理持久化数据。

在服务端复用 Backbone 模型十分易于用 Nashron 完成，就像下面的例子所展示的那样。在我们开始之前，确保你通过阅读我的 Nashorn 教程熟悉了在 Nashron 引擎中编写 JavaScript。

## Java 模型

首先，我们在 Java 中定义实体类`Product`。这个类可用于数据库的 CURD 操作（增删改查）。要记住这个类是个纯粹的 Java Bean，不实现任何业务逻辑，因为我们想让前端正确执行 UI 的业务逻辑。

```
class Product {
    String name;
    double price;
    int stock;
    double valueOfGoods;
} 
```

## Backbone 模型

现在我们定义 Backbone 模型，作为 Java Bean 的对应。Backbone 模型`Product`使用和 Java Bean 相同的数据结构，因为它是我们希望在 Java 服务器上持久存储的数据。

Backbone 模型也实现了业务逻辑：`getValueOfGoods`方法通过将`stock`与`price`相乘计算所有产品的总值。每次`stock`或`price`的变动都会使`valueOfGoods`重新计算。

```
var Product = Backbone.Model.extend({
    defaults: {
        name: '',
        stock: 0,
        price: 0.0,
        valueOfGoods: 0.0
    },

    initialize: function() {
        this.on('change:stock change:price', function() {
            var stock = this.get('stock');
            var price = this.get('price');
            var valueOfGoods = this.getValueOfGoods(stock, price);
            this.set('valueOfGoods', valueOfGoods);
        });
    },

    getValueOfGoods: function(stock, price) {
        return stock * price;
    }
}); 
```

由于 Backbone 模型不使用任何 Nashron 语言扩展，我们可以在客户端（浏览器）和服务端（Java）安全地使用同一份代码。

要记住我特意选择了十分简单的函数来演示我的意图。真实的业务逻辑应该会更复杂。

## 将二者放在一起

下一个目标是在 Nashron 中，例如在 Java 服务器上复用 Backbone 模型。我们希望完成下面的行为：把所有属性从 Java Bean 上绑定到 Backbone 模型上，计算`valueOfGoods`属性，最后将结果传回 Java。

首先，我们创建一个新的脚本，它仅仅由 Nashron 执行，所以我们这里可以安全地使用 Nashron 的扩展。

```
load('http://cdnjs.cloudflare.com/ajax/libs/underscore.js/1.6.0/underscore-min.js');
load('http://cdnjs.cloudflare.com/ajax/libs/backbone.js/1.1.2/backbone-min.js');
load('product-backbone-model.js');

var calculate = function(javaProduct) {
    var model = new Product();
    model.set('name', javaProduct.name);
    model.set('price', javaProduct.price);
    model.set('stock', javaProduct.stock);
    return model.attributes;
}; 
```

这个脚本首先加载了相关的外部脚本[Underscore](http://underscorejs.org/) 和 [Backbone](http://backbonejs.org/)（Underscore 是 Backbone 的必备条件），以及我们前面的`Product`Backbone 模型。

函数`calcute`接受`Product`Java Bean，将其所有属性绑定到新创建的 Backbone`Product`上，之后返回模型的所有属性给调用者。通过在 Backbone 模型上设置`stock`和`price`属性，`ValueOfGoods`属性由于注册在模型`initialize`构造函数中的事件处理器，会自动计算出来。

最后，我们在 Java 中调用`calculate`函数。

```
Product product = new Product();
product.setName("Rubber");
product.setPrice(1.99);
product.setStock(1337);

ScriptObjectMirror result = (ScriptObjectMirror)
    invocable.invokeFunction("calculate", product);

System.out.println(result.get("name") + ": " + result.get("valueOfGoods"));
// Rubber: 2660.63 
```

我们创建了新的`Product`Java Bean，并且将它传递到 JavaScript 函数中。结果触发了`getValueOfGoods`方法，所以我们可以从返回的对象中读取`valueOfGoods`属性的值。

## 总结

在 Nashron 中复用现存的 JavaScript 库十分简单。Backbone 适用于构建复杂的 HTML5 前端。在我看来，Nashron 和 JVM 现在是 Node.js 的优秀备选方案，因为你可以在 Nashron 的代码库中充分利用 Java 的整个生态系统，例如 JDK 的全部 API，以及所有可用的库和工具。要记住你在使用 Nashron 时并不限制于 Java -- 想想 Scala、Groovy、Clojure 和`jjs`上的纯 JavaScript。

这篇文章中可运行的代码托管在[Github](https://github.com/winterbe/java8-tutorial)上（请见[这个文件](https://github.com/winterbe/java8-tutorial/blob/master/res/nashorn6.js)）。请随意[fork 我的仓库](https://github.com/winterbe/java8-tutorial/fork)，或者在[Twitter](https://twitter.com/winterbe_)上向我反馈。