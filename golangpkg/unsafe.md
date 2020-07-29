# unsafe 包

- [unsafe 包](#unsafe-包)
  - [概述](#概述)
  - [索引](#索引)
    - [函数 Alignof](#函数-alignof)
    - [函数 Offsetof](#函数-offsetof)
    - [函数 Sizeof](#函数-sizeof)
    - [类型 ArbitraryType](#类型-arbitrarytype)
    - [类型 Pointer](#类型-pointer)
      - [1. `*T1` 转为 Pointer，再转为 `*T2`](#1-t1-转为-pointer再转为-t2)
      - [2. Pointer 转为 uintptr (不能转回 Pointer)](#2-pointer-转为-uintptr-不能转回-pointer)
      - [3. Pointer 转为 uintptr，算术运算后转回 Pointer](#3-pointer-转为-uintptr算术运算后转回-pointer)
      - [4. 调用 `syscall.Syscall` 时将 Pointer 转为 uintptr](#4-调用-syscallsyscall-时将-pointer-转为-uintptr)
      - [5. `reflect.Value.Pointer` 或 `reflect.Value.UnsafeAddr` 的结果从 uintptr 转为 Pointer](#5-reflectvaluepointer-或-reflectvalueunsafeaddr-的结果从-uintptr-转为-pointer)
      - [6. `reflect.SliceHeader` 或 `reflect.StringHeader` Data 域和 Pointer 的互相转换](#6-reflectsliceheader-或-reflectstringheader-data-域和-pointer-的互相转换)

参考 [Golang 官网文档](https://golang.org/pkg/unsafe/) 学习。

导入语句：`import "unsafe"`

## 概述

unsafe 包包含的操作围绕 Go 程序的类型安全。

导入 unsafe 的包可能是不可移植的，且不受 Go 1 兼容性指南的约束。

## 索引

[参考](https://golang.org/pkg/unsafe/#pkg-index)

### 函数 Alignof

```go
func Alignof(x ArbitraryType) uintptr
```

Alignof 接收任意类型的表达式 x，返回一个假设变量 v 所需的对齐，其中 v 类似通过 `var v = x` 声明。返回值是最大值 m，满足 v 的地址对 m 取模之后总是 0。它等同于 `reflect.TypeOf(x).Align()` 的返回值。作为特例，如果一个变量 s 是结构体类型，且 f 是结构体内的一个域，那么 `Alignof(s.f)` 会返回结构体内部该类型域所需的对齐数。这种情况等同于 `reflect.TypeOf(s.f).FieldAlign()`。Alignof 的返回值是一个 Go 常量。

### 函数 Offsetof

```go
func Offsetof(x ArbitraryType) uintptr
```

Offsetof 返回 x 代表的结构体内的域的偏移量，x 必须是 `structValue.field` 的形式。Offsetof 返回结构体起始位置到域的起始位置之间的字节数。Offsetof 的返回值是一个 Go 常量。

### 函数 Sizeof

```go
func Sizeof(x ArbitraryType) uintptr
```

Sizeof 接收任意类型的表达式 x，返回一个假设变量 v 的字节数，其中 v 类似通过 `var v = x` 声明。这个大小不包含可能被 x 引用的任何内存。比如，如果 x 是一个切片，Sizeof 返回切片描述符的大小，而不是切片引用的内存大小。Sizeof 的返回值是一个 Go 常量。

### 类型 ArbitraryType

```go
type ArbitraryType int
```

ArbitraryType 在这里仅用于文档，实际上并不是 unsafe 包的一部分。它表示一个任意 Go 表达式的类型。

### 类型 Pointer

```go
type Pointer *ArbitraryType
```

Pointer 表示一个任意类型的指针。类型 Pointer 有 4 中其他类型不支持的特殊操作：

- 一个任意类型的指针可以转为一个 Pointer
- 一个 Pointer 可以转为一个任意类型的指针
- 一个 uintptr 可以转为一个 Pointer
- 一个 Pointer 可以转为一个 uintptr

因此 Pointer 支持程序欺骗类型系统，从而读写任意内存。应该极度小心地使用 Pointer。

下面有关 Pointer 的模式是有效的。不使用这些模式的代码可能现在或者将来是无效的。下面有效的模式即使有效，也有重要的注意事项。

运行 `go vet` 可以帮助查找和这些模式不一致的 Pointer 用法，但是 `go vet` 不能保证代码是有效的。

#### 1. `*T1` 转为 Pointer，再转为 `*T2`

假定 T2 字节长度不大于 T1，而且二者使用相同的内存结构，这个转化支持将一个类型的数据重新解读为另一个类型的数据。一个示例就是 math.Float64bits 的实现：

```go
func Float64bits(f float64) uint64 {
  return *(*uint64)(unsafe.Pointer(&f))
}
```

#### 2. Pointer 转为 uintptr (不能转回 Pointer)

Pointer 转换为 uintptr 会生成一个值指向的内存地址，该值是一个整数。这种 uinptr 通常的用法是打印值。

uinptr 转回 Pointer 一般是无效的。

一个 uintptr 是一个整数，不是一个引用。Pointer 转为 uintptr 生成一个整数值，该值和指针含义无关。即使这个 uintptr 持有某个对象的地址，当对象移动时，垃圾收集器(GC) 不会更新 uintptr 的值，且 uintptr 不能一直保持对象避免被释放。

下面的模式枚举了有效的 uintptr 到 Pointer 的转换。

#### 3. Pointer 转为 uintptr，算术运算后转回 Pointer

如果 Pointer p 指向一个分配好的对象，可以通过转为 uintptr，增加一个偏移量，再转回 Pointer 来操作对象更多的内存。

```go
p = unsafe.Pointer(uintptr(p) + offset)
```

这种模式最常见的用法是访问一个结构体的域或者一个数组的元素：

```go
// 等同于 f := unsafe.Pointer(&s.f)
f := unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f))

// 等同于 e := unsafe.Pointer(&x[i])
e := unsafe.Pointer(uintptr(unsafe.Pointer(&x[0])) + i*unsafe.Sizeof(x[0]))
```

用这种方式对一个指针增加和减少偏移量都是有效的。使用 `&^` 对指针取证也是有效的，这通信是用于对齐。在所有情况下，结果必须继续指向原来的分配对象。

和 C 不同，一个指针前进到超出原有分配的末尾是无效的：

```go
// 无效：end 指向分配空间的外部
var s string
end = unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Sizeof(s))

// 无效：end 指向分配空间的外部
b := make([]byte, n)
end = unsafe.Pointer(uintptr(unsafe.Pointer(&b[0])) + uintptr(n))
```

注意两个转换必须出现在同一个表达式，转换之间只有算术运算：

```go
// 无效：uintptr 在转回 Pointer 之前不能保存在一个变量
u := uintptr(p)
p = unsafe.Pointer(u + offset)
```

注意指针必须指向一个分配好的对象，因此不能是 nil。

```go
// 无效：转换一个 nil 指针
u := unsafe.Pointer(nil)
p := unsafe.Pointer(uintptr(u) + offset)
```

#### 4. 调用 `syscall.Syscall` 时将 Pointer 转为 uintptr

syscall 包中的 Syscall 函数直接传递 uintptr 参数给操作系统，然后操作系统会依据调用的细节，重新将 uinptr 解释为指针。也就是说，系统调用的实现隐式地将一些参数从 uintptr 转回指针。

如果一个指针参数必须被转为 uintptr 作为参数使用，那么这个转换必须出现在调用表达式本身：

```go
syscall.Syscall(SYS_READ, uintprt(fd), uintptr(unsafe.Pointer(p)), uintptr(n))
```

编译器处理函数调用的参数列表中 Pointer 到 uintptr 的转换。如果有的话，实现在汇编层保留引用的分配对象不被移动直到调用完成，即使在调用期间不再需要这个对象。

为了编译器识别这种模式，转换必须出现在参数列表：

```go
// 无效：uintptr 在系统调用期间，隐式转回 Pointer 之前不能存在一个变量中
u := uintptr(unsafe.Pointer(p))
syscall.Syscall(SYS_READ, uintptr(fd), u, uintptr(n))
```

#### 5. `reflect.Value.Pointer` 或 `reflect.Value.UnsafeAddr` 的结果从 uintptr 转为 Pointer

reflect 包的 `Value` 方法将 `Pointer` 和 `UnsafeAddr` 返回类型定义为 uintptr 而不是 unsafe.Pointer，避免调用者在没有导入 `unsafe` 包时将结果修改为任意类型。然而，这意味着这个结果是不稳定的，必须在调用之后，在同一个表达式立即转为 Pointer：

```go
p := (*int)(unsafe.Pointer(reflect.Valueof(new(int)).Pointer()))
```

正如上述例子，在转换之前保存结果是无效的：

```go
// 无效：在转回 Pointer 之前不能保存 uintptr 到一个变量
u := reflect.Valueof(new(int)).Pointer
p := (*int)(unsafe.Pointer(u))
```

#### 6. `reflect.SliceHeader` 或 `reflect.StringHeader` Data 域和 Pointer 的互相转换

正如之前的例子，reflect 数据结构 SliceHeader 和 StringHeader 声明 Data 域是一个 uintptr，防止调用者在没有导入 `unsafe` 包时将结果修改为任意类型。然而，这意味着 SliceHeader 和 StringHeader 只有在诠释一个真正的切片或字符串内容时才是有效的。

```go
var s string
hdr := (*reflect.StringHeader)(unsafe.Pointer(&s)) // case 1
hdr.Data = uintptr(unsafe.Pointer(p))              // case 6
hdr.Len = n
```

在这种用法中，hdr.Data 实际上是一种可选的引用字符串头的底层指针的方式，本身并不是一个 uintptr 变量。

一般地，`reflect.SliceHeader` 和 `reflect.StringHeader` 应该只用作 `*reflect.SliceHeader` 和 `*reflect.StringHeader`，指向真正的切片或字符串，而不是普通的结构体。一个程序不应声明或分配这种结构体类型的变量。

```go
// 无效：直接声明的头不能持有 Data 作为引用
var hdr reflect.StringHeader
hdr.Data = uintpre)unasfe.Pointer(p)
hdr.Len = n
s := *(*String)(unsafe.Pointer(&hdr)) // p 可能已经释放
```
