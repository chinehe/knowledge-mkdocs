前面章节中，我们在安装Golang后，使用`go version`命令来检查安装是否成功。

我们可以使用`go`命令来执行各种操作，包括编译、运行、测试代码等。Go工具链提供了丰富的命令来帮助开发者高效地进行开发工作。本文将详细介绍Go语言的各种命令及其使用方法。

## 1. Go命令概览

要查看Go命令的帮助文档，可以在终端中执行以下命令：

```bash
go help # 查看所有可用的go命令，将显示Go工具链支持的所有命令列表
go help <command> # 查看指定命令的帮助文档
go help <topic> # 查看指定话题的帮助文档
```

go1.25.0版本`go help`命令输出如下：

```
Go is a tool for managing Go source code.

Usage:

	go <command> [arguments]

The commands are:

	bug         start a bug report
	build       compile packages and dependencies
	clean       remove object files and cached files
	doc         show documentation for package or symbol
	env         print Go environment information
	fix         update packages to use new APIs
	fmt         gofmt (reformat) package sources
	generate    generate Go files by processing source
	get         add dependencies to current module and install them
	install     compile and install packages and dependencies
	list        list packages or modules
	mod         module maintenance
	work        workspace maintenance
	run         compile and run Go program
	telemetry   manage telemetry data and settings
	test        test packages
	tool        run specified go tool
	version     print Go version
	vet         report likely mistakes in packages

Use "go help <command>" for more information about a command.

Additional help topics:

	buildconstraint build constraints
	buildjson       build -json encoding
	buildmode       build modes
	c               calling between Go and C
	cache           build and test caching
	environment     environment variables
	filetype        file types
	goauth          GOAUTH environment variable
	go.mod          the go.mod file
	gopath          GOPATH environment variable
	goproxy         module proxy protocol
	importpath      import path syntax
	modules         modules, module versions, and more
	module-auth     module authentication using go.sum
	packages        package lists and patterns
	private         configuration for downloading non-public code
	testflag        testing flags
	testfunc        testing functions
	vcs             controlling version control with GOVCS

Use "go help <topic>" for more information about that topic.
```

## 2. 基础命令

### 2.1. go version - 查看版本信息

用于查看当前安装的Go版本，或者查看Go二进制文件的构建信息：

```bash
go version # 最常用命令，查看go版本
```

输出示例：
```
go version go1.21.0 darwin/amd64
```

#### 2.1.1. 命令语法

```bash
go version [-m] [-v] [-json] [file ...]
```

#### 2.1.2. 参数说明

- `-m`: 打印每个文件内嵌的模块版本信息（如果可用）。在输出中，模块信息由版本行后面的多行组成，每行都以制表符开头。
- `-v`: 在扫描目录时报告未识别的文件。
- `-json`: 类似于-m，但以JSON格式输出runtime/debug.BuildInfo。如果指定了-json而没有-m，go version会报告错误。

#### 2.1.3. 使用示例

```bash
## 查看当前Go工具链的版本
go version

## 查看指定二进制文件的Go版本
go version myapp

## 查看二进制文件的模块版本信息
go version -m myapp

## 以JSON格式输出构建信息
go version -m -json myapp

## 查看目录下所有Go二进制文件的版本
go version /path/to/bin

## 查看目录下所有文件（包括未识别的文件）的版本信息
go version -v /path/to/bin
```

#### 2.1.4. 注意事项

- 如果命令行上没有指定文件，则`go version`打印其自身的版本信息
- 如果指定了目录名称，`go version`会递归遍历该目录，查找可识别的Go二进制文件并报告它们的版本
- 默认情况下，`go version`在目录扫描期间不会报告未识别的文件，使用`-v`标志可以显示这些文件
- 更多关于构建信息的信息，请参见：`go doc runtime/debug.BuildInfo`

### 2.2. go env - 管理环境变量

用于查看和设置Go环境变量：

#### 2.2.1. 命令语法

```bash
go env [-json] [-changed] [-u] [-w] [var ...]
```

#### 2.2.2. 功能说明

Env命令用于打印Go环境信息。默认情况下，env以shell脚本格式（在Windows上是批处理文件格式）打印信息。如果在参数中给出了一个或多个变量名，env会在单独的行上打印每个命名变量的值。

#### 2.2.3. 参数说明

- `-json`: 以JSON格式而不是shell脚本格式打印环境信息
- `-changed`: 仅打印那些有效值与在空环境中获得的默认值不同的设置，且这些设置之前没有使用过-w标志
- `-u`: 需要一个或多个参数，并取消已使用'go env -w'设置的命名环境变量的默认设置
- `-w`: 需要一个或多个NAME=VALUE形式的参数，并将命名环境变量的默认设置更改为给定值

