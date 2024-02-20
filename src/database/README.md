# 数据库

要访问一个关系数据库，你需要一个用于访问目标数据库的shard。 [crystal-lang/crystal-db](https://github.com/crystal-lang/crystal-db) 包为不同的驱动提供了统一的API。

下列包与 crystal-db 兼容：

* [crystal-lang/crystal-sqlite3](https://github.com/crystal-lang/crystal-sqlite3) 用于 sqlite
* [crystal-lang/crystal-mysql](https://github.com/crystal-lang/crystal-mysql) 用于 mysql & mariadb
* [will/crystal-pg](https://github.com/will/crystal-pg) 用于 postgres

这份指导展示了 crystal-db 的api。鉴于postgres, mysql 与 sqlite 各不相同，同样的SQL命令需要数据库驱动以合适的方式阐述。

一些驱动还有额外的功能，例如 postgres 的 `LISTEN`/`NOTIFY`。

## 安装shard

从上面选择合适的驱动，把它像普通shard一样加进你应用的 `shard.yml`。

不必显式依赖`crystal-lang/crystal-db`。

这份指南将以 `crystal-lang/crystal-mysql`为例进行展示。

```yaml
dependencies:
  mysql:
    github: crystal-lang/crystal-mysql
```

## 打开数据库

`DB.open` 允许你通过目标uri来简便地连接数据库。uri 的样式决定了所期待的驱动。这个例子连接到本地的 mysql 数据库，以root用户的身份，密码为空。

```crystal
require "db"
require "mysql"

DB.open "mysql://root@localhost/test" do |db|
  # ... 用 db 进行查询
end
```

其他的连接uri如：

* `sqlite3:///path/to/data.db`
* `mysql://user:password@server:port/database`
* `postgres://server:port/database`

你还可以选用非 yield 版的 `DB.open`，只要你最后调用 `Database#close` 来关闭数据库。

```crystal
require "db"
require "mysql"

db = DB.open "mysql://root@localhost/test"
begin
  # ... 用 db 进行查询
ensure
  db.close
end
```

## Exec

用 `Database#exec` 执行 sql 命题。

```crystal
db.exec "create table contacts (name varchar(30), age int)"
```

用括号包裹发送的数据，以避免sql注入。

```crystal
db.exec "insert into contacts values (?, ?)", "John", 30
db.exec "insert into contacts values (?, ?)", "Sarah", 33
```

注：使用 pg 驱动时，用 `$1`, `$2`, 等等，而不是 `?`。

## Query

用`Database#query`进行查询，得到结果。参数的用法如同 `Database#exec`.

`Database#query` 返回的是 `ResultSet` ，t它也需要被关闭。如同 `Database#open`，如果用含块的格式调用，这个 `ResultSet` 会隐式地被关闭。

```crystal
db.query "select name, age from contacts order by age desc" do |rs|
  rs.each do
    # ... 对 ResultSet 的每一行做一些什么。
  end
end
```

从数据库读取值时，crystal无从知道数据的类型信息，你可以用你想要的类型参数`T`调用  `rs.read(T)`来获取信息。

```crystal
db.query "select name, age from contacts order by age desc" do |rs|
  rs.each do
    name = rs.read(String)
    age = rs.read(Int32)
    puts "#{name} (#{age})"
    # => Sarah (33)
    # => John Doe (30)
  end
end
```

基于 `#query` 还有很多便捷的查询方法。

你可以一次读很多列：

```crystal
name, age = rs.read(String, Int32)
```

或者一次一行：

```crystal
name, age = db.query_one "select name, age from contacts order by age desc limit 1", as: { String, Int32 }
```

或者读取一个标量值，而不用直接处理 ResultSet：

```crystal
max_age = db.scalar "select max(age) from contacts"
```

所有用于执行命题的方法都在 `DB::QueryMethods`中定义。
