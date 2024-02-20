# struct

在`lib`中用`struct` 声明一个C结构体。

```crystal
lib C
  # C 中:
  #
  #  struct TimeZone {
  #    int minutes_west;
  #    int dst_time;
  #  };
  struct TimeZone
    minutes_west : Int32
    dst_time     : Int32
  end
end
```

你也能声明多个同类型的成员变量：

```crystal
lib C
  struct TimeZone
    minutes_west, dst_time : Int32
  end
end
```

递归的结构体如你所想的那样起效：

```crystal
lib C
  struct LinkedListNode
    prev, _next : LinkedListNode*
  end
  
  struct LinkedList
    head : LinkedListNode*
  end
end
```

用 `new`创建结构体实例：

```crystal
tz = C::TimeZone.new
```

这会在栈上分配结构体。

C 结构体刚声明是，所有的成员都会被设为 "零"： 整数和浮点数是零，指针指向地址位置 0，诸如此类。

你可以用 `uninitialized`避免这样的初始化：

```crystal
tz = uninitialized C::TimeZone
tz.minutes_west #=> 无意义值
```

它的成员都可以被读写：

```crystal
tz = C::TimeZone.new
tz.minutes_west = 1
tz.minutes_west #=> 1
```

如果被赋予的值不正好是成员的值，它会尝试匹配 [to_unsafe](to_unsafe.html)方法。

你也可以用类似于[命名参数](../default_and_named_arguments.html)的语法初始化结构体的某些域：

```crystal
tz = C::TimeZone.new minutes_west: 1, dst_time: 2
tz.minutes_west #=> 1
tz.dst_time     #=> 2
```

C 结构体按值(复制地)传入函数的方法，返回时也总是按值返回：

```crystal
def change_it(tz)
  tz.minutes_west = 1
end

tz = C::TimeZone.new
change_it tz
tz.minutes_west #=> 0
```

成员类型的声明参见 [类型语法](../type_grammar.html)。
