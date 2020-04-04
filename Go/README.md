# 安装

1. [官网](https://golang.org/dl/)下载 Go 的 tar.gz 安装包，解压到某一目录
2. 在 PATH 中加入 \$GOROOT/bin
3. 使用 `go env -w 环境变量=值` 来设置环境变量
   - Go 1.13 建议所有跟 Go 相关的环境变量都交由 `go env -w` 来管理
   - `go env -w` 不会覆盖系统环境变量
   - 设置：GOROOT（Go 的安装目录）、GOPATH、GO111MODULE、GOPROXY、GOBIN 环境变量
4. 使用 `go version` 验证是否安装成功

# 部分命令

- 运行：`go run [参数] 命令源码文件.go`
  - `-race` 在运行时检查数据访问冲突
- 构建：`go build [参数]`
  - 编译包和依赖，会生成一个可执行的文件
  - `-a` 将命令/库源码文件全部重新构建
  - `-n` 把编译器设计的命令全部打印出来，但不会真的执行，方便学习
  - `-race` 开启竞态条件检测，支持的平台有限制
  - `-x` 打印编译期间用到的命令，与 `-n` 的区别在于它不仅打印还会执行
  - `-ldflags “-X '包名.变量名=值'”` 向程序中指定包的变量传递值
  - `-o 文件名` 指定编译后输出的可执行文件名
- 使用 `go build` 生产可执行文件后，`./可执行文件名` 即可运行，使用 `-v` 参数可以打印出程序的一些信息
- 安装：`go install` 编译并安装包和依赖
- `go clean` 移除对象文件和缓存文件
- `go tool compile -S file.go` 查看代码编译后的汇编内容
  - `-S` 表示打印出生成的汇编代码。
  - 注意，此时输出的汇编代码还没有链接，呈现的地址都是偏移量。
  - 也可以使用 `go build -gcflags -S file.go` 命令
- `go tool objdump -s main.main file.exe` 查看链接之后得到的内容
  - 相当于反汇编你的程序，会丢失很多信息。
- `go get`，使用 `go get`前需要安装 git
  - 拉取依赖，会进行指定性拉取（更新），并不会更新所依赖的其它模块。
  - `-u` 更新现有的依赖，会强制更新它所依赖的其它全部模块，不包括自身。
  - `-fix` 在下载代码包后先运行一个用于根据当前 Go 语言版本修正代码的工具，然后再安装代码包
  - `-insecure` 允许通过非安全的网络协议下载和安装代码包。HTTP 就是这样的协议。
- `godoc -http :端口` 即可在浏览器中输入 `localhost:端口` 后查看标准库的文档

下载第三方库时无法访问：

1. 下载 golang.org 的包时，golang.org/x/net 对应 GitHub 上的镜像地址为 https://github.com/golang/net
2. 将其源码 $GOPATH/src/github.com/golang/net 复制到 $GOPATH/src/golang.org/x/
3. 然后 `go build net`

Q：如何向二进制文件中加入编译时间戳、go 的版本信息？

以 Linux 系统下向编译的二进制文件中添加为例，

```go
var (
  // 这三个参数从外部在源码编译时传入
	buildstamp   = "unknown"
	gitstatus    = "unknown"
	gitcommitlog = "unknown"
	// 这里使用 runtime 包的函数来初始化值，也可以类似 buildstamp 从外部向其传值
	goversion = fmt.Sprintf("%s %s/%s", runtime.Version(), runtime.GOOS, runtime.GOARCH)
)

func main() {
	args := os.Args
	if len(args) == 2 && (args[1] == "--version" || args[1] == "-v") {
		fmt.Printf("UTC Build Time : %s\n", buildstamp)
		fmt.Printf("Git Commit Hash: %s\n", gitstatus)
		fmt.Printf("Git Commit Hash: %s\n", gitcommitlog)
		fmt.Printf("Golang Version : %s\n", goversion)
		return
	}
}

```

这里使用一个 shell 脚本来完成程序内部参数赋值及可执行文件的生成：

```shell
#!/bin/bash

# 获取源码最近一次 git commit log，包含 commit sha 值，以及 commit message
gitcommitlog=`git log --pretty=oneline -n 1`
# 检查源码在git commit 基础上，是否有本地修改，且未提交的内容
gitstatus=`git status -s`
# 获取当前时间
buildstamp=`date +'%Y.%m.%d.%H%M%S'`

flags="-X 'main.buildstamp=${buildstamp}' -X 'main.gitstatus=${gitstatus}' -X 'main.gitcommitlog=${gitcommitlog}'"
# 会生成名为 add_info_to_binaryfile 的可执行文件
go build -ldflags "${flags}" -x -o add_info_to_binaryfile main.go
```

执行：`./add_info_to_binaryfile -v` 就可以看到输出了。

详见：[二进制文件加入 Git 版本的坑](https://mp.weixin.qq.com/s/VZXQeEeNNTLJPfS8tJVJZg)

# Go Modules

[Go Modules 终极入门](https://mp.weixin.qq.com/s/fNMXfpBhBC3UWTbYCnwIMg)

视频可以看：youtu.be/F8nrpe0XWRg

- 一个开关环境变量：GO111MODULE

  - auto：只在项目中包含了 go.mod 文件时启用 Go Modules，是 Go 1.14 中的默认值
  - on：启用 Go Modules，推荐
  - off：禁用 Go Modules

- 五个辅助环境变量：GOPROXY、GONOPROXY、GOSUMDB、GONOSUMDB、GOPRIVATE

- 两个辅助概念：

  - Go module proxy 集中时模块版本管理，加快模块版本的拉取速度、项目构建速度，使 Go 脱离对 VCS 的依赖，防止项目由于依赖某个模块被其作者删除而被 break 掉，淡化 Vendor 概念。
  - Go checksum database 用于保护 Go 模块的生态系统，防止 Go 从任何源头意外拉取到经过篡改的模块版本，引入了意外的代码更改。

- 两个主要文件：go.mod、go.sum

- 一个主要管理子命令：`go mod`

  - 内置在几乎所有的其他子命令中：`go get`、`go install`、`go list`、`go test`、`go run`、`go build` ......

```shell
go mod init	 # 生成 go.mod 文件，可以指定模块导入路径，如：go mod init github.com/eddycjy/module-repo
go mod download	# 下载 go.mod 文件中指明的所有依赖
go mod tidy	# 整理现有的依赖
go mod graph	# 查看现有的依赖结构
go mod edit	# 编辑 go.mod 文件
go mod vendor	# 导出项目所有的依赖到vendor目录
go mod verify	# 校验一个模块是否被篡改过
go mod why	# 查看为什么需要依赖某模块
```

- Global Caching：不同项目的相同模块版本只会在电脑上缓存一份
  - 拉去下来的模块目前都是缓存在 `$GOPATH/pkg/mod` 和 `$GOPATH/pkg/sumdb` 下
  - 同一个模块版本的数据只缓存一份，所有其它模块共享使用
  - 可使用 `go clean -modcache` 清理所有已缓存的模块版本数据

## 辅助环境变量

- GOPROXY

多个值以 `,` 分隔，用于使 Go 在后续拉取模块版本时能脱离传统的 VCS 方式而直接从镜像站点快速拉取，值也可以是 off，即禁止从任何地方拉取模块版本。

初始默认值：`https://proxy.golang.org,direct`

`proxy.golang.org` 在中国无法访问，建议替换：`go env -w GOPROXY=https://goproxy.cn,direct`

“direct” 用于指示 Go 回源到模块版本的源地址去抓取（如 Github 等）。当值列表中上一个 Go module proxy 返回 404 或 410 错误时，Go 会自动尝试列表中的下一个，遇见 “direct” 时终止并回到源地址去抓取，遇见 EOF 时终止并抛出错误。

---

- GOSUMDB

值是一个 Go checksum database，用于使 Go 在拉取模块版本时保证拉取到的模块版本数据未经篡改，值也可以是 “off” ，禁止 Go 校验任何模块版本。两种格式：

1. <SUMDB_NAME>+<PUBLIC_KEY>
2. <SUMDB_NAME>+<PUBLIC_KEY>+<SUMDB_URL>

默认值：`sum.golang.org`（不符合格式，是因为 Go 对默认值做了特殊处理）

`sum.golang.org`在中国无法访问，将 GOPROXY 设置为 goproxy.cn ，goproxy.cn 支持代理 sum.golang.org。

---

- GONOPROXY
- GONOSUMDB
- GOPRIVATE

这三个环境变量都用在当前项目依赖了私有模块的场景，即依赖了由 GOPROXY 指定的 Go module proxy 或由 GOSUMDB 指定 Go checksum database 无法访问到的模块。

值都是以 `,` 分割的模块路径前缀，匹配规则同 `path.Match`

其中，GOPRIVATE 的值会作为 GONOPROXY 和 GONOSUMDB 的默认值，所以建议只使用 GOPRIVATE。

如：`GOPRIVATE=*.corp.example.com` 表示所有以 corp.example.com 的子域名为前缀的模块路径都将不经过 Go module proxy 和 Go checksum database，而是直接回源以传统方式拉取。注意，以 corp.example.com 本身为前缀的模块路径还是会经过的。

## 主要文件

- go.mod

描述了当前项目(aka 当前模块)的元信息，每行以一个动词开头，目前有以下五个动词：

- module：用于定义当前项目的模块路径
- go：用于设置预期的 Go 版本
- require：用于设置一个特定的模块版本
  - 如果某个模块后面又 '// indirect'，表示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，可能是先手动 go get 拉取下来的，也有可能是所依赖的模块所依赖的
- exclude：用于从使用中排除一个特定的模块版本
- replace：用于将一个模块版本替换为另一个模块版本

```
module example.com/foobar

go 1.13

require(
	example.com/apple v0.1.2
  example.com/banana // indirect
  example.com/banana/v2 v2.3.4
)

exclude example.com/banana v1.2.4

replace example.com/apple v0.1.2 => example.com/rda v0.1.0
```

---

- go.sum

详细罗列了当前项目直接或间接依赖的所有模块版本，并写明了那些模块版本的 SHA-256 哈希值，以备 Go 在今后的操作中保证项目所以来的那些模块版本不会被篡改。通常一个模块版本会出现两行值，第一行是目标模块版本的 zip 文件的哈希值，第二行是 zip 文件中 go.mod 的哈希值。有时也会只出现一行，即 zip 文件中 go.mod 的哈希值。

```
github.com/gin-contrib/sse v0.1.0 h1:Y/yl/+YNO8GZSjAhjMsSuLt29uWRFHdHYUb5lYOV9qE=
github.com/gin-contrib/sse v0.1.0/go.mod h1:RHrZQHXnP2xjPF+u1gW/2HnVO7nvIa9PG3Gm+fLHvGI=
github.com/gin-gonic/gin v1.3.0/go.mod h1:7cKuhb5qV2ggCFctp2fJQ+ErvciLZrIeoOSOm6mUr7Y=
```

## 项目迁移到 Go Modules

1. 升级到 Go 1.13，之前的版本中问题太多

2. 如下：

   ```shell
   go env -w GOBIN=$HOME/bin
   go env -w GO111MODULE=on
   go env -w GOPROXY=https://goproxy.cn,direct
   ```

3. 可选：按照喜欢的目录结构重新组织项目

4. 在项目的根目录下执行 `go mod init` 生成 go.mod 文件

## Go Modules 下的 go get 行为

用 `go help module-get` 和 `go help gopath-get`分别了解 Go modules 启用和未启用两种状态下的 go get 的行为。

在拉取项目依赖时，拉取过程分为 finding、downloading、extracting 三步，且在拉取信息上共分为三段内容(以 `-` 分隔)：版本信息-所拉取版本的 commit 时间-所拉去版本的 commit 哈希。注意，时间以 UTC 时区为准。

- 用 `go get` 拉取新的依赖

  - 拉取最新的版本(优先择取 tag)：`go get golang.org/x/text@latest`
  - 拉取 `master` 分支的最新 commit：`go get golang.org/x/text@master`
  - 拉取 tag 为 v0.3.2 的 commit：`go get golang.org/x/text@v0.3.2`
  - 拉取 hash 为 342b231 的 commit，最终会被转换为 v0.3.2：`go get golang.org/x/text@342b2e`
  - 用 `go get -u` 更新现有的依赖

- `go get` 的版本选择
  - 所拉取的模块有发布 tags
    - 如果只有单个模块，那么就取主版本号最大的那个 tag；
    - 如果有多个模块，则推算相应的模块路径，取主版本号最大的那个 tag（子模块的 tag 的模块路径会有前缀要求）
  - 所拉取的模块没有发布过 tags
    - 默认取主分支最新一次 commit 的 commithash，则拉取的版本视为 v0.0.0 版本

以 mquote 项目举例多个模块：

```
mquote
├── go.mod
├── module
│   └── tour
│       ├── go.mod
│       └── tour.go
└── quote.go
```

可能会打多个 tag：

| tag               | 模块导入路径                          | 含义                                             |
| ----------------- | ------------------------------------- | ------------------------------------------------ |
| v0.0.1            | github.com/eddycjy/mquote             | mquote 项目的 v 0.0.1 版本                       |
| module/tour/v0.01 | github.com/eddycjy/mquote/module/tour | mquote 项目下的子模块 module/tour 的 v0.0.1 版本 |

```shell
# 拉取主模块
go get github.com/eddycjy/mquote@v0.0.1
# 拉取子模块
go get github.com/eddycjy/mquote/module/tour@v0.0.1
```

## 导入路径说明

Go modules 在主版本号为 v0 和 v1 的情况下省略了版本号(强制性的)，而在主版本号为 v2 及以上则需要明确指定出主版本号，否则会出现冲突。

| tag    | 模块导入路径                 |
| ------ | ---------------------------- |
| v0.0.0 | github.com/eddycjy/mquote    |
| v1.0.0 | github.com/eddycjy/mquote    |
| v2.0.0 | github.com/eddycjy/mquote/v2 |

# 内存对齐

[图解 Go 内存对齐](https://www.bilibili.com/video/BV1iZ4y1j7TT)

[在 Go 中恰到好处的内存对齐](https://segmentfault.com/a/1190000017527311)

数据结构对齐工具：

```shell
# layout
go get github.com/ajstarks/svgo/structlayout-svg
go get -u honnef.co/go/tools
go install honnef.co/go/tools/cmd/structlayout
go install honnef.co/go/tools/cmd/structlayout-pretty

# optmize
go install honnef.co/go/tools/cmd/structlayout-optmize
```

# Web

Web 部分的内容是学习[Go 语言 Web 应用开发](https://github.com/unknwon/building-web-applications-in-go)的笔记！

# 相对路径

Golang 的相对路径是**相对于执行命令时的目录**！

`go run` 执行时会将文件放到 /tmp/go-build... 目录下，编译并运行！

`go test` 只能在当前目录下执行，所以执行测试用例时，执行目录就是测试目录。

# 部分类库说明

- `unsafe`: 包含了一些打破 Go 语言“类型安全”的命令，一般的程序中不会被使用，可用在 C/C++ 程序的调用中。
- `syscall`、`os`、`os/exec`：
  - `os`: 提供给我们一个平台无关性的操作系统功能接口，采用类 Unix 设计，隐藏了不同操作系统间差异，让不同的文件系统和操作系统对象表现一致。
  - `os/exec`: 提供我们运行外部操作系统命令和程序的方式。
  - `syscall`: 底层的外部包，提供了操作系统底层调用的基本接口。
- `archive/tar` 和 `/zip-compress`：压缩(解压缩)文件功能。
- `fmt`-`io`-`bufio`-`path/filepath`-`flag`:
  - `fmt`: 提供了格式化输入输出功能。
  - `io`: 提供了基本输入输出功能，大多数是围绕系统功能的封装。
  - `bufio`: 缓冲输入输出功能的封装。
  - `path/filepath`: 用来操作在当前系统中的目标文件名路径。
  - `flag`: 对命令行参数的操作。
- `strings`-`strconv`-`unicode`-`regexp`-`bytes`:
  - `strings`: 提供对字符串的操作。
  - `strconv`: 提供将字符串转换为基础类型的功能。
  - `unicode`: 为 unicode 型的字符串提供特殊的功能。
  - `regexp`: 正则表达式功能。
  - `bytes`: 提供对字符型分片的操作。
  - `index/suffixarray`: 子字符串快速查询。
- `math`-`math/cmath`-`math/big`-`math/rand`-`sort`:
  - `math`: 基本的数学函数。
  - `math/cmath`: 对复数的操作。
  - `math/rand`: 伪随机数生成。
  - `sort`: 为数组排序和自定义集合。
  - `math/big`: 大数的实现和计算。
- `container`-`/list-ring-heap`: 实现对集合的操作。
  - `list`: 双链表。
  - `ring`: 环形链表。
- `time`-`log`:
  - `time`: 日期和时间的基本操作。
  - `log`: 记录程序运行时产生的日志
- `encoding/json`-`encoding/xml`-`text/template`:
  - `encoding/json`: 读取并解码和写入并编码 JSON 数据。
  - `encoding/xml`:简单的 XML1.0 解析器
  - `text/template`:生成像 HTML 一样的数据与文本混合的数据驱动模板
- `net`-`net/http`-`html`:
  - `net`: 网络数据的基本操作。
  - `http`: 提供了一个可扩展的 HTTP 服务器和基础客户端，解析 HTTP 请求和回复。
  - `html`: HTML5 解析器。
- `runtime`: Go 程序运行时的交互操作，例如垃圾回收和协程创建。
- `reflect`: 实现通过程序运行时反射，让程序操作任意类型的变量。

cgo：

Windows 安装 GCC-64：进入[离线安装包](https://sourceforge.net/projects/mingw-w64/files/)，选择下面的 GCC 版本的链接即可开始下载，解压后将 bin 目录加入环境变量中

通过 `import "C"` 语句启用 CGO 特性，同时包含 C 语言的 <stdio.h> 头文件。`go build` 命令会在编译和链接阶段启动 gcc 编译器。
