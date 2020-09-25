
文档地址：https://github.com/rubyhan1314/Golang-100-Days


# 语言基础
## 初识Goland
### 安装Goland开发工具
开发工具：
- 文本类编辑器：notepad,sublime text,atom...（通过命令执行程序）
- IED（集成开发环境）：Goland

#### Goland
 Goland是JetBrains公司推出的Go语言IDE，是一款功能强大，使用便捷的产品。
 下载地址：http://www.jetbrains.com/go
 Goland的激活码：http://idea.iblue.me


#### VScode + Go插件
参考：https://www.cnblogs.com/mrbug/archive/2019/12/05/11978765.html
Windows
1）安装Golang
下载Golang安装包：https://golang.google.cn/
安装：`go version`
环境变量：
- GOPATH（工作区）
- GOROOT（bin路径）

2）安装VSCode
官网下载：https://code.visualstudio.com/Download
1. 安装
2. 安装中文包：Extensions--language--中文（简体）
3. 安装Go插件：Go

3）安装Git工具
下载地址：https://git-scm.com/downloads



### 编码规范
#### 命名规范

大小写字符或下划线开头（区分大小写）
注：（包括常量、变量、类型、函数名、结构字段等等）
- 以大写字母开头的标识符的对象，可以被外部包的代码所使用（客户端程序需要先导入这个包），类似面向对象语言中的`public`。
- 以小写字母开头的标识符的对象，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的，类似像面向对象语言中的`private`。

1）包命名(package)
保持package的名字和目录保持一致。
尽量采取简短有意义的包名。
尽量和标准库不要冲突。
包名应该为小写单词，不要使用下划线或者混合大小写。
```go
package demo
package main
```

2）文件命名
尽量采取简短有意义的包名。
使用小写单词，以下划线分隔各个单词。
`my_test.go`

3）结构体命名
采用驼峰命令法。
`struct`申明和初始化格式采用多行。
```go
// 多行申明
type User struct{
    Username    string
    Email       string
}
// 多行初始化
u := User{
    Username:   "wanglaowu",
    Email:      "wanglaowu@gmail.com"
}
```

4）接口命名
命名规则基本和结构体类似。
单个函数的结构名以"er"作为后缀（如：Reader,Writer）。
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

5）变量命名
和结构体类似，变量名称一般遵循驼峰法。
若变量类型为 bool 类型，则名称最好以 Has, Is, Can 或 Allow 开头。
```go
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

6）常量命名
常量均需使用全部大写字母组成，并使用下划线分词。
`const APP_VER = "1.0"`
如果是枚举类型的常量，需要先创建相应类型：
```go
type Scheme string

const (
    HTTP    Scheme = "http"
    HTTPS   Scheme = "https"
)
```

7）关键字
这些保留字不能用作常量或变量或任何其他标识符名称。

![guanjianzi](https://camo.githubusercontent.com/9aca2ccaaf7ea1ec4189d8eba38dcf00c4913e53/687474703a2f2f3778746377642e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f6775616e6a69616e7a692e6a7067)

#### 注释
 Go提供C风格的`/* */`块注释和C++风格的`//`行注释。
1）**包注释**
每个包都应该有一个包注释，位于package子句之前。
包如果有多个go文件，只需要出现在一个go文件中（一般是和包同名的文件）即可。
包注释基本信息：
- 包的基本简介
- 创建者
- 创建时间
```go
// util 包，该包包含了项目共用的一些常量，封装了项目中一些共用函数。
// 创建人：     zhangsan
// 创建时间：   20200924
```

2）**结构（接口）注释**
每个自定义的结构体或者接口都应该有注释说明，该注释对结构进行简要介绍，放在结构体定义的前一行。
格式为： 结构体名， 结构体说明。
同时结构体内的每个成员变量都要有说明，该说明放在成员变量的后面（注意对齐）。
```go
// User, 用户对象，定义了用户的基本信息
type User struct {
    Username    string  //用户名
    Email       string  //邮箱
}
```

