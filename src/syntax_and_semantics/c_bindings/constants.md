# 常量

你也可以在 `lib` 中声明常量：

```crystal
@[Link("pcre")]
lib PCRE
  INFO_CAPTURECOUNT = 2
end

PCRE::INFO_CAPTURECOUNT #=> 2
```
