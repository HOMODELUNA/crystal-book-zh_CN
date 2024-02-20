# type

`lib` 中的 `type` 声明类似于C的 `typedef`，但是更强：

```crystal
lib X
  type MyInt = Int32
end
```

不同于 C， `Int32` 和 `MyInt` 不能互通：

```crystal
lib X
  type MyInt = Int32

  fun some_fun(value : MyInt)
end

X.some_fun 1 # 错误: 'X#some_fun' 的参数 'value'  
             # 必须是 X::MyInt, 而不是 Int32
```

因此， `type` 可以在C绑定中定义隐晦的类型。一个例子是C 的`FILE` 类型，你可以用 `fopen`获得。

这种类型的定义也如同 [类型语法](../type_grammar.html) 。