3）**函数（方法）注释**
每个函数或者方法都应该有注释说明，函数的注释格式
- 简要说明（格式：函数名，函数说明）
- 参数列表（每行一个参数：参数名，参数说明）
- 返回值（每行一个返回值）
```go
/*
NewtAttrModel, 属性数据层操作类的工厂方法
参数：
    ctx : 上下文信息
返回值：
    属性操作类指针
*/
func NewtAttrModel(ctx *common.Context) *AttrModel {
}
```

4）**代码逻辑注释**
对于一些关键位置的代码逻辑，或者局部较为复杂的逻辑，需要有相应的逻辑说明，方便其他开发者阅读该段代码。

#### 代码风格
1）缩进和折行
- 缩进直接使用 gofmt 工具格式化即可（gofmt 是使用 tab 缩进的）；
- 折行方面，一行最长不超过120个字符，超过的请使用换行展示，尽量保持格式优雅。

2）**语句结尾**
Go语言中是不需要类似于Java需要冒号结尾，默认一行就是一条数据，如果你打算将多个语句写在同一行，它们则必须使用`;`。

3）**括号空格**
括号和空格方面，也可以直接使用 gofmt 工具格式化（go 会强制左大括号不换行，换行会报语法错误），所有的运算符和操作数之间要留空格。

4）**import规范**
import在多行的情况下，goimports会自动帮你格式化，但是我们这里还是规范一下import的一些规范.
如果你在一个文件里面引入了一个package，还是建议采用如下格式：
```go
import (
    "fmt"
)
```
如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包：
```go
import (
    "encoding/json"
    "strings"

    "myproject/models"

    "github.com/astaxie/beego"
    "github.com/go-sql-driver/mysql"
)
```
5）**错误处理**
- 错误处理的原则就是不能丢弃任何有返回err的调用，不要使用 _ 丢弃，必须全部处理。接收到错误，要么返回err，或者使用log记录下来
- 尽早return：一旦有错误发生，马上返回
- 尽量不要使用panic，除非你知道你在做什么
- 错误描述如果是英文必须为小写，不需要标点结尾
- 采用独立的错误流进行处理

```GO
// 错误写法
if err != nil {
    // error handling
} else {
    // normal code
}
// 正确写法
if err != nil {
    // error handling
    return // or continue, etc.
}
// normal code
```

#### 常用工具
1）**gofmt**
大部分的格式问题可以通过gofmt解决， gofmt 自动格式化代码，保证所有的 go 代码与官方推荐的格式保持一致，于是所有格式有关问题，都以 gofmt 的结果为准。

2）**goimport**
烈建议使用 goimport ，该工具在 gofmt 的基础上增加了自动删除和引入包.
`go get golang.org/x/tools/cmd/goimports`

3）**go vet**
vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等。
`go get golang.org/x/tools/cmd/vet`



### Go的特性
 Go 语言具有很强的表达能力，它简洁、清晰而高效。得益于其并发机制， 用它编写的程序能够非常有效地利用多核与联网的计算机，其新颖的类型系统则使程序结构变得灵活而模块化。 Go 代码编译成机器码不仅非常迅速，还具有方便的垃圾收集机制和强大的运行时反射机制。 它是一个快速的、静态类型的编译型语言，感觉却像动态类型的解释型语言。(摘取自官网)
#### 核心特性
1）**并发编程**
***高并发是Golang语言最大的亮点***
不同于传统的多进程或多线程，golang的并发执行单元是一种称为goroutine的协程。
Golang语言级别支持协程(goroutine)并发（协程又称微线程，比线程更轻量、开销更小，性能更高），操作起来非常简单，语言级别提供关键字（go）用于启动协程，并且在同一台机器上可以启动成千上万个协程。
协程经常被理解为轻量级线程，一个线程可以包含多个协程，共享堆不共享栈。协程间一般由应用程序显式实现调度，上下文切换无需下到内核层，高效不少。协程间一般不做同步通讯，而golang中实现协程间通讯有两种：1）共享内存型，即使用全局变量+mutex锁来实现数据共享；2）消息传递型，即使用一种独有的channel机制进行异步通讯。