#### 2.2.4. 使用示例

```bash
## 查看所有环境变量
go env

## 查看特定环境变量
go env GOROOT
go env GOPATH
go env GOMODCACHE

## 以JSON格式查看环境变量
go env -json
go env -json GOPATH GOROOT

## 设置环境变量
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOPATH=$HOME/go
go env -w GOBIN=$HOME/go/bin

## 恢复环境变量默认值
go env -u GOPROXY
go env -u GOPATH GOBIN

## 查看已更改的环境变量设置
go env -changed
```

#### 2.2.5. 常用环境变量

Go有许多环境变量，以下是一些常用的环境变量：

- `GOROOT`: Go的安装路径
- `GOPATH`: Go的工作区路径（Go 1.11版本后使用Go Modules时可不设置）
- `GOBIN`: 可执行文件的存放路径
- `GOPROXY`: 模块代理地址
- `GOMODCACHE`: 模块缓存目录
- `GOOS`: 目标操作系统
- `GOARCH`: 目标架构
- `CGO_ENABLED`: 是否启用CGO

#### 2.2.6. 注意事项

- 使用`go env -w`设置的环境变量会保存在配置文件中，持久生效
- 使用`go env -u`可以恢复环境变量到默认值
- 更多关于环境变量的信息，请参见：`go help environment`

### 2.3. go help - 获取帮助信息

用于获取特定命令或者话题的帮助信息：

```bash
## 获取go命令的总体帮助
go help

## 获取特定子命令的帮助
go help build # 获取build命令的帮助
go help run # 获取run命令的帮助
go help test # 获取test命令的帮助

## 获取特定话题的帮助
go help gopath # 获取gopath话题的帮助
go help packages # 获取packages话题的帮助
go help environment # 获取environment话题的帮助
```

### 2.4. go telemetry - 管理遥测数据

用于管理Go遥测数据和设置。

#### 2.4.1. 命令语法

```bash
go telemetry [off|local|on]
```

#### 2.4.2. 功能说明

Telemetry命令用于管理Go遥测数据和设置。遥测可以处于三种模式之一：off（关闭）、local（本地）或on（开启）。

#### 2.4.3. 遥测数据介绍

Go遥测系统收集有关Go工具使用情况的匿名数据，以帮助Go团队改进工具链和相关工具。收集的数据包括：

1. **使用统计**: 收集有关Go命令使用频率的计数器数据，例如build、run、test等命令的使用情况
2. **配置信息**: 包括Go版本、操作系统、架构等基本信息，但不包含任何个人身份信息
3. **功能使用情况**: 了解哪些Go功能和特性被广泛使用，帮助团队确定开发优先级
4. **性能数据**: 收集工具性能指标，帮助优化Go工具链的性能

遥测数据完全匿名，不会收集任何个人身份信息、项目代码或敏感数据。所有收集的数据仅用于改进Go工具链和相关工具。

#### 2.4.4. 模式说明

- **off模式**: 当遥测处于off模式时，既不收集本地计数器数据，也不上传数据。
- **local模式**: 当遥测处于local模式时，计数器数据会写入本地文件系统，但不会上传到远程服务器。
- **on模式**: 当遥测处于on模式时，遥测数据会写入本地文件系统，并定期发送到https://telemetry.go.dev/。上传的数据用于帮助改进Go工具链和相关工具，并将作为公共数据集的一部分发布。

#### 2.4.5. 使用示例

```
## 查看当前遥测模式
go telemetry

## 禁用遥测上传，但保持本地数据收集
go telemetry local

## 启用数据收集和上传
go telemetry on

## 禁用数据收集和上传
go telemetry off
```

#### 2.4.6. 环境变量

- `GOTELEMETRY`: 当前遥测模式的值，这是一个不可设置的go env变量
- `GOTELEMETRYDIR`: 遥测数据写入的本地文件系统目录，这是一个不可设置的go env变量

#### 2.4.7. 注意事项

- 遥测数据完全匿名，不包含任何个人身份信息或项目代码
- 数据收集遵循Google隐私政策：https://policies.google.com/privacy
- 用户可以随时通过命令控制遥测模式
- 更多详细信息，请参见：https://telemetry.go.dev/privacy
- 更多关于遥测的信息，请参见：https://go.dev/doc/telemetry

## 3. 项目构建命令

### 3.1. go build - 编译源代码

用于编译Go程序，生成可执行文件。

#### 3.1.1. 命令语法

```bash
go build [-o output] [build flags] [packages]
```

#### 3.1.2. 功能说明

Build命令编译由导入路径命名的包及其依赖项，但不安装结果。

当编译包时，build会忽略以'_test.go'结尾的文件。

