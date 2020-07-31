# cgo 命令

- [cgo 命令](#cgo-命令)
  - [go 命令使用 cgo](#go-命令使用-cgo)
  - [Go 引用 C](#go-引用-c)
  - [C 引用 Go](#c-引用-go)
  - [传递指针](#传递指针)

参考 [Golang 官网文档](https://golang.org/cmd/cgo/) 学习。

cgo 使能创建可以调用 C 代码的 Go 包。

## go 命令使用 cgo

使用 cgo 编写正常的 Go 代码，导入一个伪包 `"C"`。之后 Go 代码可以引用例如 `C.size_t` 类型、例如 `C.stdout` 变量，或者例如 `C.putchar` 函数。

如果 `"C"` 导入之前紧跟着一个注释，改注释称为序言，当编译包的 C 部分时用作头文件。比如：

```go
// #include <stdio.h>
// #include <errno.h>
import "C"
```

序言可以包含任何 C 代码，包括函数。变量声明和定义。这些之后可以从 Go 代码中引用，就像它们是在包 `"C"` 中定义。序言中声明的所有名称都可以被使用，即使是以小写字母开头。**例外**：序言中的静态变量不能从 Go 代码中引用；静态函数可以。

查看 `$GOROOT/misc/cgo/stdio` 和 `$GOROOT/misc/cgo/gmp` 找到更多例子。查看 [C? Go? Cgo!](https://golang.org/doc/articles/c_go_cgo.html) 找到使用 cgo 的一个介绍文档。

`CFLAGS`、`CPPFLAGS`、`CXXFLAGS`、`FFLAGS` 和 `LDFLAGS` 可以使用伪 `#cgo` 命令在这些注释中定义，用来调整 C、C++ 或 Fortran 的行为。在多个命令中定义的值会被连接在一起。命令可以包含一系列编译约束限制其对满足其中一个约束的系统生效(查看[编译约束](https://golang.org/pkg/go/build/#hdr-Build_Constraints)找到更多约束语法的细节)。例如：

```go
// #cgo CFLAGS: -DPNG_DEBUG=1
// #cgo amd64 386 CFLAGS: -DX86=1
// #cgo LDFLAGS: -lpng
// #include <png.h>
import "C"
```

可选地，`CPPFLAGS` 和 `LDFLAGS` 可以使用 `#cgo pkg-config` 命令通过 pkg-config 工具获得，命令之后跟的包名。比如：

```go
// #cgo pkg-config: png cairo
// #include <png.h>
import "C"
```

默认的 pkg-config 工具可以通过设置 `PKG_CONFIG` 环境变量修改。

出于安全因素，只支持有限的标识，即 `-D`、`-U`、`-I` 和 `-l`。为了支持其他标识，设置 `CGO_CFLAGS_ALLOW` 为匹配新标识的正则表达式。另外也支持禁用标识，设置 `CGO_CFLAGS_DISALLOW` 为匹配禁用标识的正则表达式。在两种情况下，正则表达式必须匹配完整的参数：比如支持 `-mfoo=bar`，使用 `CGO_CFLAGS_ALLOW='-mfoo.*'`，而不是`CGO_CFLAGS_ALLOW='-mfoo'`。类似地，命名变量控制支持 `CPPFLAGS`、`CXXFLAGS`、`FFLAGS` 和 `LDFLAGS`。

也是出于安全考虑，只支持有限的字符，即字母字符和一些符号，比如 `'.'`，这些不会被解释为其他行为。尝试使用禁止的字符会报 `"malformed #cgo argument"` 错。

构建时，`CGO_CFLAGS`、`CGO_CPPFLAGS`、`CGO_CXXFLAGS`、`CGO_FFLAGS` 和 `CGO_LDFLAGS` 环境变量增加到这些命令衍生的标识。作用于包的标识应用使用这些命令设置，而不是环境变量，以便在未修改的环境中构建也会生效。从环境变量得到的标识不受上述的安全限制约束。

一个包中所有的 cgo `CPPFLAGS` 和 `CFLAGS` 命令被连接到一起，并用于编译该包中的 C 文件。一个包中所有的 `CPPFLAGS` 和 `CXXFLAGS` 命令被连接到一起，用于编译该包中的 C++ 文件。一个包中所有的 `CPPFLAGS` 和 `FFLAGS` 命令被连接到一起，用于编译该包中的 Fortran 文件。一个程序所有包中所有的 `LDFLAGS` 命令被连接到一起，链接时使用。所有的 `pkg-config` 命令被连接到一起，同时发送给 pkg-config 增加各自的命令行标识。

解析 cgo 命令时，出现的任何字符串 `${SRCDIR}` 会被替换成包含该源文件的绝对路径。这支持包目录包含的预编译的静态库被正确链接。比如一个包 foo 在目录 `/go/src/foo`：

```go
// #cgo LDFLAGS: -L${SRCDIR}/libs -lfoo
```

会被扩展为：

```go
// #cgo LDFLAGS: -L/go/src/foo/libs -lfoo
```

当 Go tool 看到一个或多个 Go 文件使用特殊导入 `"C"` 时，它会查找该目录下的其他非 Go 文件，并编译这些文件作为 Go 包的一部分。所有 `.c`、`.s`、`.S` 或 `.sx` 文件会被 C 编译器编译。所有 `.cc`、`.cpp` 或 `.cxx` 会被 C++ 编译器编译。所有 `.f`、`.F`、`.for` 或 `.f90` 文件会被 fortran 编译器编译。所有 `.h`、`.hh`、`.hpp` 或 `.hxx` 文件会被单独编译，但是如果这些头文件修改，该包(包含非 Go 源码文件)会被重新编译。注意，修改其他目录的文件不会导致当前目录的包被重编译，因此一个包的所有非 Go 源码应该在包目录，而不是子目录中。默认的 C 和 C++ 编译器也可以使用 CC 和 CXX 环境变量修改；这些环境变量可以包含命令行选项。

cgo 工具在工作系统上原生的编译是默认启用的。交叉编译时默认是禁用的。你可以在运行 go tool 时通过设置 `CGO_ENABLED` 环境变量控制它：设置为 1 是启用 cgo，设置为 0 则是禁用 cgo。如果启用了 cgo，go tool 会设置编译约束 `"cgo"`。因此，如果禁用 cgo，导入 `"C"` 的文件不会被 go tool 编译。(查看[编译约束](https://golang.org/pkg/go/build/#hdr-Build_Constraints)找到更多编译约束的信息)。

交叉编译时，你必须知道一个 C 交叉编译器给 cgo 使用。当使用 `make.bash` 编译工具链时，你可以通过设置通用的 `CC_FOR_TARGET` 或更具体的 `CC_FOR_${GOOD}_${GOARCH}` (比如 `CC_FOR_linux_arm`) 环境变量，或者你可以在任何时候运行 go tool 时设置 `CC` 环境变量。

类似地，`CXX_FOR_TARGET`、`CXX_FOR_${GOOD}_${GOARCH}` 和 `CXX` 环境变量对于 C++ 代码生效。

## Go 引用 C

在 Go 文件内，C 的结构体字段名称是 Go 的关键字时，可以通过增加下换线前缀访问：如果 x 指向一个 C 结构体，包含一个字段名 `"type"`，`x._type` 可以访问该字段。C 结构体字段不能在 Go 中表达的，比如位字段或未对齐的数据，在 Go 结构体中被忽视，通过合适的填充访问下个字段或结构体末尾。

标准的 C 数值类型可通过下面的名称访问：`C.char`、`C.schar` (signed char)、`C.uchar` (unsigned char)、`C.short`、`C.ushort` (unsigned short)、`C.int`、`C.uint` (unsigned int)、`C.long`、`C.ulong` (unsigned long)、`C.longlong` (long long)、`C.ulonglong` (unsigned long long)、`C.float`、`C.double`、`C.complexfloat` (complex float) 和 `C.complexdouble` (complex double)。C 的 `void*` 类型表示为 Go 的 `unsafe.Pointer`。C 的 `__int128_t` 和 `__uint128_t` 表示为 `[16]byte`。

通常会用 Go 的指针类型表示的一些特殊的 C 类型会用一个 `uintptr` 表示。查看下面的“特例”章节。

要直接访问一个结构体、联合或枚举类型，使用 `strcut_`、`union_`或 `enum_` 前缀，比如 `C.strcut_stat`。

C 类型 T 的字节长度通过 `C.sizeof_T` 获得，比如 `C.sizeof_struct_stat`。

一个 C 函数可以在 Go 文件声明，带一个特殊的 `_GoString_` 类型的参数。这个函数可以使用一个普通的 Go 字符串值调用。字符串长度，指向字符串内容的指针，可以通过调用 C 函数获得。

```go
size_t _GoStringLen(_GoString_ s);
const char *_GoStringPtr(_GoString_ s);
```

这些函数只在序言中可以用，不能在其它 C 文件使用。C 代码一定不能修改 `_GoStringPtr` 返回的指针的内容。注意字符串内容可能尾部不包含 NUL 字节。

因为 Go 一般不支持 C 的联合类型，C 的联合类型表示为相同长度的 Go 字节数组。

Go 结构体不能嵌套 C 类型的字段。

Go 代码不能引用非空 C 结构体末尾的零长度的字段。要获取这个字段的地址(唯一可以对该零长度字段进行的操作)，你必须获取结构体的地址，加上结构体的字节长度。

cgo 将 C 类型解释为等价的未导出的 Go 类型。因为这个解释是未导出的，一个 Go 包不应在导出的 API 中暴露 C 类型：一个包中使用的 C 类型不同于另一个包中相同的 C 类型。

任何 C 函数(甚至是 void 函数)可以在一个多赋值上下文调用，以获取返回值(如果有的话)和 C 的错误码变量作为 error(使用 `_` 跳过返回 void 函数的返回值)。比如：

```go
n, err = C.sqrt(-1)
_, err := C.voidFunc()
var n, err = C.sqrt(1)
```

目前不支持调用 C 函数指针，然而你可以声明 Go 变量持有 C 函数指针，在 Go 和 C 之间来回传递。C 代码可以调用从 Go 接收到的函数指针。比如：

```go
package main

// typedef int (*intFunc) ();
//
// int
// bridge_int_func(intFunc f)
// {
//    return f();
// }
//
// int fortytwo()
// {
//    return 42;
// }
import "C"
import "fmt"

func main() {
  f := C.intFunc(C.fortytwo)
  fmt.Println(int(C.bridge_int_func(f)))
  // Output: 42
}
```

在 C 中，一个函数的参数写作固定长度数组，实际上需要一个指针指向数组的第一个元素。C 编译器知道这种调用管理，会调整函数调用，但是 Go 不会。在 Go 中，你必须传递显式传递第一个元素的指针：`C.f(&c.x[0])`。

不支持调用可变参数的 C 函数。可以通过使用一个 C 函数封装规避这种。比如：

```go
package main

// #include <stdio.h>
// #include <stdlib.h>
//
// static void myprint(char* s) {
//   printf("%s\n", s);
// }
import "C"
import "unsafe"

func main() {
  cs := C.CString("Hello from stdio")
  C.myprint(cs)
  C.free(unsafe.Pointer(cs))
}
```

一些特殊的函数通过生成数据副本在 Go 和 C 类型之间转换。在伪 Go 定义中：

```go
// Go string to C string
// The C string is allocated in the C heap using malloc.
// It is the caller's responsibility to arrange for it to be
// freed, such as by calling C.free (be sure to include stdlib.h
// if C.free is needed).
func C.CString(string) *C.char

// Go []byte slice to C array
// The C array is allocated in the C heap using malloc.
// It is the caller's responsibility to arrange for it to be
// freed, such as by calling C.free (be sure to include stdlib.h
// if C.free is needed).
func C.CBytes([]byte) unsafe.Pointer

// C string to Go string
func C.GoString(*C.char) string

// C data with explicit length to Go string
func C.GoStringN(*C.char, C.int) string

// C data with explicit length to Go []byte
func C.GoBytes(unsafe.Pointer, C.int) []byte
```

作为特例，`C.malloc` 不会直接调用 C 库的 `malloc`，而是调用 Go 的辅助函数，它封装了 C 库的 `malloc` 但是确保永远不返回 nil。如果 C 的 `malloc` 提示内存不足，辅助函数会崩溃在程序中，就像 Go 本身运行内存不足。因为 `C.malloc` 不能失败，没有双值形式返回错误码。

## C 引用 Go

可以按照下面的方式导出 Go 函数给 C 代码使用：

```go
//export MyFunction
func MyFunction(arg1, arg2 int, arg3 string) int64 {...}

//export MyFunction2
func MyFunction2(arg1, arg2 int, arg3 string) (int64, *C.char) {...}
```

C 代码可以这样使用：

```c
extern GoInt64 MyFunction(int arg1, int arg2, GoString arg3);
extern struct MyFunction2_return MyFunction2(int arg1, int arg2, GoString arg3);
```

上述函数会在 `_cgo_export.h` 生成的头文件中，前面是从 cgo 输入文件拷贝的序言。带有多返回值的函数映射为返回结构体的函数。

不是所有的 Go 类型可以有实用的方式映射到 C 类型。不支持 Go 结构类型；使用 C 结构体类型。不支持 Go 数组，使用 C 指针。

接收字符串类型参数的 Go 函数可以使用上述的 C 的 `_GoString_` 类型调用。`_GoString_` 会在序言中自动定义。注意 C 代码不会创建这种类型的值；这只在 Go 传递字符串给 C 再传回 Go 有用。

在一个文件中使用 `//export` 会在序言增加一个限制：因为会被拷贝到不同的 C 输出文件，不能包含任何定义，只能包含声明。如果一个文件包含定义和声明，那么两个输出文件会生成重复的符号，链接会失败。为了避免这个，定义必须放在其他文件的序言，或者在 C 源码文件。

## 传递指针
