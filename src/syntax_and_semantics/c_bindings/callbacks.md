# 回调

你可以在C 库中声明函数类型：

```crystal
lib X
  # In C:
  #
  #    void callback(int (*f)(int));
  fun callback(f : Int32 -> Int32)
end
```

而后你可以这样传递一个函数 ([Proc](http://crystal-lang.org/api/Proc.html))：

```crystal
f = ->(x : Int32) { x + 1 }
X.callback(f)
```

如果你在调用时只写了一个块，那你可以不写类型，让编译器根据`fun`的签名替你标出来：

```crystal
X.callback ->(x) { x + 1 }
```

但是，注意，这些传递给C的函数不能形成闭包。如果编译器发现这个函数是一个闭包，它会报错：

> 因为C的函数不过是一个函数指针,它就没有带其运行所需的环境。
```crystal
y = 2
X.callback ->(x) { x + y } # 错误: 不能向C的函数传递闭包
```

如果编译器在编译时没有发现，那就会在运行时产生异常。

回调所需的函数类型详见 [类型语法](../type_grammar.html) 

如果你不想传递回调函数，而只是传递 `NULL`(空指针)，你可以传 `nil`：

```crystal
# 等同于C中的 callback(NULL) 
X.callback nil
```

### 向 C 函数中传递闭包

C 在设置回调函数时往往允许附带一个void指针，用来传递这个函数所需的环境。举个例子，如果有个C函数每一刻都要执行某个函数，同时每次都需要刻的序号：

```crystal
lib LibTicker
  fun on_tick(callback : (Int32, Void* ->), data : Void*)
end
```

To properly define a wrapper for this function we must send the Proc as the callback data, and then convert that callback data to the Proc and finally invoke it.

```crystal
module Ticker
  @@box : Box(Int32 ->)

  # 用户用的回调不用带 Void*
  def self.on_tick(&callback : Int32 ->)
    # 因为 Proc 本质上就是 {Void*, Void*}, 我们不能把缩减成一个 Void*,
    # 所以我们把它"包起来" : 我们在这里分配内存,存储 Proc 的环境
    boxed_data = Box.box(callback)

    # 我们必须在 Crystal的地盘存储这个函数和它的内存,以免 GC 收走他(*)
    @@box = boxed_data

    # 我们把Process分成两份传递,一份是单独的函数指针,另一份是函数所需的环境
    LibTicker.on_tick(->(tick, data) {
      # 我们用 Box.unbox 把数据还给 Proc
      data_as_callback = Box(typeof(callback)).unbox(data)
      # 现在我们才调用用户层的回调函数
      data_as_callback.call(tick)
    }, boxed_data)
  end
end

Ticker.on_tick do |tick|
  puts tick
end
```

注意我们用在 `@@box`中存储了包装起来的回调函数。这是因为如果我们不这么做，我们的代码就引用不到它了， GC就会把它收走。C库当然会存储这个回调，但是 Crystal 的 GC 不可能知道这些。

## Raises 属性

如果 C 函数所执行的用户回调可能抛异常，他就必须加以 `@[Raises]` 属性。

即使你没有加`@[Raises]` ，却在回调(或是回调的子函数)中确实抛出了异常，编译器也会替你加上它。

但是，有些C函数存储回调函数，以供其他C函数使用。例如，考虑一个假想的库：

```crystal
lib LibFoo
  fun store_callback(callback : ->)
  fun execute_callback
end

LibFoo.store_callback ->{ raise "OH NO!" }
LibFoo.execute_callback
```

如果`store_callback`存储的这个回调函数会抛异常，那么 `execute_callback` 也会抛异常。然而，编译器不知道这个 `execute_callback` 也可能抛异常(因为他没有标注`@[Raises]`，编译器也没有其他的线索猜出来)，这时你必须手动把它标出来：

```crystal
lib LibFoo
  fun store_callback(callback : ->)

  @[Raises]
  fun execute_callback
end
```

如果你不标它，这个函数周围的 `begin/rescue` 块可能会产生意想不到的麻烦。
