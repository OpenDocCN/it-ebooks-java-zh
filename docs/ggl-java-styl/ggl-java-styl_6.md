# 编程实践

## 编程实践

# @Override：能用则用

### 6.1 @Override：能用则用

只要是合法的，就把`@Override`注解给用上。

# 捕获的异常：不能忽视

### 6.2 捕获的异常：不能忽视

除了下面的例子，对捕获的异常不做响应是极少正确的。(典型的响应方式是打印日志，或者如果它被认为是不可能的，则把它当作一个`AssertionError`重新抛出。)

如果它确实是不需要在 catch 块中做任何响应，需要做注释加以说明(如下面的例子)。

```java
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // it's not numeric; that's fine, just continue
}
return handleTextResponse(response); 
```

**例外**：在测试中，如果一个捕获的异常被命名为`expected`，则它可以被不加注释地忽略。下面是一种非常常见的情形，用以确保所测试的方法会抛出一个期望中的异常， 因此在这里就没有必要加注释。

```java
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
} 
```

# 静态成员：使用类进行调用

### 6.3 静态成员：使用类进行调用

使用类名调用静态的类成员，而不是具体某个对象或表达式。

```java
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad 
```

# Finalizers: 禁用

### 6.4 Finalizers: 禁用

极少会去重载`Object.finalize`。

> > Tip：不要使用 finalize。如果你非要使用它，请先仔细阅读和理解[Effective Java](http://books.google.com/books?isbn=8131726592) 第 7 条款：“Avoid Finalizers”，然后不要使用它。