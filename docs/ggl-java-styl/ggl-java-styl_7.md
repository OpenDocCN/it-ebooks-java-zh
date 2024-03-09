# Javadoc

## Javadoc

# 格式

### 7.1 格式

# 一般形式

#### 7.1.1 一般形式

Javadoc 块的基本格式如下所示：

```java
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... } 
```

或者是以下单行形式：

```java
/** An especially short bit of Javadoc. */ 
```

基本格式总是 OK 的。当整个 Javadoc 块能容纳于一行时(且没有 Javadoc 标记@XXX)，可以使用单行形式。

# 段落

#### 7.1.2 段落

空行(即，只包含最左侧星号的行)会出现在段落之间和 Javadoc 标记(@XXX)之前(如果有的话)。 除了第一个段落，每个段落第一个单词前都有标签`<p>`，并且它和第一个单词间没有空格。

# Javadoc 标记

#### 7.1.3 Javadoc 标记

标准的 Javadoc 标记按以下顺序出现：`@param`, `@return`, `@throws`, `@deprecated`, 前面这 4 种标记如果出现，描述都不能为空。 当描述无法在一行中容纳，连续行需要至少再缩进 4 个空格。

# 摘要片段

### 7.2 摘要片段

每个类或成员的 Javadoc 以一个简短的摘要片段开始。这个片段是非常重要的，在某些情况下，它是唯一出现的文本，比如在类和方法索引中。

这只是一个小片段，可以是一个名词短语或动词短语，但不是一个完整的句子。它不会以`A {@code Foo} is a...`或`This method returns...`开头, 它也不会是一个完整的祈使句，如`Save the record...`。然而，由于开头大写及被加了标点，它看起来就像是个完整的句子。

> > Tip：一个常见的错误是把简单的 Javadoc 写成`/** @return the customer ID */`，这是不正确的。它应该写成`/** Returns the customer ID. */`。

# 哪里需要使用 Javadoc

### 7.3 哪里需要使用 Javadoc

至少在每个 public 类及它的每个 public 和 protected 成员处使用 Javadoc，以下是一些例外：

# 例外：不言自明的方法

#### 7.3.1 例外：不言自明的方法

对于简单明显的方法如`getFoo`，Javadoc 是可选的(即，是可以不写的)。这种情况下除了写“Returns the foo”，确实也没有什么值得写了。

单元测试类中的测试方法可能是不言自明的最常见例子了，我们通常可以从这些方法的描述性命名中知道它是干什么的，因此不需要额外的文档说明。

> > Tip：如果有一些相关信息是需要读者了解的，那么以上的例外不应作为忽视这些信息的理由。例如，对于方法名`getCanonicalName`， 就不应该忽视文档说明，因为读者很可能不知道词语`canonical name`指的是什么。

# 例外：重载

#### 7.3.2 例外：重载

如果一个方法重载了超类中的方法，那么 Javadoc 并非必需的。

# 可选的 Javadoc

#### 7.3.3 可选的 Javadoc

对于包外不可见的类和方法，如有需要，也是要使用 Javadoc 的。如果一个注释是用来定义一个类，方法，字段的整体目的或行为， 那么这个注释应该写成 Javadoc，这样更统一更友好。