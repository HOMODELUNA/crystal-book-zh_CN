# 传递块

为了把代码块传给其他方法，你应当注明块参数，在其前面加 `&`：

```crystal
def capture(&block)
  block
end

def invoke(&block)
  block.call
end

proc = capture { puts "Hello" }
invoke(&proc) # 打印 "Hello"
```

上例中， `invoke` 接收块，我们不能把闭包直接传进它，因为它不接受一般的参数，只接受块参数。我们用 `&` 指定“我们真的要把 `proc`当做块参数传进去”，否则会：

```crystal
invoke(proc) # 错误: 'invoke' 参数数量匹配 (给出 1, 应有 0)
```

你可以把闭包传给使用 yield 的方法：

```crystal
def capture(&block)
  block
end

def twice
  yield
  yield
end

proc = capture { puts "Hello" }
twice &proc
```

上述等同于：

```crystal
proc = capture { puts "Hello" }
twice do
  proc.call
end
```

或者，结合 `&` 和 `->` 的句法：

```crystal
twice &->{ puts "Hello" }
```

Or:

```crystal
def say_hello
  puts "Hello"
end

twice &->say_hello
```

## 传递非捕获的块

要传递非捕获的块，你必须用 `yield`：

```crystal
def foo
  yield 1
end

def wrap_foo
  puts "Before foo"
  foo do |x|
    yield x
  end
  puts "After foo"
end

wrap_foo do |i|
  puts i
end

# 输出:
# Before foo
# 1
# After foo
```

你也可以用 `&block` 来传递块，但是你至少要指出块的输入类型，并且，产生的代码会包含一个闭包，因此也更慢。

```crystal
def foo
  yield 1
end

def wrap_foo(&block : Int32 -> _)
  puts "Before foo"
  foo(&block)
  puts "After foo"
end

wrap_foo do |i|
  puts i
end

# Output:
# Before foo
# 1
# After foo
```

如果 `yield` 够用，就尽量避免传递块。同时注意，在捕获的块中 `break` and `next` 都不能用。所以`&block`不能在接下来的代码中正确运行：

```crystal
foo_forward do |i|
  break # 错误
end
```

总之，当有 `yield` 时避免传递块`&block` 。