当编译单个main包时，build将生成的可执行文件写入以包导入路径的最后一个非主版本组件命名的输出文件中。在Windows上会添加'.exe'后缀。
例如：'go build example/sam'会生成'sam'或'sam.exe'；'go build example.com/foo/v2'会生成'foo'或'foo.exe'，而不是'v2.exe'。

当从.go文件列表编译包时，可执行文件以第一个源文件命名。
例如：'go build ed.go rx.go'会生成'ed'或'ed.exe'。

当编译多个包或单个非main包时，build会编译包但丢弃生成的对象，仅作为检查包是否可以构建的手段。

#### 3.1.3. 主要参数

- `-o output`: 强制build将生成的可执行文件或对象写入指定的输出文件或目录，而不是默认行为

#### 3.1.4. 常用构建标志

- `-a`: 强制重新构建已经是最新的包
- `-n`: 打印命令但不运行它们
- `-v`: 打印编译的包名
- `-x`: 打印命令
- `-race`: 启用数据竞争检测（仅支持特定平台）
- `-cover`: 启用代码覆盖率检测
- `-work`: 打印临时工作目录的名称且在退出时不删除它
- `-asmflags '[pattern=]arg list'`: 传递给每个go tool asm调用的参数
- `-buildmode mode`: 使用的构建模式（参见'go help buildmode'）
- `-compiler name`: 使用的编译器名称（gccgo或gc）
- `-gcflags '[pattern=]arg list'`: 传递给每个go tool compile调用的参数
- `-ldflags '[pattern=]arg list'`: 传递给每个go tool link调用的参数
- `-tags tag,list`: 逗号分隔的附加构建标签列表
- `-trimpath`: 从生成的可执行文件中删除所有文件系统路径
- `-mod mode`: 模块下载模式（readonly、vendor或mod）

#### 3.1.5. 使用示例

```
## 编译当前目录下的main包
go build

## 编译指定包
go build package_name

## 编译并指定输出文件名
go build -o myapp main.go

## 编译时添加编译标签
go build -tags tag_name

## 编译时添加版本信息
go build -ldflags "-X main.version=1.0.0"

## 启用竞态检测
go build -race

## 启用代码覆盖率检测
go build -cover

## 打印编译过程中的命令
go build -x

## 强制重新构建所有包
go build -a

## 指定构建标签
go build -tags "linux,prod"

## 修剪路径信息
go build -trimpath

## 指定构建模式
go build -buildmode=shared
```

#### 3.1.6. 注意事项

- 构建标志由build、clean、get、install、list、run和test命令共享
- 更多关于指定包的信息，请参见'go help packages'
- 更多关于包和二进制文件安装位置的信息，请运行'go help gopath'
- 更多关于Go和C/C++之间调用的信息，请运行'go help c'

### 3.2. go install - 编译并安装包

编译并安装由导入路径命名的包。

#### 3.2.1. 命令语法

```bash
go install [build flags] [packages]
```

#### 3.2.2. 功能说明

Install命令编译并安装由导入路径命名的包。

可执行文件安装在GOBIN环境变量命名的目录中，默认为$GOPATH/bin或$HOME/go/bin（如果未设置GOPATH环境变量）。$GOROOT中的可执行文件安装在$GOROOT/bin或$GOTOOLDIR中，而不是$GOBIN。

#### 3.2.3. 版本后缀处理

如果参数具有版本后缀（如@latest或@v1.0.0），"go install"会在模块感知模式下构建包，忽略当前目录或任何父目录中的go.mod文件（如果存在）。这对于安装可执行文件而不影响主模块的依赖关系很有用。

使用版本后缀时，参数必须满足以下约束：

1. 参数必须是包路径或包模式（带"..."通配符），不能是标准包（如fmt）、元模式（std、cmd、all）或相对或绝对文件路径
2. 所有参数必须具有相同的版本后缀，不允许不同的查询，即使它们引用相同的版本
3. 所有参数必须引用同一模块中相同版本的包
4. 包路径参数必须引用main包，模式参数只会匹配main包
5. 没有模块被视为"主"模块
6. 任何模块中都不使用vendor目录

#### 3.2.4. 模块模式

如果参数没有版本后缀，"go install"可能在模块感知模式或GOPATH模式下运行，具体取决于GO111MODULE环境变量和go.mod文件的存在。如果启用了模块感知模式，"go install"会在主模块的上下文中运行。

禁用模块感知模式时，非主包安装在目录$GOPATH/pkg/$GOOS_$GOARCH中。启用模块感知模式时，非主包会被构建和缓存但不会被安装。

#### 3.2.5. 标准库处理

