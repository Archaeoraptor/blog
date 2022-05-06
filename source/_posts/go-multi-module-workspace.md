---
title: Go Workspace：本地可以愉快的replace了
date: 2022-03-18 10:45:03
tags:
- go
- workspace
abbrlink: 'go-multi-module-workspace'
categories:
- Concurrency&Go
---
昨天go2:1.18终于发布了，万众期待的泛型终于来了。不过我一直期待的是 go mod 的改动，新增了workspace和go vendor那几个命令不再默认更新`go.mod`和`go.sum`，不用再苦哈哈的replace和固定版本了。
~~然而这并不能改变go包管理器依旧鸡肋的现状~~
<!-- more -->

## 终于等到workspace

之前用 go module 我们有各种奇奇怪怪的理由要用replace：

1. 我在本地新建了一个项目，然后我`go mod init github/archaeoraptor/XXX`， 但是我暂时不想push上去
2. 写的比较乱我暂时不想commit
3. 用到了别人的依赖但是我魔改了点内容
4. 哎哎哎，go mod tidy 怎么自动把我modules版本升上去了，我不想升啊
5. 我想把github上面的私有项目迁移到gitea，想把url换成这个

我自己写就在`go.mod`里面replace就好了，手写几行 replace 又不是太麻烦的事情。  
然而我和我室友一起写点项目的时候麻烦就来了，replace 的本地路径不一样，我是`/home/xxx/xxx`，他是`C:\XXX\XXX`，这样在 commit 的时候就有问题。但是把`go.mod`放到`.gitignore`里面忽略掉又不好。

如果有一个像vscode那样的workspace，workspace中的本地json设置可以覆盖全局的json设置就好了。

## go work 使用

举例说明一下, 新建一个叫hello的repo, hello.go和main.go内容:

```go
//hello.go
package hello

import (
    "fmt"
)

func Hello() {
    fmt.Println("Hello, World!")
}
```

```go
// main.go
package main

import "github.com/archaeoraptor/hello"

func main() {
    hello.Hello()
}
```

如果你想不commit和push, 也不想修改go.mod文件加入replace，在本地能跑，那么需要将这两个文件放到同一个 go module 里面：

```bash
$ go mod init github.com/archaeoraptor/hello_repo
$ tree
.
├── go.mod
├── hello
│   ├── go.mod
│   └── hello.go
└── main.go
```

repo根目录的`go.mod`:

```
module github.com/archaeoraptor/hello_repo

go 1.18
```

这个时候不commit并push到远程，直接执行`go run main.go`是可以正常运行出`Hello, World!`的。

不过有的时候我们想要设置嵌套模块(nested modules)或者一个repo里面多个module（**比如万恶的grpc，一个项目里面可能要依赖不同版本的grpc**, 这时候就需要一个项目中多个`go.mod`来兼容不同的版本）

如果你想给hello单独设为嵌套模块
目录结构变成这样:

```bash
module github.com/archaeoraptor/hello

go 1.18
```

此时：hello目录的`go.mod`长这样：

```
module github.com/archaeoraptor/hello/hello

go 1.18
```

此时我们还没有上传，就会报错`go: finding module for package github.com/archaeoraptor/hello/hello  ... fatal: repository 'https://github.com/archaeoraptor/hello/' not found`或者`can't request version "latest" of the main module (github.com/archaeoraptor/hello)`

这个时候我们以前会改`go.mod`用replace替换成本地的：

```config
module github.com/archaeoraptor/hello

go 1.18

require github.com/archaeoraptor/hello/hello v0.0.0-20220318143722-96fbd621879d

replace github.com/archaeoraptor/hello/hello => ../hello/hello
```

有了workspace，可以将replace写在`go.work`中，然后将`go.work`添加到`.gitignore`。这样不同人对replace的更改就不会影响repo中的`go.mod`了

新建workspace(在module的外层目录)

```go
cd ..
go work init ./hello
```

会产生一个`go.work`文件夹，加入replace

```
go 1.18

use ./hello

replace github.com/archaeoraptor/hello/hello => ./hello/hello
```

## 终于不再默认更新依赖

1.18终于更改了之前`go mod vendor`等命令默认更新`go.mod`和`go.sum`的迷惑行为。

什么？这难道不好吗？理论上来说这很好，如果这些包都按照语义化版来。但是有的包它不讲武德，小版本升级有breaking changes，最爱干这种事的还不是那些野鸡包，**就是是谷歌自家的grpc**

按照语义化版本号一般的习惯，只有大版本更新（v1更新到v2, v2更新到v3）才有 breaking changes， 小版本的更新是向下兼容的（v1.1更新到v1.2），grpc基本每次小版本更新总有 breaking changes。我好几次arch滚出问题都是因为grpc。

每次想执行`go mod vendor`等命令的时候，之前为了不让它自动更新`go.mod`的依赖版本，只能用各种扭曲的办法，比如加`-mod=readonly`参数。

## 闲话包管理器

相比广为诟病的错误处理啊，没有泛型啊，我一直觉得go的包管理器是个更麻烦的问题。包管理器的问题很容易成为历史包袱的。包管理器一开始留下的问题，再想改就很麻烦，有兴趣可以看看npm之类的历史包袱。

## 链接

[Tutorial: Getting started with multi-module workspaces](https://go.dev/doc/tutorial/workspaces)  
<https://github.com/golang/go/issues/45713>  

[go-module-knobs](https://github.com/thepudds/go-module-knobs/blob/master/README.md)  
[Go modules cheat sheet](https://encore.dev/guide/go.mod)  

### 包管理器相关

[Node.js 包管理器发展史](https://wxsm.space/2021/npm-history/)  
[Proposal: Multi-Module Workspaces in cmd/go](https://go.googlesource.com/proposal/+/master/design/45713-workspace.md)   
