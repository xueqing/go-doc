# 内置函数

- [内置函数](#内置函数)
  - [关闭](#关闭)
  - [长度和容量](#长度和容量)
  - [分配](#分配)
  - [创建切片、map 和 channel](#创建切片map-和-channel)
  - [追加和拷贝切片](#追加和拷贝切片)
  - [删除 map 元素](#删除-map-元素)
  - [操作复数](#操作复数)
  - [处理 panic](#处理-panic)
  - [bootstrapping](#bootstrapping)

内置函数是预先声明的。可以像其他函数一样调用这些内置函数，但是一些内置函数的第一个参数是一个类型而不是一个表达式。

内置函数没有标准的 Go 类型，因此只能出现在调用表达式中；它们不能作为一个函数值。

## 关闭

对于一个 channel `c`，内置函数 `close(c)` 记录不会有更多数据在这个 channel 发送。如果 `c` 是一个只读的 channel 会出错。在一个关闭的 channel 上执行发送或关闭导致“运行时恐慌(run-time panic)”。关闭 `nil` channel 也会导致“运行时恐慌”。调用 `close` 之后，且之前发送的数据都被接收之后，接收操作会无阻塞地返回该 channel 类型的零值。多值接收操作返回一个接收到的值和一个指示 channel 是否关闭的标志。

## 长度和容量

内置函数 `len` 和 `cap` 接受不同类型的参数，并返回一个 `int` 类型的结果。实现保值结果总是适用一个 `int`：

```go
调用       参数类型         结果

len(s)    string 类型      字符串的字节长度
          [n]T, *[n]T      数组长度 (== n)
          []T              切片长度
          map[K]T          map 长度 (定义的键的数目)
          chan T           channel 缓存中排队的元素数目

cap(s)    [n]T, *[n]T      数组长度 (== n)
          []T              切片容量
          chan T           channel 缓存容量
```

切片的容量是底层数组分配空间的元素数目。下面的关系总是成立的：

```txt
0 <= len(s) <= cap(s)
```

`nil` 切片、map 或 channel 的长度为 0。`nil` 切片或 channel 的容量是 0.

如果 `s` 是字符串，表达式 `len(s)` 是常量。如果 `s` 的类型是数组或指向数组的指针，表达式 `len(s)` 和 `cap(s)` 是常量，且表达式 `s` 不包含 channel 接收者或(非常量的)函数调用；在这种情况下 `s` 不是计算求值的。否则，调用 `len` 和 `cap` 不是常量，且 `s` 是计算求值的。

```go
const (
  c1 = imag(2i)                    // imag(2i) = 2.0 是一个常量
  c2 = len([10]float64{2})         // [10]float64{2} 不包括函数调用
  c3 = len([10]float64{c1})        // [10]float64{c1} 不包括函数调用
  c4 = len([10]float64{imag(2i)})  // imag(2i) 是一个常量，没有函数调用
  c5 = len([10]float64{imag(z)})   // 无效: imag(z) 是一个(非常量)函数调用
)
var z complex128
```

## 分配

内置函数 `new` 接收一个类型 `T`，在运行时为该类型的变量分配存储，并返回一个类型 `*T` 的值指向该存储。变量的初始化正如“初始化值”章节描述的。

```go
new(T)
```

比如：

```go
type S struct { a int; b float64 }
new(S)
```

为类似 `S` 的变量分配存储，初始化变量(`a=0,b=0.0`)，并返回一个类型 `*S` 的值包含了这个位置的地址。

## 创建切片、map 和 channel

内置函数 `make` 接收一个类型 `T`，该类型必须是一个切片、map 或 channel 类型，后面可选地跟一个类型相关的表达式列表。函数返回一个类型 `T` (而不是 `*T`) 的值。内存的初始化正如“初始化值”章节描述的。

```txt
调用             类型 T      结果

make(T, n)       切片       类型 T 切片，长度 n，容量 n
make(T, n, m)    切片       类型 T 切片，长度 n，容量 m

make(T)          map        类型 T map
make(T, n)       map        类型 T map，初始空间大约 n 个元素

make(T)          channel    无缓存类型 T channel
make(T, n)       channel    有缓存类型 T channel，缓存大小是 n
```

大小参数 `n` 和 `m` 必须都是整数类型或无类型的常量。一个常量大小参数必须是非负数且可用一个 `int` 类型值表示；如果是一个无类型的常量，会指定它的类型 `int`。如果 `n` 和 `m` 都提供且为常量，那么`n` 必须不大于 `m`。如果运行时 `n` 是负数或大于 `m`，会发生“运行时恐慌(run-time panic)”。

```go
s := make([]int, 10, 100)       // 切片 len(s) == 10, cap(s) == 100
s := make([]int, 1e3)           // 切片 len(s) == cap(s) == 1000
s := make([]int, 1<<63)         // 不合法: len(s) 不能用 int 类型值表示
s := make([]int, 10, 0)         // 不合法: len(s) > cap(s)
c := make(chan int, 10)         // channel 缓存大小是 10
m := make(map[string]int, 100)  // map 初始空间大约是 100 个元素
```

使用 map 类型和大小 `n` 调用 `make` 会创建一个 map，且初始空间可保存 `n` 个 map 元素。精确的行为是取决于实现的。

## 追加和拷贝切片

内置函数 `append` 和 `copy` 辅助常见的切片操作。两个函数的结果和参数引用的内存是否重叠无关。

可变参数函数 `append` 追加 0 个或更多值 `x` 到类型 `S` 的值 `s`，`S` 必须是切片类型，并返回一个生成类型 `S` 的切片。值 `x` 传递给参数类型 `...T`，`T` 是 `s` 的元素类型，且应用了各自参数传递规则。作为特例，`append` 也接收第一个可赋值给类型 `[]byte`，第二个参数是字符串类型。这种形式追加的是字符串的字节。

```go
append(s S, x ...T) S  // T is the element type of S
```

如果 `s` 的容量不足以容纳额外的值，`append` 分配一个新的，足够大的底层数组，这个数组可以容纳已有的切片元素和额外的值。除此以外，`append` 复用了底层数组。

```go
s0 := []int{0, 0}
s1 := append(s0, 2)                // 追加一个元素    s1 == []int{0, 0, 2}
s2 := append(s1, 3, 5, 7)          // 追加多个元素    s2 == []int{0, 0, 2, 3, 5, 7}
s3 := append(s2, s0...)            // 追加一个切片    s3 == []int{0, 0, 2, 3, 5, 7, 0, 0}
s4 := append(s3[3:6], s3[2:]...)   // 追加重叠的切片  s4 == []int{3, 5, 7, 2, 3, 5, 7, 0, 0}

var t []interface{}
t = append(t, 42, 3.1415, "foo")   //                t == []interface{}{42, 3.1415, "foo"}

var b []byte
b = append(b, "bar"...)            // 追加字符串内容  b == []byte{'b', 'a', 'r' }
```

函数 `copy` 从源 `src` 拷贝切片元素到目的 `dst`，且返回拷贝的元素数目。两个参数必须有相同的元素类型 `T`，且必须可赋值给一个类型 `T[]` 的切片。拷贝的元素数目是 `len(src)` 和 `len(dst)` 的较小值。作为特例，拷贝也接收目的参数可赋值给类型 `[]byte`，源参数是一个字符串类型。这种形式从字符串拷贝字节到字节切片。

```go
copy(dst, src []T) int
copy(dst []byte, src string) int
```

示例：

```go
var a = [...]int{0, 1, 2, 3, 4, 5, 6, 7}
var s = make([]int, 6)
var b = make([]byte, 5)
n1 := copy(s, a[0:])            // n1 == 6, s == []int{0, 1, 2, 3, 4, 5}
n2 := copy(s, s[2:])            // n2 == 4, s == []int{2, 3, 4, 5, 4, 5}
n3 := copy(b, "Hello, World!")  // n3 == 5, b == []byte("Hello")
```

## 删除 map 元素

内置函数 `delete` 从 map `m` 中移除带有键 `k` 的元素。`k` 的类型必须可赋值给 `m` 的键类型。

```go
delete(m, k)  // 从 map m 中删除元素 m[k]
```

如果 map `m` 是 `nil` 或元素 `m[k]` 不存在，`delete` 是一个不执行的操作(no-op)。

## 操作复数

有三个函数组装和拆解复数。内置函数 `complex` 从一个浮点的实部和虚部构造一个复数值，`real` 和 `img` 提取一个复数值的实部和虚部。

```go
complex(realPart, imaginaryPart floatT) complexT
real(complexT) floatT
imag(complexT) floatT
```

参数类型和返回类型是对应的。对于 `complex`，两个参数必须都是同一浮点类型，返回类型是对应浮点类型的复数类型：`complex64` 对应 `float32` 参数，`complex128` 对应 `float64` 参数。如果其中一个参数是无类型常量，首先隐式地将其转换为另一个参数的类型。如果两个参数是无类型常量，他们必须是非复数，或者他们的虚部必须是 0，那么函数的返回值是无类型的复数常量。

对于 `real` 和 `img`，参数必须是复数类型，反水类型是对应的浮点类型：`float32` 对应 `complex64` 参数，`float64` 对应 `complex128` 参数。如果参数是一个无类型常量，它必须是一个数字，函数的返回值是一个无类型的浮点常量。

反过来，`real` 和 `img` 函数一起构造 `complex`，所以对于一个复数类型 `Z` 的值 `z`，`z == Z(complex(real(z), img(z)))`。

如果这些函数的操作子都是常量，返回值也是常量。

```go
var a = complex(2, -2)             // complex128
const b = complex(1.0, -1.4)       // 无类型的复数常量 1 - 1.4i
x := float32(math.Cos(math.Pi/2))  // float32
var c64 = complex(5, -x)           // complex64
var s int = complex(1, 0)          // 无类型的复数常量 1 + 0i 可以转换为 int
_ = complex(1, 2<<s)               // 无效的: 2 应该为浮点类型，不能转换
var rl = real(c64)                 // float32
var im = imag(a)                   // float64
const c = imag(b)                  // 无类型常量 -1.4
_ = imag(3 << s)                   // 无效的: 3 应该为复数类型，不能转换
```

## 处理 panic

两个内置函数 `panic` 和 `recover`，辅助报告和处理“运行时恐慌(run-time panic)”和程序定义的错误情况。

```go
func panic(interface{})
func recover() interface{}
```

执行一个函数 `F` 时，一个对 `panic` 的显式调用或一个运行时恐慌终止 `F` 的执行。然后任何 `F` defer 的函数会如常执行。接下来，任何 `F` 调用者 defer 的函数运行，以此类推到执行 goroutine 的顶级函数 defer 的函数。那时，程序被终止且报告错误情况，包括 `panic` 的参数值。这个终止序列叫做“恐慌(panicking)”。

```go
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
```

函数 `recover` 运行一个程序管理一个恐慌的 goroutine 的行为。假定一个函数 `G` defer 一个函数 `D`，函数 `D` 调用 `recover`，且 执行 `G` 的 goroutine 内的某个函数发生了恐慌。当 defer 的函数运行到 `D` 时，`D` 调用 `recover` 的返回值会是调用 `panic` 时传递的值。如果 `D` 正常返回，没有开启一个新的恐慌，这个恐慌序列就停止了。这种情况下，在 `G` 和调用 `panic` 之间的函数状态被丢弃，恢复正常的执行。`G` 在 `D` 之前 defer 的函数被运行，且 `G` 的执行终止返回给它的调用者。

如果下面任意一个条件满足，`recover` 的返回值为 `nil`：

- `panic` 的参数是 `nil`
- goroutine 没有恐慌
- `recover` 没有被 defer 的函数直接调用

下面示例中的`protect` 函数调用函数参数 `g`，且保护调用者没有因为 `g` 导致的运行时恐慌。

```go
func protect(g func()) {
  defer func(){
    log.Println("done")  // Println executes normally even if there is a panic
    if x := recover(); x != nil {
      log.Println("run time panic: %v", x)
    }
  }()
  log.Println("start")
  g()
}
```

## bootstrapping

当前的实现在 bootstrapping 期间提供一个有用的内置函数。这些函数用于记录完整性，但不保证保留在语言。它们不返回结果。

```txt
函数       行为

print      打印所有参数；格式化参数是实现相关的
println    类似于 print，但是参数之间有空格，且末尾有换行
```

实现限制：`print` 和 `println` 不需要接收任何参数类型，但是打印布尔值、数值和字符串类型必须支持。