2）**内存回收(GC)**
- 内存自动回收，再也不需要开发人员管理内存
- 开发人员专注业务实现，降低了心智负担
- 只需要new分配内存，不需要释放

3）**内存分配**
初始化阶段直接分配一块大内存区域，大内存被切分成各个大小等级的块，放入不同的空闲list中，对象分配空间时从空闲list中取出大小合适的内存块。内存回收时，会把不用的内存重放回空闲list。空闲内存会按照一定策略合并，以减少碎片。

4）**编辑**
目前Golang的两种编译器：
- 建立在GCC基础上的Gccgo
- 分别针对64位x64和32位x86计算机的一套编译器(6g和8g)
依赖管理方面，由于golang绝大多数第三方开源库都在github上，在代码的import中加上对应的github路径就可以使用了，库会默认下载到工程的pkg目录下。

补充：编译时会默认检查代码中所有实体的使用情况，凡是没使用到的package或变量，都会编译不通过。这是golang挺严谨的一面。

5）**网络编程**
由于golang诞生在互联网时代，因此它天生具备了去中心化、分布式等特性，具体表现之一就是提供了丰富便捷的网络编程接口，比如socket用net.Dial(基于tcp/udp，封装了传统的connect、listen、accept等接口)、http用http.Get/Post()、rpc用client.Call('class_name.method_name', args, &reply)，等等。

6）**函数多返回值**

7）**语言交互性**
golang可以和C程序交互，但不能和C++交互。可以有两种替代方案：1）先将c++编译成动态库，再由go调用一段c代码，c代码通过dlfcn库动态调用动态库（记得export LD_LIBRARY_PATH）；2）使用swig(没玩过)

8）**异常处理**
golang不支持try...catch这样的结构化的异常解决方式，因为觉得会增加代码量，且会被滥用，不管多小的异常都抛出。golang提倡的异常处理方式是：
- 普通异常：被调用方返回error对象，调用方判断error对象。
- 严重异常：指的是中断性panic（比如除0），使用defer...recover...panic机制来捕获处理。严重异常一般由golang内部自动抛出，不需要用户主动抛出，避免传统try...catch写得到处都是的情况。当然，用户也可以使用panic('xxxx')主动抛出，只是这样就使这一套机制退化成结构化异常机制了。

### Go的第一个程序
#### Go项目的工程结构
1）**gopath目录**
gopath目录就是我们存储我们所编写源代码的目录。该目录下往往要有3个子目录：src，bin，pkg。
- src：里面每一个子目录就是一个包，包内是Go的源码文件。
- pkg：编译后生成的，包的目标文件。
- bin：生成的可执行文件。

2）**包的说明**





#### Helloworld
```go
package main

import "fmt"

func main() {
   fmt.Println("Hello, World!")
}
/* 说明
package：
- 在同一个包下面的文件属于同一个工程文件，不用import包，可以直接使用。
- 在同一个包下面的所有文件的package名，都是一样的。
- 在同一个包下面的文件package名都建议设为是该目录名。

import：
import "fmt" 告诉 Go 编译器这个程序需要使用 fmt 包的函数，fmt 包实现了格式化 IO（输入/输出）的函数。
可以是相对路径也可以是绝对路径，推荐使用绝对路径（起始于工程根目录）
1. 点操作
点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名:
import(
    .   "fmt"
)
fmt.Println("hello world") ==> Println("hello world")

2. 别名操作
把包命名成另一个我们用起来容易记忆的名字。
import(
    f   "fmt"
)
fmt.Println("hello world") ==> f.Println("hello world")

3. _操作
引入该包，不直接使用包里面的函数，而是调用了该包里面的init函数。
import (
  "database/sql"
  _ "github.com/ziutek/mymysql/godrv"
) 

main():
程序运行的入口。
*/
```