在Go 1.20之前，标准库安装到$GOROOT/pkg/$GOOS_$GOARCH。
从Go 1.20开始，标准库会被构建和缓存但不会被安装。
设置GODEBUG=installgoroot=all可以恢复使用$GOROOT/pkg/$GOOS_$GOARCH。

#### 3.2.6. 使用示例

```
## 安装当前目录下的包
go install

## 安装指定路径的包
go install path/to/package

## 安装并指定版本
go install github.com/user/package@latest

## 安装特定版本
go install github.com/user/package@v1.2.3

## 安装多个包
go install github.com/user/package1@latest github.com/user/package2@latest

## 安装包并启用竞态检测
go install -race github.com/user/package@latest

## 安装包并指定构建标签
go install -tags "linux,prod" github.com/user/package@latest
```

#### 3.2.7. 注意事项

- 更多关于构建标志的信息，请参见'go help build'
- 更多关于指定包的信息，请参见'go help packages'
- 使用版本后缀时，确保所有参数都引用同一模块中的包
- 安装的可执行文件默认放在$GOBIN目录中

### 3.3. go run - 编译并运行程序

编译并立即运行Go程序，不会生成可执行文件。

#### 3.3.1. 命令语法

```bash
go run [build flags] [-exec xprog] package [arguments...]
```

#### 3.3.2. 功能说明

Run命令编译并运行指定的主Go包。通常包被指定为单个目录中的.go源文件列表，但它也可以是导入路径、文件系统路径或匹配单个已知包的模式，如'go run .'或'go run my/cmd'。

#### 3.3.3. 版本后缀处理

如果包参数具有版本后缀（如@latest或@v1.0.0），"go run"会在模块感知模式下构建程序，忽略当前目录或任何父目录中的go.mod文件（如果存在）。这对于运行程序而不影响主模块的依赖关系很有用。

如果包参数没有版本后缀，"go run"可能在模块感知模式或GOPATH模式下运行，具体取决于GO111MODULE环境变量和go.mod文件的存在。如果启用了模块感知模式，"go run"会在主模块的上下文中运行。

#### 3.3.4. 执行方式

默认情况下，'go run'直接运行编译后的二进制文件：'a.out arguments...'。

如果给出了-exec标志，'go run'会使用xprog调用二进制文件：
'xprog a.out arguments...'。

如果未给出-exec标志，GOOS或GOARCH与系统默认值不同，并且在当前搜索路径上可以找到名为go_$GOOS_$GOARCH_exec的程序，'go run'会使用该程序调用二进制文件，例如'go_js_wasm_exec a.out arguments...'。这允许在模拟器或其他执行方法可用时执行交叉编译的程序。

#### 3.3.5. 调试信息

默认情况下，'go run'编译二进制文件时不生成调试器使用的信息，以减少构建时间。要在二进制文件中包含调试器信息，请使用'go build'。

#### 3.3.6. 退出状态

Run的退出状态不是编译后二进制文件的退出状态。

#### 3.3.7. 使用示例

```
## 运行程序
go run main.go

## 运行多个文件
go run main.go utils.go

## 运行时传递参数
go run main.go arg1 arg2

## 运行当前目录下的程序
go run .

## 运行指定路径的程序
go run ./cmd/myapp

## 运行特定版本的程序
go run github.com/user/package@latest

## 运行程序并启用竞态检测
go run -race main.go

## 运行程序并指定构建标签
go run -tags "linux,prod" main.go

## 运行程序并启用代码覆盖率检测
go run -cover main.go

## 使用指定程序执行编译后的二进制文件
go run -exec mydebugger main.go
```

#### 3.3.8. 注意事项

- 更多关于构建标志的信息，请参见'go help build'
- 更多关于指定包的信息，请参见'go help packages'
- 使用版本后缀时，确保遵循相关约束条件
- go run不会生成可执行文件，适用于快速测试和运行程序

### 3.4. go clean - 清理编译文件

清理编译生成的文件。

#### 3.4.1. 命令语法

```
go clean [-i] [-r] [-cache] [-testcache] [-modcache] [-fuzzcache] [build flags] [packages]
```

#### 3.4.2. 功能说明

Clean命令从包源目录中删除对象文件。go命令在临时目录中构建大多数对象，因此go clean主要关注其他工具或手动调用go build留下的对象文件。

如果给出了包参数或设置了-i或-r标志，clean会从与导入路径对应的每个源目录中删除以下文件：

- `_obj/`：旧对象目录，来自Makefiles
- `_test/`：旧测试目录，来自Makefiles
- `_testmain.go`：旧gotest文件，来自Makefiles
- `test.out`：旧测试日志，来自Makefiles
- `build.out`：旧测试日志，来自Makefiles
- `*.[568ao]`：对象文件，来自Makefiles
- `DIR(.exe)`：来自go build
- `DIR.test(.exe)`：来自go test -c
- `MAINFILE(.exe)`：来自go build MAINFILE.go
- `*.so`：来自SWIG

