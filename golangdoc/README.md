# Golang 官方文档

- [Golang 官方文档](#golang-官方文档)
  - [入门](#入门)
    - [安装 Go](#安装-go)
    - [教程：入门](#教程入门)
    - [教程：创建一个模块](#教程创建一个模块)
  - [学习 Go](#学习-go)
    - [Go 语言之旅](#go-语言之旅)
    - [如何编写 Go 代码](#如何编写-go-代码)
    - [编辑器插件和 IDE](#编辑器插件和-ide)
    - [实效 Go 编程](#实效-go-编程)
    - [诊断](#诊断)
    - [常见问题解答](#常见问题解答)
    - [教程](#教程)
    - [Go 维基](#go-维基)
    - [更多学习资料](#更多学习资料)
  - [参考](#参考)
    - [包文档](#包文档)
    - [命令文档](#命令文档)
    - [语言规范](#语言规范)
    - [Go 内存模型](#go-内存模型)
    - [发布历史](#发布历史)
  - [文章](#文章)
    - [Go 博客](#go-博客)
    - [代码漫步](#代码漫步)
    - [语言](#语言)
    - [包](#包)
    - [模块](#模块)
    - [工具](#工具)
    - [更多文章](#更多文章)
  - [演讲](#演讲)
    - [Go 的视频之旅](#go-的视频之旅)
    - [优雅增长的代码](#优雅增长的代码)
    - [Go 的并发模式](#go-的并发模式)
    - [高级的 Go 并发模式](#高级的-go-并发模式)
    - [更多演讲](#更多演讲)
  - [非英文文章](#非英文文章)

参考 [Golang 官网文档](https://golang.org/doc/) 学习。

Go 编程语言是一个开源的项目，旨在提高程序员的生产力。

Go 富有表现力、简练、整洁且高效。它的并发机制使得易于编写可以最大化利用多核和联网机器的程序，而其新颖的类型系统可以实现灵活的模块化的程序构造。Go 可以快速编译为机器代码，但是具有垃圾回收的便利性和运行时反射的能力。它是一个快速的、静态类型的编译语言，感觉像是一种动态类型的解释语言。

## 入门

### [安装 Go](install.md)

下载和安装 Go 的教程。

### [教程：入门](tutorial/getting_started.md)

一个简单的 `Hello, World` 教程用于入门。学习一些关于 Go 代码、工具、包和模块。

### [教程：创建一个模块](tutorial/create_module.md)

简短话题的教程，包括函数、错误处理、数组、映射、单元测试和编译。

## 学习 Go

### [Go 语言之旅](../gotour/README.md)

用三个部分对 Go 进行了交互式介绍。第一部分覆盖了基本语法和数据结构；第二部分讨论方法和接口；第三部分介绍 Go 的并发原语。每个部分以一些练习结束，以便你可以练习所学内容。你可以[在线](https://tour.golang.org/)游览，也可以使用下面的命令安装在本地：

```sh
go get golang.org/x/tour
```

它会将 `tour` 二进制放在你的工作区的 `bin` 目录。

### [如何编写 Go 代码](code.md)

这篇文章解释如何在一个模块内开发一个简单的 Go 包集合，它也展示了如何使用 [go 命令](../command/README.md)来构建和测试包。

### [编辑器插件和 IDE](editors.md)

该文档总结了 Go 支持的常用编辑器插件和 IDE。

### [实效 Go 编程](effective_go.md)

提供有关编写清晰、惯用的 Go 代码的建议文档。所有新的 Go 程序员必须阅读的文章。它扩充了游览和语言规范，这两部分都应该先阅读。

### [诊断](diagnostics.md)

总结了诊断 Go 程序问题的工具和方法。

### [常见问题解答](faq.md)

关于 Go 的常见问题解答。

### [教程](tutorial/README.md)

Go 入门的一系列教程。

### [Go 维基](../golangwiki/README.md)

Go 社区维护的一个维基。

### 更多学习资料

查阅[维基](../golangwiki/README.md)的[学习](../golangwiki/learn.md)页获取更多 Go 的学习资源。

## 参考

### [包文档](../golangpkg/README.md)

Go 标准库的文档。

### [命令文档](cmd/README.md)

Go 工具的文档。

### [语言规范](../golangref/spec/README.md)

官方的 Go 语言规范。

### [Go 内存模型](../golangref/mem.md)

该文档指定了一个条件，在这种情况下，可以确保在一个 goroutine 中读一个变量可以观察到不同 goroutine 中对同一变量写入的值。

### [发布历史](https://golang.org/doc/devel/release.html)

Go 版本之间的更改摘要。

## 文章

### Go 博客

Go 项目的官方博客，包含 Go 团队和宾客的新闻和深入的文章。

### 代码漫步

Go 程序的指导之旅。

- [Go 的一级函数](https://golang.org/doc/codewalk/functions)
- [生成任意文本：一个 Markov 链算法](https://golang.org/doc/codewalk/markov)
- [通过共享内存](https://golang.org/doc/codewalk/sharemem)
- [编写 Web 应用](articles/wiki.md)——构建一个简单的 web 应用。

### 语言

- [JSON RPC：接口的故事](https://golang.org/blog/json-rpc-tale-of-interfaces)
- [Go 的声明语法](https://golang.org/blog/gos-declaration-syntax)
- [Defer、Panic 和 Recover](https://golang.org/blog/defer-panic-and-recover)
- [Go 并发模式：超时，继续](https://golang.org/blog/go-concurrency-patterns-timing-out-and)
- [Go 切片：用法和内部原理](https://golang.org/blog/go-slices-usage-and-internals)
- [一个 GIF 解码器：Go 接口的一个练习](https://golang.org/blog/gif-decoder-exercise-in-go-interfaces)
- [错误处理和 Go](https://golang.org/blog/error-handling-and-go)
- [组织 Go 代码](https://golang.org/blog/organizing-go-code)

### 包

- [JSON 和 Go](https://golang.org/blog/json-and-go) ——使用 [json](https://golang.org/pkg/encoding/json/) 包
- [数据块](https://golang.org/blog/gobs-of-data)—— [gob](https://golang.org/pkg/encoding/gob/) 包的设计和用法
- [反射定律](https://golang.org/blog/laws-of-reflection)—— [reflect](https://golang.org/pkg/reflect/) 包的基础
- [Go image 包](https://golang.org/blog/go-image-package)—— [image](https://golang.org/pkg/image/) 包的基础
- [Go image/draw 包](https://golang.org/blog/go-imagedraw-package)—— [image/draw](https://golang.org/pkg/image/draw/) 包的基础

### 模块

- [使用 Go 模块](https://golang.org/blog/using-go-modules)——在简单项目中使用模块的介绍
- [迁移到 Go 模块](https://golang.org/blog/migrating-to-go-modules)——将一个现存项目转成使用模块
- [发布 Go 模块](https://golang.org/blog/publishing-go-modules)——如何使其他人可以使用模块的新版本
- [Go 模块：v2 及以上](https://golang.org/blog/v2-go-modules)——创建主版本是 2 及以上
- [保持你的模块兼容](https://golang.org/blog/module-compatibility)——如何保持你的模块兼容以前的次要的版本或补丁版本

### 工具

- [关于 Go 命令](articles/go_command.md)——为什么我们编写它？它是什么？它不是什么？如何使用它？
- [使用 GDB 调试 Go 代码](gdb.md)
- [数据竞争检测器](articles/race_detector.md)——关于数据竞争检测器的手册
- [快速入门 Go 的汇编器](asm.md)——介绍了 Go 使用的汇编器
- [C?Go?Cgo?](https://golang.org/blog/c-go-cgo)——使用 [cgo](https://golang.org/cmd/cgo/) 链接 C 代码
- [Godoc：Go 代码的文档](https://golang.org/blog/godoc-documenting-go-code)——为 [godoc](https://golang.org/cmd/godoc/) 编写好的文档
- [分析 Go 程序](https://golang.org/blog/profiling-go-programs)
- [介绍 Go 的竞争探测器](https://golang.org/blog/race-detector)——对竞争探测器的一个介绍

### 更多文章

查看[维基](../golangwiki/README.md)的[文章](../golangwiki/articles.md)页面获取更多 Go 的文章。

## 演讲

### [Go 的视频之旅](https://research.swtch.com/gotour)

三件事使得 Go 快速、有趣和高效：接口、反射和并发。构建一个玩具网络爬虫来演示这些。

### [优雅增长的代码](https://vimeo.com/53221560)

Go 的主要设计目标之一是代码适应性；即它应该易于采用一个简单的设计，并以干净自然的方式构建。在本次演讲中，Andrew Gerrand 描述了一个简单的“聊天轮赌盘”服务器，该服务器匹配成对的到达的 TCP 连接，然后使用 Go 的并发机制、接口和标准库通过 web 界面和其他功能将其扩展。虽然程序的功能发生了巨大变化，但是 Go 的灵活性随着原始设计的增长而保留。

### [Go 的并发模式](https://www.youtube.com/watch?v=f6kdp27TYZs)

并发是设计高性能网络服务的关键。Go 的并发原语(goroutine 和 channel)提供了一种表达并发执行的简单高效的方法。在这个演讲中，我们看到如何使用简单的 Go 代码优雅地解决棘手的并发问题。

### [高级的 Go 并发模式](https://www.youtube.com/watch?v=QDDwwePbDtw)

这个演讲扩展了 `Go 的并发模式`，更深入地研究了 Go 的并发原语。

### 更多演讲

查看[Go 演讲网站](https://golang.org/talks)和[维基](../golangwiki/gotalks.md)页面获取更多 Go 的演讲。

## 非英文文章

查看维基的[非英文](../golangwiki/nonenglish.md)页面获取更多本地化文档。
