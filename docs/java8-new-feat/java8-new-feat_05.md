# 五、重复注解（repeating annotations）

> 来源：[Java 8 新特性探究（五）重复注解（repeating annotations）](http://my.oschina.net/benhaile/blog/180932)

### **知识回顾**

前面介绍了：

*   lambda 表达式和默认方法 （JEP 126）
*   批量数据操作（JEP 107）
*   类型注解（JEP 104）

注：JEP=JDK Enhancement-Proposal (JDK 增强建议 )，每个 JEP 即一个新特性。

在 java 8 里面，注解一共有 2 个改进，一个是类型注解，在上篇已经介绍了，本篇将介绍另外一个注解的改进：重复注解（JEP 120）。

### **什么是重复注解**

允许在同一申明类型（类，属性，或方法）的多次使用同一个注解

### **一个简单的例子**

java 8 之前也有重复使用注解的解决方案，但可读性不是很好，比如下面的代码：

```java
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseOldVersion {

    @Authorities({@Authority(role="Admin"),@Authority(role="Manager")})
    public void doSomeThing(){
    }
} 
```

由另一个注解来存储重复注解，在使用时候，用存储注解 Authorities 来扩展重复注解，我们再来看看 java 8 里面的做法：

```java
@Repeatable(Authorities.class)
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseNewVersion {
    @Authority(role="Admin")
    @Authority(role="Manager")
    public void doSomeThing(){ }
} 
```

不同的地方是，创建重复注解 Authority 时，加上@Repeatable,指向存储注解 Authorities，在使用时候，直接可以重复使用 Authority 注解。从上面例子看出，java 8 里面做法更适合常规的思维，可读性强一点

### **总结**

JEP120 没有太多内容，是一个小特性，仅仅是为了提高代码可读性。这次 java 8 对注解做了 2 个方面的改进（JEP 104,JEP120），相信注解会比以前使用得更加频繁了。