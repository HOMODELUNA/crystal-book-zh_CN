# fun

在 `lib` 内部，用 `fun`声明一个 C 函数。

```crystal
lib C
  # 在 C 中: double cos(double x)
  fun cos(value : Float64) : Float64
end
```

只要绑定完成，这个函数就能像`C`类型的一个类方法一样被调用：

```crystal
C.cos(1.5) #=> 0.0707372
```

如果函数没有参数，你可以省略括号 (当然在调用中也可以省略)：

```crystal
lib C
  fun getch : Int32
end

C.getch
```

如果返回类型是空的，你可以省略它：

```crystal
lib C
  fun srand(seed : UInt32)
end

C.srand(1_u32)
```

你可以绑定到可变参数数目的函数：

```crystal
lib X
  fun variadic(value : Int32, ...) : Int32
end

X.variadic(1, 2, 3, 4)
```

注意，调用C函数时参数不会进行任何的隐式转换 (除了 `to_unsafe`，下面将会解释)：你必须准确地传递所需类型。对于整数和浮点数，你可以使用 `to_...`系列方法。

## 函数名

不同于 `lib`外的方法，`lib` 内的函数名可以以大写字母开头(相对地，`lib`外的方法只能以小写字母开头)。

Crystal 里的函数名可以与 C 中的函数名不同。下面的例子展示了如何把 C 函数 `SDL_Init` 绑定到 Crystal 的`LibSDL.init`。

```crystal
lib LibSDL
  fun init = SDL_Init(flags : UInt32) : Int32
end
```

如果 C 函数名不是有效的标识符，他们应该用引号括起来：

```crystal
lib LLVMIntrinsics
  fun ceil_f32 = "llvm.ceil.f32"(value : Float32) : Float32
end
```

这也用于给C函数起一个更短，更好听的名字，因为它们往往很长，前面还带有库的名称。

##  C 绑定中的类型

C 绑定中可以用的类型有：
* 原始类型 (`Int8`, ..., `Int64`, `UInt8`, ..., `UInt64`, `Float32`, `Float64`)
* 指针类型 (`Pointer(Int32)`, 也可以写成 `Int32*`)
* 静态数组 (`StaticArray(Int32, 8)`, 也可以写成 `Int32[8]`)
* 函数类型 (`Function(Int32, Int32)`, 也可以写成 `Int32 -> Int32`)
* 其他先前已经定义的 `struct`, `union`, `enum`, `type` 或 `alias` 。
* `Void`: 表示没有返回值。
* `NoReturn`: 类似于 `Void`，但编译器知道它调用后绝不返回。
* 标有 `@[Extern]` 属性的Crystal结构体

fun 中类型的记名参见 [类型语法](../type_grammar.html)。

标准库定义了 [LibC](https://github.com/crystal-lang/crystal/blob/master/src/lib_c.cr) 库，内含C中常见的类型，如 `int`, `short`, `size_t`。 它们可以这样使用：

```crystal
lib MyLib
  fun my_fun(some_size : LibC::SizeT)
end
```

**注意:** C 中的 `char` 在 Crystal 中是`UInt8` , 因此 `char*` 或 `const char*` 应当是 `UInt8*`。 Crystal中的 `Char`类型表示 unicode codepoint ，所以它占四个字节，类似于 `Int32`，而不是 `UInt8`。如果搞不明白，直接用别名 `LibC::Char`就行了。
