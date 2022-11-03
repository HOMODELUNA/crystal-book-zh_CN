# 测试Crystal代码

Crystal 的 [`Spec`模块](https://crystal-lang.org/api/latest/Spec.html)自带一个功能完全的测试库。它提供了描述代码如何运行的语法结构。

启发于 [Rspec](http://rspec.info/)，它包含一种领域特定语言 (DSL)来用类似于英语的方法来描述测试用例。

一个最基本的样例长这样：

```crystal
require "spec"

describe Array do
  describe "#size" do
    it "正确表示数组中元素的数目" do
      [1, 2, 3].size.should eq 3
    end
  end

  describe "#empty?" do
    it "当数组中没有元素时为真" do
      ([] of Int32).empty?.should be_true
    end

    it "当数组中有元素时为假" do
      [1].empty?.should be_false
    end
  end
end
```

## 剖析测试文件

为了使用 spec 模块和 DSL，你需要在测试文件中加入 `require "spec"` 。许多项目都有一个自定义的 [spec helper](#spec-helper)来组织这些包含文件。

具体的测试内容在 `it`块从定义。一个可选的(但强烈建议写) 描述字符串表示它的目的，后面接一个块，包含测试的基本逻辑。

已经定义或预订，但是还没有保证能工作的情形可以以 `pending`定义，而不必是 `it`。 它们不会被运行，但是会在测试报告中以“待实现”的形式表示出来。

`it` 块包含一段代码。运行它，然后指定期望的结果。每个例子可以指定多个期待，但是最好每个块测试一种特定的行为。

当 `spec`被包含时, 每个对象都会含有实例方法 `#should`和 `#should_not`。这些方法被所期待的元素调用。如果期待满足，代码就会继续运行；否则这个测试宣告失败，这个块中后续的代码就不再会运行。

在测试文件中，样例以样例组的格式整理起来。每组样例以 `describe`或 `context`开头。 基本上，一个顶层 `describe`定义一个外部单元(such 比如一个类)的行为。 `describe` 可以更深入地嵌套起来，以指定一个更小单元的行为 (比如一个实例方法)。

单元测试中有个被一直遵守的传统：外部 `describe`是指定类，内部 `describe`指定方法。实例方法以 `#`开头, 类方法以 `.`开头。

为了建立特定的情景——比如 *空数组* 对于 *有元素的数组*—— `context`方法有助于向读者解释这一点。他和 `describe`名字不一样，但是行为一样。

`describe` 和 `context` 用一个元素做描述(往往是一个字符串) ，后带一个块，包含样例或是嵌套的样例组。

## 期望

期待为测试的值设下规范，看测试值是不是*真的*符合预期。

### 等价，同一，类型匹配
这些方法用于创建等价 (`eq`), 同一 (`be`), 类型匹配 (`be_a`), 空 (`be_nil`)方面的期望。

注意，判断同一性用的是 `.same?` 方法，它判断两个对象的 [`#object_id`](https://crystal-lang.org/api/latest/Reference.html#object_id%3AUInt64-instance-method) 是否是一样的。这仅当两个引用指向的是 *同一个对象* ，而非 *等价的两个对象*时，才会成立. 这仅能检查引用类型的对象，不能检查值类型的对象，比如结构体或数字。
```crystal
actual.should eq(expected)    # 当 actual == expected     时匹配
actual.should be(expected)    # 当 actual.same?(expected) 时匹配
actual.should be_a(expected)  # 当 actual.is_a?(expected) 时匹配
actual.should be_nil          # 当 actual.nil?            时匹配
```

### 真性
```crystal
actual.should be_true         # 当 actual == true   时匹配
actual.should be_false        # 当 actual == false  时匹配
actual.should be_truthy       # 当 actual 是真的     时匹配 (不是 nil, false, 或 Pointer.null)
actual.should be_falsey       # 当 actual 是假的     时匹配 (nil, false 或 Pointer.null)
```

### 比较
```crystal
actual.should be <  expected  # 当 actual <  expected 时匹配
actual.should be <= expected  # 当 actual <= expected 时匹配
actual.should be >  expected  # 当 actual >  expected 时匹配
actual.should be >= expected  # 当 actual >= expected 时匹配
```

### 其他的匹配方式
```crystal
actual.should be_close(expected, delta) # 当 actual 在 expected 的 delta 邻域内 时匹配:
                                        #   即 (actual - expected).abs <= delta
actual.should contain(expected)         # 当 actual.includes?(expected)   时匹配
actual.should match(expected)           # 当 actual =~ expected           时匹配
```

### 期待错误

这些标志声明的块当抛出特定异常时匹配。

```crystal
expect_raises(MyError) do
  # 当块抛出 MyError 类型的异常时匹配。
end

expect_raises(MyError, "error message") do
  # 要求块抛出 MyError 类型的异常
  # 并且错误内容包含 "error message"
end

expect_raises(MyError, /error \w{7}/) do
  # 要求块抛出 MyError 类型的异常
  # 并且错误内容匹配于这个表达式
end
```

它们返回那个被挽救的异常，以用于后续的其他期许，如指定这个异常的属性等。

## 运行测试

Crystal 编译器有一个 `spec` 命令来运行测试，同时指定测试范围，裁剪输出格式。 `crystal spec` 会编译运行项目内的所有测试文件。

传统上，测试文件在项目的 `spec/` 目录。测试文件必须以 `_spec.cr` 结尾，来让编译器认识到。

你可以从目录树，单独的文件，文件的某一行来指定测试的范围。

```bash
# 运行匹配于 spec/**/*_spec.cr 的所有文件
crystal spec

# 运行匹配于 spec/my/test/**/*_spec.cr 的所有文件
crystal spec spec/my/test/

# 运行 spec/my/test/file_spec.cr 内的所有测试
crystal spec spec/my/test/file_spec.cr

# 运行 spec/my/test/file_spec.cr 于 14 行定义的一组测试
crystal spec spec/my/test/file_spec.cr:14
```

如果指定的行号是 `describe`或 `context`分区的开头，里面的所有测试例子都会被检验。

基础的格式化器会输出失败样例的文件位置和行号，以便于重试这个错误的例子。

## 测试辅助文件

许多项目都包含一个测试辅助文件，通常叫 `spec/spec_helper.cr`。

这个文件常用于包含`spec` 和其他项目中所需的包含文件。这里也适合定义测试用全局变量和方法，以简化测试过程，减少代码重复。 

```crystal
# spec/spec_helper.cr
require "spec"
require "../src/my_project.cr"

def create_test_object(name)
  project = MyProject.new(option: false)
  object = project.create_object(name)
  object
end

# spec/my_project_spec.cr
require "./spec_helper"

describe "MyProject::Object" do
  it "被创建" do
    object = create_test_object(name)
    object.should_not be_nil
  end
end
```