#### Go的源码文件

![yuanmawenjian1](https://github.com/rubyhan1314/Golang-100-Days/raw/master/Day01-15(Go%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80)/img/yuanmawenjian1.png)

1）**命令源码文件**
命令源码文件是 Go 程序的入口。声明自己属于 main 代码包。同一个代码包中最好也不要放多个命令源码文件。
命令源码文件被安装以后，GOPATH 如果只有一个工作区，那么相应的可执行文件会被存放当前工作区的 bin 文件夹下；如果有多个工作区，就会安装到 GOBIN 指向的目录下。
2）**库源码文件**
库源码文件就是不具备命令源码文件上述两个特征的源码文件。存在于某个代码包中的普通的源码文件。
库源码文件被安装后，相应的归档文件（.a 文件）会被存放到当前工作区的 pkg 的平台相关目录下。
3）**测试源码文件**
名称以 _test.go 为后缀的代码文件，并且必须包含 Test 或者 Benchmark 名称前缀的函数：
- 名称以 Test 为名称前缀的函数，只能接受 *testing.T 的参数，这种测试函数是功能测试函数。
- 名称以 Benchmark 为名称前缀的函数，只能接受 *testing.B 的参数，这种测试函数是性能测试函数。






#### Go命令
1）**Go程序的两种方式**
- `go run`:进入helloworld.go所在目录，执行`go run helloworld.go`
- `go build`:在任意路径执行`go build path/to/helloworld.go`生成二进制文件，再执行。

2）**通用命令标记**

| 名称  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| -a    | 用于强制重新编译所有涉及的 Go 语言代码包（包括 Go 语言标准库中的代码包），即使它们已经是最新的了。该标记可以让我们有机会通过改动底层的代码包做一些实验。 |
| -n    | 使命令仅打印其执行过程中用到的所有命令，而不去真正执行它们。如果不只想查看或者验证命令的执行过程，而不想改变任何东西，使用它正好合适。 |
| -race | 用于检测并报告指定 Go 语言程序中存在的数据竞争问题。当用 Go 语言编写并发程序的时候，这是很重要的检测手段之一。 |
| -v    | 用于打印命令执行过程中涉及的代码包。这一定包括我们指定的目标代码包，并且有时还会包括该代码包直接或间接依赖的那些代码包。这会让你知道哪些代码包被执行过了。 |
| -work | 用于打印命令执行时生成和使用的临时工作目录的名字，且命令执行完成后不删除它。这个目录下的文件可能会对你有用，也可以从侧面了解命令的执行过程。如果不添加此标记，那么临时工作目录会在命令执行完毕前删除。 |
| -x    | 使命令打印其执行过程中用到的所有命令，并同时执行它们。       |


## 语法
### 基本语法
#### 变量
 变量是为存储特定类型的值而提供给内存位置的名称。所以变量的本质就是一小块内存，用于存储数据，在程序运行过程中数值可以改变。
1）**变量声明**
单变量声明：
- 指定变量类型：`var name type`
- 根据值自动判断变量类型：`var name = value`
- 简短定义方式`:=`：`name := value`
  注：该方式只能被用在函数体内，而不可以用于全局变量的声明与赋值，且该变量是新的变量。
多变量声明：
- 以逗号分隔：`var name1, name2, name3 = v1, v2, v3`
- 集合类型
  ```
   var (
    name1 type1
    name2 type2
  )
  ```

2）**注意事项**
1. 变量必须先定义才能使用
2. go语言是静态语言，要求变量的类型和赋值的类型必须一致。
3. 变量名不能冲突。(同一个作用于域内不能冲突)
4. 简短定义方式，左边的变量名至少有一个是新的
5. 简短定义方式，不能定义全局变量。
6. 变量的零值。也叫默认值。
7. 变量定义了就要使用，否则无法通过编译。

#### 常量
常量是一个简单值的标识符，在程序运行时，不会被修改的量。
1）**常量声明**
- 显示类型定义：`const b string = "abc"`
- 隐式类型定义：`const b = "abc"`

2）**注意事项**
1. 常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。
2. 不曾使用的常量，在编译的时候，是不会报错的。
3. 显示指定类型的时候，必须确保常量左右值类型一致，需要时可做显示类型转换。这与变量就不一样了，变量是可以是不同的类型值。

3）**iota**
iota，特殊常量，可以认为是一个可以被编译器修改的常量。
第一个 iota 等于 0，每当 iota 在新的一行被使用时，它的值都会自动加 1。

#### 数据类型
1）**Go中可用的基本数据类型**
- 布尔型bool
- 数值型（整数型/浮点型等）
- 字符串

![002shujuleixng](https://github.com/rubyhan1314/Golang-100-Days/raw/master/Day01-15(Go%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80)/img/002shujuleixng.jpg)

补充：数据类型转换（Type Convert）
语法格式：`Type(Value)`
- 常量：在有需要时，会自动转型
- 变量：需要手动转型

2）**复合类型（派生类型）**
1、指针类型（Pointer） 2、数组类型 3、结构化类型(struct) 4、Channel 类型 5、函数类型 6、切片类型 7、接口类型（interface） 8、Map 类型

#### 运算符
表达式：`(a + b) * c    //a,b,c叫做操作数；+，*，叫做运算符`
1)**算术运算符**
`+ - * / %(求余) ++ --`
2)**关系运算符**
`== != > < >= <=`
3)**逻辑运算符**

