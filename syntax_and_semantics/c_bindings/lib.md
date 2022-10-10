# lib

用`lib`声明某个库的一系列C函数和类型。

```crystal
@[Link("pcre")]
lib LibPCRE
end
```

尽管编译器不强制，一个 `lib`的名称往往以 `Lib`开头。

编译器会根据`lib`头上的属性找到它要链接的外部库：

* `@[Link("pcre")]` 会把 `-lpcre` 标志传递给链接器，但编译器会首先尝试使用 [pkg-config](http://en.wikipedia.org/wiki/Pkg-config)。
* `@[Link(ldflags: "...")]` 会直接把标志原样传给链接器。例如： `@[Link(ldflags: "-lpcre")]`。常见的方法是使用反引号来执行命令： ``@[Link(ldflags: "`pkg-config libpcre --libs`")]``。
* `@[Link(framework: "Cocoa")]` 会把 `-framework Cocoa` 传给链接器 (只在 Mac OS X 上有用)。

当这个库已经被隐式地链接时，属性会被忽略。例如libc。
