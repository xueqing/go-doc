# 表达式

- [表达式](#表达式)
  - [转换](#转换)

表达式通过将运算符和函数应用于操作数来指定值的计算。

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
