# C? Go? cgo!

- [C? Go? cgo!](#c-go-cgo)
  - [介绍](#介绍)
  - [字符串](#字符串)
  - [构建 cgo 包](#构建-cgo-包)
  - [更多 cgo 资源](#更多-cgo-资源)

参考 [Go 博客——C? Go? cgo?](https://blog.golang.org/cgo)学习。

```txt
  作者：Andrew Gerrand
  日期：2011/3/17
```

## 介绍

cgo 支持 Go 包调用 C 代码。假设有一个实现部分特性的 Go 源码文件，cgo 将 Go 和 C 文件结合输出到一个单独的 Go 包。

用一个例子开始，有一个 Go 包提供两个函数——`Random` 和 `Seed`——分别封装了 C 语言的 `random` 和 `srandom` 函数。

```go
package rand

/*
#include <stdlib.h>
*/
import "C"

func Random() int {
  return int(C.random())
}

func Seed(i int) {
  C.srandom(C.uint(i))
}
```

从 import 语句开始看看发生了什么。

`rand` 包导入 `"C"`，但是你会发现标准 Go 库没有这样的包。这是因为 `"C"` 是一个 “伪包”，是 cgo 用来解释链接到 C 的命名空间的特殊名字。

`rand` 包包含四个 `"C"` 的连接：调用 `C.random` 和 `C.srandom`，转换 `C.uint(i)`，以及 `import` 语句。

`Random` 函数调用标准 C 库的 `random` 函数并返回结果。在 C 中，`random` 返回一个 C 类型 `long` 的值，cgo 解释为类型 `C.long`。在包外部被 Go 代码使用结果之前必须使用普通的 Go 类型转换：

```go
func Random() int {
  return int(C.random())
}
```

有一个等价的函数，使用一个临时变量来更加显式地演示这个类型转换：

```go
func Random() int {
  var r C.long = C.random()
  return int(r)
}
```

`Seed` 函数在某种程度上做了相反的事情。它接收一个普通的 Go `int`，转换为 C 的 `unsigned int` 类型，并传递给 C 函数 `srandom`。

```go
func Seed(i int) {
  C.srandom(C.uint(i))
}
```

注意 cgo 知道 `unsigned int` 是 `C.uint`；浏览 [cgo](https://golang.org/cmd/cgo/) 文档查看这些数值类型名称的完整列表。

我们还没解释的这个例子的一个细节就是 `import` 语句上面的注释。

```go
/*
#include <stdlib.h>
*/
import "C"
```

cgo 识别这种注释。任何以 `#cgo` 开头之后跟着一个空格的行被移除；这些是 cgo 的命令。剩下的行用作编译包的 C 部分的头。在这种情况下，这些行只是一个 `#include` 语句，但是它们可以是几乎任意 C 代码。`#cgo` 命令用于在编译包的 C 部分时为编译器和链接器提供标识。

有一个限制：如果你的程序使用任何 `//export` 命令，注释中的 C 代码可能只包含声明(`extern int f();`)而不包含定义(`int f() { return 1; }`)。你可以使用 `//export` 命令使得 C 代码可以访问 Go 函数。

`#cgo` 和 `//export` 命令在 [cgo](https://golang.org/cmd/cgo/) 文档有说明。

## 字符串

和 Go 不同，C 没有显式的字符串类型。C 中的字符串表示为一个以零结束的字符数组。

Go 和 C 的字符串转换使用 `C.CString`、`C.GoString` 和 `C.GoStringN` 函数。这些转换生成字符串数据的副本。

下面的例子实现了一个 `Print` 函数，使用 C 的 `stdio` 库的 `fputs` 函数写字符串到标准输出：

```go
package print

// #include <stdio.h>
// #include <stdlib.h>
import "C"
import "unsafe"

func Print(s string) {
  cs := C.CString(s)
  C.fputs(cs, (*C.FILE)(C.stdout))
  C.free(unsafe.Pointer(cs))
}
```

通过 C 代码分配的内存对 Go 的内存管理器是不可知的。当使用 `C.CString` 创建一个 C 字符串(获取任何 C 内存分配)时，你必须记得使用结束之后通过调用 `C.free` 释放内存。

调用 `C.CString` 返回指针指向一个字符数组的起始位置，因此在函数退出之前，我们将其转化为 `unsafe.Pointer` 并且使用 `C.free` 释放分配的内存。cgo 程序中一个惯用做法是在分配(尤其是之后的代码比一个函数调用复杂)之后紧接着调用 `defer` 进行释放，正如下面重写的 `Print`：

```go
func Print(s string) {
  cs := C.CString(s)
  defer C.free(unsafe.Pointer(cs))
  C.fputs(cs, (*C.FILE)(C.stdout))
}
```

## 构建 cgo 包

要构建 cgo 包，照常使用 `go build` 和 `go install`。go tool 识别特殊的 `"C"` 导入并自动对这些文件使用 cgo.

## 更多 cgo 资源

[cgo 命令](https://golang.org/cmd/cgo/)文档有更多关于 C 伪包和构建过程的细节。Go 树中的 [cgo 示例](https://golang.org/misc/cgo/)演示更多高级的概念。

最后，如果你对这些内部是如何工作的感兴趣，查看 runtime 包的 [cgocall.go](https://golang.org/src/runtime/cgocall.go) 的介绍注释。
