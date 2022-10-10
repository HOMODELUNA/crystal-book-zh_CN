# out

考虑 [waitpid](http://www.gnu.org/software/libc/manual/html_node/Process-Completion.html) 这个函数：

```crystal
lib C
  fun waitpid(pid : Int32, status_ptr : Int32*, options : Int32) : Int32
end
```

这个函数的文档说：

```
子程序的状态信息会储存到status_ptr指向的对象，除非status_ptr是空指针。
```

我们可以这样使用它：

```crystal
pid = ...
options = ...
status_ptr = uninitialized Int32

C.waitpid(pid, pointerof(status_ptr), options)
```

这样我们给 `status_ptr`传递了一个指针，让他填充值。

不过有一种简便的方法，那就是 `out`参数：

```crystal
pid = ...
options = ...

C.waitpid(pid, out status_ptr, options)
```

编译器会自动声明一个`Int32`类型的变量 `status_ptr`，因为参数类型是 `Int32*`。

这对任何类型都适用，只要参数是指针类型。 (当然这个函数要真的填充这个变量)。
