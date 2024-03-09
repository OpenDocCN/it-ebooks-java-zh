# 十五、15.Java NIO Path 路径

> 原文链接：[`tutorials.jenkov.com/java-nio/path.html`](http://tutorials.jenkov.com/java-nio/path.html)

Java 的 path 接口是作为 Java NIO 2 的一部分是 Java6,7 中 NIO 的升级增加部分。Path 在 Java 7 新增的。相关接口位于 java.nio.file 包下，所以 Javaz 内 Path 接口的完整名称是 java.nio.file.Path.

一个 Path 实例代表一个文件系统内的路径。path 可以指向文件也可以指向目录。可以使相对路径也可以是绝对路径。绝对路径包含了从根目录到该文件（目录）的完整路径。相对路径包含该文件（目录）相对于其他路径的路径。相对路径听起来可能有点让人头晕。但是别急，稍后我们会详细介绍。

不要把文件系统中路径和环境变量的路径混淆。java.nio.file.Path 和环境变量没有任何关系。

在很多情况下 java.no.file.Path 接口和 java.io.File 比较相似，但是他们之间存在一些细微的差异。尽管如此，在大多数情况下，我们都可以用 File 相关类来替换 Path 接口。

## 创建 Path 实例（Creating a Path Instance）

为了使用 java.nio.file.Path 实例我们必须创建 Path 对象。创建 Path 实例可以通过 Paths 的工厂方法 get（）。下面是一个实例：

```java
import java.nio.file.Path;
import java.nio.file.Paths;

public classs PathExample {
  public static void mian(String[] args) {
    Path = path = Paths.get("c:\\data\\myfile.txt");
  }
}
```

注意上面的两个 import 声明。需要使用 Path 和 Paths 的接口,毕现先把他们引入。

其次注意 Paths.get("c:\data\myfile.txt")的调用。这个方法会创建一个 Path 实例，换句话说 Paths.get()是 Paths 的一个工厂方法。

### 创建绝对路径（Creating an Absolute Path）

创建绝对路径只需要调动 Paths.get()这个工厂方法，同时传入绝对文件。这是一个例子：

```java
Path path = Paths.get("c:\\data\\myfile.txt");
```

对路径是 c:\data\myfile.txt，里面的双斜杠\字符是 Java 字符串中必须的，因为\是转义字符，表示后面跟的字符在字符串中的真实含义。双斜杠\表示\自身。

上面的路径是 Windows 下的文件系统路径表示。在 Unixx 系统中（Linux, MacOS,FreeBSD 等）上述的绝对路径长得是这样的：

```java
Path path = Paths.get("/home/jakobjenkov/myfile.txt");
```

他的绝对路径是/home/jakobjenkov/myfile.txt。 如果在 Windows 机器上使用用这种路径，那么这个路径会被认为是相对于当前磁盘的。例如：

```java
/home/jakobjenkov/myfile.txt
```

这个路径会被理解其 C 盘上的文件，所以路径又变成了

```java
C:/home/jakobjenkov/myfile.txt
```

### 创建相对路径（Creating a Relative Path）

相对路径是从一个路径（基准路径）指向另一个目录或文件的路径。完整路径实际上等同于相对路径加上基准路径。

Java NIO 的 Path 类可以用于相对路径。创建一个相对路径可以通过调用 Path.get(basePath, relativePath),下面是一个示例：

```java
Path projects = Paths.get("d:\\data", "projects");

Path file     = Paths.get("d:\\data", "projects\\a-project\\myfile.txt");
```

第一行创建了一个指向 d:\data\projects 的 Path 实例。第二行创建了一个指向 d:\data\projects\a-project\myfile.txt 的 Path 实例。 在使用相对路径的时候有两个特殊的符号：

*   .
*   ..

.表示的是当前目录，例如我们可以这样创建一个相对路径：

```java
Path currentDir = Paths.get(".");
System.out.println(currentDir.toAbsolutePath());
```

currentDir 的实际路径就是当前代码执行的目录。 如果在路径中间使用了.那么他的含义实际上就是目录位置自身，例如：

```java
Path currentDir = Paths.get("d:\\data\\projects\.\a-project");
```

上诉路径等同于：

```java
d:\data\projects\a-project
```

..表示父目录或者说是上一级目录：

```java
Path parentDir = Paths.get("..");
```

这个 Path 实例指向的目录是当前程序代码的父目录。 如果在路径中间使用..那么会相应的改变指定的位置：

```java
String path = "d:\\data\\projects\\a-project\\..\\another-project";
Path parentDir2 = Paths.get(path);
```

这个路径等同于：

```java
d:\data\projects\another-project
```

.和..也可以结合起来用，这里不过多介绍。

## Path.normalize()

Path 的 normalize()方法可以把路径规范化。也就是把.和..都等价去除：

```java
String originalPath = "d:\\data\\projects\\a-project\\..\\another-project";

Path path1 = Paths.get(originalPath);
System.out.println("path1 = " + path1);

Path path2 = path1.normalize();
System.out.println("path2 = " + path2);
```

这段代码的输出如下：

```java
path1 = d:\data\projects\a-project\..\another-project
path2 = d:\data\projects\another-project
```