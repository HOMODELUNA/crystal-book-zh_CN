# 命令字面量

命令字面量是一个以反引号 `` ` `` 或百分号式字面量 `%x` 包裹的字符串。在运行时，程序在一个子终端中运行此命令，表示运行结果的字符串即为命令字面量的值。

命令和常规字符串一样进行[转义](string.md#转义)和[插值](string.md#插值)。

类似于字符串字面量， `%x` 可用的分隔符是括号 `()`，方括号 `[]`，花括号 `{}`, 尖括号 `<>` 和竖杠 `||`。除了竖杠，所有的分隔符都可以嵌套，每个始分隔符转义最近的一个终分隔符。

特殊变量 `$?` 以 `Process::Status` 类型保留了命令的执行结果。它只在该字面量的同一个作用域有效。

```crystal
`echo foo`  # => "foo"
$?.success? # => true
```

在内部，编译器会将命令字面量转换为一次对顶层方法 的调用，参数即为命令字符串的内容。```echo #{argument}``` 和 `%x(echo #{argument})` 被重写为 `` `("echo #{argument}")``。


## 安全问题

命令行字面量用于脚本程序非常方便，但是使用时应注意检查用户输入，因为它容易导致命令注入：

```crystal
user_input = "hello; rm -rf *"
`echo #{user_input}`
```

这个命令会写调用`hello`，然后删除当前工作目录下的所有文件。


为了避免这种情况,最好不要让用户命令字符串中插值.标准库的 `Process` 提供了一种将用户输入用于命令行参数的安全做法：

```crystal
user_input = "hello; rm -rf *"
process = Process.new("echo", [user_input], output: Process::Redirect::Pipe)
process.output.gets_to_end # => "hello; rm -rf *"
process.wait.success?      # => true
```