在列表中，DIR代表目录的最终路径元素，MAINFILE是目录中任何Go源文件的基名，这些文件在构建包时不包含在内。

#### 3.4.3. 参数说明

- `-i`：导致clean删除相应的已安装归档文件或二进制文件（即'go install'会创建的内容）
- `-r`：导致clean递归应用于导入路径命名的包的所有依赖项
- `-cache`：导致clean删除整个go构建缓存
- `-testcache`：导致clean使go构建缓存中的所有测试结果过期
- `-modcache`：导致clean删除整个模块下载缓存，包括版本化依赖项的解压源代码
- `-fuzzcache`：导致clean删除存储在Go构建缓存中用于模糊测试的文件
- `-n`：导致clean打印它将执行的删除命令，但不运行它们
- `-x`：导致clean在执行删除命令时打印它们

#### 3.4.4. 使用示例

```
## 清理当前目录下的编译文件
go clean

## 清理并删除安装的包
go clean -i

## 删除编译时生成的模块缓存
go clean -modcache

## 递归清理所有依赖项
go clean -r

## 删除整个go构建缓存
go clean -cache

## 使所有测试结果过期
go clean -testcache

## 删除模糊测试缓存文件
go clean -fuzzcache

## 打印将要执行的命令但不实际执行
go clean -n

## 打印执行的删除命令
go clean -x

## 清理指定包
go clean ./cmd/...
```

#### 3.4.5. 注意事项

- 模糊测试引擎缓存扩展代码覆盖率的文件，因此删除它们可能会降低模糊测试的效果，直到找到提供相同覆盖率的新输入
- 存储在testdata目录中的文件与模糊测试缓存文件不同，clean不会删除这些文件
- 更多关于构建标志的信息，请参见'go help build'
- 更多关于指定包的信息，请参见'go help packages'

## 4. 包管理命令

### 4.1. go mod - 模块管理

Go 1.11版本引入的模块管理工具。

#### 4.1.1. 命令语法

```
go mod <command> [arguments]
```

#### 4.1.2. 功能说明

Go mod提供对模块操作的访问。需要注意的是，对模块的支持内置于所有go命令中，而不仅仅是'go mod'。例如，日常的添加、删除、升级和降级依赖项应该使用'go get'。模块功能的概述请参见'go help modules'。

#### 4.1.3. 子命令

- `download`：将模块下载到本地缓存
- `edit`：从工具或脚本编辑go.mod
- `graph`：打印模块需求图
- `init`：在当前目录中初始化新模块
- `tidy`：添加缺失的模块并删除未使用的模块
- `vendor`：制作依赖项的供应商副本
- `verify`：验证依赖项是否具有预期内容
- `why`：解释为什么需要包或模块

#### 4.1.4. 使用示例

```
## 初始化新模块
go mod init module_name

## 下载依赖
go mod download

## 查看依赖列表
go mod graph

## 验证依赖
go mod verify

## 整理依赖
go mod tidy

## 查看为什么需要某个依赖
go mod why package_name

## 编辑go.mod文件
go mod edit -require=example.com/module@v1.0.0

## 生成vendor目录
go mod vendor

## 查看模块信息
go mod download -json
```

#### 4.1.5. 注意事项

- 更多关于模块的信息，请参见：https://golang.org/ref/mod

### 4.2. go get - 获取包

下载并安装指定的包，更新go.mod以要求这些版本，并将源代码下载到模块缓存中。

#### 4.2.1. 命令语法

```
go get [-t] [-u] [-tool] [build flags] [packages]
```

#### 4.2.2. 功能说明

Get将其命令行参数解析为特定模块版本的包，更新go.mod以要求这些版本，并将源代码下载到模块缓存中。

在早期版本的Go中，'go get'用于构建和安装包。现在，'go get'专门用于调整go.mod中的依赖项。'go install'可用于构建和安装命令。当指定了版本时，'go install'在模块感知模式下运行，并忽略当前目录中的go.mod文件。

#### 4.2.3. 参数说明

- `-t`：指示get考虑构建命令行上指定的包的测试所需的模块
- `-u`：指示get更新提供命令行上命名的包的依赖项的模块，以在可用时使用更新的次要或补丁版本
- `-u=patch`：也指示get更新依赖项，但将默认值更改为选择补丁版本
- `-tool`：指示go为每个列出的包在go.mod中添加匹配的工具行。如果-tool与@none一起使用，则该行将被删除
- `-x`：在执行时打印命令。这对于调试版本控制命令（当模块直接从存储库下载时）很有用

#### 4.2.4. 使用示例

