# reflect 包

- [reflect 包](#reflect-包)
  - [概述](#概述)
  - [索引](#索引)
    - [func Copy](#func-copy)
    - [type SliceHeader](#type-sliceheader)
    - [type StringHeader](#type-stringheader)
    - [type Type](#type-type)
      - [func ArrayOf(count int, elem Type) Type](#func-arrayofcount-int-elem-type-type)
      - [func TypeOf(i interface{}) Type](#func-typeofi-interface-type)
    - [type Value](#type-value)

参考 [Golang 官网文档](https://golang.org/pkg/reflect/) 学习。

导入语句：`import "reflect"`

## 概述

reflect 包实现了运行时反射，从而支持程序处理任意类型的对象。典型的用法是使用静态类型 `interface{}` 获取一个值，然后通过调用 `TypeOf` 提取其动态类型信息，函数返回一个 `Type`。

调用 `ValueOf` 返回一个代表运行时数据的 `Value`。零采用一种 `Type`，并返回一个 `Value`，表示该类型的零值。

有关 Go 语言反射的介绍，参阅“反射法则”：<https://golang.org/doc/articles/laws_of_reflection.html>。

## 索引

[参考](https://golang.org/pkg/reflect/#pkg-index)

### func Copy

```go
func Copy(dst, src Value) int
```

`Copy` 拷贝 `src` 的内容到 `dst` 直至 `dst` 填充满或者 `src` 消耗完。函数返回拷贝元素的数目。`dst` 和 `src` 都必须是切片或数组类型，且二者元素类型相同。

作为特例，如果 `dst` 元素类型是 uint8，`src` 可以是字符串类型。

### type SliceHeader

`SliceHeader` 是切片的运行时表示形式。它不能安全或跨平台使用，并且其表现形式可能在以后的版本中更改。此外，`Data` 字段不足以保证避免其引用的数据被垃圾回收，因此程序必须保留一个单独的、正确类型的指针指向底层数据。

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

### type StringHeader

`StringHeader` 是字符串的运行时表示形式。它不能安全或跨平台使用，并且其表现形式可能在以后的版本中更改。此外，`Data` 字段不足以保证避免其引用的数据被垃圾回收，因此程序必须保留一个单独的、正确类型的指针指向底层数据。

```go
type StringHeader struct {
    Data uintptr
    Len  int
}
```

### type Type

`Type` 表示 Go 类型。

并非所有方法适用于所有类型。每种方法的文档中都注明了其限制(如果有的话)。在调用特定类型的方法之前使用 `Kind` 方法找出类型。调用类型不适用的方法会导致运行时恐慌。

`Type` 值是可比较的，比如使用 `==` 操作符，因此可将其用作映射的键。如果两个 `Type` 值表示同一类型则二者相等。

```go
type Type interface {

    // Align returns the alignment in bytes of a value of
    // this type when allocated in memory.
    Align() int

    // FieldAlign returns the alignment in bytes of a value of
    // this type when used as a field in a struct.
    FieldAlign() int

    // Method returns the i'th method in the type's method set.
    // It panics if i is not in the range [0, NumMethod()).
    //
    // For a non-interface type T or *T, the returned Method's Type and Func
    // fields describe a function whose first argument is the receiver.
    //
    // For an interface type, the returned Method's Type field gives the
    // method signature, without a receiver, and the Func field is nil.
    //
    // Only exported methods are accessible and they are sorted in
    // lexicographic order.
    Method(int) Method

    // MethodByName returns the method with that name in the type's
    // method set and a boolean indicating if the method was found.
    //
    // For a non-interface type T or *T, the returned Method's Type and Func
    // fields describe a function whose first argument is the receiver.
    //
    // For an interface type, the returned Method's Type field gives the
    // method signature, without a receiver, and the Func field is nil.
    MethodByName(string) (Method, bool)

    // NumMethod returns the number of exported methods in the type's method set.
    NumMethod() int

    // Name returns the type's name within its package for a defined type.
    // For other (non-defined) types it returns the empty string.
    Name() string

    // PkgPath returns a defined type's package path, that is, the import path
    // that uniquely identifies the package, such as "encoding/base64".
    // If the type was predeclared (string, error) or not defined (*T, struct{},
    // []int, or A where A is an alias for a non-defined type), the package path
    // will be the empty string.
    PkgPath() string

    // Size returns the number of bytes needed to store
    // a value of the given type; it is analogous to unsafe.Sizeof.
    Size() uintptr

    // String returns a string representation of the type.
    // The string representation may use shortened package names
    // (e.g., base64 instead of "encoding/base64") and is not
    // guaranteed to be unique among types. To test for type identity,
    // compare the Types directly.
    String() string

    // Kind returns the specific kind of this type.
    Kind() Kind

    // Implements reports whether the type implements the interface type u.
    Implements(u Type) bool

    // AssignableTo reports whether a value of the type is assignable to type u.
    AssignableTo(u Type) bool

    // ConvertibleTo reports whether a value of the type is convertible to type u.
    ConvertibleTo(u Type) bool

    // Comparable reports whether values of this type are comparable.
    Comparable() bool

    // Bits returns the size of the type in bits.
    // It panics if the type's Kind is not one of the
    // sized or unsized Int, Uint, Float, or Complex kinds.
    Bits() int

    // ChanDir returns a channel type's direction.
    // It panics if the type's Kind is not Chan.
    ChanDir() ChanDir

    // IsVariadic reports whether a function type's final input parameter
    // is a "..." parameter. If so, t.In(t.NumIn() - 1) returns the parameter's
    // implicit actual type []T.
    //
    // For concreteness, if t represents func(x int, y ... float64), then
    //
    //  t.NumIn() == 2
    //  t.In(0) is the reflect.Type for "int"
    //  t.In(1) is the reflect.Type for "[]float64"
    //  t.IsVariadic() == true
    //
    // IsVariadic panics if the type's Kind is not Func.
    IsVariadic() bool

    // Elem returns a type's element type.
    // It panics if the type's Kind is not Array, Chan, Map, Ptr, or Slice.
    Elem() Type

    // Field returns a struct type's i'th field.
    // It panics if the type's Kind is not Struct.
    // It panics if i is not in the range [0, NumField()).
    Field(i int) StructField

    // FieldByIndex returns the nested field corresponding
    // to the index sequence. It is equivalent to calling Field
    // successively for each index i.
    // It panics if the type's Kind is not Struct.
    FieldByIndex(index []int) StructField

    // FieldByName returns the struct field with the given name
    // and a boolean indicating if the field was found.
    FieldByName(name string) (StructField, bool)

    // FieldByNameFunc returns the struct field with a name
    // that satisfies the match function and a boolean indicating if
    // the field was found.
    //
    // FieldByNameFunc considers the fields in the struct itself
    // and then the fields in any embedded structs, in breadth first order,
    // stopping at the shallowest nesting depth containing one or more
    // fields satisfying the match function. If multiple fields at that depth
    // satisfy the match function, they cancel each other
    // and FieldByNameFunc returns no match.
    // This behavior mirrors Go's handling of name lookup in
    // structs containing embedded fields.
    FieldByNameFunc(match func(string) bool) (StructField, bool)

    // In returns the type of a function type's i'th input parameter.
    // It panics if the type's Kind is not Func.
    // It panics if i is not in the range [0, NumIn()).
    In(i int) Type

    // Key returns a map type's key type.
    // It panics if the type's Kind is not Map.
    Key() Type

    // Len returns an array type's length.
    // It panics if the type's Kind is not Array.
    Len() int

    // NumField returns a struct type's field count.
    // It panics if the type's Kind is not Struct.
    NumField() int

    // NumIn returns a function type's input parameter count.
    // It panics if the type's Kind is not Func.
    NumIn() int

    // NumOut returns a function type's output parameter count.
    // It panics if the type's Kind is not Func.
    NumOut() int

    // Out returns the type of a function type's i'th output parameter.
    // It panics if the type's Kind is not Func.
    // It panics if i is not in the range [0, NumOut()).
    Out(i int) Type
    // contains filtered or unexported methods
}
```

#### func ArrayOf(count int, elem Type) Type

```go
func ArrayOf(count int, elem Type) Type
```

`ArrayOf` 返回数组类型，包含指定数目和元素类型。比如，如果 `t` 表示 `int`，`ArrayOf(5, t)` 表示 `[5]int`。

如果结果类型大于可用的地址空间，导致运行时恐慌。

#### func TypeOf(i interface{}) Type

```go
func TypeOf(i interface{}) Type
```

`TypeOf` 返回反射类型，表示 `i` 的动态类型。如果 `i` 是 `nil` 接口类型值，`TypeOf` 返回 `nil`。

### type Value

`Value` 是 Go 值的反射接口。

并非所有方法适用所有值类型。妹子方法的文档中都注明了其限制(如果有的话)。在调用特定类型的方法之前使用 `Kind` 方法找出类型。调用类型不适用的方法会导致运行时恐慌。

零值表示没有值。其 `IsValid` 方法返回 `false`，`Kind` 方法返回 `Invalid`，`String` 方法返回 “\<invalid Value\>”，其他所有方法都会发生恐慌。大多数函数和方法从不返回 `invalid` 值。否则，文档会显式说明这种情况。

多个 goroutine 可并发使用 `Value`，前提是底层的 Go 值可同时用于等效的直接操作。

要比较两个 `Value`，比较 `Interface` 方法的结果。在两个 `Value` 上调用 `==` 并非比较它们表示的底层值。

```go
type Value struct {
    // contains filtered or unexported fields
}
```