| 运算符 | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| &&     | 所谓逻辑与运算符。如果两个操作数都非零，则条件变为真         |
| \|\|   | 所谓的逻辑或操作。如果任何两个操作数是非零，则条件变为真     |
| !      | 所谓逻辑非运算符。使用反转操作数的逻辑状态。如果条件为真，那么逻辑非操后结果为假 |

4）**位运算符**
略

5）**赋值运算符**

| 运算符 | 描述                                                         | 示例                         |
| ------ | ------------------------------------------------------------ | ---------------------------- |
| =      | 简单的赋值操作符，分配值从右边的操作数左侧的操作数           | C = A + B 将分配A + B的值到C |
| +=     | 相加并赋值运算符，它增加了右操作数左操作数和分配结果左操作数 | C += A 相当于 C = C + A      |
| -=     | 减和赋值运算符，它减去右操作数从左侧的操作数和分配结果左操作数 | C -= A 相当于 C = C - A      |
| *=     | 乘法和赋值运算符，它乘以右边的操作数与左操作数和分配结果左操作数 | C *= A 相当于 C = C * A      |
| /=     | 除法赋值运算符，它把左操作数与右操作数和分配结果左操作数     | C /= A 相当于 C = C / A      |
| %=     | 模量和赋值运算符，它需要使用两个操作数的模量和分配结果左操作数 | C %= A 相当于 C = C % A      |
| <<=    | 左移位并赋值运算符                                           | C <<= 2 相同于 C = C << 2    |
| >>=    | 向右移位并赋值运算符                                         | C >>= 2 相同于 C = C >> 2    |
| &=     | 按位与赋值运算符                                             | C &= 2 相同于 C = C & 2      |
| ^=     | 按位异或并赋值运算符                                         | C ^= 2 相同于 C = C ^ 2      |
| \|=    | 按位或并赋值运算符                                           | C \|= 2 相同于 C = C \| 2    |
6）**运算符优先级**
由上至下代表优先级由高到低：

| 优先级 | 运算符           |
| ------ | ---------------- |
| 7      | ~ ! ++ --        |
| 6      | * / % << >> & &^ |
| 5      | + - ^            |
| 4      | == != < <= >= >  |
| 3      | <-               |
| 2      | &&               |
| 1      | \|\|             |

