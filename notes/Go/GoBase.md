# Go语言快速入门

目录介绍：
- GOROOT 目录
  解压压缩包存放的目录
  (建议：linux 环境解压到 /usr/local 目录; windows 环境解压到 C:\ProgramFiles 目录)
- GOPATH 目录
  “工作目录”，该目录可以有多个，建议只设置一个。
  GOPATH 目录下 新建 {src,bin,pkg} 子目录：
  ```shell
   src 目录，该目录用于存放第三方库源码，以及存放我们的项目的源码。
   bin 目录，该目录用于存放项目中所生成的可执行文件。
   pkg 目录，该目录用于存放编译生成的库文件。
  ```
## Go 环境配置
### 安装

***Yum 安装：***
`yum install golang -y`

***源码安装：***

```shell
# 源码下载
# 下载地址：https://golang.org/dl/
cd /usr/local
wget https://dl.google.com/go/go1.14.3.linux-amd64.tar.gz
tar xf go1.14.3.linux-amd64.tar.gz
# 新建GOPATH目录
mkdir -p $HOME/gopath
```

### 环境变量配置
- 查看默认环境变量：
  `go env` 显示go环境变量详细信息
  ```shell
  # go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/root/go"
GOPRIVATE=""
GOPROXY="direct"
GOROOT="/usr/lib/golang"
GOSUMDB="off"
GOTMPDIR=""
GOTOOLDIR="/usr/lib/golang/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build108887598=/tmp/go-build -gno-record-gcc-switches"
  ```
- 修改环境变量
  ```shell
  vim ~/.bash_profile 或者 vim ~/.zshrc
  #设置为go安装的路径
  export GOROOT=/usr/local/go
  #默认安装包的路径
  export GOPATH=/code/goDemo
  export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
  # 立即生效配置
  source /etc/profile
  mkdir -p /code/goDemo/{src,bin,pkg}
  ```
### 卸载 Go
- 删除 Go 安装目录，如：`/usr/local/go`
- 删除环境变量配置
### 运行示例
1. 编写代码
   ```go
   cat src/hello.go
   package main
   import "fmt" 
   func main() {
     fmt.Println("Hello, World!")
   }
   ```
2. 直接运行输出
   ```shell
   go run src/hello.go
   Hello, World!
   ```
3. 编译运行
   ```shell
   go build src/hello.go
   ./hello
   Hello, World!
   ```
补充：
在线运行go代码网站：https://play.golang.org/

### 依赖管理工具Go Modules
#### 介绍

