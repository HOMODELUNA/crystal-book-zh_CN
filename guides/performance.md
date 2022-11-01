# 性能

这些提示有助于你发挥程序的全部能力,不论是速度还是内存占用。

## 过早优化

Donald Knuth 说过：

> 我们应当放过细小的瑕疵，因为 97% 的场合中，过早优化都是万恶之源。但是也不要错过最关键的 3%。

不过，如果你在写程序的时候就意识到两种方法是等效的，那就直接写成更好的那种。这样后来的改动就会更少。

同时也记得测试你的程序，以得知他的瓶颈。在Mac上， [Instruments Time Profiler](https://developer.apple.com/library/prerelease/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/Instrument-TimeProfiler.html)是不错的测试工具，一般与 XCode 配合使用。 Linux上， 任何能测试 C/C++ 程序的工具都可以用，比如 [perf](https://perf.wiki.kernel.org/index.php/Main_Page) 或者 [Callgrind](http://valgrind.org/docs/manual/cl-manual.html)。

记着测试程序的时候以 `--release`模式编译，这会开启优化。

## 避免内存分配

最好的优化方法之一就是避免过多的、无用的内存分配。当创建 **class** 实例时，就会有堆上的内存被分配。而创建 **struct** 在栈上分配内存，这样就能避免一部分性能损失。如果你不知道堆内存和栈内存的区别， [看这里](https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)。

分配堆内存是缓慢的，也会给垃圾收集器带来更多压力，因为你每一片分配的内存总是要让垃圾收集器来收场。

有多种方法来避免分配内存，标准库的设计也帮助你这么做。

### 写 IO 时不要创建中间字符串

你可以这样向标准输出打印数字：

```
puts 123
```

在许多语言中，这会导致一个对象调用自己的 `to_s` 方法——或是什么把自己转成字符串的操作——然后向标准输出打印这个字符串。这么做可以，但是有个缺点：它在堆内存里创建了中间的字符串，并且写一遍就扔了。这白白浪费了一次内存分配，GC又有活干了。 

Crystal中, `puts` 会调用对象的 `to_s(io)`操作，把IO传给这个对象，让它自己决定如何输出。

所以，你永远不用这么做：

```
puts 123.to_s
```

因为这样创建了一个中间字符串，反而增加了开销。直接让对象向IO写就完事了。

当输出自定义类型时，一定要重载 `to_s(io)`,而不是 `to_s`, 并且避免在这个方法里创建中间字符串。例如：

```crystal
class MyClass
  # 好
  def to_s(io)
    # 向IO 中追加了 "1, 2" ，而不用创建中间字符串
    x = 1
    y = 2
    io << x << ", " << y
  end

  # 坏
  def to_s(io)
    x = 1
    y = 2
    # 用字符串插值创建了中间字符串，这应该避免。
    io << "#{x}, #{y}"
  end
end
```

相比创建中间字符串，这种直接向IO写的思想能得到更好的性能，你在定义自己的API时也应该这么做。

比较一下用时：

```crystal
# io_benchmark.cr
require "benchmark"

io = IO::Memory.new

Benchmark.ips do |x|
  x.report("不用 to_s") do
    io << 123
    io.clear
  end

  x.report("用 to_s") do
    io << 123.to_s
    io.clear
  end
end
```

输出:

```
$ crystal run --release io_benchmark.cr
    不用 to_s  77.11M ( 12.97ns) (± 1.05%)       fastest
      用 to_s  18.15M ( 55.09ns) (± 7.99%)  4.25× slower
```

不仅减少了用时，还减少了内存消耗。

### 用字符串插值，而不是拼接

有时你要把字符串字面量和其他的值拼接起来，你不要用 `String#+(String)` ，而应该用 [字符串插值](syntax_and_semantics/literals/string.html) ，后者还能被表达式嵌到字符串里面。 `"Hello, #{name}"` 比 `"Hello, " +  name.to_s`好。

字符串插值会被编译器转换成 IO ，这样就不会涉及中间字符串了。上面的例子会变成： `(StringBuilder.new << "Hello, " << name).to_s`。

### 创建字符串时避免IO分配

`String.build` 对字符串有优化，所以不要另外分配 `IO::Memory`。

```crystal
require "benchmark"

Benchmark.ips do |x|
  x.report("String.build") do
    String.build do |io|
      99.times do
        io << "hello world"
      end
    end
  end
  x.report("IO::Memory") do
    io = IO::Memory.new
    99.times do
      io << "hello world"
    end
    io.to_s
  end
end
```

输出：

```
$ crystal run --release str_benchmark.cr
String.build 597.57k (  1.67µs) (± 5.52%)       fastest
  IO::Memory 423.82k (  2.36µs) (± 3.76%)  1.41× slower
```


### 避免重复创建临时对象

考虑这个程序：

```crystal
lines_with_language_reference = 0
while line = gets
  if ["crystal", "ruby", "java"].any? { |string| line.includes?(string) }
    lines_with_language_reference += 1
  end
end
puts "Lines that mention crystal, ruby or java: #{lines_with_language_reference}"
```

这个程序能运行，但是有一个重大的性能问题：每次迭代时都会创建一个包含 `["crystal", "ruby", "java"]`的数组。记住：数组字面量主不过是创建数组，把值放进去的一个语法糖。这会在，，在此迭代时都发生一遍。

有两种方法来解决它：

1. 用元组。 如果在程序里写 `{"crystal", "ruby", "java"}` ，它仍然会这么运行，但是因为元组不涉及堆内存，它会快一些，编译器也有更多机会去优化程序。

  ```crystal
  lines_with_language_reference = 0
  while line = gets
    if {"crystal", "ruby", "java"}.any? { |string| line.includes?(string) }
      lines_with_language_reference += 1
    end
  end
  puts "Lines that mention crystal, ruby or java: #{lines_with_language_reference}"
  ```

2. 把数组拖出去，写成一个常量

  ```crystal
  LANGS = ["crystal", "ruby", "java"]

  lines_with_language_reference = 0
  while line = gets
    if LANGS.any? { |string| line.includes?(string) }
      lines_with_language_reference += 1
    end
  end
  puts "Lines that mention crystal, ruby or java: #{lines_with_language_reference}"
  ```

一般推荐写元组。

循环中，显式的数组字面量是一种创建临时对象的方法，同样的规律也适用于方法调用。例如，`Hash#keys` 每次调用都会返回一个包含键的数组。你可以用 `Hash#each_key`, `Hash#has_key?` 或其他方法来替换它。

### 尽可能用结构体

如果你把一个类型声明为 **struct** 而不是 **class**， 创建一个实例只会使用栈内存，这比对内存便宜得多，也不会增加GC压力。

不过你不应该总是用结构体。结构体按值传递。如果你把它传给一个方法，这个方法又修改了它，调用者不会知道有这些修改。这可能会产生bug。最好的办法是对笔可变对象用结构体，尤其对于细小的对象。

例如：

```crystal
# class_vs_struct.cr
require "benchmark"

class PointClass
  getter x
  getter y

  def initialize(@x : Int32, @y : Int32)
  end
end

struct PointStruct
  getter x
  getter y

  def initialize(@x : Int32, @y : Int32)
  end
end

Benchmark.ips do |x|
  x.report("class") { PointClass.new(1, 2) }
  x.report("struct") { PointStruct.new(1, 2) }
end
```

Output:

```
$ crystal run --release class_vs_struct.cr
 class  28.17M (± 2.86%) 15.29× slower
struct 430.82M (± 6.58%)       fastest
```

## 字符串迭代

Crystal 的字符串包含的是以 UTF-8 格式编码的字节。 UTF-8 是变长编码：尽管ASCII字符都是一个字节的， 另一些字符(比如汉字)可能会占据多个字节。因此，用 `String#[]` 索引字符串不是 `O(1)` 操作，因为找到对应位置的字符总是要把之前的字符都解码一遍。虽然 Crystal的 `String` 做了一些优化：如果他知道每个字符都是 ASCII的，那么 `String#[]` 就可以以 `O(1)`实现。然而，这不总是成立。

印制，这样遍历一个字符串不是优化的。相反，它有 `O(n^2)`复杂度：

```crystal
string = ...
while i < string.size
  char = string[i]
  # ...
end
```

上面的写法还有一个问题：计算String的 `size` 也是很慢的，因为它不是字符串中字节的数量(`bytesize`)。不过，只要它被计算过一次，它就会被缓存起来。

要提升性能,要么用迭代方法 (`each_char`, `each_byte`, `each_codepoint`), 要么用更底层的 `Char::Reader`结构体。例如，用 `each_char`：

```crystal
string = ...
string.each_char do |char|
  # ...
end
```
