# if 变量

如果一个变量被用作 `if` 的条件，那么在 `then`分支内他会被认为不可能是 `Nil`类型：

```crystal
a = some_condition ? nil : 3
# a is Int32 or Nil

if a
  # 既然我们只能通过 a 为真一种方式到达这里,
  # a 就不能是. 所以在此， a 是 Int32.
  a.abs
end
```

同样的思维用于当一个值在if表达式中被赋值时：

```crystal
if a = some_expression
  # 此处 a 不会是 nil
end
```

逻辑与运算符(`&&`)也适用此规律： 

```crystal
if a && b
  # 这里 a 和 b 保证都不是 Nil
end
```

这里， 在求值 `&&` 表达式右边的 `b` 时，`a`已经保证不是 `Nil`。

当然，在 `then` 分支重赋值变量会导致变量有新的类型。

## 限制

这个逻辑 **不适用于** 成员变量，类变量和其他在闭包中绑定的变量。这些变量的值有可能在条件判断之后又被另外的纤程修改，导致它变成 `nil`。

```crystal
if @a
  # 此处 `@a` 可以是 nil
end

if @@a
  # 此处 `@@a` 可以是 nil
end

a = nil
closure = ->(){ foo = "foo" } # 这个闭包绑定了前面所有的变量，包括 a

if a
  # 此处 `a` 可以是 nil
end
```

这个缺陷可以通过新赋值一个局部变量来绕过：

```crystal
if a = @a
  # 此处 `a` 一定不是 nil
end
```

另一个方法是使用标准库的 [`Object#try`](https://crystal-lang.org/api/Object.html#try%28%26block%29-instance-method) ，仅当该元素不是 `nil`时才执行后面的块：

```crystal
@a.try do |a|
  # 此处 `a` 一定不是 nil
end
```

## 方法调用

这个逻辑同样不适用于过程和方法调用，比如读取方法(getters)和属性(properties)，因为一个可空（广泛地说，返回联合体的方法）的方法并不保证在接连的调用时返回同样的精确类型。

```crystal
if method # 这个方法的第一次调用返回 Int32 或 Nil
          # 好，我们知道第一次调用返回的不是 Nil了
  method  # 第二次调用仍返回 Int32 或 Nil
end
```


> 译注:想想这个方法
> ```crystal 
> class Flipper
>   def flip
>     if @v
>       v = nil
>     else
>       @v = 1
>     end
>     v
>   end
> end
> ``` 
> 这个方法会交替返回 1 和 nil，没法保证两次方法调用返回的是同一个值

同样的迂回手法可以用于保存某次调用的值，让它在if的子句中不为空。
