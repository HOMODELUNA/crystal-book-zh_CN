# 变量

C库中暴露的变量可以用全局变量的形式在 `lib` 中定义：

```crystal
lib C
  $errno : Int32
end
```

而后他可以进行读写：

```crystal
C.errno #=> some value
C.errno = 0
C.errno #=> 0
```

可以用属性把变量设为线程局部变量：

```crystal
lib C
  @[ThreadLocal]
  $errno : Int32
end
```

外部变量的类型定义也如同 [类型语法](../type_grammar.html)。
