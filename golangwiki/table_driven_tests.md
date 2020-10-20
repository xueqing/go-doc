# 表驱动测试

- [表驱动测试](#%e8%a1%a8%e9%a9%b1%e5%8a%a8%e6%b5%8b%e8%af%95)
  - [介绍](#%e4%bb%8b%e7%bb%8d)
  - [表驱动测试的例子](#%e8%a1%a8%e9%a9%b1%e5%8a%a8%e6%b5%8b%e8%af%95%e7%9a%84%e4%be%8b%e5%ad%90)
  - [参考](#%e5%8f%82%e8%80%83)

参考 [Go 维基官网——表驱动测试](https://github.com/golang/go/wiki/TableDrivenTests)学习。

原网页由 Martin Tournoij 在 2018/4/11 编辑。[第 3 次修订](https://github.com/golang/go/wiki/TableDrivenTests/_history)。

## 介绍

编写好的测试不是烦琐的，在很多情况下大量领域可以使用表驱动测试覆盖：每个表条目是一个包含输入和预期结果的测试用例，而且有时候包含一些额外的信息，比如测试名称，使得测试输出易于阅读。如果你曾经发现自己在编写测试时使用拷贝和粘贴，考虑是否重构为表驱动测试，或者把拷贝的代码放在一个辅助函数可能是一个更好的选择。

给定一个测试用例表，真正的测试简单地迭代遍历所有表条目，且未每个条目执行必要的测试。测试代码只编写一次且被分摊到所有的表条目，因此精心编写一个带有好的错误消息的测试是有意义的。

## 表驱动测试的例子

这里是一个来自 [`fmt` 包](http://golang.org/pkg/fmt/)测试代码的好例子：

```go
var flagtests = []struct {
  in  string
  out string
}{
  {"%a", "[%a]"},
  {"%-a", "[%-a]"},
  {"%+a", "[%+a]"},
  {"%#a", "[%#a]"},
  {"% a", "[% a]"},
  {"%0a", "[%0a]"},
  {"%1.2a", "[%1.2a]"},
  {"%-1.2a", "[%-1.2a]"},
  {"%+1.2a", "[%+1.2a]"},
  {"%-+1.2a", "[%+-1.2a]"},
  {"%-+1.2abc", "[%+-1.2a]bc"},
  {"%-1.2abc", "[%-1.2a]bc"},
}
func TestFlagParser(t *testing.T) {
  var flagprinter flagPrinter
  for _, tt := range flagtests {
    t.Run(tt.in, func(t *testing.T) {
      s := Sprintf(tt.in, &flagprinter)
      if s != tt.out {
        t.Errorf("got %q, want %q", s, tt.out)
      }
    })
  }
}
```

注意使用 `t.Errorf` 提供的详细的错误消息：提供了函数结果和预期结果；输入是子测试的名字。当测试失败时，哪个错误失败以及为什么失败是显然的，甚至不用阅读测试代码。

`t.Errorf` 调用不是一个断言。即使打印一个错误日志，测试仍会继续。比如，当使用整数输入测试一些代码时，知道函数对所有输入失败，还是只对奇数失败，或者是对 2 的幂失败是有意义的。

## 参考

- [如何编写 Go 代码——测试](golangdoc/code.md#测试)
- [常见问题解答——断言](golangdoc/faq.md#为什么-Go-没有断言)
- [常见问题解答——测试框架](golangdoc/faq.md#我最喜欢的测试辅助函数在哪里)
- [testing 包](golangpkg/testing.md)
