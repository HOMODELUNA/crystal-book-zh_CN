# union
在 `lib` 中用`union`声明C的联合体：

```crystal
lib U
  # C中:
  #
  #  union IntOrFloat {
  #    int some_int;
  #    double some_float;
  #  };
  union IntOrFloat
    some_int : Int32
    some_float : Float64
  end
end
```

用 `new`创建联合体的实例:

```crystal
value = U::IntOrFloat.new
```

这会在栈上分配联合体。

C联合体初始化时所有的域都是 "零": 整数和浮点数是零，指针指向地址零。

你可以用`uninitialized`纸面这样的初始化：

```crystal
value = uninitialized U::IntOrFloat
value.some_int #=> 无意义值
```

你可以读写其中的成员：

```crystal
value = U::IntOrFloat.new
value.some_int = 1
value.some_int #=> 1
value.some_float #=> 4.94066e-324
```

如果这个赋值的类型并不正好是属性的值，那么会尝试调用 [to_unsafe](to_unsafe.html)方法，然后把返回值传进去。

 C联合体按值(复制地)传递给函数和方法，并且也总是按值返回：

```crystal
def change_it(value)
  value.some_int = 1
end

value = U::IntOrFloat.new
change_it value
value.some_int #=> 0
```

成员类型的声明详见 [类型语法](../type_grammar.html)。
