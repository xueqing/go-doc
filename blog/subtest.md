# 使用子测试和子基准测试

- [使用子测试和子基准测试](#%e4%bd%bf%e7%94%a8%e5%ad%90%e6%b5%8b%e8%af%95%e5%92%8c%e5%ad%90%e5%9f%ba%e5%87%86%e6%b5%8b%e8%af%95)
  - [介绍](#%e4%bb%8b%e7%bb%8d)
  - [表驱动测试基础](#%e8%a1%a8%e9%a9%b1%e5%8a%a8%e6%b5%8b%e8%af%95%e5%9f%ba%e7%a1%80)
  - [表驱动的基准测试](#%e8%a1%a8%e9%a9%b1%e5%8a%a8%e7%9a%84%e5%9f%ba%e5%87%86%e6%b5%8b%e8%af%95)
  - [使用子测试的表驱动测试](#%e4%bd%bf%e7%94%a8%e5%ad%90%e6%b5%8b%e8%af%95%e7%9a%84%e8%a1%a8%e9%a9%b1%e5%8a%a8%e6%b5%8b%e8%af%95)
  - [运行指定的测试或基准测试](#%e8%bf%90%e8%a1%8c%e6%8c%87%e5%ae%9a%e7%9a%84%e6%b5%8b%e8%af%95%e6%88%96%e5%9f%ba%e5%87%86%e6%b5%8b%e8%af%95)
  - [设置和清理](#%e8%ae%be%e7%bd%ae%e5%92%8c%e6%b8%85%e7%90%86)
  - [并行控制](#%e5%b9%b6%e8%a1%8c%e6%8e%a7%e5%88%b6)
  - [并行运行一组测试](#%e5%b9%b6%e8%a1%8c%e8%bf%90%e8%a1%8c%e4%b8%80%e7%bb%84%e6%b5%8b%e8%af%95)
  - [一组并行测试之后的清理](#%e4%b8%80%e7%bb%84%e5%b9%b6%e8%a1%8c%e6%b5%8b%e8%af%95%e4%b9%8b%e5%90%8e%e7%9a%84%e6%b8%85%e7%90%86)
  - [结论](#%e7%bb%93%e8%ae%ba)
  - [相关文章](#%e7%9b%b8%e5%85%b3%e6%96%87%e7%ab%a0)

参考 [Go 博客——使用子测试和子基准测试](https://blog.golang.org/subtests)学习。

```txt
  作者：Marcel van Lohuizen
  日期：2016/10/3
```

## 介绍

在 Go1.7，testing 包引入了一个 Run 方法，作用于 [T](https://golang.org/pkg/testing/#T.Run) 和 [B](https://golang.org/pkg/testing/#B.Run) 类型，允许创建子测试和子基准测试。子测试和子基准测试的引入使得支持更好的失败处理，从命令行对运行哪个测试的细粒度控制，并行控制，并且经常生成更简单和可维护的代码。

## 表驱动测试基础

在深入细节之前，让我们先讨论用 Go 编写测试的常用方式。可以通过对一个测试用例切片的循环实现一系列相关的检查：

```go
func TestTime(t *testing.T) {
    testCases := []struct {
        gmt  string
        loc  string
        want string
    }{
        {"12:31", "Europe/Zuri", "13:31"},     // incorrect location name
        {"12:31", "America/New_York", "7:31"}, // should be 07:31
        {"08:08", "Australia/Sydney", "18:08"},
    }
    for _, tc := range testCases {
        loc, err := time.LoadLocation(tc.loc)
        if err != nil {
            t.Fatalf("could not load location %q", tc.loc)
        }
        gmt, _ := time.Parse("15:04", tc.gmt)
        if got := gmt.In(loc).Format("15:04"); got != tc.want {
            t.Errorf("In(%s, %s) = %s; want %s", tc.gmt, tc.loc, got, tc.want)
        }
    }
}
```

这个方法常备表驱动测试提及。且相比为每个测试重复相同的代码，此方法减少了大量的重复代码，且使得增加测试用例更加直接。

## 表驱动的基准测试

在 Go1.7 之前，不能为基准测试使用相同的表驱动方法。一个基准测试测试整个函数的性能，因此遍历基准测试只是将它们作为一个整体测试。

一个常用的变通方案是定义单独的顶层基准测试，每个基准测试使用不同参数调用一个公共的函数。比如，在 1.7 之前，strconv 包地狱塔 AppendFloat 的基准测试看起来像这样：

```go
func benchmarkAppendFloat(b *testing.B, f float64, fmt byte, prec, bitSize int) {
    dst := make([]byte, 30)
    b.ResetTimer() // Overkill here, but for illustrative purposes.
    for i := 0; i < b.N; i++ {
        AppendFloat(dst[:0], f, fmt, prec, bitSize)
    }
}

func BenchmarkAppendFloatDecimal(b *testing.B) { benchmarkAppendFloat(b, 33909, 'g', -1, 64) }
func BenchmarkAppendFloat(b *testing.B)        { benchmarkAppendFloat(b, 339.7784, 'g', -1, 64) }
func BenchmarkAppendFloatExp(b *testing.B)     { benchmarkAppendFloat(b, -5.09e75, 'g', -1, 64) }
func BenchmarkAppendFloatNegExp(b *testing.B)  { benchmarkAppendFloat(b, -5.11e-95, 'g', -1, 64) }
func BenchmarkAppendFloatBig(b *testing.B)     { benchmarkAppendFloat(b, 123456789123456789123456789, 'g', -1, 64) }
...
```

Go1.7 可使用 Run 方法，相同的基准测试现在可以表示为一个顶层的基准测试：

```go
func BenchmarkAppendFloat(b *testing.B) {
    benchmarks := []struct{
        name    string
        float   float64
        fmt     byte
        prec    int
        bitSize int
    }{
        {"Decimal", 33909, 'g', -1, 64},
        {"Float", 339.7784, 'g', -1, 64},
        {"Exp", -5.09e75, 'g', -1, 64},
        {"NegExp", -5.11e-95, 'g', -1, 64},
        {"Big", 123456789123456789123456789, 'g', -1, 64},
        ...
    }
    dst := make([]byte, 30)
    for _, bm := range benchmarks {
        b.Run(bm.name, func(b *testing.B) {
            for i := 0; i < b.N; i++ {
                AppendFloat(dst[:0], bm.float, bm.fmt, bm.prec, bm.bitSize)
            }
        })
    }
}
```

每次调用 Run 方法创建一个单独的基准测试。调用 Run 方法的闭包的基准测试函数只允许一次且不被测量。

新代码行数更多，但是更易维护，更易读，且与测试常用的表驱动方法是一致的。此外，现在可以在运行时共享设置代码，同时不再需要重置计时器。

## 使用子测试的表驱动测试

Go1.7 也引入了用于创建子测试的 Run 方法。这个测试是使用子测试对之前的例子重新的版本：

```go
func TestTime(t *testing.T) {
    testCases := []struct {
        gmt  string
        loc  string
        want string
    }{
        {"12:31", "Europe/Zuri", "13:31"},
        {"12:31", "America/New_York", "7:31"},
        {"08:08", "Australia/Sydney", "18:08"},
    }
    for _, tc := range testCases {
        t.Run(fmt.Sprintf("%s in %s", tc.gmt, tc.loc), func(t *testing.T) {
            loc, err := time.LoadLocation(tc.loc)
            if err != nil {
                t.Fatal("could not load location")
            }
            gmt, _ := time.Parse("15:04", tc.gmt)
            if got := gmt.In(loc).Format("15:04"); got != tc.want {
                t.Errorf("got %s; want %s", got, tc.want)
            }
        })
    }
}
```

第一件要注意的事情是两个实现的输出不同。原本的实现打印：

```go
--- FAIL: TestTime (0.00s)
    time_test.go:62: could not load location "Europe/Zuri"
```

即使有两个错误，测试执行终止在对 Fatalf 的调用，且第二个测试永远不会运行。

使用 Run 的实现打印两个错误：

```go
--- FAIL: TestTime (0.00s)
    --- FAIL: TestTime/12:31_in_Europe/Zuri (0.00s)
        time_test.go:84: could not load location
    --- FAIL: TestTime/12:31_in_America/New_York (0.00s)
        time_test.go:88: got 07:31; want 7:31
```

Fatal 及其同属函数导致子测试被跳过，但是不会跳过父测试或后续的子测试。

另外一件要注意的事情是新版本中的错误信息更短。因为子测试的名字唯一标识了一个子测试，因此不再需要在错误信息内部识别该测试。

使用子测试或子基准测试还有其他的益处，下面的部分会阐明。

## 运行指定的测试或基准测试

子测试和子基准测试可以在命令行使用 [`-run 或 -bench 标识`](../command/test_flag.md)选择。两个标识都接收一个斜线分隔的正则表达式列表，匹配了子测试或子基准测试的完整名字的对应部分。

子测试或子基准测试的完整名字是一个斜线分隔的列表，包括自身的名字以及所有父测试的名字，从顶层测试开始。名字是顶层测试和基准测试对应的名字，且第一个参数必须是 Run。为了避免显示和解析问题，名字使用下划线替换空格，且忽视不可打印字符。相同的处理适用于传递给 -run 或 -bench 表示的正则表达式。

一些例子：

使用欧洲时区运行测试：

```sh
$ go test -run=TestTime/"in Europe"
--- FAIL: TestTime (0.00s)
    --- FAIL: TestTime/12:31_in_Europe/Zuri (0.00s)
        time_test.go:85: could not load location
```

只运行时间在午后的测试：

```go
$ go test -run=Time/12:[0-9] -v
=== RUN   TestTime
=== RUN   TestTime/12:31_in_Europe/Zuri
=== RUN   TestTime/12:31_in_America/New_York
--- FAIL: TestTime (0.00s)
    --- FAIL: TestTime/12:31_in_Europe/Zuri (0.00s)
        time_test.go:85: could not load location
    --- FAIL: TestTime/12:31_in_America/New_York (0.00s)
        time_test.go:89: got 07:31; want 7:31
```

可能有点奇怪，使用 -run=TestTime/NewYork 没有匹配任何测试。这是因为出现在位置名字的斜线被当做一个分割符。反之使用：

```go
$ go test -run=TestTime//New_York
--- FAIL: TestTime (0.00s)
    --- FAIL: TestTime/12:31_in_America/New_York (0.00s)
        time_test.go:88: got 07:31; want 7:31
```

注意传递给 -run 的字符串中的 //。时区名字 America/New_York 中的 / 被当做是来自子测试的一个分隔符处理。第一个正则表达式模式 (TestTime) 匹配顶层测试。第二个正则表达式(空字符串)匹配所有，这种情况匹配时间和位置的大洲部。第三部分正则表达式(New_york)匹配位置的城市部分。

把名字中的斜线当做分隔符允许用户重构测试的层次结构，而不用修改名字。它也简化了避免规则。如果这暴露一个问题的话，用户应该避免名字中的斜线，比如使用下划线替代。

一个唯一的序列号被增加到不唯一的测试名字末尾。因此如果子测试没有明显的名字结构，可以只传递一个空字符串给 Run，且子测试可以简单地通过序列号识别。

## 设置和清理

子测试和自己准测试可用于管理公共的设置和清理代码：

```go
func TestFoo(t *testing.T) {
    // <setup code>
    t.Run("A=1", func(t *testing.T) { ... })
    t.Run("A=2", func(t *testing.T) { ... })
    t.Run("B=1", func(t *testing.T) {
        if !test(foo{B:1}) {
            t.Fail()
        }
    })
    // <tear-down code>
}
```

如果任一闭包的子测试运行，设置和清理代码会被允许且最多只运行一次。即使任何子测试调用 Skip、FAIL 或 Fatal 也适用。

## 并行控制

子测试支持细粒度控制并行，为了理解如何用这种方式使用子测试，理解并行测试的语法是重要的。

每个子测试和一个测试函数相关。如果一个测试的测试函数在其 testing.T 实例中调用 Parallel 方法，那么这个测试被称为并行测试。一个并行测试不会和一个顺序测试并发运行，且并行测试的执行被中止直到调用它的测试方法，即父测试返回。-parallel 标识定义了可以并行运行的并行测试的最大数目。

一个测试会阻塞直到它的测试函数返回且它所有的子测试结束。这意味着顺序测试运行的并行测试会在任何其他一连串的顺序测试运行之前完成。

这种行为对使用 Run 创建的测试和顶层测试是一样的。事实上，在底层，顶层测试被实现为一个隐藏的主测试的子测试。

## 并行运行一组测试

上述语义支持并行运行一组测试，这组测试内部是并行的，但不与其他并行测试并行：

```go
func TestGroupedParallel(t *testing.T) {
    for _, tc := range testCases {
        tc := tc // capture range variable
        t.Run(tc.Name, func(t *testing.T) {
            t.Parallel()
            if got := foo(tc.in); got != tc.out {
                t.Errorf("got %v; want %v", got, tc.out)
            }
            ...
        })
    }
}
```

外部测试一直到所有通过 Run 启动的并行测试完成之后才会结束。因此，不会有其他并行测试可以和这些并行测试并行运行。

注意我们需要捕获 range 变量以确保 tc 与正确的实例绑定。

## 一组并行测试之后的清理

在上述例子中，我们在开始其他测试之前使用语义等待一组并行测试结束。相同的技术可用于在一组共享公共资源的并行测试之后清理：

```go
func TestTeardownParallel(t *testing.T) {
    // <setup code>
    // This Run will not return until its parallel subtests complete.
    t.Run("group", func(t *testing.T) {
        t.Run("Test1", parallelTest1)
        t.Run("Test2", parallelTest2)
        t.Run("Test3", parallelTest3)
    })
    // <tear-down code>
}
```

等待一组并行测试的行为和之前的例子是相同的。

## 结论

Go1.7 对子测试和子基准测试的增加允许你用正常的方式编写结构化的测试和基准测试，可以优雅的融入现有的工具。一种思考方式是 testing 包之前的版本有 1 层结构：包级别的测试被组织为一个单独的测试和基准测试的集合。现在这种组织可以递归扩展到这些单独的测试和基准测试。事实上，在实现中，顶层测试和基准测试被作为一个隐藏的主测试和基准测试的子测试和基准测试：这种处理在每一层都是相同的。

对于测试来说，定义这种结构的能力使能细粒度执行指定的测试用例、共享设置和清理，以及更好地控制测试并行。我们很高兴看到人们发现其他用途。享受它！

## 相关文章

- [Go 可测试的示例函数](examples.md)
- [关于覆盖的故事](cover.md)
