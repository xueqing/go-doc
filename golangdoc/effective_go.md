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
  - [控制结构](#%e6%8e%a7%e5%88%b6%e7%bb%93%e6%9e%84)
    - [if](#if)
    - [重新声明和重新赋值](#%e9%87%8d%e6%96%b0%e5%a3%b0%e6%98%8e%e5%92%8c%e9%87%8d%e6%96%b0%e8%b5%8b%e5%80%bc)
  - [函数](#%e5%87%bd%e6%95%b0)
    - [多返回值](#%e5%a4%9a%e8%bf%94%e5%9b%9e%e5%80%bc)
    - [命名结果参数](#%e5%91%bd%e5%90%8d%e7%bb%93%e6%9e%9c%e5%8f%82%e6%95%b0)
    - [defer](#defer)
  - [数据](#%e6%95%b0%e6%8d%ae)
    - [使用 new 分配](#%e4%bd%bf%e7%94%a8-new-%e5%88%86%e9%85%8d)
    - [构造函数和复合字面量](#%e6%9e%84%e9%80%a0%e5%87%bd%e6%95%b0%e5%92%8c%e5%a4%8d%e5%90%88%e5%ad%97%e9%9d%a2%e9%87%8f)
    - [使用 make 分配](#%e4%bd%bf%e7%94%a8-make-%e5%88%86%e9%85%8d)
    - [数组](#%e6%95%b0%e7%bb%84)
    - [切片](#%e5%88%87%e7%89%87)
    - [二维切片](#%e4%ba%8c%e7%bb%b4%e5%88%87%e7%89%87)
    - [映射](#%e6%98%a0%e5%b0%84)
    - [打印](#%e6%89%93%e5%8d%b0)
    - [追加](#%e8%bf%bd%e5%8a%a0)
  - [初始化](#%e5%88%9d%e5%a7%8b%e5%8c%96)
    - [常数](#%e5%b8%b8%e6%95%b0)

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
    name string // 对象的名字
    value int // 对象的值
}
```

gofmt 会列对齐：

```go
type T struct {
    name    string // 对象的名字
    value   int    // 对象的值
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
包 regexp 实现了正则表达式的一个简单库。

接收正则表达式的语法是:

    正则表达式:
        连接 { '|' 连接 }
    连接:
        { 闭包 }
    闭包:
        项 [ '*' | '+' | '?' ]
    项:
        '^'
        '$'
        '.'
        字符
        '[' [ '^' ] 字符范围 ']'
        '(' 正则表达式 ')'
*/
package regexp
```

如果是一个简单包，包注释可以是简洁的。

```go
// 包 path 实现了功能代码，用于操作斜线分隔的文件名路径。
```

注释不需要额外的格式比如一行星号。生成的输出可能不能显式为固定宽度的字体，因此不要依赖空格对齐——godoc 像 gofmt 一样，会注意对齐问题。注释是无解释的普通文本，因此 HTML 和其他的注解，比如 \_this\_，会逐字重复，不应该使用。godoc 会做的一个调整是按固定宽度的字体显示缩进文本，适用于代码片段。[fmt 包](../golangpkg/fmt.md)对包注释的使用恰到好处。

视上下文而定，godoc 甚至可能不会重新格式化注释，因此确保他们直接看起来是格式好的：使用正确的拼写、标点符号和句子结构，折叠长行等等。

在包内部，任何紧紧出现在顶层声明之前的注释作为该声明的一个文档注释。程序中每个导出的(大写开头的)名字应该有一个文档注释。

文档注释最好是完整的句子，允许不同的自动化显示。第一个句子应该是一个总结句，以声明的名字开头。

```go
// Compile 解析一个正则表达式，且成功时返回一个可用于匹配文本的 Regexp 对象。
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
// 解析表达式失败时返回的错误代码。
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

类似 C，Go 规范的语法使用分号来终止一个语句，但是和 C 不同的是，这些分号不会出现在源文件。反之，词法分析器使用一个简单的规则在扫描时自动插入分号，因此输入文本可免除大部分分号。

规则如下。如果新行之前的最后一个符号是一个标识符(包括像 int 和 float64 的单词)，一个基本字面量，比如一个数字、字符串常量，或者下面的一个符号

```txt
break continue fallthrough return ++ -- ) }
```

词法分析器总是在这个符号之后插入一个分号。这可以概括为，“如果在一个可以结束一句话的符号之后有一个新行，插入一个分号”。

紧挨着在一个右大括号之前出现的分号也可以忽略，因此一个类似下面的语句不需要分号：

```go
go func() {for { dts <- <- src }} ()
```

习惯上，Go 程序只在类似于 for 循环子句中有分号，用于分隔初始化、条件和连续元素。如果你在一行中写多个语句，也需要分号来分隔语句。

插入分号规则的一个结果是你不能将一个控制结构(if/for/switch/select)的左大括号放在下一行。如果你这样做，会在一个大括号之前插入分号，这会导致不想出现的影响。像这样编写代码：

```go
if i < f() {
    g()
}
```

不要像这样：

```go
if i < f()  // 错误!
{           // 错误!
    g()
}
```

## 控制结构

Go 的控制结构和 C 的控制结构相关，但是很不相同。Go 没有 do 或 while 循环，只有一个稍微普遍的 for；switch 更加灵活；if 和 switch 接受一个可选的类似 for 中的初始化语句；break 和 continue 语句使用一个可选的标签来识别从哪里跳出或继续循环；Go 也有新的控制结构，包括 type switch 和多向通讯复用器 select。语法也有一点不同：Go 没有小括号，且控制结构体必须使用大括号分隔。

### if

在 Go 中，一个简单的 if 看起来像这样：

```go
if x > 0 {
    return y
}
```

强制的大括号鼓励将一个简单的 if 语句分为多行。无论如何，这样编写是一个好的风格，尤其是当代码体包含一个控制语句，比如 return 或 break。

因为 if 和 switch 接受一个初始化语句，常见的是用于设置一个局部变量：

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在 Go 的库中，你会发现当一个 if 没有流入下一句——即代码体以 break、continue、goto 或 return 结束——会忽略不需要的 else。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

这是一个常见情形的例子，即代码必须防止一系列错误条件。如果成功的控制流沿着页面向下，而错误出现的时候消除它们时，代码阅读体验更好。因为错误情况倾向于以 return 语句结束，生成的diamante不需要 else 语句。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### 重新声明和重新赋值

## 函数

### 多返回值

Go 其中一个非凡的特性时函数和方法可以返回多个值。这个性质可用于改善 C 程序中的一些笨拙的写法：in-band 错误返回类似 -1 的值表示错误码并修改通过地址传递的参数。

在 C 语言，使用一个负的计数器标记一个写入错误，且错误码隐藏在一个不固定位置。在 Go 语言，`Write` 可以返回一个计数器和一个错误：“是的，你写了一部分但非全部的字节，因为你已经填满了设备”。`os` 包中作用于文件的 `Write` 方法签名：

```go
func (file *File) Write(b []byte) (n int, err error)
```

且如文档所说，当 n 不等于 b 时这个方法返回写入的字节数和一个非空的错误。这是常见的风格；查看错误处理部分获得更多例子。

一个类似的方法不需要传递一个指针给返回值来模拟一个引用参数。下面是一个简单的函数，从一个字节切片的某个位置起捕获一个数字，返回该数字和下一个位置。

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

你可以使用这个方法像下面这样来扫描一个输入切片 `b` 的数字：

```go
for i := 0; i < len(b); {
    x, i = nextInt(b, i)
    fmt.Println(x)
}
```

### 命名结果参数

Go 函数的返回或结果“参数”可以指定名字并作为普通变量使用，就像使用传入参数。当函数开始时，命名的参数被初始化对应类型的零值；如果函数执行一个不带参数的 `return` 语句，返回参数的当前值被作为返回值。

名字不是必须的，但是名字可以使得代码更加简短清晰：名字即是文档。如果我们将 `nextInt` 的结果命名，很显然返回的 `int` 含义。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名的结果会被初始化且绑定在一个简单的 `return`，它们可以既简单又清晰。下面是 `oi.ReadFull` 使用命名结果良好的版本：

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

### defer

Go 的 `defer` 语句安排执行 `defer` 的函数返回之前立即运行一个函数调用(即推迟的函数)。这是一个处理一些场景特别而高效的方式，比如无论函数使用哪条路径返回都必须释放的资源。经典的例子是解锁一个互斥锁或关闭一个文件。

```go
// Contents 将文件内容作为字符串返回。
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // 函数结束时会运行 f.Close。

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // 后面会讨论 append。
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // 如果在这里返回，会关闭 f。
        }
    }
    return string(result), nil // 如果在这里返回，会关闭 f。
}
```

推迟一个类似于 `Close` 的函数调用有两个优点。其一，它保证你永远不会忘记关闭一个文件，如果你之后编辑这个函数增加一个新的返回路径，这是很容易犯的一个错误。其二，它意味着关闭挨着打开操作，这比放在函数末尾更加清晰。

推迟的函数参数(当函数是一个方法时还包括接收者)在执行 `defer` 时计算值，而不是执行调用时计算。除了避免担心在函数执行时修改变量值，这还意味着一个单一的推迟调用可以推迟多个函数执行。这里有一个丑陋的示例。

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

推迟的函数按照 LIFO (后进先出)的顺序执行，因此上述代码函数返回时，会打印“ 4 3 2 1 0”。一个更加合乎情理的例子是使用一个简单的方式来跟踪程序的函数执行。我们可以写一些像这样的简单的跟踪代码：

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// 像这样使用它们:
func a() {
    trace("a")
    defer untrace("a")
    // 做一些事情....
}
```

我们可以利用延迟函数的参数在执行 `defer` 时计算这一事实做的更好。跟踪代码可以设置不跟踪代码的参数。下面的例子

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

打印

```text
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

对于习惯块级别资源管理的其他语言的编程人员，`defer` 可能看起来怪异的，但是它最有趣且强大的应用正来自它不是块级别而是函数级别的事实。在 `panic` 和 `recover` 部分，我们会看到另一个可能使用 `defer` 的例子。

## 数据

### 使用 new 分配

Go 有两种分配原语，即内置函数 `new` 和 `make`。它们做了不同的事情且适用于不同类型，这可能有点难以理解，但是规则很简单。我们首先讨论 `new`。它是一个分配内存的内置函数，但是和一些其他语言的同名函数不同，它不会初始化内存，它只是将内存置零。也就是说，`new(T)` 为类型 T 的新条目分配置零的存储，并返回存储地址(值为类型 T*)。在 Go 的术语中， 它返回一个指针指向一个新分配的类型 T 的零值。

因为 `new` 返回的内存是置零的，当将你的数据结构设计为每个类型的零值都可以直接使用不需要进一步初始化，在安排的时候是很有用的。这意味着数据结构的使用者可以使用 `new` 创建一个对象并正常工作。比如，`bytes.Buffer` 的文档声明“ Buffer 的零值是一个就绪的空缓冲”。类似的，`sync.Mutex` 没有一个显式的构造函数或 `Init` 方法。反之，`sync.Mutex` 的零值被定义为一个未上锁的互斥锁。

“零值是有用的”这一属性可以传递。考虑这个类型声明：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer` 类型的值也是分配或声明时就绪的。在下一个片段中，`p` 和 `v` 都可以正确工作而不用进一步安排。

```go
p := new(SyncedBuffer)  // *SyncedBuffer 类型
var v SyncedBuffer      // SyncedBuffer 类型
```

### 构造函数和复合字面量

有时候零值不够好，且需要一个初始化构造函数，正如下面从 `os` 包衍生的一个例子：

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

这里有很多模板式代码。我们可以使用一个“复合字面量”来简化代码。“复合字面量”是一个表达式，它在每次求值时创建一个新的实例。

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

注意，和 C 不同，返回一个局部变量的地址是完全可以的；和变量相关的存储在函数返回时仍存在。事实上，使用复合字面量的地址在每次求值时分配一个新的实例，因此我们可以合并后面两行代码：

```go
return &File{fd, name, nil, 0}
```

复合字面量的域按顺序放置且必须都要出现。然而，通过显式给域打像 `field:value` 的标签，初始化列表可以按任何顺序出现，且缺失的域会分别使用对应的零值。因此我们可以写

```go
return &File{fd: fd, name: name}
```

作为一个限制性场景，如果一个复合字面量不包含任何域，它会为类型创建零值。表达式 `new(File)` 和 `&File{}` 是等价的。

复合字面量也可用于创建数组、切片和映射，使用索引或合适的键给域打标签，在这些例子中，无论 `Enone`、`Eio` 和 `Einval` 的值是什么，只要它们是唯一的，初始化器都可以工作。

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### 使用 make 分配

回到分配。内置函数 `make(T, args)` 提供一个不同于 `new(T)` 的目的。它只能创建切片、映射和 channel。

### 数组

### 切片

### 二维切片

### 映射

### 打印

### 追加

## 初始化

### 常数