```
## 添加依赖或将其升级到最新版本
go get example.com/pkg

## 升级或降级包到特定版本
go get example.com/pkg@v1.2.3

## 移除对模块的依赖并降级需要它的模块
go get example.com/mod@none

## 升级最低要求的Go版本到最新的发布版本
go get go@latest

## 升级Go工具链到当前Go工具链的最新补丁版本
go get toolchain@patch

## 升级依赖项
go get -u example.com/pkg

## 升级依赖项到补丁版本
go get -u=patch example.com/pkg

## 考虑测试依赖项
go get -t example.com/pkg

## 构建和安装特定版本的命令
go install example.com/pkg@v1.2.3
go install example.com/pkg@latest
```

#### 4.2.5. 注意事项

- 更多关于模块的信息，请参见：https://golang.org/ref/mod
- 更多关于使用'go get'更新最低Go版本和建议的Go工具链的信息，请参见：https://go.dev/doc/toolchain

### 4.3. go list - 列出包或模块

列出命名的包，每行一个。

#### 4.3.1. 命令语法

```
go list [-f format] [-json] [-m] [list flags] [build flags] [packages]
```

#### 4.3.2. 功能说明

List列出命名的包，每行一个。最常用的标志是-f和-json，它们控制为每个包打印的输出格式。其他列表标志控制更具体的细节。

#### 4.3.3. 参数说明

- `-f`：为列表指定替代格式，使用包模板的语法。默认输出等效于-f '.ImportPath'
- `-json`：使包数据以JSON格式打印，而不是使用模板格式
- `-m`：导致list列出模块而不是包
- `-deps`：导致list不仅迭代命名的包，还迭代它们的所有依赖项
- `-e`：更改对错误包的处理
- `-export`：导致list将Export字段设置为包含给定包的最新导出信息的文件名
- `-find`：导致list识别命名的包但不解析它们的依赖项
- `-test`：导致list不仅报告命名的包，还报告它们的测试二进制文件
- `-compiled`：导致list将CompiledGoFiles设置为呈现给编译器的Go源文件
- `-versions`：导致list将Module的Versions字段设置为该模块的所有已知版本的列表
- `-retracted`：导致list报告有关已撤回模块版本的信息
- `-u`：添加有关可用升级的信息

#### 4.3.4. 使用示例

```
## 列出当前目录下的所有包
go list ./...

## 列出所有依赖
go list -m all

## 以JSON格式显示模块信息
go list -m -json all

## 显示包的导入路径
go list -f .ImportPath encoding/json

## 显示包的目录
go list -f .Dir encoding/json

## 显示包的依赖项
go list -f .Deps encoding/json

## 显示包的导入
go list -f .Imports encoding/json

## 以JSON格式显示包信息
go list -json encoding/json

## 显示模块路径和版本信息
go list -m -f .String all

## 显示可用的模块升级
go list -m -u all

## 显示模块的所有版本
go list -m -versions example.com/module

## 显示已撤回的模块版本
go list -m -retracted all
```

#### 4.3.5. 注意事项

- 更多关于指定包的信息，请参见'go help packages'
- 更多关于模块的信息，请参见：https://golang.org/ref/mod

## 5. 代码生成和维护命令

### 5.1. go fmt - 格式化代码

格式化Go源代码文件。

#### 5.1.1. 命令语法

```bash
go fmt [-n] [-x] [packages]
```

#### 5.1.2. 功能说明

Fmt在导入路径命名的包上运行'gofmt -l -w'命令。它会打印被修改的文件名。

#### 5.1.3. 参数说明

- `-n`：打印将要执行的命令
- `-x`：在执行时打印命令
- `-mod`：设置要使用的模块下载模式，值可以是readonly或vendor

#### 5.1.4. 使用示例

```bash
## 格式化当前目录下的所有Go文件
go fmt ./...

## 格式化指定文件
go fmt main.go

## 显示将要格式化的文件但不执行格式化
go fmt -n ./...

## 格式化时打印执行的命令
go fmt -x ./...

## 格式化指定包
go fmt example.com/package
```

#### 5.1.5. 注意事项

- 要使用特定选项运行gofmt，请直接运行gofmt本身
- 更多关于gofmt的信息，请参见'go doc cmd/gofmt'
- 更多关于指定包的信息，请参见'go help packages'

### 5.2. go vet - 检查代码问题

检查Go源代码中可能存在的错误。

#### 5.2.1. 命令语法

```bash
go vet [build flags] [-vettool prog] [vet flags] [packages]
```

#### 5.2.2. 功能说明

Vet在导入路径命名的包上运行Go vet命令。

#### 5.2.3. 参数说明

