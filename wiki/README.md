# Go 维基学习

- [Go 维基学习](#go-%e7%bb%b4%e5%9f%ba%e5%ad%a6%e4%b9%a0)
  - [Go 之家](#go-%e4%b9%8b%e5%ae%b6)
  - [贡献](#%e8%b4%a1%e7%8c%ae)
  - [内容列表](#%e5%86%85%e5%ae%b9%e5%88%97%e8%a1%a8)
    - [Go 入门](#go-%e5%85%a5%e9%97%a8)
    - [用 Go 工作](#%e7%94%a8-go-%e5%b7%a5%e4%bd%9c)
    - [更多关于 Go 的学习](#%e6%9b%b4%e5%a4%9a%e5%85%b3%e4%ba%8e-go-%e7%9a%84%e5%ad%a6%e4%b9%a0)
    - [Go 社区](#go-%e7%a4%be%e5%8c%ba)
    - [使用 go 工具链](#%e4%bd%bf%e7%94%a8-go-%e5%b7%a5%e5%85%b7%e9%93%be)
    - [其他的 Go 编程维基](#%e5%85%b6%e4%bb%96%e7%9a%84-go-%e7%bc%96%e7%a8%8b%e7%bb%b4%e5%9f%ba)
    - [网上用 Go 的服务](#%e7%bd%91%e4%b8%8a%e7%94%a8-go-%e7%9a%84%e6%9c%8d%e5%8a%a1)
    - [生产环境的 Go 程序疑难解答](#%e7%94%9f%e4%ba%a7%e7%8e%af%e5%a2%83%e7%9a%84-go-%e7%a8%8b%e5%ba%8f%e7%96%91%e9%9a%be%e8%a7%a3%e7%ad%94)
    - [为 Go 项目做贡献](#%e4%b8%ba-go-%e9%a1%b9%e7%9b%ae%e5%81%9a%e8%b4%a1%e7%8c%ae)
    - [平台特定的信息](#%e5%b9%b3%e5%8f%b0%e7%89%b9%e5%ae%9a%e7%9a%84%e4%bf%a1%e6%81%af)
    - [发布特定的信息](#%e5%8f%91%e5%b8%83%e7%89%b9%e5%ae%9a%e7%9a%84%e4%bf%a1%e6%81%af)
    - [问题](#%e9%97%ae%e9%a2%98)

参考 [Go 维基官网](https://github.com/golang/go/wiki) 学习。

原网页由 HunterQ 在 2019/4/12 编辑。[第 99 次修订](https://github.com/golang/go/wiki/Home/_history)。

## Go 之家

欢迎来到 Go 维基，集中了关于 [Go 编程语言](https://golang.org/)的信息。[Awesome Go](http://awesome-go.com/) 是另外一个给 Go 编程人员的丰富的资源，由 Go 社区管理。

## 贡献

- 这个维基可被拥有 Github 账号的 Go 社区的任意成员编辑。
- 如果你想要新增一个页面，请首先在 [Go issue 跟踪页面](https://github.com/golang/go/issues) 打开一个 issue，以前缀 “wiki” 开头来提议新增的内容。清楚地说明为什么这个内容不适用任何现有的页面。
- 因为维基页面的重命名会破坏外部链接，请在重命名或删除任何维基页面之前打开一个 issue。

## 内容列表

### Go 入门

- [x] [Go 语言之旅](gotour/README.md)是入门最好的地方。
- [ ] [实效 Go 编程](https://golang.org/doc/effective_go.html)将会帮助学习如何编写惯用的 Go 代码。
- [ ] [Go 标准库文档](https://golang.org/pkg/)使你熟悉标准库。
- [ ] 使用 [Go Playground](http://play.golang.org/) 用于在你的浏览器测试 Go 程序。
- [ ] 仍然不确信？查看这份 [Go 使用者](https://github.com/golang/go/wiki/GoUsers)清单以及他们的一些[成功案例](https://github.com/golang/go/wiki/SuccessStories)。我们也收集了一份长长的原因清单，关于[你为什么应该尝试 Go](https://github.com/golang/go/wiki/whygo)。
- [ ] 了解更多已经[从其他语言转到 Go](https://github.com/golang/go/wiki/FromXToGo) 的公司。
这里是一些帮助你入门的链接。

### 用 Go 工作

准备好自己写一些 Go 代码了吗？这里是一些帮助你入门的链接。

- [ ] 安装和设置你的环境
  - [ ] 由此开始：[官方安装文档](https://golang.org/doc/install)
  - [ ] 如果你更喜欢源码安装，[先阅读此文档](https://golang.org/doc/install/source)
  - [ ] [从源码安装](https://github.com/golang/go/wiki/InstallFromSource)——其他关于源码安装的建议
  - [ ] Windows 用户？[为 Windows 安装和配置 Go、Git和 Atom](https://github.com/abourget/getting-started-with-golang)
  - [ ] Mac 用户？[如何开始-Go](https://howistart.org/posts/go/1)——安装 Go 和编译你的以第一个 web 服务的分步指南
  - [ ] 安装遇到问题？[安装疑难解答](https://github.com/golang/go/wiki/InstallTroubleShooting)
  - [ ] 确保你已经[正确设置了 $GOPATH 环境变量](https://golang.org/doc/install/source#gopath)
    - [ ] 如果需要其他关于[使用 $GOPATH 的建议，浏览这里](https://github.com/golang/go/wiki/GOPATH)
  - [ ] [多个 GOROOT](https://github.com/golang/go/wiki/MultipleGoRoots)——更多高级信息关于在安装多个 go 以及 `$GOROOT` 变量的环境工作
- [ ] [Go 集成开发环境和编辑器](https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins)——一些关于如何使用你最喜欢的编辑器开发 Go 的信息
- [ ] [为开发 Go 代码的工具](https://github.com/golang/go/wiki/CodeTools)——格式化、语言分析、代码检查、代码重构、代码导航和可视化
- [ ] 查找 Go 库和包
  - [ ] 由此开始：[Go 开源工程](https://github.com/golang/go/wiki/Projects)
  - [ ] 查找 Go 包：[go 文档官网](http://godoc.org/)
  - [ ] [Go 开源包图](https://anvaka.github.io/pm/#/galaxy/gosearch?l=1)的可视化
- [ ] [管理你的依赖](https://github.com/golang/go/wiki/PackageManagementTools)——一个你可以用来管理第三方包 (vendoring) 的工具纵览
- [ ] 发布开源的 Go 包
  - [ ] 准备好发布你的包了？[由此开始](https://github.com/golang/go/wiki/PackagePublishing)
  - [ ] [Go 检查清单](https://github.com/matttproud/gochecklist)——发布一个项目的完全指南
  - [ ] [如何设计你的 Github 仓库](https://github.com/golang/go/wiki/GitHubCodeLayout) 以便其他 Go 编程人员更容易使用 `go get` 命令
  - [ ] [Go 包](https://johnsto.co.uk/blog/go-package-go)——一些使得 Go 包更易用的建议

### 更多关于 Go 的学习

当你对这门语言有一个概览之后，这里有一些资源供你使用学习更多关于 Go：

- [ ] [学习 Go](https://github.com/golang/go/wiki/Learn)—— Go 入门到高级的资料集合
  - [ ] [测试](learn_testing.md)
- [ ] [书籍](https://github.com/golang/go/wiki/Books)——一份已经出版的(电子书，论文)关于 Go 的的书籍清单
- [ ] [博客](https://github.com/golang/go/wiki/Blogs)——关于 Go 的博客
  - [ ] [播客]

### Go 社区

### 使用 go 工具链

### 其他的 Go 编程维基

### 网上用 Go 的服务

### 生产环境的 Go 程序疑难解答

### 为 Go 项目做贡献

### 平台特定的信息

### 发布特定的信息

### 问题