#### 输入输出
1）**fmt包**
fmt包实现了类似C语言printf和scanf的格式化I/O。
官方文档：https://golang.google.cn/pkg/fmt/

2）**常用打印函数**
- 打印：`func Print(a ...interface{}) (n int, err error)`
- 格式化打印：`func Printf(format string, a ...interface{}) (n int, err error)`
- 打印后换行：`func Println(a ...interface{}) (n int, err error)`

**补充1：**格式化打印中的常用占位符
```GO
格式化打印占位符：
			%v,原样输出
			%T，打印类型
			%t,bool类型
			%s，字符串
			%f，浮点
			%d，10进制的整数
			%b，2进制的整数
			%o，8进制
			%x，%X，16进制
				%x：0-9，a-f
				%X：0-9，A-F
			%c，打印字符
			%p，打印地址
			. . . 
```

**示例1：**
```GO
package main

import (
        "fmt"
)

func main() {
        a := 100           //int
        b := 3.14          //float64
        c := true          // bool
        d := "Hello World" //string
        e := `Ruby`        //string
        f := 'A'
        fmt.Printf("%T,%b\n", a, a)
        fmt.Printf("%T,%f\n", b, b)
        fmt.Printf("%T,%t\n", c, c)
        fmt.Printf("%T,%s\n", d, d)
        fmt.Printf("%T,%s\n", e, e)
        fmt.Printf("%T,%d,%c\n", f, f, f)
        fmt.Println("-----------------------")
        fmt.Printf("%v\n", a)
        fmt.Printf("%v\n", b)
        fmt.Printf("%v\n", c)
        fmt.Printf("%v\n", d)
        fmt.Printf("%v\n", e)
        fmt.Printf("%v\n", f)

}
// 输出：
int,1100100
float64,3.140000
bool,true
string,Hello World
string,Ruby
int32,65,A
-----------------------
100
3.14
true
Hello World
Ruby
65
```
3）**fmt包读取键盘输入**
常用方法：
- `func Scan(a ...interface{}) (n int, err error)`
- `func Scanf(format string, a ...interface{}) (n int, err error)`
- `func Scanln(a ...interface{}) (n int, err error)`
**示例2：**
```GO
package main

import (
        "fmt"
)

func main() {
        var x int
        var y float64
        fmt.Println("请输入一个整数，一个浮点类型：")
        fmt.Scanln(&x,&y)//读取键盘的输入，通过操作地址，赋值给x和y   阻塞式
        fmt.Printf("x的数值：%d，y的数值：%f\n",x,y)

        fmt.Scanf("%d,%f",&x,&y)
        fmt.Printf("x:%d,y:%f\n",x,y)
}

// 输出：
请输入一个整数，一个浮点类型：
100 1.11
x的数值：100，y的数值：1.110000
200,2.22
x:200,y:2.220000
```
4）**bufio包读取**
官方文档：https://golang.google.cn/pkg/bufio/
先创建Reader对象，在读取。
**示例3:**
```GO
package main

import (
        "fmt"
        "os"
        "bufio"
)

func main() {
        fmt.Println("请输入一个字符串：")
        reader := bufio.NewReader(os.Stdin)
        s1, _ := reader.ReadString('\n')
        fmt.Println("读到的数据：", s1)

}

// 输出：
请输入一个字符串：
Hello Go.
读到的数据： Hello Go.
```

### 流程控制结构
程序的流程控制结构一共有三种：顺序结构，选择结构，循环结构。
- 顺序结构：从上向下，逐行执行。
- 选择结构：条件满足，某些代码才会执行。0-1次
- ​分支语句：if，switch，select
- 循环结构：条件满足，某些代码会被反复的执行多次。0-N次
- 循环语句：for

