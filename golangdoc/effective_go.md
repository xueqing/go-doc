# 实效 Go 编程

- [实效 Go 编程](#%e5%ae%9e%e6%95%88-go-%e7%bc%96%e7%a8%8b)
  - [介绍](#%e4%bb%8b%e7%bb%8d)
    - [例子](#%e4%be%8b%e5%ad%90)
  - [格式化](#%e6%a0%bc%e5%bc%8f%e5%8c%96)
  - [注释](#%e6%b3%a8%e9%87%8a)
  - [名字](#%e5%90%8d%e5%ad%97)
    - [包名](#%e5%8c%85%e5%90%8d)
    - [getter](#getter)
    - [接口名](#%e6%8e%a5%e5%8f%a3%e5%90%8d)
    - [驼峰](#%e9%a9%bc%e5%b3%b0)
  - [分号](#%e5%88%86%e5%8f%b7)

参考 [Golang 官网文档——Effective Go](https://golang.org/doc/effective_go.html) 学习。

## 介绍

Go 是一门新语言。虽然它从现有语言借鉴了想法，但是它有独特的属性使得实效的 Go 程序和使用其他语言编写的程序特点不同。直接将一个 C++ 或 Java 程序翻译成 Go 不太可能生成令人满意的结果——Java 程序是用 Java 写的，而不是 Go。另一方面，从 Go 的角度思考一个问题可能产生一个成功的但是完全不同的程序。换句话说，要写好 Go，理解它的特性和惯用语是很重要的。而且，了解用 Go 编程已有的惯例，比如命名、格式化、程序结构等等也很重要，以便你编写的程序容易被其他 Go 开发人员理解。

此文档给出关于编写清晰、惯用的 Go 代码的建议。它增补了[语言规范](https://golang.org/ref/spec)、[Go 语言之旅](https://tour.golang.org/)以及[如何编写 Go 代码](code.md)，所有这些你应该先阅读。

### 例子

[Go 包源码](https://golang.org/src/)用于作为和核心库，也作为如何使用语言的例子。此外，很多包包含可工作的、自包含的可执行例子，你可以直接从 [golang.org](https://golang.org/) 网站运行，比如[这个网站](https://golang.org/pkg/strings/#example_Map)(如果需要，点击单词 “Example” 打开它)。如果你对于如何处理一个问题或一些东西是如何实现的有疑问，这个库中的文档、代码和例子可以给出答案、思想和背景。

## 格式化

格式化问题是最有争议却最不重要的。人们可以选择不同的格式风格，但是如果每个人遵循相同的风格，那么人们不需要且可以花费更少的时间在这个问题上是更好的。问题是如何在没有一个长长的风格规范指南的情况下实现这个想法。

对于 Go，我们使用了一种特别的方法且交由机器注意大多数格式化问题。gofmt 程序(也可以通过 `go fmt` 使用，它作用于包级别而不是源文件级别)读入一个 Go 程序并且以标准的风格缩进、垂直对齐、保持或者需要的话重新格式化注释，然后发出源文件。如果你想要知道如何处理一些新的格式场景，运行 gofmt；如果答案看起来不正确，重新组织你的程序(或提出一个 gofmt 的错误)，不要绕过这个问题。

作为一个例子，不必花时间对结构体的域做注释对齐。gofmt 将会为你做这些。给出一个声明

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

gofmt 会列对齐：

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

标准库中所有的 Go 代码都已经使用 gofmt 格式化过。

还有一些格式化细节。非常简洁：

```txt
缩进
  我们使用 tab 键缩进，且 gofmt 默认使用 tab 键。只在必要的时候使用空格。
行长度
  Go 没有行长度限制。不要担心溢出穿孔卡片。如果感觉一行太长，包裹它并使用额外的 tab 键缩进
括号
  Go 比 C 和 Java 需要更少的括号：控制结构 (if/for/switch) 的语法没有括号。同时，操作符优先级层次更短更清晰。
  因此不像其他语言， `x<<8 + y<<16` 就是空格暗示的含义。
```

## 注释

Go 提供 C-风格的块注释 /**/ 和 C++-风格的行注释 //。行注释是常态；块注释大多出现在包注释，但是在一个表达式内部或禁用大段代码是有用的。

godoc 程序，也是 web 服务器，处理 Go 源文件以提取关于包内容的文档。在顶层声明之前出现的注释，中间没有新行，和声明一起提取作为该元素的解释文本。这些注释的本性和风格决定了 godoc 生成的文档的质量。

每个包应该有一个包注释，即 package 语句之前的一个块注释。对于多文件的包，包注释只需要出现在一个文件，且每个文件都可以看到。包注释应该介绍包并提供和包有关的信息作为一个整体。它会先出现在 godoc 页面，并且应该设置后面的详细文档。

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果是一个简单包，包注释可以是简洁的。

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

注释不需要额外的格式比如一行星号。生成的输出可能不能显式为固定宽度的字体，因此不要依赖空格对齐——godoc 像 gofmt 一样，会注意对齐问题。注释是无解释的普通文本，因此 HTML 和其他的注解，比如 \_this\_，会逐字重复，不应该使用。godoc 会做的一个调整是按固定宽度的字体显示缩进文本，适用于代码片段。[fmt 包](../golangpkg/fmt.md)对包注释的使用恰到好处。

视上下文而定，godoc 甚至可能不会重新格式化注释，因此确保他们直接看起来是格式好的：使用正确的拼写、标点符号和句子结构，折叠长行等等。

在包内部，任何紧紧出现在顶层声明之前的注释作为该声明的一个文档注释。程序中每个导出的(大写开头的)名字应该有一个文档注释。

文档注释最好是完整的句子，允许不同的自动化显示。第一个句子应该是一个总结句，以声明的名字开头。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每个文档注释以描述的元素名字开头，你可以使用 [go](../command/README.md) 工具的 [doc](../command/show_doc.md) 子命令并通过 grep 运行输出。设想你不能急的 “Compile” 名字但是正在查找正则表达式的解析函数，因此你运行命令：

```sh
go doc -all regexp | grep -i parse
```

如果包内所有的文档注释以“这个函数……”开头，grep 不会帮助你记得那个名字。但是因为包的每个文档注释以名字开始，你会看到类似下面的内容，这会回忆起你正在寻找的单词。

```go
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
```

Go 的声明语法允许分组声明。一个单一的文档注释可以介绍一组相关的常量或变量。因为显示了整个声明，这样的注释通常是敷衍了事的。

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

分组也可以指示元素之间的关系，比如被一个所保护的变量集合的事实。

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

## 名字

Go 中的名字和其他语言中的一样重要。它们甚至有语义影响：一个名字在包外的可见性取决于它的第一个字母是否是大写。因此值得花费一些时间讨论 Go 编程中的命名惯例。

### 包名

当导入一个包时，包名成为这些内容的一个访问器。在 `import "bytes"` 之后，导入包可以讨论 `bytes.buffer`。每个使用该包的人可以使用相同的名字来引用包内容是有帮助的，这意味着包名应该是好的：短、简明、引起共鸣的。按照惯例，包使用小写的、单一单词的名字；不应该需要使用下划线或驼峰。Err 就是简洁的，因为每个人使用你的包都会输入那个名字。并且不予担心与先前的冲突。包名只是导入的默认名字；它不需要在所有源码范围内唯一，并且在极少冲突的情况下，导入包可选择一个不同的名字在局部使用。无论如何，混淆是稀少的，因为这个导入的文件名只决定正在使用的包。

另外一个惯例是包名是源路径的基础名；在 src/encoding/base64 中的包作为 “encoding/base64” 导入，但名字是 base64，而不是 encoding_base64 或者 encodingBase64。

包的导入者将会使用包名来引用它的内容，因此包中导出的名字可使用这个事实来避免停顿。(不要使用 `import .` 符号，这可以简化必须在被测试包之外的测试，但应该被避免。)比如， bufio 包中的带缓冲的 reader 类型叫做 Reader，而不是 BufReader，因为使用者看到的是 bufio.Reader，这是一个更加清晰简洁的名字。此外，因为导入的实体总是用包名处理，bufio.Reader 和 io.Reader 不会冲突。类似的，生成 ring.Ring 实例的函数——这是 Go 中构造函数的定义——通常会使用 NewRing 调用，但是因为 Ring 是这个包导出的唯一类型，且这个包叫做 ring，这个函数只用 New 调用，这个包的使用者看到的是 ring.New。使用包结构来帮助你选择好名字。

另外一个简单的例子是 once.Do；once.Do(setup) 读着不错，并且不会被写做 once.DoOrWaitUntilDone(setup) 而有改善。长名字不会自动使得东西更易读。一个有用的文档注释通常比一个特别长的名字更有价值。

### getter

Go 不提供对 getter 和 setter 的自动支持。自己提供 getter 和 setter 是没有问题的，且通常这样做事合适的。但是将 Get 放在 getter 名字中既不是惯例也非必要的。如果你有一个域叫 owner(小写的，不导出)，它的 getter 方法应叫做 Owner(大写，导出的)，而不是 GetOwner。使用大写名字导出可以区分域名和方法名。如果有必要，一个 setter 方法可能叫做 SetOwner。两个名字实际上也是易读的：

```go
owner := obj.Owner()
if owner != user {
  obj.SetOwner(user)
}
```

### 接口名

按照惯例，一个方法的接口用方法名和一个 -er 后缀或类似的修改器命名，用以构造一个代理名词：Reader，Writer，Formatter，CloseNotifier 等。

有许多类似的名字，且尊重这些名字及其捕获的函数名是富有成效的，Read，Write，Close，Flush，String 等等有规范的签名和含义。为了避免混淆，除非方法具有相同的签名和含义，不要使用上述这些名字给方法命名。相反地，如果你的类型实现的方法与一个熟悉的类型的方法有相同的含义，使用这个相同的名字和签名；将你的字符串转换方法命名为 String 而不是 ToString。

### 驼峰

最后，Go 的惯例是使用 MixedCaps 或 mixedCaps 而不是下划线来写多单词的名字。

## 分号
