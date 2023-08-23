
## 常用命令

**go version**: 查看 go 版本，常用来验证 go 是否正确安装

**go env [变量名]**: 查看环境变量。不加参数打印所有变量，加参数就打印指定变量

**go env -w GOPROXY=https://goproxy.cn,direct**: 设置模块来源，goproxy.cn 是七牛云提供的国内源，速度快

**go build xxx.go**：编译 go 源码文件，生成一个可执行文件，它可以运行在没有安装环境的电脑上

**go run xxx.go**：编译 go 源码文件并运行可执行文件，通常用于开发环境

## 风格规范

### 文件名

源文件总是用全小写字母形式的短小单词命名，并且以.go 扩展名结尾

包名通常使用单个的小写单词命名

### 缩进

推荐把左花括号与函数声明置于同一行并以空格分隔

标准 Go 代码风格使用 Tab 而不是空格来实现缩进的

### 分号

大多数分号都是可选的，常常被省略，不过在源码编译时，Go 编译器会自动插入这些被省略的分号

## 概念

### 包

`package xyz` 定义本文件的代码属于包 xyz，包名通常使用单个的小写单词命名。代码都隶属于某个包，包是 Go 语言的基本组成单元，一个 Go 程序本质上就是一组包的集合。

整个 Go 程序中仅允许存在一个名为 main 的包。main 包中的主要代码是一个名为 main 的函数：当你运行一个可执行的 Go 程序的时候，所有的代码都会从这个入口函数开始运行。

### import

import "xxx" 中 "xxx" 代表包的导入路径

xxx.func 中 "xxx" 代表包名

注意，main 包不能被导入

### Module

通常一个代码仓库对应一个 Go Module。一个 Go Module 的顶层目录下会放置一个 go.mod 文件，每个 go.mod 文件会定义唯一一个 module

go.mod 文件所在的顶层目录也被称为 module 的根目录，module 根目录以及它子目录下的所有 Go 包均归属于这个 Go Module，这个 module 也被称为 main module

一个 Go Module 是一个 Go 包的集合。module 是有版本的，所以 module 下的包也就有了版本属性。

依赖包的管理：

- 语义导入版本机制：主版本号不相同，不兼容；次版本号大兼容次版本号小的版本；补丁版本号不影响兼容性。在包导入路径引入主版本号的方式来区别同一个包的不兼容版本。
- 最小版本选择：从所有可选的版本中，选择符合项目整体要求的『最小版本』

## 工具

**Gofmt**：将代码自动格式化为约定的风格。作为 Go 开发人员，请在提交你的代码前使用 Gofmt 格式化你的 Go 源码。

## 构建

### 安装外部依赖

执行 `go mod init moudlepath` 创建一个名为 go.mod 的文件，在这个文件中存储了这个 module 对第三方依赖的全部信息

执行 `go mod tidy` 会自动安装 module 直接依赖和间接依赖的包，并把它们的信息记录到 go.mod。还会生成一个go.sum 文件，这个文件记录了 module 的直接依赖和间接依赖包的相关版本的 hash 值，用来校验本地包的真实性

go mod tidy下载的第三方包一般在$GOPATH/pkg/mod下面。如果没有设置GOPATH环境变量，其默认值为你的home路径下的go文件夹。这样第三方包就在go文件夹的pkg/mod下面。

go.sum 存放了特定版本 module 内容的哈希值。这是 Go Module 的一个安全措施。当将来这里的某个 module 的特定版本被再次下载的时候，go 命令会使用 go.sum 文件中对应的哈希值，和新下载的内容的哈希值进行比对，只有哈希值比对一致才是合法的，这样可以确保你的项目所依赖的 module 内容，不会被恶意或意外篡改。

### 引用本地模块

在Go 1.17版本及之前版本的解决方法是使用go mod的replace指示符(directive)。假如你的module a要import的module b将发布到github.com/user/repo中，那么你可以手动在module的go.mod中的require块中手工加上一条：

