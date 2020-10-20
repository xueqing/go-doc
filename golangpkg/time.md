# time 包

- [time 包](#time-包)
  - [概述](#概述)
  - [Ticker 类型](#ticker-类型)
    - [func NewTicker](#func-newticker)
    - [func (*Ticker) Reset](#func-ticker-reset)
    - [func (*Ticker) Stop](#func-ticker-stop)
  - [Timer 类型](#timer-类型)
    - [func AfterFunc](#func-afterfunc)
    - [func NewTimer](#func-newtimer)
    - [func (*Timer) Reset](#func-timer-reset)
    - [func (*Timer) Stop](#func-timer-stop)
  - [索引](#索引)
  - [例子](#例子)

参考 [Golang 官网文档](https://golang.org/pkg/time/) 学习。

导入语句：`import "time"`

## 概述

time 包提供了用于测量和显示时间的功能。

日历计算始终采用公历，没有闰秒(leap second)。

## Ticker 类型

一个 Ticker 持有一个通道，该通道每隔一段时间变发送一个 “滴答(tick)”。

```go
type Ticker struct {
    C <-chan Time // The channel on which the ticks are delivered.
    // contains filtered or unexported fields
}
```

### func NewTicker

```go
func NewTicker(d Duration) *Ticker
```

`NewTicker` 返回一个新的包含一个通道的 ticker，该通道会在指定的 `duration` 参数周期发送时间。它可以调整健哥或丢掉 tick 以弥补接收慢的问题。持续时间 `d` 必须大于零；否则，`NewTicker` 会导致恐慌。停止 ticker 以释放相关的资源。

### func (*Ticker) Reset

```go
func (t *Ticker) Reset(d Duration)
```

`Reset` 停止一个 ticker，并重置其周期为指定的持续时间。下一个 tick 会在新的周期过去之后到达。

### func (*Ticker) Stop

```go
func (t *Ticker) Stop()
```

`Stop` 关闭一个 ticker。`Stop` 之后，不会再发送 tick。`Stop` 不会关闭通道，以防止并发的 goroutine 从通道读到错误的 tick。

## Timer 类型

Timer 类型表示一个单一的事件。当计时器到期时，会发送当前时间到 `C`，除非计时器是通过 `AfterFunc` 创建的。必须使用 `NewTimer` 或 `AfterFunc` 创建一个计时器。

```go
type Timer struct {
    C <-chan Time
    // contains filtered or unexported fields
}
```

### func AfterFunc

```go
func AfterFunc(d Duration, f func()) *Timer
```

`AfterFunc` 等待持续时间过去，然后在自己的 goroutine 中调用 `f`。它返回一个计时器，可以使用其 `Stop` 方法取消该调用。

### func NewTimer

```go
func NewTimer(d Duration) *Timer
```

`NewTimer` 创建一个新的计时器，它将在至少持续时间 `d` 之后发送当前时间到它的通道。

### func (*Timer) Reset

```go
func (t *Timer) Reset(d Duration) bool
```

`Reset` 将计时器更改为在持续时间 `d` 之后过期。如果计时器处于活动状态则返回 true；如果计时器已经过期或已经停止，则返回 false。

只能在已经停止或到期且通道耗尽的计时器上调用 `Reset`。如果程序已经从 `t.C` 接收到一个值，那么这个计数器是到期的且通道耗尽的，因此可以直接使用 `t.Reset`。然而，如果程序尚未从 `t.C` 接收到值，则必须停止计数器，且——如果 `Stop` 报告计时器在停止之前已经到期——需要显式清空其通道。

```go
if !t.Stop() {
  <-t.C
}
t.Reset(d)
```

这个操作不应该和计时器通道的其他接收操作同时进行。

注意，由于在耗尽通道和新计时器到期之间存在竞争条件，不能正确使用 `Reset` 的返回值。如上所述，应始终在已经停止或到期的通道上调用 `Reset`。返回值的存在是为了保持与现有程序的兼容性。

### func (*Timer) Stop

```go
func (t *Timer) Stop() bool
```

`Stop` 可防止计时器触发。如果调用停止了计时器，返回 true；如果计时器已经到期或停止，则返回 false。`Stop` 不会关闭通道，以防止后续错误地从通道读取数据。

为了确保调用 `Stop` 之后通道是空的，请检查返回值并清空通道。例如，假定程序尚未从 `t.C` 接收到值：

```go
if !t.Stop() {
  <-t.C
}
```

这个操作不能和计时器通道的其他接收操作或计时器的 `Stop` 方法的调用同时进行。

对于使用 `AfterFunc(d, f)` 创建的计时器，如果 `t.Stop` 返回 false，那么计时器已经到期且函数 `f` 已经在其自己的 goroutine 启动；`Stop` 不会等待 `f` 完成再返回。如果调用者需要知道 `f` 是否完成，那么它必须与 `f` 显式协调。

## 索引

[参考](https://golang.org/pkg/time/#pkg-index)

## 例子

[参考](https://golang.org/pkg/time/#pkg-examples)
