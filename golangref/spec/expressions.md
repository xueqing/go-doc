# 表达式

- [表达式](#表达式)
  - [切片表达式](#切片表达式)
    - [简单的切片表达式](#简单的切片表达式)
    - [完整的切片表达式](#完整的切片表达式)
  - [转换](#转换)

表达式通过将运算符和函数应用于操作数来指定值的计算。

## 切片表达式

切片表达式从字符串、数组、指向数组或切片的指针构造子字符串或切片。有两种变体：一种简单的形式，指定一个下限和一个上限；一个完整的形式，另外指定一个容量界限。

### 简单的切片表达式

对于字符串、数组、指向数组的指针或切片 `a`，基本表达式

```go
a[low : high]
```

构造一个子字符串或切片。*索引* `low` 和 `high` 选择操作数 `a` 中哪些元素出现在结果中。结果的索引从 0 开始，长度等于 `high - low`。对数组 `a` 进行切片后：

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]
```

切片 `s` 的类型是 `[]int`，长度为 3，容量为 4，且元素为：

```go
s[0] == 2
s[1] == 3
s[2] == 4
```

方便起见，可以忽略任一索引。缺少 `low` 则默认为 0；缺少 `high` 默认是被切片的操作数的长度：

```go
a[2:]  // 等同于 a[2 : len(a)]
a[:3]  // 等同于 a[0 : 3]
a[:]   // 等同于 a[0 : len(a)]
```

如果 `a` 是指向数组的指针，`a[low : high]` 是 `(*a)[low : high]` 的简写。

对于数组或字符串，如果 `0 <= low <= high <= len(a)`，则索引在范围内，否则索引超出范围。对于切片，索引上界是切片容量 `cap(a)` 而非长度。[常数](https://golang.org/ref/spec#Constants)索引必须是非负数，且可以由 `int` 类型的值[表示](https://golang.org/ref/spec#Representability)；对于数组或常量字符串，常量索引也必须在范围内。如果两个索引都是常数，则必须满足 `low <= high`。如果运行时索引超出范围，会发生[运行时恐慌](https://golang.org/ref/spec#Run_time_panics)。

除了[无类型的字符串](https://golang.org/ref/spec#Constants)，如果切片操作数是字符串或切片，则切片操作的结果是一个非恒定值，且类型与操作数相同。对于无类型的字符串操作数，结果是字符串类型的非恒定值。如果切片的操作数是数组，它必须是[可寻址](https://golang.org/ref/spec#Address_operators)的，且切片操作的结果是一个切片，此切片元素类型和数组相同。

如果一个有效切片表达式的切片操作数是 `nil` 切片，结果是 `nil` 切片。否则，如果结果是切片，此切片和操作数共享底层数组。

```go
var a [10]int
s1 := a[3:7]   // s1 底层数组是 a; &s1[2] == &a[5]
s2 := s1[1:4]  // s2 底层数组是 s1 底层数组，即数组 a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; 它们都引用同一个底层数组元素
```

### 完整的切片表达式

对于数组、指向数组的指针或切片 `a` (但不是字符串)，基本表达式

```go
a[low : high : max]
```

构造一个切片，和简单的切片表达式 `a[low : high]` 具有相同类型、相同长度和元素。此外，此表达式控制生成的切片容量，将其设置为 `max - low`。只有第一个索引可以忽略，默认是 0。对数组 `a` 切片之后：

```go
a := [5]int{1, 2, 3, 4, 5}
t := a[1:3:5]
```

切片 `t` 的类型是 `[]int`，长度为 2，容量为 4，且元素为：

```go
t[0] == 2
t[1] == 3
```

和简单的切片表达式相同，如果 `a` 是指向数组的指针。`a[low : high : max]` 是 `(*a)[low : high : max]` 的简写。如果被切片的操作符是一个数组，那么它必须是[可寻址](https://golang.org/ref/spec#Address_operators)的。

对于数组或字符串，如果 `0 <= low <= high <= cap(a)`，则索引在范围内，否则索引超出范围。[常数](https://golang.org/ref/spec#Constants)索引必须是非负数，且可以由 `int` 类型的值[表示](https://golang.org/ref/spec#Representability)；对于数组，常量索引也必须在范围内。如果多个索引都是常数，则出现的常数必须在彼此相对的范围内。如果运行时索引超出范围，会发生[运行时恐慌](https://golang.org/ref/spec#Run_time_panics)。

## 转换

转换将表达式的类型更改为转换指定的类型。转换可能显示在源的字面上，或者*隐含*在表达式的上下文中。

*显式*转换是一个形如 `T(x)` 的表达式，其中 `T` 是类型，`x` 是可以转换为类型 `T` 的表达式。

```txt
Conversion = Type "(" Expression [ "," ] ")" .
```

如果类型以操作符 `*` 或 `<-` 开始，或者以关键字 `func` 开始且没有结果列表，则在必要时必须使用括号以避免歧义。

```txt
*Point(p)        // 等同于 *(Point(p))
(*Point)(p)      // p 被转为 *Point
<-chan int(c)    // 等同于 <-(chan int(c))
(<-chan int)(c)  // c 被转为 <-chan int
func()(x)        // 函数签名 func() x
(func())(x)      // x 被转为 func()
(func() int)(x)  // x 被转为 func() int
func() int(x)    // x 被转为 func() int (有歧义)
```

[todo](https://golang.org/ref/spec#Conversions)
