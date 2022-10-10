# to_unsafe

如果类型定义了 `to_unsafe`方法，当传递给 C 函数时，这个方法会被自动调用。例如：

```crystal
lib C
  fun exit(status : Int32) : NoReturn
end

class IntWrapper
  def initialize(@value)
  end

  def to_unsafe
    @value
  end
end

wrapper = IntWrapper.new(1)
C.exit(wrapper) # 不是 Int32, 但是它的 to_unsafe 方法返回值是
                # 所以 wrapper.to_unsafe会被传进去
```

这用于定义C类型的包装类，而对C使用它们的时候又不用手动拆包。

例如 `String` 类实现了 `to_unsafe` 方法，返回 `UInt8*`：

```crystal
lib C
  fun printf(format : UInt8*, ...) : Int32
end

a = 1
b = 2
C.printf "%d + %d = %d\n", a, b, a + b
```