require github.com/user/repo v1.0.0

注意v1.0.0这个版本号是一个临时的版本号。

然后在module a的go.mod中使用replace将上面对module b的require替换为本地的module b:

replace github.com/user/repo v1.0.0 => module b本地路径

这样go命令就会使用你本地正在开发、尚未提交github的module b了。

### 包路径

module 下每个包的导入路径都是由 module path 和包所在子目录的名字结合在一起构成

## 结构布局

### 可执行程序项目

```shell
exe-layout
├── cmd/
│   ├── app1/
│   │   └── main.go
│   └── app2/
│       └── main.go
├── go.mod
├── go.sum
├── internal/
│   ├── pkga/
│   │   └── pkg_a.go
│   └── pkgb/
│       └── pkg_b.go
├── pkg1/
│   └── pkg1.go
├── pkg2/
│   └── pkg2.go
└── vendor/
```

- 放在项目顶层的 Go Module 相关文件，包括 go.mod 和 go.sum；
- cmd 目录：存放项目要编译构建的可执行文件所对应的 main 包的源码文件；
- 项目包目录：每个项目下的非 main 包都“平铺”在项目的根目录下，每个目录对应一个 Go 包；
- internal 目录：存放仅项目内部引用的 Go 包，这些包无法被项目之外引用；
- vendor 目录：这是一个可选目录，为了兼容 Go 1.5 引入的 vendor 构建模式而存在的。这个目录下的内容均由 Go 命令自动维护，不需要开发者手工干预。

### 包

```shell
lib-layout
├── go.mod
├── internal/
│   ├── pkga/
│   │   └── pkg_a.go
│   └── pkgb/
│       └── pkg_b.go
├── pkg1/
│   └── pkg1.go
└── pkg2/
    └── pkg2.go
```

在 Go 可执行程序项目的基础上去掉 cmd 目录和 vendor 目录

## 其它

静态语言和动态语言的区别，静态语言在执行前需要先经过编译器进行编译，编译完成会生成可执行文件，可以运行在没有安装环境的电脑上。 动态语言执行前不需要编译，但是只能运行在安装了环境的电脑上

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

go默认是开启CGO_ENABLED的，即CGO_ENABLED=1。但编译出来的二进制程序究竟有无动态链接，取决于你的程序使用了什么包。如果就是一个hello，world，那么你编译出来的将是一个纯静态程序。

如果你依赖了网络包或一些系统包，比如用http包编写了一个web server(见第9讲示例），那么你编译出来的二进制程序又会是一个包含动态链接的程序。

原因就在于目前的go标准库中，某些功能具有两份实现，一份是c语言实现的，一份是go语言实现的。在cgo_enable开启的情况下，go链接器会链接c语言的版本，于是就有了依赖动态链接库的情况。如果你将cgo_enabled置为0，你再重新编译链接，那么go链接器会使用go版本的实现，这样你将得到一个没有动态链接的纯静态二进制程序。

xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

在Go中我们针对包（package）编写测试代码。测试代码与包代码放在同一个包目录下，并且Go要求所有测试代码都存放在以*_test.go结尾的文件中。这使Go开发人员一眼就能分辨出哪些文件存放的是包代码，哪些文件存放的是针对该包的测试代码。

执行单元测试时，go test命令会将所有包目录下的*_test.go文件编译成一个临时二进制文件（我们可以通过go test -c显式编译出该文件），并执行该文件，后者将执行各个测试源文件中的名字格式为TestXxx函数所代表的测试用例并输出测试执行结果。

当然go不仅有_test，还有以os、cpu架构为特殊后缀的文件，比如：signal_linux_amd64.go(Go runtime包下的文件)。文件名中的linux、amd64用于限制该文件参与编译的os和平台。以signal_linux_amd64.go为例，该文件仅在linux x86-64平台上才会参与编译。