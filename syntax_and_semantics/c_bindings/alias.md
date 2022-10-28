# 类型别名

`lib` 中的 `alias` 声明类似于C的 `typedef`：


```crystal
lib X
  alias MyInt = Int32
end
```

这里 `Int32` 和 `MyInt` 是互通的：

```crystal
lib X
  alias MyInt = Int32

  fun some_fun(value : MyInt)
end

X.some_fun 1 # OK
```

`alias` 最适合用来避免写超长类型声明，同时创建一个编译时类型标志：

```crystal
lib C
  {% if flag?:(x86_64) %}
    alias SizeT = Int64
  {% else %}
    alias SizeT = Int32
  {% end %}

  fun memcmp(p1 : Void*, p2 : Void*, size : C::SizeT) : Int32
end
```

类型别名的定义也如同 [类型语法](../type_grammar.html)。
