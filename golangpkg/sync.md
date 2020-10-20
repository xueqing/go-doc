# sync 包

- [sync 包](#sync-包)
  - [概述](#概述)
  - [Cond 类型](#cond-类型)
    - [func NewCond](#func-newcond)
    - [func (*Cond) Broadcast](#func-cond-broadcast)
    - [func (*Cond) Signal](#func-cond-signal)
    - [func (*Cond) Wait](#func-cond-wait)
  - [Locker 类型](#locker-类型)
  - [Map 类型](#map-类型)
    - [func (*Map) Delete](#func-map-delete)
    - [func (*Map) Load](#func-map-load)
    - [func (*Map) LoadAndDelete](#func-map-loadanddelete)
    - [func (*Map) LoadOrStore](#func-map-loadorstore)
    - [func (*Map) Range](#func-map-range)
    - [func (*Map) Store](#func-map-store)
  - [Mutex 类型](#mutex-类型)
    - [func (*Mutex) Lock](#func-mutex-lock)
    - [func (*Mutex) Unlock](#func-mutex-unlock)
  - [Once 类型](#once-类型)
    - [func (*Once) Do](#func-once-do)
  - [Pool 类型](#pool-类型)
    - [func (*Pool) Get](#func-pool-get)
    - [func (*Pool) Put](#func-pool-put)
  - [RWMutex 类型](#rwmutex-类型)
    - [func (*RWMutex) Lock](#func-rwmutex-lock)
    - [func (*RWMutex) RLock](#func-rwmutex-rlock)
    - [func (*RWMutex) RLocker](#func-rwmutex-rlocker)
    - [func (*RWMutex) RUnlock](#func-rwmutex-runlock)
    - [func (*RWMutex) Unlock](#func-rwmutex-unlock)
  - [WaitGroup 类型](#waitgroup-类型)
    - [func (*WaitGroup) Add](#func-waitgroup-add)
    - [func (*WaitGroup) Done](#func-waitgroup-done)
    - [func (*WaitGroup) Wait](#func-waitgroup-wait)
  - [索引](#索引)
  - [例子](#例子)

参考 [Golang 官网文档](https://golang.org/pkg/sync/) 学习。

导入语句：`import "sync"`

## 概述

sync 包提供基本的同步原语，比如互斥锁。除了 `Once` 和 `WaitGroup` 类型，大多数供底层库例程使用。更高级别的同步最好使用通道和通信完成。

包含此包中定义的类型的值不应复制。

## Cond 类型

`Cond` 实现了一个条件变量，它是 goroutine 等待或宣布时间发生的集合点。

每个 `Cond` 有一个关联的 `Locker` L (通常是 `*Mutex` 或 `*RWMutex`)，在修改条件和调用 `Wait` 方法时必须持有该锁。

第一次使用 `Cond` 之后，不能将其复制。

```go
type Cond struct {

    // L is held while observing or changing the condition
    L Locker
    // contains filtered or unexported fields
}
```

### func NewCond

```go
func NewCond(l Locker) *Cond
```

`NewCond` 返回一个新的 `Cond`，它带有锁 `l`。

### func (*Cond) Broadcast

```go
func (c *Cond) Broadcast()
```

`Broadcast` 唤醒所有等待 `c` 的 goroutine。

在调用期间，调用者可以但是不要求持有 `c.L`。

### func (*Cond) Signal

```go
func (c *Cond) Signal()
```

`Signal` 唤醒一个等待 `c` 的 goroutine，如果有的话。

在调用期间，调用者可以但是不要求持有 `c.L`。

### func (*Cond) Wait

```go
func (c *Cond) Wait()
```

`Wait` 自动解锁 `c.L`，并挂起调用的 goroutine 的执行。在之后恢复执行之后，`Wait` 在返回之前加锁 `c.L`。和其他系统不同，`Wait` 不能返回除非被 `Broadcast` 或 `Signal` 唤醒。

因为 `Wait` 第一次恢复时，`c.L` 未加锁，调用者一般不能假定 `Wait` 返回时条件为真。相反地，调用者应该在一个循环中 `Wait`：

```go
c.L.Lock()
for !condition() {
    c.Wait()
}
... make use of condition ...
c.L.Unlock()
```

## Locker 类型

`Locker` 表示一个对象，它可以被加锁和解锁。

```go
type Locker interface {
    Lock()
    Unlock()
}
```

## Map 类型

`Map` 类似 Go 的 `map[interface{}]interface{}`，但是可以安全地被多个 goroutine 并发使用，且不用额外的锁或者协调。加载、存储和删除以分摊的常量时间运行。

`Map` 类型是专用的。大多数代码应该使用普通的 Go `map`，带有单独的锁或协作，以提高类型安全性，并使得更易于维护和这个 `map` 内容一起的其他不变量。

`Map` 类型针对两个常见用例进行了优化：

1. 当给定键的条目仅写入一次单读取多次，比如仅在增长的缓存中
2. 当多个 goroutine 读、写和覆盖不相交的键集的条目

在这两种情况下，与带有单独的 `Mutex` 或 `RWMutex` 的 Go `map` 相比。使用 `Map` 可显著减少锁竞争。

`Map` 的零值是空的，可立即使用的。第一次使用 `Map` 之后不能将其复制。

```go
type Map struct {
    // contains filtered or unexported fields
}
```

### func (*Map) Delete

```go
func (m *Map) Delete(key interface{})
```

`Delete` 删除 `key` 的值。

### func (*Map) Load

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool)
```

`Load` 返回存储在 map 中的 `key` 的值。`ok` 结果表示是否在 map 中找到该值。

### func (*Map) LoadAndDelete

```go
func (m *Map) LoadAndDelete(key interface{}) (value interface{}, loaded bool)
```

`LoadAndDelete` 删除`key` 的值，返回前一个值，如果有的话。 `loaded` 结果报告了 `key` 是否存在。

### func (*Map) LoadOrStore

```go
func (m *Map) LoadOrStore(key, value interface{}) (actual interface{}, loaded bool)
```

`LoadOrStore` 返回 `key` 的现有值，如果存在的话。否则，它存储并返回给定的值。如果已加载该值，`loaded` 为 true，如果存储为 false。

### func (*Map) Range

```go
func (m *Map) Range(f func(key, value interface{}) bool)
```

`Range` 为每个出现在 map 中的键和值依次调用 `f`。如果 `f` 返回 false，`range` 停止迭代。

`Range` 不一定和 `Map` 内容的任何一致的快照对应：不会多次访问任何键，但是如果同时存储或删除
任何键的值，`Range` 可能会在 `Range` 期间从任何点反映该键的任何映射。

`Range` 对于 map 中元素的数量可能为 O(N)，即使 `f` 在常数次调用之后返回 false。

### func (*Map) Store

```go
func (m *Map) Store(key, value interface{})
```

`Store` 设置 `key` 的值。

## Mutex 类型

`Mutex` 是一个互斥锁。一个 `Mutex` 的零值是一个未加锁的互斥锁。

第一次使用 `Mutex` 之后不能将其复制。

```go
type Mutex struct {
    // contains filtered or unexported fields
}
```

### func (*Mutex) Lock

```go
func (m *Mutex) Lock()
```

`Lock` 对 `m` 加锁。如果这个锁已经被使用，调用的 goroutine 会阻塞直到这个互斥锁可用。

### func (*Mutex) Unlock

```go
func (m *Mutex) Unlock()
```

`Unlock` 对 `m` 解锁。如果 `m` 在进入 `Unlock` 时未加锁，则是运行时错误。

锁定的互斥锁与特定的 goroutine 没有关联。允许一个 goroutine 锁定互斥锁，然后安排另一个 goroutine 对其解锁。

## Once 类型

`Once` 是一个对象，它将只执行一次操作。

```go
type Once struct {
    // contains filtered or unexported fields
}
```

### func (*Once) Do

```go
func (o *Once) Do(f func())
```

当且仅当 `Once` 的一个实例第一次调用 `Do`，`Do` 才会调用函数 `f`。也就是说，给定

```go
var once Once
```

如果多次调用 `once.Do(f)`，即使 `f` 在每次调用中的值不同，只有第一次调用会调用 `f`。每个函数执行需要一个新的 `Once` 实例。

`Do` 旨在用于必须只运行一次的初始化。由于 `f` 是无参数也无返回值的，因此可能需要使用一个函数字面量来捕获 `Do` 调用的函数的参数：

```go
config.once.Do(func() { config.init(filename) })
```

因为直到对 `f` 的一个调用返回，对 `Do` 的调用才会返回。如果 `f` 导致 `Do` 被调用，会发生死锁。

如果 `f` 恐慌，`Do` 认为已经返回；之后对 `Do` 的调用会返回而不再调用 `f`。

## Pool 类型

`Pool` 是一个临时对象的集合，该集合可以单独保存和检索。

`Pool` 中保存的任何元素都可能随时自动删除且没有通知。当这种情况发生时，如果 `Pool` 持有唯一的引用，元素可能被释放。

`Pool` 可安全地同时用于多个 goroutine。

`Pool` 的目的是缓存已分配但是未使用的元素以便之后重用，减小垃圾收集器的压力。也就是说，它使得易于构建高效、线程安全的空闲列表。然而，它不适用于所有空闲列表。

`Pool` 的一个合适的用途是管理一组临时元素，它们由一个包的并发独立客户端之间静态共享，且有可能被重用。`Pool` 提供了一种摊销许多客户端分配负载的方式。

一个较好的使用 `Pool` 的示例是 `fmt` 包，它维护了一个动态大小的临时输出缓存存储。这个存储在负载时扩展(当很多 goroutine 活跃打印时)，并在静止时收缩。

另一方面，作为短期对象的一部分维护的空闲列表不适用于 `Pool`，因为在这种场景下，该负载无法很好的分摊。使用此对象实现自己的空闲列表更有效。

在第一次使用 `Pool` 之后不能将其复制。

```go
type Pool struct {

    // New optionally specifies a function to generate
    // a value when Get would otherwise return nil.
    // It may not be changed concurrently with calls to Get.
    New func() interface{}
    // contains filtered or unexported fields
}
```

### func (*Pool) Get

```go
func (p *Pool) Get() interface{}
```

`Get` 从池中选择一个任意的元素，将其从池中移除，然后将其返回给调用者。`Get` 可能选择忽略池，并将其视为空的。调用者不应假定传递给 `Put` 的值和 `Get` 返回的值之间的任何关系。

如果 `Get` 本来将会返回 `nil` 且 `p.New` 是非 `nil`，`Get` 返回调用 `p.New` 的结果。

### func (*Pool) Put

```go
func (p *Pool) Put(x interface{})
```

`Put` 将 `x` 增加到池中。

## RWMutex 类型

一个 `RWMutex` 是一个读写互斥锁。这个锁可被任意数量的读者持有或被一个写者持有。`RWMutex` 的零值是一个未锁定的互斥锁。

第一次使用 `RWMutex` 之后不能将其拷贝。

如果一个 goroutine 持有一个 `RWMutex` 用于读，且另一个 goroutine 可能调用 `Lock`，那么没有 goroutine 可以期待获得读锁直到原始的读锁被释放。特别是，这禁止了递归读锁加锁。这是为了确保锁最终可用；阻塞的 `Lock` 调用使得新读者无法获得锁。

```go
type RWMutex struct {
    // contains filtered or unexported fields
}
```

### func (*RWMutex) Lock

```go
func (rw *RWMutex) Lock()
```

`Lock` 锁定 `rm` 用于写。如果这个锁已经被锁定用于读或写，`Lock` 会阻塞直到该锁可用。

### func (*RWMutex) RLock

```go
func (rw *RWMutex) RLock()
```

`RLock` 锁定 `rw` 用于读。

不应将 `RLock` 用于递归读锁定；阻塞的 `Lock` 调用使得新读者无法获得锁。参阅有关 `RWMutex` 类型的文档。

### func (*RWMutex) RLocker

```go
func (rw *RWMutex) RLocker() Locker
```

`RLocker` 返回一个 `Locker` 接口，通过调用 `rw.RLock` 和 `rw.RUnlock` 分别实现了 `Lock` 和 `Unlock` 方法。

### func (*RWMutex) RUnlock

```go
func (rw *RWMutex) RUnlock()
```

`RUnlock` 撤销单个 `RLock` 调用；它不会影响其他的同步读者。如果 `rw` 在进入 `RUnlock` 读时没有锁定，是一个运行时错误。

### func (*RWMutex) Unlock

```go
func (rw *RWMutex) Unlock()
```

`Unlock` 解锁 `rw` 进行写入。如果 `rw` 在进入 `Unlock` 写时未锁定，是一个运行时错误。

和互斥锁一样，一个锁定的 `RWMutex` 和特定的 goroutine 没有关联。一个 goroutine 可以 `RLock (Lock)` 一个 `RWMutex`，然后安排另一个 goroutine `RUnlock (Unlock)` 它。

## WaitGroup 类型

一个 `WaitGroup` 等待一个 goroutine 的集合结束。主 goroutine 调用 `Add` 设置需要等待的 goroutine 的数目。然后每个 goroutine 运行并在结束之后调用 `Done`。同事，`Wait` 可用于阻塞直到所有的 goroutine 结束。

第一次使用 `WaitGroup` 之后不能将其复制。

```go
type WaitGroup struct {
    // contains filtered or unexported fields
}
```

### func (*WaitGroup) Add

```go
func (wg *WaitGroup) Add(delta int)
```

`Add` 将可能为负的增量添加到 `WaitGroup` 计数器。如果计数器变成零，则所有阻塞在 `Wait` 的 goroutine被释放。如果计数器变成负数，`Add` 发生恐慌。

注意，当计数器为零时，调用增量为正必须发生在 `Wait` 之前。调用增量为负，或者调用增量为正开始时计数器大于零的，可以在任何时候发生。通常，这意味着调用 `Add` 应该在创建 goroutine 或等待其他事件之前执行。如果一个 `WaitGroup` 被复用等待几个独立的事件集，之前所有的 `Wait` 调用已经返回之后才可以进行新的 `Add` 调用。参阅 `WaitGroup` 示例。

### func (*WaitGroup) Done

```go
func (wg *WaitGroup) Done()
```

`Done` 将 `WaitGroup` 的计数器减 1。

### func (*WaitGroup) Wait

```go
func (wg *WaitGroup) Wait()
```

`Wait` 阻塞到 `WaitGroup` 的计数器变为零。

## 索引

[参考](https://golang.org/pkg/sync/#pkg-index)

## 例子

[参考](https://golang.org/pkg/sync/#pkg-examples)
