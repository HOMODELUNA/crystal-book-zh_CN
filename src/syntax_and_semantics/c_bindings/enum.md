# enum

在`lib` 中用`enum`声明C的枚举：

```crystal
lib X
  # C 中:
  #
  #  enum SomeEnum {
  #    Zero,
  #    One,
  #    Two,
  #    Three,
  #  };
  enum SomeEnum
    Zero
    One
    Two
    Three
  end
end
```

如同C的枚举, 第一个成员的值为零，随后每个成员数目增加1。

使用一个值：

```crystal
X::SomeEnum::One #=> One
```

你可以指定成员的值：

```crystal
lib X
  enum SomeEnum
    Ten = 10
    Twenty = 10 * 2
    ThirtyTwo = 1 << 5
  end
end
```

如你所见，枚举的值可以用成员的值可以做一些基本的数值运算： `+`, `-`, `*`, `/`, `&`, `|`, `<<`, `>>` 和 `%`。

枚举成员的类型默认是 `Int32` ，即使你在单个成员中使用了不同的类型：

```crystal
lib X
  enum SomeEnum
    A = 1_u32
  end
end

X::SomeEnum #=> 1_i32
```

不过你可以改变基础类型：

```crystal
lib X
  enum SomeEnum : Int8
    Zero,
    Two = 2
  end
end

X::SomeEnum::Zero #=> 0_i8
X::SomeEnum::Two  #=> 2_i8
```

你可以被枚举作为 `fun` 的参数，和 `struct` 或 `union` 的成员:

```crystal
lib X
  enum SomeEnum
    One
    Two
  end

  fun some_fun(value : SomeEnum)
end
```