在Go Modules之前，Go常用依赖管理工具主要有三个：[godep](https://github.com/tools/godep)、[vendor](https://github.com/kardianos/govendor)和[db](https://github.com/constabulary/gb/)

go modules 在go1.11版推出，go1.12版功能不断改进，再到go1.13版完善优化，正式扶正（官方正式推荐）。
***特点：***
- Go Modules 是官方正式推出的包依赖管理项目。
- Go modules 出现的目的之一就是为了解决 GOPATH 的问题。以前项目必须在$GOPATH/src 里进行，现在Go 允许在$GOPATH/src外的任何目录下使用 go.mod 创建项目。
- 模块代理协议（Module proxy protocol），通过这个协议我们可以实现 Go 模块代理（Go module proxy），也就是依赖镜像。
- Tag必须遵循语义化版本控制，如果没有将忽略 Tag，然后根据你的 Commit 时间和哈希值再为你生成一个假定的符合语义化版本控制的版本号。
- Go modules 还默认认为，只要你的主版本号不变，那这个模块版本肯定就不包含 Breaking changes，因为语义化版本控制就是这么规定的。
- Global Caching 模块的全局缓存，同一个模块版本的数据只缓存一份，所有其他模块共享使用。（清理缓存：`go clean -modcache`）
#### 配置
```shell
#修改 GOBIN 路径（可选）
go env -w GOBIN=$HOME/bin
#打开 Go modules
go env -w GO111MODULE=on
#设置 GOPROXY
go env -w GOPROXY=https://goproxy.cn,direct
或
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
```
**GO111MODULE：**
Go modules 的开关，（auto|on|off）
**GOPROXY:**
设置 Go 模块代理，以英文逗号分割Go module proxy 列表。默认是proxy.golang.org(国内无法访问)。
`GOPROXY=https://mirrors.aliyun.com/goproxy/,direct`
“direct” 为特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取(比如 GitHub 等)，当值列表中上一个 Go module proxy 返回 404 或 410 错误时，Go 自动尝试列表中的下一个，遇见 “direct” 时回源，遇见 EOF 时终止并抛出类似 “invalid version: unknown revision…” 的错误。
#### 实例
```shell
## 创建项目根目录
mkdir -p /var/www/demo
cd /var/www/demo
## 新建测试程序
cat main.go
package main

import (
	"github.com/gin-gonic/gin"
	"fmt"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		fmt.Println("hello world!")
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}

## 在/var/www/demo根目录下生成go.mod文件
go mod init demo
cat go.mod
module demo

go 1.14
## build 构建项目
go build
ls
demo  go.mod  go.sum  main.go
```
**两个生成的文件：**
go.mod (初始化项目生成的文件)
启用了 Go moduels 的项目所必须的最重要的文件，它描述了当前项目（也就是当前模块）的元信息，每一行都以一个动词开头，目前有以下 5 个动词:
- module：用于定义当前项目的模块路径。
- go：用于设置预期的 Go 版本。
- require：用于设置一个特定的模块版本。
- exclude：用于从使用中排除一个特定的模块版本。
- replace：用于将一个模块版本替换为另外一个模块版本。

go.sum (构建项目生成的文件)
类似dep 的 Gopkg.lock 一类文件，它详细罗列了当前项目直接或间接依赖的所有模块版本，并写明了那些模块版本的 SHA-256 哈希值以备 Go在今后的操作中保证项目所依赖的那些模块版本不会被篡改。

**两个命令：**
go get
go mod

## 编程基础
### 格式化输入输出
Usage：

`%[标记][宽度][.精度][arg索引]动词`

`Print(arg列表)`、`Println(arg列表)`、`Printf(格式字符串, arg列表)`

- 标记
  
  | 符号 | 说明                                                     |
  | ---- | -------------------------------------------------------- |
  | +    | 总打印数值的正负号                                       |
  | -    | 左对齐该区域（左侧填充空格）                             |
  | #    | 备用格式，0（%#o）八进制；0x（%#x）或  0X（%#X）十六进制 |
  | ' '  | （空格），数值时省略的正负号留出空白（% d）；            |
  | 0    | 填充前导的0而非空格；                                    |
  
- `[宽度][.精度]`
  宽度值：用于设置最小宽度。
  精度值：对于浮点型，用于控制小数位数，对于字符串或字节数组，用于控制字符个数。
  
  | 形式     | 说明                                          |
  | -------- | --------------------------------------------- |
  | 数值     | 指定的数值作为宽度值或精度值                  |
  | *        | 使用当前正在处理的 arg 的值作为宽度值或精度值 |
  | arg索引* | 使用指定 arg 的值作为宽度值或精度值           |
  
- arg 索引
  用于指定当前要处理的 arg 的序号（由中括号和 arg 序号组成），如：`"abc%+ #8.3[3]vdef"`中的[3]
  
- 动词
  
  v    默认格式，不同类型的默认格式如下：
  
  - 布尔型：`t`
  - 整　型：`d`
  - 浮点型：`g`
  - 复数型：`g`
  - 字符串：`s`
  - 通　道：`p`
  - 指　针：`p`
  
  #v   默认格式，以符合 Go 语法的方式输出。
  
  T    输出 arg 的类型而不是值。
  
  **注：**`动词`不能省略，不同的数据类型支持的动词不一样。




