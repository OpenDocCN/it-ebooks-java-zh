# 数据库

## 数据库

Clojure Contrib 里面的 jdbc 库简化了 clojure 对于关系型数据库的访问. 它通过 commit 和 rollback 来支持事务, 支持 prepared statements, 支持创建/删除表, 插入/更新/删除行, 以及查询。下面的例子链接到一个 Postgres 数据库并且执行了一个查询。代码的注释里面还提到了怎么使用 jdbc 来连接 mysql。

```java
(use 'clojure.java.jdbc)

(let [db-host "localhost"
      db-port 5432 ; 3306
      db-name "HR"]

  ; The classname below must be in the classpath.
  (def db {:classname "org.postgresql.Driver" ; com.mysql.jdbc.Driver
           :subprotocol "postgresql" ; "mysql"
           :subname (str "//" db-host ":" db-port "/" db-name)
           ; Any additional map entries are passed to the driver
           ; as driver-specific properties.
           :user "mvolkmann"
           :password "cljfan"})

  (with-connection db ; closes connection when finished
    (with-query-results rs ["select * from Employee"] ; closes result set when finished
      ; rs will be a non-lazy sequence of maps,
      ; one for each record in the result set.
      ; The keys in each map are the column names retrieved and
      ; their values are the column values for that result set row.
      (doseq [row rs] (println (row :lastname)))))) 
```

`clj-record` 提供了一个类似 Ruby on Rails 的 ActiveRecord 的数据库访问包. 更多关于它的信息看这里： [`github.com/duelinmarkers/clj-record/tree/master`](http://github.com/duelinmarkers/clj-record/tree/master) .