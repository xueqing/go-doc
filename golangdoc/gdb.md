# 使用 GDB 调试 Go 代码

- [使用 GDB 调试 Go 代码](#使用-gdb-调试-go-代码)
  - [概述](#概述)
  - [介绍](#介绍)
    - [常见操作](#常见操作)
    - [Go 扩展](#go-扩展)
    - [已知的问题](#已知的问题)
  - [教程](#教程)
    - [开始](#开始)
    - [检查源码](#检查源码)
    - [命名](#命名)
    - [设置断点](#设置断点)
    - [检查栈](#检查栈)
    - [优雅打印](#优雅打印)

参考 [Go 官方文档——使用 GDB 调试 Go 代码](https://golang.org/doc/gdb)学习。

## 概述

以下说明适用于标准的工具链(gc Go 编译器和工具)。gccgo 具有原生 gdb 支持。

请注意，在调试使用标准工具链构建的 Go 程序时，[Delve](https://github.com/go-delve/delve) 是 GDB 的更好的替代方案。它比 GDB 更了解 Go 运行时、数据结构和表达式。目前 Delve 支持 amd64 上的 Linux、OSX 和 Windows。有关受支持的平台的最新列表，请参阅 [Delve 文档](https://github.com/go-delve/delve/tree/master/Documentation/installation)。

GDB 不太了解 Go 程序。堆栈管理、线程和运行时有一些方面，与 GDB 预期的执行模型有很大的不同，当使用 gccgo 编译程序时，它们会混淆调试器并导致错误的结果。因此，尽管 GDB 在一些场景很有用(例如，调试 Cgo 代码，或调试运行时本身)，但调试 Go 程序是不可靠的，尤其是高并发程序。此外，解决这些问题并不是 Go 项目的优先事项。

简言之，以下说明应该仅作为 GDB 正常工作时如何使用的指南，而不能确保成功。除了这个概述，你可能想查阅 [GDB 手册](https://sourceware.org/gdb/current/onlinedocs/gdb/)。

## 介绍

当你在 Linux、macOS、FreeBSD 或 NetBSD 上使用 gc 工具链编译和链接 Go 程序时，生成的二进制文件包含 DWARFv4 跳水信息，GDB 调试器的最新版本(>=7.5)可以使用这些信息来检查实时进程或核心转储。

将 `-w` 标志传递给链接器，以忽略调试信息(例如，`go build -ldflags=-w prog.go`)。

gc 编译器生成的代码包括函数调用的内联和变量的注册。这些优化有时使得使用 gdb 调试变得更加困难。如果你发现需要禁用这些优化，使用 `go build -gcflags=all="-N -l"` 构建程序。

如果你想使用 gdb 检查核心转储，你可以通过在环境中设置 `GOTRACEBACK=crash` (有关更多信息，参阅[运行时包文档](https://golang.org/pkg/runtime/#hdr-Environment_Variables))，在支持核心转储的系统上在程序崩溃时触发转储。

### 常见操作

- 显示代码的文件和行号，设置断点和反汇编

  ```sh
  (gdb) list
  (gdb) list line
  (gdb) list file.go:line
  (gdb) break line
  (gdb) break file.go:line
  (gdb) disas
  ```

- 显示调用栈和展开堆栈帧

  ```sh
  (gdb) bt
  (gdb) frame n
  ```

- 显示局部变量、参数和返回值的堆栈帧的名称、类型和位置

  ```sh
  (gdb) info locals
  (gdb) info args
  (gdb) p variable
  (gdb) whatis variable
  ```

- 显示全局变量的名称、类型和位置

  ```sh
  (gdb) info variables regexp
  ```

### Go 扩展

GDB 的最新扩展机制允许它为给定的二进制文件加载扩展脚本。工具链使用它通过一些命令扩展 GDB，来检查运行时代码(例如 goroutines)的内部，并优雅地打印内置的 map、切片和通道类型。

- 优雅地打印字符串、切片、map、通道或接口

  ```sh
  (gdb) p var
  ```

- 用于字符串、切片和 map 的 `$len()` 和 `$cap()`

  ```sh
  (gdb) p $len(var)
  ```

- 将接口转为动态类型的函数

  ```sh
  (gdb) p $dtype(var)
  (gdb) iface var
  ```

  **已知问题**：如果接口的长名称和短名称不同，GDB 不能自动查找接口值的动态类型(打印堆栈跟踪时很烦人，优雅的打印会打印短类型名称和指针)

- 检查 goroutine

  ```sh
  (gdb) info goroutines
  (gdb) goroutine n cmd
  (gdb) help goroutine
  ```

  比如

  ```sh
  (gdb) goroutine 12 bt
  ```

  你可以通过传递 `all` 而不是特定的 goroutine 的 ID 来检查所有的 goroutine。比如

  ```sh
  (gdb) goroutine all bt
  ```

如果你想了解它如何工作，或希望扩展它，查看 Go 源码仓库中的 [src/runtime/runtime-gdb.py](https://golang.org/src/runtime/runtime-gdb.py)。它依赖链接器([src/cmd/link/internal/ld/dwarf.go](https://golang.org/src/cmd/link/internal/ld/dwarf.go))确保的一些特殊的魔法类型(`hash<T,U>`)和变量(`runtime.m` 和 `runtime.g`)，这些类型和变量在 DWARD 代码中描述。

如果你对调试信息的外观感兴趣，运行 `objdump -W a.out` 并浏览 `.debug_*` 部分。

### 已知的问题

1. 只有字符串类型会触发优雅打印字符串，从字符串衍生的类型不会触发
2. 运行时库的 C 部分缺少类型信息
3. GDB 不理解 Go 的名称限定，将 `fmt.Print` 视为带有 `.` 的需要引用的非结构字面量。它更强烈地反对 `pkg.(*MyType).Meth` 形式的方法名称
4. 从 Go 1.11 开始，默认压缩调试信息。gdb 的较旧版本，例如 MacOS 上默认可用的版本，不理解压缩。你可以使用 `go build -ldflags=-compressdwarf=false` 生成未压缩的调试信息。(方便起见，你可以将 `-ldflags` 选项放在 GOFLAGS 环境变量中，这样你就不必每次都指定它。)

## 教程

在此教程中，我们将检查 [regexp](https://golang.org/pkg/regexp/) 包的单元测试。为了构建二进制文件，切换到 `$GOROOT/src/regexp` 并运行 `go test -c`。这应该生成一个名为 `regexp.test` 的可执行文件。

### 开始

启动 GDB，调试 `regexp.test`：

```sh
$ gdb regexp.test
GNU gdb (GDB) 7.2-gg8
Copyright (C) 2010 Free Software Foundation, Inc.
License GPLv  3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
Type "show copying" and "show warranty" for licensing/warranty details.
This GDB was configured as "x86_64-linux".

Reading symbols from  /home/user/go/src/regexp/regexp.test...
done.
Loading Go Runtime support.
(gdb)
```

`Loading Go Runtime support` 信息表示 GDB 从 `$GOROOT/src/runtime/runtime-gdb.py` 加载扩展。

为了帮助 GDB 查找 Go 的运行时源码，以及附带的支持脚本，将 `-d` 标志传递给你的 `$GOROOT`:

```sh
$ gdb regexp.test -d $GOROOT
```

如果出于某种原因 GDB 仍然找不到该目录或脚本，你可以告诉 gdb 手动加载它(假定你的 go 源码在 `~/go/`)：

```sh
(gdb) source ~/go/src/runtime/runtime-gdb.py
Loading Go Runtime support.
```

### 检查源码

使用 `l` 或 `list` 目录检查源代码：

```sh
(gdb) l
```

列出源代码的特定部分，`list` 参数带有函数名称(必须用其包名限定)：

```sh
(gdb) l main.main
```

列出特定文件和行号：

```sh
(gdb) l regexp.go:1
(gdb) # Hit enter to repeat last command. Here, this lists next 10 lines.
```

### 命名

变量和函数名称必须用其所属的包名限定。`regexp` 包中的 `Compile` 函数在 GDB 中对应 `regexp.Compile`。

方法必须使用其接收器类型的名称限定。比如，`*Regexp` 类型的 `String` 方法对应 `regexp.(*Regexp).String`。

隐藏其他变量的变量在调试信息中带有一个数字后缀。闭包引用的变量将显示为带有 `&` 前缀的指针。

### 设置断点

在 `TestFind` 函数设置断点：

```sh
(gdb) b 'regexp.TestFind'
Breakpoint 1 at 0x424908: file /home/user/go/src/regexp/find_test.go, line 148.
```

运行程序：

```sh
(gdb) run
Starting program: /home/user/go/src/regexp/regexp.test

Breakpoint 1, regexp.TestFind (t=0xf8404a89c0) at /home/user/go/src/regexp/find_test.go:148
148 func TestFind(t *testing.T) {
```

执行暂停在断点处。查看哪些 goroutine 在运行，以及它们在做的事情：

```sh
(gdb) info goroutines
  1  waiting runtime.gosched
* 13  running runtime.goexit
```

带有 `*` 标记的是当前的 goroutine。

### 检查栈

查看堆栈，我们暂停在程序的位置：

```sh
(gdb) bt  # backtrace
#0  regexp.TestFind (t=0xf8404a89c0) at /home/user/go/src/regexp/find_test.go:148
#1  0x000000000042f60b in testing.tRunner (t=0xf8404a89c0, test=0x573720) at /home/user/go/src/testing/testing.go:156
#2  0x000000000040df64 in runtime.initdone () at /home/user/go/src/runtime/proc.c:242
#3  0x000000f8404a89c0 in ?? ()
#4  0x0000000000573720 in ?? ()
#5  0x0000000000000000 in ?? ()
```

其他 goroutine，编号 1，卡在 `runtime.gosched`，阻塞在通道接收：

```sh
(gdb) goroutine 1 bt
#0  0x000000000040facb in runtime.gosched () at /home/user/go/src/runtime/proc.c:873
#1  0x00000000004031c9 in runtime.chanrecv (c=void, ep=void, selected=void, received=void) at  /home/user/go/src/runtime/chan.c:342
#2  0x0000000000403299 in runtime.chanrecv1 (t=void, c=void) at/home/user/go/src/runtime/chan.c:423
#3  0x000000000043075b in testing.RunTests (matchString={void (struct string, struct string, bool *, error *)} 0x7ffff7f9ef60, tests=  []testing.InternalTest = {...}) at /home/user/go/src/testing/testing.go:201
#4  0x00000000004302b1 in testing.Main (matchString={void (struct string, struct string, bool *, error *)} 0x7ffff7f9ef80, tests= []testing.InternalTest = {...}, benchmarks= []testing.InternalBenchmark = {...}) at /home/user/go/src/testing/testing.go:168
#5  0x0000000000400dc1 in main.main () at /home/user/go/src/regexp/_testmain.go:98
#6  0x00000000004022e7 in runtime.mainstart () at /home/user/go/src/runtime/amd64/asm.s:78
#7  0x000000000040ea6f in runtime.initdone () at /home/user/go/src/runtime/proc.c:243
#8  0x0000000000000000 in ?? ()
```

栈帧显示我们目前正在按预期执行 `regexp.TestFind` 函数：

```sh
(gdb) info frame
Stack level 0, frame at 0x7ffff7f9ff88:
 rip = 0x425530 in regexp.TestFind (/home/user/go/src/regexp/find_test.go:148);
    saved rip 0x430233
 called by frame at 0x7ffff7f9ffa8
 source language minimal.
 Arglist at 0x7ffff7f9ff78, args: t=0xf840688b60
 Locals at 0x7ffff7f9ff78, Previous frame's sp is 0x7ffff7f9ff88
 Saved registers:
  rip at 0x7ffff7f9ff80
```

`info locals` 命令列举函数的所有局部变量及其值，但使用起来有点危险，因为它还会尝试打印未初始化的变量。未初始化的切片可能会导致 gdb 尝试打印任意大的数组。

函数的参数：

```sh
(gdb) info args
t = 0xf840688b60
```

在打印参数时，请注意，它是一个指向 `Regexp` 值的指针。注意，GDB 错误地将 `*` 放在在类型名称右侧，并以传统的 C 风格组成 `struct` 关键字。

```sh
(gdb) p re
(gdb) p t
$1 = (struct testing.T *) 0xf840688b60
(gdb) p t
$1 = (struct testing.T *) 0xf840688b60
(gdb) p *t
$2 = {errors = "", failed = false, ch = 0xf8406f5690}
(gdb) p *t->ch
$3 = struct hchan<*testing.T>
```

`struct hchan<*testing.T>` 是通道的运行时内部表示。目前它是空的，否则 gdb 会优雅地打印其内容。

向前运行：

```sh
(gdb) n  # execute next line
149             for _, test := range findTests {
(gdb)    # enter is repeat
150                     re := MustCompile(test.pat)
(gdb) p test.pat
$4 = ""
(gdb) p re
$5 = (struct regexp.Regexp *) 0xf84068d070
(gdb) p *re
$6 = {expr = "", prog = 0xf840688b80, prefix = "", prefixBytes =  []uint8, prefixComplete = true,
  prefixRune = 0, cond = 0 '\000', numSubexp = 0, longest = false, mu = {state = 0, sema = 0},
  machine =  []*regexp.machine}
(gdb) p *re->prog
$7 = {Inst =  []regexp/syntax.Inst = {{Op = 5 '\005', Out = 0, Arg = 0, Rune =  []int}, {Op =
    6 '\006', Out = 2, Arg = 0, Rune =  []int}, {Op = 4 '\004', Out = 0, Arg = 0, Rune =  []int}},
  Start = 1, NumCap = 2}
```

我们可以使用 `s` 进入 `String` 函数：

```sh
(gdb) s
regexp.(*Regexp).String (re=0xf84068d070, noname=void) at /home/user/go/src/regexp/regexp.go:97
97      func (re *Regexp) String() string {
```

获取堆栈跟踪查看我们的位置：

```sh
(gdb) bt
#0  regexp.(*Regexp).String (re=0xf84068d070, noname=void) at /home/user/go/src/regexp/regexp.go:97
#1  0x0000000000425615 in regexp.TestFind (t=0xf840688b60) at /home/user/go/src/regexp/find_test.go:151
#2  0x0000000000430233 in testing.tRunner (t=0xf840688b60, test=0x5747b8) at /home/user/go/src/testing/testing.go:156
#3  0x000000000040ea6f in runtime.initdone () at /home/user/go/src/runtime/proc.c:243
....
```

查看源代码：

```sh
(gdb) l
92              mu      sync.Mutex
93              machine []*machine
94      }
95
96      // String returns the source text used to compile the regular expression.
97      func (re *Regexp) String() string {
98              return re.expr
99      }
100
101     // Compile parses a regular expression and returns, if successful,
```

### 优雅打印

GDB 的优雅打印机制由类型名称上的正则表达式匹配触发。切片示例：

```sh
(gdb) p utf
$22 =  []uint8 = {0 '\000', 0 '\000', 0 '\000', 0 '\000'}
```

因为切片、数组和字符串不是 C 指针，GDB 不能为你解释下标操作，但你可以查看运行时表示来执行此操作(制表符补全在这里有帮助)：

```sh
(gdb) p slc
$11 =  []int = {0, 0}
(gdb) p slc-><TAB>
array  slc    len
(gdb) p slc->array
$12 = (int *) 0xf84057af00
(gdb) p slc->array[1]
$13 = 0
```

扩展函数 `$len` 和 `$cap` 对字符串、数组和切片有效：

```sh
(gdb) p $len(utf)
$23 = 4
(gdb) p $cap(utf)
$24 = 4
```

通道和 map 是“引用”类型，gdb 将其显示为指向类似 C++ 类型 `hash<int, string>*` 的指针。解引用会触发优雅的打印。

接口在运行时表示为指向类型描述符的指针和指向值的指针。Go GDB 运行时扩展对此进行解释并自动触发运行时类型的优雅打印。扩展函数 `$dtype` 为你解释动态类型(示例取自 regexp.go 第 293 行的断点。)

```sh
(gdb) p i
$4 = {str = "cbb"}
(gdb) whatis i
type = regexp.input
(gdb) p $dtype(i)
$26 = (struct regexp.inputBytes *) 0xf8400b4930
(gdb) iface i
regexp.input: struct regexp.inputBytes *
```
