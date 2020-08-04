# builtin 包

- [builtin 包](#builtin-包)
  - [概述](#概述)
  - [索引](#索引)
    - [常量](#常量)
    - [变量](#变量)
    - [函数 append](#函数-append)
    - [函数 cap](#函数-cap)
    - [函数 close](#函数-close)
    - [函数 complex](#函数-complex)
    - [函数 copy](#函数-copy)
    - [函数 delete](#函数-delete)
    - [函数 imag](#函数-imag)
    - [函数 len](#函数-len)
    - [函数 make](#函数-make)
    - [函数 new](#函数-new)
    - [函数 panic](#函数-panic)
    - [函数 print](#函数-print)
    - [函数 println](#函数-println)
    - [函数 real](#函数-real)
    - [函数 recover](#函数-recover)
    - [类型 ComplexType](#类型-complextype)
    - [类型 FloatType](#类型-floattype)
    - [类型 IntegerType](#类型-integertype)
    - [类型 Type](#类型-type)
    - [类型 Type1](#类型-type1)
    - [类型 bool](#类型-bool)
    - [类型 byte](#类型-byte)
    - [类型 complex128](#类型-complex128)
    - [类型 complex64](#类型-complex64)
    - [类型 error](#类型-error)
    - [类型 float32](#类型-float32)
    - [类型 float64](#类型-float64)
    - [类型 int](#类型-int)
    - [类型 int16](#类型-int16)
    - [类型 int32](#类型-int32)
    - [类型 int64](#类型-int64)
    - [类型 int8](#类型-int8)
    - [类型 rune](#类型-rune)
    - [类型 string](#类型-string)
    - [类型 uint](#类型-uint)
    - [类型 uint16](#类型-uint16)
    - [类型 uint32](#类型-uint32)
    - [类型 uint64](#类型-uint64)
    - [类型 uint8](#类型-uint8)
    - [类型 uintptr](#类型-uintptr)

参考 [Golang 官网文档](https://golang.org/pkg/builtin/) 学习。

## 概述

builtin 包为 Go 预先声明的标识符提供文档。包中记录的条目实际上不在这个包，但是它们的描述在这里，以支持 godoc 展示 Go 的特殊标识符的文档。

## 索引

[参考](https://golang.org/pkg/builtin/#pkg-index)

### 常量

`true` 和 `false` 是两个无类型的布尔值。

```go
const (
    true  = 0 == 0 // Untyped bool.
    false = 0 != 0 // Untyped bool.
)
```

`iota` 是一个预先声明的标识符，在一个(通常是带括号的)常量声明中表示当前常量规范的无类型整型序列号。从 0 开始索引。

```go
const iota = 0 // Untyped int.
```

### 变量

`nil` 是一个预先声明的标识符，表示一个指针、channel、函数、接口、map 或切片类型的零值。

```go
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type
```

### 函数 append

```go
func append(slice []Type, elems ...Type) []Type
```

内置的 `append` 函数追加元素到一个切片的末尾，如果切片有足够容量，目标切片被重组以容纳新元素。如果没有足够容量，底层数组被重新分配。`append` 返回更新的切片。因此需要保存 `append` 的结果，通常是持有的切片本身变量：

```go
slice = append(slice, elem1, elem2)
slice = append(slice, anotherSlice...)
```

作为特例，允许追加一个字符串到一个字节切片，像这样：

```go
slice = append([]byte("hello "), "world"...)
```

### 函数 cap

```go
func cap(v Type) int
```

内置的 `cap` 函数根据 `v` 的类型返回其容量：

```txt
数组：v 中元素的数目(等同于 len(v))。
数组的指针： *v 中元素的数目(等同于 len(v))。
切片：切片重组时切片可以达到的最大长度；如果 v 是 nil，cap(v) 是 0。
channel：channel 缓存容量，以元素为单位；如果 v 是 nil，cap(v) 是 0。
```

对于一些参数，比如一个简单的数组表达式，结果可以是一个常量。查看 Go 语言规范的“[长度和容量](https://golang.org/ref/spec#Length_and_capacity)”章节获取更多细节。

### 函数 close

```go
func close(c chan<- Type)
```

内置的 `close` 函数关闭一个 channel，该 channel 是双向的或者只写的。应该只能由发送者只写，接收者不能执行，且有接收到最后一个发送值之后关闭 channel 的效果。从关闭的 channel `c` 中接收到的最后一个值之后，`c` 的所有接收者都会成功而不会阻塞，返回 channel 元素的零值。形式：

```go
x, ok := <-c
```

对于一个关闭的 channel 会将 `ok` 置为 `false`。

### 函数 complex

```go
func complex(r, i FloatType) ComplexType
```

内置的 `complex` 函数从两个浮点数值构造一个复数值。实部和虚部必须是相同字节长度，即 `float32` 或 `float64`(或者可赋值为这些类型)，且返回值是对应的复数类型(float32 是 complex64，float64 是 complex128)。

### 函数 copy

```go
func copy(dst, src []Type) int
```

内置的 `copy` 函数从源切片拷贝元素到目标切片。(作为特例，也会从字符串拷贝字节到一个字节切片。)源和目标可能重叠。`copy` 返回拷贝的元素数目，即 `len(src)` 和 `len(dst)` 的较小值。

### 函数 delete

```go
func delete(m map[Type]Type1, key Type)
```

内置的 `delete` 函数从 map 删除带有指定键的元素(`m[key]`)。如果 `m` 是 nil 或没有这样的元素，`delete` 不执行操作(no-op)。

### 函数 imag

```go
func imag(c ComplexType) FloatType
```

内置的 `imag` 函数返回复数 `c` 的虚部。返回值是 `c` 的类型对应的浮点类型。

### 函数 len

```go
func len(v Type) int
```

内置的 `len` 函数根据 `v` 的类型返回其长度：

```txt
数组：v 中的元素数目。
数组的指针：*v 中的元素数目(即使 v 是 nil)。
切片/map：v 中的元素数目；如果 v 是 nil，len(v) 是 0。
字符串：v 中的字节数目。
channel：channel 缓存中排队(未读)的元素数目；如果 v 是 nil，len(v) 是 0。
```

对于一些参数，比如字符串字面量或一个简单的数组表达式，结果可以是一个常量。查看 Go 语言规范的“[长度和容量](https://golang.org/ref/spec#Length_and_capacity)”章节获取更多细节。

### 函数 make

```go
func make(t Type, size ...IntegerType) Type
```

内置的 `make` 函数为切片、map 或 chan (只有这些类型) 类型的对象分配和初始化。像 `new` ，第一个参数是类型，而不是一个值。和 `new` 不同，`make` 的返回类型和第一个参数的类型相同，而不是该参数类型的一个指针。结果的规范取决于类型：

```txt
切片：size 指定长度。切片容量和长度相等，可以提供第二个整数参数指定不同的容量；必须不小于长度。比如，make([]int, 0, 10) 分配一个底层数组大小是 10，返回一个切片长度是 0，容量是 10 且以这个底层数组支持的切片。
map：分配一个空的 map，带有足够空间保存指定数目的元素。size 可以忽略，这种情况下分配一个小的起始大小。
channe：使用指定的缓存容量初始化 channel 的缓存。如果是 0，或者忽略 size，channel 是不带缓存的。
```

### 函数 new

```go
func new(Type) *Type
```

内置的 `new` 函数分配内存，第一个参数是类型，而不是一个值，且返回值是一个指针，指向新分配的该类型的零值。

### 函数 panic

```go
func panic(v interface{})
```

内置的 `panic` 函数停止当前 goroutine 的正常指向。当一个函数 `F` 调用 `panic`，`F` 的正常执行立刻停止。`F` 中 defer 执行的所有函数以通常的方式运行，然后 `F` 返回给它的调用者。对于调用者 `G`，调用 `F` 像是调用 `panic`，终止 `F` 的执行，运行所有 defer 的函数。这种过程继续直到正在执行的 goroutine 的所有函数按照逆序停。这时，程序终止返回一个非零的退出码。这个终止序列称为“恐慌(panicking)”，可由内置的 `recover` 函数控制。

### 函数 print

```go
func print(args ...Type)
```

内置的 `print` 函数按照具体实现的方式格式化它的参数，并将结果写到标准错误。`print` 对于引导(bootstrap)和调试是有用的；不能保证保留这个语言。

### 函数 println

```go
func println(args ...Type)
```

内置的 `println` 函数按照具体实现的方式格式化它的参数，并将结果写到标准错误。参数之间总是会增加空格，并在最后追加一个新行。`println` 对于引导(bootstrap)和调试是有用的；不能保证保留这个语言。

### 函数 real

```go
func real(c ComplexType) FloatType
```

内置的 `real` 函数返回复数 `c` 的实部。返回值是 `c` 的类型对应的浮点类型。

### 函数 recover

```go
func recover() interface{}
```

内置的 `recover` 函数支持一个程序管理一个“恐慌”的 goroutine 的行为。在一个 defer 的函数(但不是它调用的任何函数)内执行调用 `recover`，通过恢复正常执行并获取传递给 `panic` 调用的错误码来停止“恐慌”序列。在这种情况下，或当 goroutine 没有“恐慌”时，或者提供给 `panic` 的参数是 nil，`recover` 会返回 nil。因此 `recover` 的返回值报告了 goroutine 是否是“恐慌”的。

### 类型 ComplexType

`ComplexType` 在此只是为了文档。它是复数类型的代替：`complex64` 或者 `complex128`。

```go
type ComplexType complex64
```

### 类型 FloatType

`FloatType` 在此只是作为文档。它是浮点类型代替：`float32` 或者 `float64`。

```go
type FloatType float32
```

### 类型 IntegerType

`IntegerType` 在此只是作为文档。它是整型的代替：`int`、`uint`、`int8` 等。

```go
type IntegerType int
```

### 类型 Type

`Type` 在此只是作为文档。它是任何 Go 类型的代替，表示任何指定的函数调用的相同类型。

```go
type Type int
```

### 类型 Type1

`Type1` 在此只是作为文档。它是任何 Go 类型的代替，表示任何指定的函数调用的相同类型。

```go
type Type1 int
```

### 类型 bool

`bool` 是布尔值的集合，`true` 和 `false`。

```go
type bool bool
```

### 类型 byte

`byte` 是 `uint8` 的别名，且在所有方面都等同于 `uint8`。按照惯例，它用于区分字节值和 8 位的无符号整数值。

```go
type byte = uint8
```

### 类型 complex128

`complex128` 是所有带 `float64` 的实部和虚部的复数集。

```go
type complex128 complex128
```

### 类型 complex64

`complex64` 是所有带 `float32` 的实部和虚部的复数集。

```go
type complex64 complex64
```

### 类型 error

内置的 `error` 接口类型是传统的接口，用于表示一个错误条件，`nil` 值表示没有错误。

```go
type error interface {
    Error() string
}
```

### 类型 float32

`float32` 是所有 IEEE-754 32 位浮点数集。

```go
type float32 float32
```

### 类型 float64

`float64` 是所有 IEEE-754 64 位浮点数集。

```go
type float64 float64
```

### 类型 int

`int` 是一个有符号整数类型，至少 32 位。但它是一个不同的类型，也不是 `int32` 的别名。

```go
type int int
```

### 类型 int16

`int16` 是有符号的 16 位整数集。范围：-32768-32767。

```go
type int16 int16
```

### 类型 int32

`int32` 是有符号的 32 位整数集。范围：-2147483648-2147483647。

```go
type int32 int32
```

### 类型 int64

`int64` 是有符号的 64 位整数集。范围：-9223372036854775808-9223372036854775807。

```go
type int64 int64
```

### 类型 int8

`int8` 是有符号的 8 位整数集。范围：-128-127。

```go
type int8 int8
```

### 类型 rune

`rune` 是 `int32` 的别名，且在所有方面都等同于 `int32`。按照惯例，它用于区分字符值和整数值。

```go
type rune = int32
```

### 类型 string

`string` 是所有 8 位字节的字符串集，传统但是必须表示 UTF-8 编码的文本。一个字符串可以为空，但不能是 nil。字符串类型的值不可改变的。

```go
type string string
```

### 类型 uint

`uint` 是无符号整数类型，至少 32 位。但它是一个不同的类型，也不是 `uint32` 的别名。

```go
type uint uint
```

### 类型 uint16

`uint16` 是所有无符号的 16 位整数集。范围：0-65535。

```go
type uint16 uint16
```

### 类型 uint32

`uint32` 是所有无符号的 32 位整数集。范围：0-4294967295。

```go
type uint32 uint32
```

### 类型 uint64

`uint64` 是所有无符号的 64 位整数集。范围：0-18446744073709551615。

```go
type uint64 uint64
```

### 类型 uint8

`uint8` 是所有无符号的 8 位整数集。范围：0-255。

```go
type uint8 uint8
```

### 类型 uintptr

`uintptr` 是一个整数类型，足够大可以支持所有位数模式的指针。

```go
type uintptr uintptr
```