- `-vettool=prog`：选择具有替代或附加检查的不同分析工具
- 构建标志：支持的构建标志包括控制包解析和执行的标志，如-C、-n、-x、-v、-tags和-toolexec

#### 5.2.4. 使用示例

```bash
## 检查当前目录下的代码
go vet

## 检查指定包
go vet package_name

## 检查所有包
go vet ./...

## 使用特定的检查工具
go vet -vettool=$(which shadow)

## 显示详细信息
go vet -v ./...

## 打印将要执行的命令
go vet -n ./...

## 打印执行的命令
go vet -x ./...
```

#### 5.2.5. 注意事项

- 更多关于vet及其标志的信息，请参见'go doc cmd/vet'
- 更多关于指定包的信息，请参见'go help packages'
- 有关检查器列表及其标志，请参见'go tool vet help'
- 有关特定检查器（如'printf'）的详细信息，请参见'go tool vet help printf'
- 更多关于构建标志的信息，请参见'go help build'

### 5.3. go generate - 通过注释执行命令

通过源码中的注释执行命令生成代码。

#### 5.3.1. 命令语法

```
go generate [-run regexp] [-n] [-v] [-x] [build flags] [file.go... | packages]
```

#### 5.3.2. 功能说明

Generate运行现有文件中指令描述的命令。这些命令可以运行任何进程，但目的是创建或更新Go源文件。

Go generate永远不会被go build、go test等自动运行，必须显式运行。

#### 5.3.3. 指令格式

Go generate扫描文件中的指令，这些指令是以下形式的行：

```
//go:generate command argument...
```

注意：没有前导空格且"//go"中没有空格，其中command是要运行的生成器，对应于可以在本地运行的可执行文件。

#### 5.3.4. 参数说明

- `-run=""`：如果非空，指定一个正则表达式来选择完整原始源文本（不包括任何尾随空格和最终换行符）匹配该表达式的指令
- `-skip=""`：如果非空，指定一个正则表达式来抑制完整原始源文本（不包括任何尾随空格和最终换行符）匹配该表达式的指令
- `-n`：打印将要执行的命令
- `-v`：打印正在处理的包和文件名
- `-x`：在执行时打印命令

#### 5.3.5. 环境变量

Go generate在运行生成器时设置以下变量：

- `$GOARCH`：执行架构（arm、amd604等）
- `$GOOS`：执行操作系统（linux、windows等）
- `$GOFILE`：文件的基本名称
- `$GOLINE`：源文件中指令的行号
- `$GOPACKAGE`：包含指令的文件的包名
- `$GOROOT`：调用生成器的'go'命令的GOROOT目录，包含Go工具链和标准库
- `$DOLLAR`：美元符号
- `$PATH`：父进程的$PATH，$GOROOT/bin放在开头

#### 5.3.6. 命令别名

可以使用以下形式的指令为源文件的其余部分指定命令别名：

```
//go:generate -command xxx args...
```

这指定字符串xxx代表由参数标识的命令。这可以用于创建别名或处理多字生成器。

例如：

```
//go:generate -command foo go tool foo
```

指定命令"foo"代表生成器"go tool foo"。

#### 5.3.7. 使用示例

```
## 执行源码中的generate指令
go generate

## 执行指定包中的generate指令
go generate package_name

## 显示将要执行的命令但不执行
go generate -n

## 显示执行的命令
go generate -x

## 显示处理的包和文件
go generate -v

## 执行匹配正则表达式的指令
go generate -run="stringer"

## 跳过匹配正则表达式的指令
go generate -skip="stringer"

## 处理指定文件
go generate main.go
```

#### 5.3.8. 注意事项

- 生成的源代码应该有一行匹配正则表达式`^// Code generated .* DO NOT EDIT\.$`，以表明代码是生成的
- 该行必须出现在文件中第一个非注释、非空白文本之前
- 如果任何生成器返回错误退出状态，"go generate"将跳过该包的所有进一步处理
- 生成器在包的源目录中运行
- 更多关于构建标志的信息，请参见'go help build'
- 更多关于指定包的信息，请参见'go help packages'

## 6. 测试命令

### 6.1. go test - 运行测试

运行Go测试文件。

#### 6.1.1. 命令语法

```bash
go test [build/test flags] [packages] [build/test flags & test binary flags]
```

#### 6.1.2. 功能说明

'Go test'自动化测试由导入路径命名的包。它以以下格式打印测试结果摘要：

```
ok   archive/tar   0.011s
FAIL archive/zip   0.022s
ok   compress/gzip 0.033s
...
```

'Go test'会重新编译每个包以及任何文件名匹配文件模式"*_test.go"的文件。这些附加文件可以包含测试函数、基准函数、模糊测试和示例函数。

#### 6.1.3. 测试模式

Go test在两种不同的模式下运行：