#### IF语句
1）**基本语法**
语法格式一.`if{...}`
```GO
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
}
```
语法格式二.`if{...} else{...}`
```GO
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
} else {
  /* 在布尔表达式为 false 时执行 */
}
```
语法格式三.`if{...} else if{...} else{...}`
```GO
if 布尔表达式1 {
   /* 在布尔表达式1为 true 时执行 */
} else if 布尔表达式2{
   /* 在布尔表达式1为 false ,布尔表达式2为true时执行 */
} else{
   /* 在上面两个布尔表达式都为false时，执行*/
}
```
**示例1：**
```GO
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 10
 
   /* 使用 if 语句判断布尔表达式 */
   if a < 20 {
       /* 如果条件为 true 则执行以下语句 */
       fmt.Printf("a 小于 20\n" )
   }
   fmt.Printf("a 的值为 : %d\n", a)
}
```
2）**IF变体**
如果其中包含一个可选的语句组件(在评估条件之前执行)，则还有一个变体。
```GO
if statement; condition {  
}

if condition{
    
    
}
```
**示例2：**
```GO
package main

import (  
    "fmt"
)

func main() {  
    if num := 10; num % 2 == 0 { //checks if number is even
        fmt.Println(num,"is even") 
    }  else {
        fmt.Println(num,"is odd")
    }
}
// 注：num的定义在if里，那么只能够在该if..else语句块中使用，否则编译器会报错的。
```

#### switch语句
 Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码。

1）**语法格式**
```GO
switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
// VALUE必须是同一个类型，测试多个可能符合条件的值，使用逗号分割它们，例如：case val1, val2, val3。
```
**示例1：**
```GO
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var grade string = "B"
   var marks int = 90

   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"  //case 后可以由多个数值
      default: grade = "D"
   }

   switch {
      case grade == "A" :
         fmt.Printf("优秀!\n" )
      case grade == "B", grade == "C" :
         fmt.Printf("良好\n" )
      case grade == "D" :
         fmt.Printf("及格\n" )
      case grade == "F":
         fmt.Printf("不及格\n" )
      default:
         fmt.Printf("差\n" );
   }
   fmt.Printf("你的等级是 %s\n", grade );
}
// 输出：
优秀!
你的等级是 A
```
2）**如需贯通后续的`case`，就添加`fallthrough`**
```GO
package main

import (
        "fmt"
)

type data [2]int

func main() {
        switch x := 5; x {
        default:
                fmt.Println(x)
        case 5:
                x += 10
                fmt.Println(x)
                fallthrough
        case 6:
                x += 20
                fmt.Println(x)

        }

}
// 输出：
15
35
```
3）**case中的表达式省略时，表示`switch true`**
**示例3：**
```GO
package main

import (
    "fmt"
)

func main() {
    num := 75
    switch { // expression is omitted
    case num >= 0 && num <= 50:
        fmt.Println("num is greater than 0 and less than 50")
    case num >= 51 && num <= 100:
        fmt.Println("num is greater than 51 and less than 100")
    case num >= 101:
        fmt.Println("num is greater than 100")
    }

}
// 输出：
num is greater than 51 and less than 100
```
**注1：**
- case后的常量值不能重复
- case后可以有多个常量值
- fallthrough应该是某个case的最后一行。如果它出现在中间的某个地方，编译器就会抛出错误。
4）**Type Switch**
switch 语句还可以被用于 type-switch 来判断某个 interface 变量中实际存储的变量类型。
```GO
switch x.(type){
    case type:
       statement(s);      
    case type:
       statement(s); 
    /* 你可以定义任意个数的case */
    default: /* 可选 */
       statement(s);
}
```
**示例1：**
```GO
package main

import "fmt"

func main() {
   var x interface{}

   switch i := x.(type) {
      case nil:
         fmt.Printf(" x 的类型 :%T",i)
      case int:
         fmt.Printf("x 是 int 型")
      case float64:
         fmt.Printf("x 是 float64 型")
      case func(int) float64:
         fmt.Printf("x 是 func(int) 型")
      case bool, string:
         fmt.Printf("x 是 bool 或 string 型" )
      default:
         fmt.Printf("未知型")
   }
}
// 输出：
x 的类型 :<nil>
```



