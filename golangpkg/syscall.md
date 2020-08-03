# syscall 包

- [syscall 包](#syscall-包)
  - [概述](#概述)
  - [索引](#索引)
  - [子目录](#子目录)

参考 [Golang 官网文档](https://golang.org/pkg/syscall/) 学习。

## 概述

syscall 包包含对底层操作系统原语的接口。细节依据底层系统有所不同，而且默认情况下，godoc 会显示当前操作系统的 syscall 文档。如果你想要的 godoc 显示另外一个系统的 syscall 文档，设置 `$GOOS` 和 `$GOARCH` 到想要的系统。例如，如果你想显示 在 linux/amd64 上 freebas/arm 的文档，设置 `$GOOS` 为 freebsd，`$GOARCH` 为 arm。syscall 的主要用于提供更加可移植的接口给系统的其他包内部，比如 “os”、“time” 和 “net”。如果可以使用这些包而不是 syscall 包。查阅合适的操作系统受测得到抓个包内的函数和数据类型的细节。这些调用返回 `err==nil` 表示成功；否则 err 是一个描述失败的操作系统错误。在大多数系统上，错误的类型是 `syscall.Errno`。

弃用的：这个包是锁定的。调用者应该使用 golang.org/x/sys 仓库下对应的包。这也适用于新系统和版本应该更新的。查看[https://golang.org/s/go1.4-syscall](https://golang.org/s/go1.4-syscall)获得更多信息。

## 索引

[参考](https://golang.org/pkg/syscall/#pkg-index)

## 子目录

| 名称 | 简介 |
| --- | --- |
| [js](https://golang.org/pkg/syscall/js/) | 使用 js/wasm 架构时，js 包提供访问 WebAssembly 宿主环境 |
