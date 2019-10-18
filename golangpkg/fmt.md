# fmt 包

参考 [Golang 官网文档](https://golang.org/pkg/fmt/) 学习。

导入语句：`import "fmt"`

## 概述

fmt 包用类似于 C 的 printf 和 scanf 的函数实现了格式化的 I/O。格式 “verbs” 衍生自 C 但是更简单。

### 打印

verbs 包括：

通用的：

```txt
%v  默认格式的值
    当打印结构体时，增加标识 (%+v) 会增加域名。
%#v 值的一个 Go 语法显示
%T  值类型的一个 Go 语法显示
%%  一个字面百分比符号；不消费任何值
```

布尔型：

```txt
%t  单词是 true 还是 false
```

整型：

```txt
%b  base 2
%c  the character represented by the corresponding Unicode code point
%d  base 10
%o  base 8
%O  base 8 with 0o prefix
%q  a single-quoted character literal safely escaped with Go syntax.
%x  base 16, with lower-case letters for a-f
%X  base 16, with upper-case letters for A-F
%U  Unicode format: U+1234; same as "U+%04X"
```

## 索引

[参考](https://golang.org/pkg/fmt/#pkg-index)

## 例子

[参考](https://golang.org/pkg/fmt/#pkg-examples)
