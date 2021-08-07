# runtime 包

- [runtime 包](#runtime-包)
  - [概述](#概述)
  - [索引](#索引)
  - [例子](#例子)
  - [函数](#函数)
    - [SetFinalizer 函数](#setfinalizer-函数)

参考 [Golang 官网文档](https://pkg.go.dev/runtime) 学习。

导入语句：`import "runtime"`

## 概述

runtime 包包含与 Go 的运行时系统交互的操作，例如控制 goroutine 的函数。它还包含 reflect 包使用的底层类型信息；查看 reflect 的文档获取有关运行时类型系统的可编程接口。

## 索引

[参考](https://pkg.go.dev/runtime/#pkg-index)

## 例子

[参考](https://pkg.go.dev/runtime/#pkg-examples)

## 函数

### SetFinalizer 函数

```go
func SetFinalizer(obj interface{}, finalizer interface{})
```

SetFinalizer 将与对象关联的终结器设置为提供的的 finalizer 函数。当垃圾收集器发现一个无法访问的块有关联的 finalizer 时，垃圾收集器会清除关联并在单独的 goroutine 运行 finalizer(obj)。这使得对象再次可达，但现在没有关联的 finalizer。假定没有再次调用 SetFinalizer，下次垃圾收集器发现对象不可达时，会释放对象。

SetFinalizer(obj, nil) 清除与 obj 关联的所有 finalizer。

obj 参数必须是指向一个对象的指针，该对象通过调用 new、获取复合字面量的地址，或获取局部变量的地址分配。finalizer 参数必须是一个函数，该函数接受一个可以赋值给 obj 类型的参数，且可以有任意被忽略的返回值。如果这些任何一个不满足，SetFinalizer 可能会终止程序。

finalizer 按照依赖顺序运行：如果 A 指向 B，二者都有 finalizer，则两者都不可达时，只会运行 A 的 finalizer；一旦 A 被释放，可以运行 B 的 finalizer。如果循环结构包含一个带有 finalizer 的块，则不能保证该循环被垃圾回收，且不能保证会运行 finalizer，因为没有关于依赖的顺序。

当程序不能再访问 obj 指向的对象后，计划在任意时间运行 finalizer。不能保证在程序退出之前运行 finalizer，因此一般它们只用于在长期运行的程序中，释放与对象关联的非内存资源。例如，当程序丢弃 os.File 对象没有调用 Close 时，os.File 对象可以使用 finalizer 来关闭关联的操作系统文件描述符，但是不要依赖 finalizer 来刷新内存中的 I/O 缓冲区，例如 bufio.Writer，因为在程序退出时不会刷新缓冲区。

如果 *obj 的大小为零字节，则不能保证运行 finalizer。

对于在初始化函数中未包级别变量分配的对象，不能保证 finalizer 会运行。这样的对象可能是链接器分配的，而不是堆分配的。

一旦对象变得不可访问，就可以允许 finalizer。为了正确使用 finalizer，程序必须确保对象在不再需要之前是可访问的。存储在全局变量中的对象，或者可以通过追踪全局变量的指针找到的对象，都是可访问的。对于其他对象，将对象传递给 KeepAlice 的函数的调用，以标记函数中该对象必须可达的最后一个点。

例如，如果 p 指向一个结构，例如 os.File，该结构包含一个文件描述符 d，且 p 有一个 finalizer 来关闭该文件描述符，那么如果 p 在函数中的最后一个适用是调用 syscall.Write(p.d, buf, size)，则程序一进入 syscall.Write，p 可能变得不可达。finalizer 此时可能允许，关闭 p.d，导致 syscall.Write 失败，因为它正在写入一个关闭的文件描述符(或者更糟糕的是，写入一个由不同 goroutine 打开的完全不同的文件描述符)。为了避免这个问题，在调用 syscall.Write 之后调用 runtime.KeepAlive(p)。

单个 goroutine 按顺序允许一个程序的所有 finalizer。如果一个 finalizer 必须运行很久，应该通过启动一个新的 goroutine 运行它。
