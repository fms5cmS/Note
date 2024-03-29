# 安装

1. [官网](https://golang.org/dl/)下载 Go 的 tar.gz 安装包，解压到某一目录
2. 在 PATH 中加入 \$GOROOT/bin
3. 使用 `go env -w 环境变量=值` 来设置环境变量
   - Go 1.13 建议所有跟 Go 相关的环境变量都交由 `go env -w` 来管理
   - `go env -w` 不会覆盖系统环境变量
   - 设置：GOROOT（Go 的安装目录）、GOPATH、GO111MODULE、GOPROXY、GOBIN 环境变量
4. 使用 `go version` 验证是否安装成功

[教程](http://topgoer.com/)

## 多个版本

https://golang.org/doc/manage-install

正常安装一个版本后，

```shell
# 通过 go get 指定特定的版本，如 1.16 版本
go get golang.org/dl/go1.16
# 此时在 $GOPATH/bin 下会有一个 go1.16 的可执行文件
# 下载 sdk，SDK 会下载到 $HOME/sdk 目录下！！之后 GoLand 需要在配置中的 GOROOT 中找到该 SDK
go1.16 download
# 验证是否下载成功
go1.16 version
```



# 代码组织

Go 语言是通过包（目录）来组织代码的。环境变量 GOPATH 指向的目录是工作目录，该变量可以包含多个目录路径，Windows 下，多个值之间使用`;`来分隔，Unix 下，值之间使用`:`分隔。每个路径代表一个工作区（workspace）。工作区下的目录结构：

- src：源码所在的目录，该目录下可以有子目录。
- pkg：保存归档文件（扩展名为 .a 的文件，即 archive 文件）这是程序编译后生成的静态库文件。
  - 注意：在 pkg 下实际上还有一个平台相关目录（目标操作系统对应的目录），所有的归档文件都在这个目录下。
- bin：保存编译后的命令。每个命令都是**使用源文件所在直接目录名命名**的。

共有三种源码文件：命令源码文件、库源码文件、测试源码文件

## 示例

以下两个文件都是 Windows 系统 GOPATH 下的！

src/foo/bar/x.go：

```go
package hello  //声明其所属的包

import "fmt" //导入其他的包

func Hello(name string) {  //函数
  fmt.Printf("Hello %s",name)
}
```

src/foo/quux/y.go：

```go
package main   //命令源码文件必须使用 main 作为包 name

import "foo/bar"  //使用文件所在路径来导入

func main() {
  hello.Hello("bx") //使用时则根据声明的包来使用！
}
```

对以上两个源码文件执行 go install 命令后发现此时 GOPATH 下的目录结构为：

```
src/
   foo/
       bar/
           x.go    库源码文件
       quux/
           y.go    命令源码文件
bin/
   quux       可执行文件。注意：这里并没有 foo 目录！且使用源文件所在的直接目录来命名
pkg/
   windows_amd64/        这是一个与平台相关的目录，即操作系统对应的目录
       foo/
           bar.a
```



# 包名

在 Go 程序中的第一行必须声明程序所在包名：`package name`。

**同一个包内的所有文件必须使用相同的 name。name 一般与源码文件所在的目录同名，可以不同名。**如上面的 x.go 文件所在目录名为 bar，但声明的包名为 hello。

**例外：Go test 支持在同一个目录下使用不同包名，只要包名是以 package_test 形式命名即可。详见 strings 包。**

这里以上面的示例来说明包名的使用：

- 当外部要导入 x.go 文件中的内容（变量、函数等）时，**必须使用 x.go 所在的路径为导入路径**，即 y.go 中的 `import "foo/bar"`
  - 标准库内包的导入路径是给定了的，如`fmt`、`net/http`。
  - 对于自定义的包，必须选择一个不会与标准库冲突的基本路径。
  - 如果你将代码保存在远程的代码仓库，那就必须使用远程仓库作为基本路径
- y.go 在导入 x.go 之后，如果要使用 x.go 中的内容，**必须通过 `包名.` 的方式来使用**，即 y.go 中通过 `hello.Hello("bx")` 的方式来调用 x.go 的 Hello 函数

# 编译相关命令

和应用编译相关的命令由如下三个：

`go run [build flags] [-exec xprog] package [arguments...]`

`go build [-o output] [-i] [build flags] [packages]`

`go install [-i] [build flags] [packages]`

不同点：

- run 编译并运行、build 仅编译、install 编译后将相关文件安装到指定目录
- 执行后的结果文件保存路径
  - run 编译后的二进制文件放到临时目录下
  - build 对库源码文件的结果文件放在临时目录，而命令源码文件的结果文件(与其目录名同名)放在与源码文件相同的目录下
  - install 对库源码文件的结果文件放在 `$GOPATH/pkg/$GOOS_$GOARCH` 下(子目录为项目名)，而命令源码文件的结果文件放在 `$GOPATH/bin` 下(没有子目录)
    - 如果设置了环境变量 `$GOBIN` 时，优先安装到 `$GOBIN` 下
- run 仅接收 main 包下的文件作为参数(一个或多个)！build、install 不需要指定文件

常用参数基本相同：

- `-x` 和 `-n` 参数都是打印编译过程中所有执行命令，`-n` 不执行编译后的二进制文件，而 `-x` 会执行
- `-a` 强制重新编译所有涉及的依赖
- `-o 文件名` 指定编译后输出的可执行文件名
- `-race` 开启竞态条件检测
- `-p` 指定编译中可并发运行程序的数量，默认值为可用的 CPU 数量
- `-work` 打印临时工作目录的完整路径，在退出时不删除该目录
- build 结合`-ldflags “-X '包名.变量名=值'”` 向程序中指定包的变量传递值
  - 通常用于将一些编译信息打包进二进制文件中
- `-gcflags '-N -l'` 禁止内联
  - gcflags 代表 go compiler flags
  - `-N` 禁止编译优化
  - `-l` 禁止内联，可以在一定程度上减小可执行程序大小
  - `-m` 查看编译优化信息（逃逸分析等），最多可以加三个 `-m`，`-m` 越多输出信息就越多



# 补充命令

- `go clean` 移除对象文件和缓存文件，`go clean -cache` 清理编译缓存
- `go get`，使用 `go get`前需要安装 git
  - 拉取依赖，会进行指定性拉取（更新），并不会更新所依赖的其它模块。
  - `-u` 更新现有的依赖，会强制更新它所依赖的其它全部模块，不包括自身。
  - `-fix` 在下载代码包后先运行一个用于根据当前 Go 语言版本修正代码的工具，然后再安装代码包
  - `-insecure` 允许通过非安全的网络协议下载和安装代码包。HTTP 就是这样的协议。
- `go fmt` 格式化代码
- `golint` 识别代码中的样式错误
  - `golint file.go` 扫描单个文件
  - `golint .` 扫描当前文件夹下所有文件
  - `golint ./...` 扫描当前项目中的所有文件

下载第三方库时无法访问：

1. 下载 golang.org 的包时，golang.org/x/net 对应 GitHub 上的镜像地址为 https://github.com/golang/net
2. 将其源码 \$GOPATH/src/github.com/golang/net 复制到 \$GOPATH/src/golang.org/x/
3. 然后 `go build net`

# Go Modules

[Go Modules 终极入门](https://mp.weixin.qq.com/s/fNMXfpBhBC3UWTbYCnwIMg)

[Go module 官方文档](https://golang.org/ref/mod)

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
go mod tidy	# 修改 go.mod 和 go.sum 以匹配实际依赖
go mod graph	# 查看现有的依赖结构
go mod edit	# 编辑 go.mod 文件
go mod vendor	# 增加 build&test 依赖到vendor 文件夹下
go mod verify	# 校验本地缓存的文件自下载后是否被篡改过
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

go.mod 描述了当前项目(aka 当前模块)的元信息，每行以一个动词开头，目前有以下五个动词：

- module：用于定义当前项目的模块路径
- go：用于设置预期的 Go 版本
- require：用于设置一个特定的模块版本
  - 如果某个模块后面又 '// indirect'，表示该模块为间接依赖，也就是在当前应用程序中的 import 语句中，并没有发现这个模块的明确引用，可能是先手动 go get 拉取下来的，也有可能是所依赖的模块所依赖的
- exclude：用于从使用中排除一个特定的模块版本
- replace：用于将一个模块版本替换为另一个模块版本，可以解决包的别名问题，也能替我们解决 golang.org/x 无法下载的的问题。

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
   iam59!z$
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

## MVS

[Minimal Version Selection](https://research.swtch.com/vgo-mvs) 是构建 main module 时每个依赖的版本选择算法。

遍历找到每个 module 最大的版本，这些版本的 module 就是满足所有需求的最小版本。 

# linter

codecov 可以给出测试覆盖率的报告

[golangci-lint](https://golangci-lint.run/) 集成了很多的静态代码分析工具，可以对代码静态扫描检查代码规范，常见的静态代码分析工具：

- errcheck：检查错误是否被处理
- funlen：检查函数是否过长
- bodyclose：检查 http body 是否 close
- sqlrows：检查 sql.Rows 是否 close

```shell
golangci-lint linters # 查看所有支持的 linter
golangci-lint -h # 帮助
golangci-lint run # 进行静态代码检查
golangci-lint run -h # 查看 run 的子命令帮助信息
golangci-lint cache clean # 清除 cache
golangci-lint cache status # 查看 cache 状态
golangci-lint config path # 查看配置文件路径
# 默认配置文件名为 .golangci.yaml .golangci.toml .golangci.json 三种
```



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



# 调试

## dlv

Delve 是 Go 程序的源代码级调试器，是 Go 实现的。能够通过控制流程的执行与程序进行交互，查看变量，提供线程、goroutine、CPU 状态等信息。

```shell
# Go 1.16 及以上版本时
go install github.com/go-delve/delve/cmd/dlv@latest
# 也可以通过 git clone 方式安装
git clone https://github.com/go-delve/delve
cd delve
go install github.com/go-delve/delve/cmd/dlv
# 安装完成后，查看
dlv version
```

使用：

```shell
dlv exec ./xx   # 后面是可执行文件的路径，对该程序进行调试
dlv debug xx.go  # 调试
h  # help 操作
s  # 单独执行程序代码，可以跳转进调用的函数里面
si # step-instruction 单步执行一条 CPU 指令
b xx # break 打断点，后面可以是函数名，如：b procresize，可以是 *地址，如：b *0x455780，可以是 文件名:行数，如 b main.go:6
c  # continue 跳转到下一个断点处
bt # 或 stack 可以查看调用栈，从下面调用上面，这里可以看到每次的函数调用前面都有一个序号
up、down # 可以在栈帧的调用关系上下切换
frame xx # 后面跟栈帧序号跳转
n  # next 单步执行程序的下一步，无法跳转进调用的函数里面
disass # 反汇编	
```



示例代码：

```go
package main

import "fmt"

// 反转字符串
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}

func main() {
	fmt.Println(Reverse("猜猜看"))
}
```



```shell
# 在目录下进入 dlv 的 debug 模式交互
$ dlv debug .
# 使用关键字 b（break） 对 main.main 方法设置断点
(dlv) b main.main
Breakpoint 1 set at 0xb065ba for main.main() ./main.go:14
# 使用关键字 c（continue）跳转到下一个断点处，在断点处可以看到代码块、goroutine、CPU 寄存器地址等运行时信息
(dlv) c
> main.main() ./main.go:14 (hits goroutine(1):1 total:1) (PC: 0xb065ba)
     9:                 r[i], r[j] = r[j], r[i]
    10:         }
    11:         return string(r)
    12: }
    13: 
=>  14: func main() {
    15:         fmt.Println(Reverse("猜猜看"))
    16: }
# 使用关键字 n（next）单步执行程序的下一步，可以看到调用了 Reverse() 方法
(dlv) n
> main.main() ./main.go:15 (PC: 0xb065c8)
    10:         }
    11:         return string(r)
    12: }
    13: 
    14: func main() {
=>  15:         fmt.Println(Reverse("猜猜看"))
    16: }
# 使用关键字 s（step）进入到函数中继续调试
(dlv) s
> main.Reverse() d:/modules/local/test/questions/dlv/main.go:6 (PC: 0xb0638a)
     1: package main
     2: 
     3: import "fmt"
     4: 
     5: // 反转字符串
=>   6: func Reverse(s string) string {
     7:         r := []rune(s)
     8:         for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
     9:                 r[i], r[j] = r[j], r[i]
    10:         }
    11:         return string(r)
# 使用关键字 p（print）打印所传入变量 s 的值
(dlv) p s
"猜猜看"
# 针对 Reverse() 方法交换值得位置继续打断点查看
(dlv) b 9
Breakpoint 2 set at 0xb0643a for main.Reverse() ./main.go:9
# 继续查看
(dlv) c
> main.Reverse() ./main.go:9 (hits goroutine(1):1 total:1) (PC: 0xb0643a)
     5: // 反转字符串
     6: func Reverse(s string) string {
     7:         r := []rune(s)
     8:         for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
=>   9:                 r[i], r[j] = r[j], r[i]
    10:         }
    11:         return string(r)
    12: }
    13: 
# 执行关键字 locals，可以看到对应得变量值，并以此分析程序是否符合预期
(dlv) locals
r = []int32 len: 3, cap: 32, [...]
j = 2
i = 0
# 执行关键字 set 针对特定变量设置期望得值
(dlv) set i = 1
(dlv) locals
r = []int32 len: 3, cap: 32, [...]
j = 2
i = 1
# 排查完后，执行关键字 r（reset）重置整个调试，比如上面设置得变量值就会恢复
(dlv) r
Process restarted with PID 8876
# 指定关键字 bp 查看设置的断点情况
(dlv) bp
Breakpoint runtime-fatal-throw (enabled) at 0xa7ca00 for runtime.throw() c:/users/xxx/sdk/go1.16/src/runtime/panic.go:1107 (0)
Breakpoint unrecovered-panic (enabled) at 0xa7cc80 for runtime.fatalpanic() c:/users/xxx/sdk/go1.16/src/runtime/panic.go:1190 (0)
        print runtime.curg._panic.arg
Breakpoint 1 (enabled) at 0xb065ba for main.main() ./main.go:14 (0)
Breakpoint 2 (enabled) at 0xb0643a for main.Reverse() ./main.go:9 (0)
# 若有些部分已经排除，使用关键之 clearall 对一些断定清除，若不指定断点则会清除所有断点
(dlv) clearall main.main
Breakpoint 1 cleared at 0xb065ba for main.main() ./main.go:14
# exit 退出！
(dlv) exit
```

```shell
# 也可以直接借助函数名进行调试定位
(dlv) funcs Reverse
main.Reverse
(dlv) b Reverse
Breakpoint 3 set at 0xb0638a for main.Reverse() ./main.go:6
```

```shell
# 借助 Go 的公共函数计算
(dlv) p len(r)-1
2
# 借助关键字 vars 查看某个包下所有全局变量的值
(dlv) vars main
```

## gdb

GDB 是一个类 UNIX 系统下的程序调试工具，允许你看到另一个程序在执行时 "内部 "发生了什么，或者程序在崩溃时正在做什么。

主要可以做四类事情：

1. 启动你的程序，指定任何可能影响其行为的东西。
2. 使你的程序在指定的条件下停止。
3. 检查当你的程序停止时发生了什么。
4. 改变你程序中的东西，这样你就可以试验纠正一个错误的影响，并继续了解另一个错误。

[学会使用 GDB 调试 Go 代码](https://mp.weixin.qq.com/s/cZoebWQy5smaZac6W7NViw)
