# 不安全的(Unsafe) 代码

语言的这些部分被认为是不安全的：

* 涉及裸指针： [Pointer](http://crystal-lang.org/api/Pointer.html) 类型和 [pointerof](pointerof.html).
* [allocate](new,_initialize_and_allocate.html) 类方法.
* 涉及C绑定
* [声明未初始化的变量](declare_var.html)

"Unsafe" 意味着可能出现内存崩坏，段错误或程序崩溃。例如：

```crystal
a = 1
ptr = pointerof(a)
ptr[100_000] = 2   # 未定义行为，可能出现段错误
```

然而，正常的程序往往用不到手动操纵指针或是未初始化的变量。C函数的安全绑定也往往包括空指针和界限检查。

没有语言 100% 安全：总有部分注定要和底层打交道，与操作系统交互，涉及指针操作。但是只要你建立了抽象，在更高的层面进行操作，并且假设底下的地基是安全的(通过测试和形式证明来证实)，那么你就可以认为整个代码也是安全的。 

