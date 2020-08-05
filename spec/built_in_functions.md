# 内置函数

- [内置函数](#内置函数)
  - [长度和容量](#长度和容量)

内置函数是预先声明的。可以像其他函数一样调用这些内置函数，但是一些内置函数的第一个参数是一个类型而不是一个表达式。

内置函数没有标准的 Go 类型，因此只能出现在调用表达式中；它们不能作为一个函数值。

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
  c1 = imag(2i)                    // imag(2i) = 2.0 is a constant
  c2 = len([10]float64{2})         // [10]float64{2} contains no function calls
  c3 = len([10]float64{c1})        // [10]float64{c1} contains no function calls
  c4 = len([10]float64{imag(2i)})  // imag(2i) is a constant and no function call is issued
  c5 = len([10]float64{imag(z)})   // invalid: imag(z) is a (non-constant) function call
)
var z complex128
```
