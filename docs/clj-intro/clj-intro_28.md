# Web 应用

## Web 应用

有很多 Clojure 类库可以帮助我们创建 web 应用。现在比较流行使用 Chris Granger 写的 [Noir](http://webnoir.org/) 。另外一个简单的，基于 MVC 的框架， 使用 Christophe Grand 写的? [Enlive](https://github.com/cgrand/enlive) 来做页面的 template, 是 Sean Corfield 写的 [Framework One](https://github.com/seancorfield/fw1-clj) 。另一个流行的选择是 James Reeves 写的 Compojure，你可以在这里下载： [`github.com/weavejester/compojure/tree/master`](http://github.com/weavejester/compojure/tree/master) 。所有这些框架都是基于 Mark McGranahan 写的 [Ring](https://github.com/mmcgrana/ring) (James Reeves 同学现在在维护). 我们以 Compojure 为例子来稍微介绍一下 web 应用开发。最新的版本可以通过 git 来获取:

```java
git clone git://github.com/weavejester/compojure.git 
```

这个命令会在当前目录创建一个叫做 `compojure` 的目录. 另外你还需要从 [`cloud.github.com/downloads/weavejester/compojure/deps.zip`](http://cloud.github.com/downloads/weavejester/compojure/deps.zip) 下载所有依赖的 JAR 包，把 `deps.zip` 下载之后解压在 `compojure` 目录里面的 `deps` 子目录里面：

要获取 `compojure.jar` , 在 compojure 里面运行 `ant` 命令。

要获取 Compojure 的更新, 切换到 `compojure` 目录下面执行下面的命令:

```java
git pull
ant clean deps jar 
```

所有的 `deps` 目录里面的 jar 包都必须包含在 classpath 里面。一个方法是修改我们的 `clj` 脚本，然后用这个脚本来运行 web 应用. 把 " `-cp $CP` " 添加到 `java` 命令后面去 执行 `clojure.main 添加下面这些行到脚本里面去，以把那些 jar 包包含在` `CP` 里面。

```java
# Set CP to a path list that contains clojure.jar
# and possibly some Clojure contrib JAR files.
COMPOJURE_DIR=<em>path-to-compojure-dir</em>
COMPOJURE_JAR=$COMPOJURE_DIR/compojure.jar
CP=$CP:$COMPOJURE_JAR
for file in $COMPOJURE_DIR/deps/*.jar
do
  CP=$CP:$file
done 
```

下面是他一个简单的 Compojure web 应用：

![Compojure input page](img/QNN7Rj.png) ![Compojure output page](img/RVjmqe.png)

```java
(ns com.ociweb.hello
  (:use compojure))

(def host "localhost")
(def port 8080)
(def in-path "/hello")
(def out-path "/hello-out")

(defn html-doc
  "generates well-formed HTML for a given title and body content"
  [title & body]
  (html
    (doctype :html4)
    [:html
      [:head [:title title]]
      [:body body]]))

; Creates HTML for input form.
(def hello-in
  (html-doc "Hello In"
    (form-to [:post out-path]
      "Name: "
      (text-field {:size 10} :name "World")
      [:br]
      (reset-button "Reset")
      (submit-button "Greet"))))

; Creates HTML for result message.
(defn hello-out [name]
  (html-doc "Hello Out"
    [:h1 "Hello, " name "!"]))

(defroutes hello-service
  ; The following three lines map HTTP methods
  ; and URL patterns to response HTML.
  (GET in-path hello-in)
  (POST out-path (hello-out (params :name)))
  (ANY "*" (page-not-found))) ; displays ./public/404.html by default

(println (str "browse http://" host ":" port in-path))
; -> browse http://localhost:8080/hello
(run-server {:port port} "/*" (servl 
```