# 自动化测试

## 自动化测试

Clojure 里面主要的主要自动化测试框架是 clojure core 里面自带的。下面的代码演示了它的一些主要特性：

```java
(use 'clojure.test)

; Tests can be written in separate functions.
(deftest add-test
  ; The "is" macro takes a predicate, arguments to it,
  ; and an optional message.
  (is (= 4 (+ 2 2)))
  (is (= 2 (+ 2 0)) "adding zero doesn't change value"))

(deftest reverse-test
  (is (= [3 2 1] (reverse [1 2 3]))))

; Tests can verify that a specific exception is thrown.
(deftest division-test
  (is (thrown? ArithmeticException (/ 3.0 0))))

; The with-test macro can be used to add tests
; to the functions they test as metadata.
(with-test
  (defn my-add [n1 n2] (+ n1 n2))
  (is (= 4 (my-add 2 2)))
  (is (= 2 (my-add 2 0)) "adding zero doesn't change value"))

; The "are" macro takes a predicate template and
; multiple sets of arguments to it, but no message.
; Each set of arguments are substituted one at a time
; into the predicate template and evaluated.
(deftest multiplication
  (are [n1 n2 result]
    (= (* n1 n2) result) ; a template
    1 1 1,
    1 2 2,
    2 3 6))

; Run all the tests in the current namespace.
; This includes tests that were added as function metadata using with-test.
; Other namespaces can be specified as quoted arguments.
(run-tests) 
```

为了限制运行一个 test 的时候抛出来的异常的深度，bind 一个数字到 special symbol: `*stack-trace-depth*` 。

当你要把 Clojure 代码编译成 bytecode 以部署到生成环境的时候， 你可以给 `*load-tests*` symbol bind 一个 `false` 值，以避免把测试代码编译进去。

虽然和自动化测试不是同一个层面的东西，还是值得一提的是 Clojure 提供一个宏： `assert` 。它测试一个表达式， 如果这个表达式的值为 false 的话，他们它会抛出异常。这可以警告我们这种情况从来都不应该发生。看例子:

```java
(assert (>= dow 7000)) 
```

自动化测试的另外一个重要的特性是 fixtures。fixture 其实就是 JUnit 里面的 setup 和 tearDown 方法。fixture 分为两种，一种是在每个测试方法的开始，结束的时候 执行。一种是在整个测试（好几个测试方法）的开始和结束的时候执行。

照下面的样子编写 fixture:

```java
(defn fixture-name [test-function]
  ; Perform setup here.
  (test-function)
  ; Perform teardown here.
) 
```

这个 fixture 函数会在执行每个测试方法的时候执行一次。这里这个 `test-function` 及时要被执行的测试方法。

用下面的方法去注册这些 fixtures 去包裹每一个测试方法:

```java
(use-fixtures :each fixture-1 fixture-2 ...) 
```

执行的顺序是:

1.  fixture-1 setup
2.  fixture-2 setup
3.  ONE test function
4.  fixture-2 teardown
5.  fixture-1 teardown

用下面的方法去注册这些 fixtures 去包裹整个一次测试:

```java
(use-fixtures  nce fixture-1 fixture-2 ...) 
```

执行顺序是这样的:

1.  fixture-1 setup
2.  fixture-2 setup
3.  ALL test functions
4.  fixture-2 teardown
5.  fixture-1 teardown

Clojure 本身的测试在 `test` 子目录里面. 要想运行他们的话, cd 到包含 `src` 和 `test` 的目录下面去，然后执行： " `ant test` "。