1. **本地目录模式**：当go test在没有包参数的情况下调用时发生（例如，'go test'或'go test -v'）。在此模式下，go test编译当前目录中的包源和测试，然后运行生成的测试二进制文件。在此模式下，缓存（如下所述）被禁用。

2. **包列表模式**：当go test使用显式包参数调用时发生（例如'go test math'，'go test ./...'，甚至'go test .'）。在此模式下，go test编译并测试命令行上列出的每个包。

#### 6.1.4. 测试缓存

在包列表模式下，go test会缓存成功的包测试结果以避免不必要的重复运行测试。当测试结果可以从缓存中恢复时，go test将重新显示以前的输出而不是再次运行测试二进制文件。

可缓存的测试标志包括：-benchtime, -coverprofile, -cpu, -failfast, -fullpath, -list, -outputdir, -parallel, -run, -short, -skip, -timeout 和 -v。

要禁用测试缓存，可以使用除可缓存标志之外的任何测试标志或参数。显式禁用测试缓存的惯用方法是使用-count=1。

#### 6.1.5. 代码检查

作为构建测试二进制文件的一部分，go test在包及其测试源文件上运行go vet以识别重大问题。如果go vet发现问题，go test会报告这些问题并且不运行测试二进制文件。

#### 6.1.6. 参数说明

- `-args`：将命令行的其余部分（-args之后的所有内容）传递给测试二进制文件，不进行解释和更改
- `-c`：将测试二进制文件编译到当前目录中的pkg.test，但不运行它
- `-exec xprog`：使用xprog运行测试二进制文件
- `-json`：将测试输出转换为适合自动化处理的JSON格式
- `-o file`：将测试二进制文件编译到指定文件

#### 6.1.7. 使用示例

```
## 运行当前目录下的所有测试
go test

## 运行指定包的测试
go test package_name

## 运行测试并显示详细信息
go test -v

## 运行基准测试
go test -bench=.

## 运行覆盖率测试
go test -cover

## 生成覆盖率报告
go test -coverprofile=coverage.out
go tool cover -html=coverage.out

## 禁用测试缓存
go test -count=1

## 运行特定测试函数
go test -run TestFunctionName

## 设置测试超时时间
go test -timeout 30s

## 并行运行测试
go test -parallel 4

## 运行短时间测试
go test -short

## 跳过特定测试
go test -skip TestSkipped

## 显示详细输出
go test -v

## 仅列出测试函数
go test -list .

## 生成测试二进制文件但不运行
go test -c

## 指定输出文件名
go test -o mytest

## 传递参数给测试二进制文件
go test -args -testArg=value

## 以JSON格式输出
go test -json

## 禁用vet检查
go test -vet=off

## 运行所有vet检查
go test -vet=all
```

#### 6.1.8. 注意事项

- 测试文件名以"_"（包括"_test.go"）或"."开头的文件将被忽略
- go工具将忽略名为"testdata"的目录，使其可用于保存测试所需的辅助数据
- 所有测试输出和摘要行都打印到go命令的标准输出
- 更多关于构建标志的信息，请参见'go help build'
- 更多关于指定包的信息，请参见'go help packages'
- 更多关于测试函数的信息，请参见'go help testfunc'
- 更多关于测试标志的信息，请参见'go help testflag'

## 7. 文档和工具命令

### 7.1. go doc - 显示文档

显示Go程序实体的文档：

```bash
## 显示当前目录的文档
go doc

## 显示指定包的文档
go doc package_name

## 显示函数或类型的文档
go doc package_name.function_name
```

### 7.2. go tool - 调用底层工具

调用Go工具链中的底层工具：

```bash
## 查看可用的工具
go tool

## 使用pprof进行性能分析
go tool pprof cpu.prof

## 使用trace分析程序执行
go tool trace trace.out

## 查看汇编代码
go tool compile -S main.go
```

## 8. 环境和配置命令

### 8.1. go bug - 报告bug

打开浏览器报告Go语言的bug：

```bash
go bug
```

### 8.2. go fix - 更新旧代码

更新旧版本Go代码以适应新版本：

```bash
go fix
```

## 9. 实用示例

### 9.1. 创建新项目

```bash

## 创建项目目录
mkdir myproject
cd myproject

## 初始化Go模块
go mod init myproject

## 创建main.go文件
cat > main.go << EOF
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
EOF

## 运行程序
go run main.go
```

### 9.2. 添加外部依赖

```bash
## 添加Gin框架
go get -u github.com/gin-gonic/gin

## 整理依赖
go mod tidy
```

### 9.3. 构建和部署

```bash

## 构建项目
go build -o myapp main.go

## 跨平台编译
GOOS=linux GOARCH=amd64 go build -o myapp main.go
```