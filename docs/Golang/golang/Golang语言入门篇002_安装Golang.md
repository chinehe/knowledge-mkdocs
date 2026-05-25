Go语言（又称Golang）是由Google开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言。本文将详细介绍在不同操作系统上安装Go语言的方法，以及如何管理多个Go版本。


## 1. 官方安装教程

* [Download and install - The Go Programming Language](https://go.dev/doc/install)

## 2. Windows系统安装Go

### 2.1. 下载安装包

1. 访问[Go官方下载页面](https://go.dev/dl/)
2. 找到Windows对应的版本，下载`.msi`安装包（如：`go1.21.0.windows-amd64.msi`）

### 2.2. 安装步骤

1. 双击下载的`.msi`文件启动安装程序
2. 按照安装向导的指示进行操作，默认会安装到`C:\Go\`目录
3. 安装程序会自动将`C:\Go\bin`添加到系统的PATH环境变量中

### 2.3. 验证安装

打开命令提示符（cmd）或PowerShell，输入以下命令：

```bash
go version
```

如果显示版本信息，说明安装成功。

## 3. Linux系统安装Go

### 3.1. 下载安装包

访问[Go官方下载页面](https://go.dev/dl/)，下载Linux对应的`.tar.gz`压缩包，或者使用wget命令：

```bash
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
```

### 3.2. 安装步骤

1. 解压下载的压缩包到`/usr/local`目录：

```bash
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
```

2. 配置环境变量，将Go的bin目录添加到PATH中：

编辑`~/.bashrc`或`~/.profile`文件：

```bash
export PATH=$PATH:/usr/local/go/bin
```

3. 使配置生效：

```bash
source ~/.bashrc
```

或者重新登录系统。

### 3.3. 验证安装

在终端中执行：

```bash
go version
```

## 4. macOS系统安装Go

### 4.1. 方法一：使用安装包

1. 访问[Go官方下载页面](https://go.dev/dl/)
2. 下载macOS对应的`.pkg`安装包
3. 双击安装包，按照安装向导完成安装

### 4.2. 方法二：使用Homebrew

如果你已经安装了Homebrew，可以使用以下命令安装：

```bash
brew install go
```

### 4.3. 验证安装

在终端中执行：

```bash
go version
```

## 5. 配置Go环境变量

安装Go后，还需要配置一些重要的环境变量：

- `GOROOT`：Go的安装路径，默认情况下不需要设置
- `GOPATH`：工作区路径（Go 1.11版本后，使用Go Modules时可不设置）
- `GOBIN`：可执行文件存放路径
- `PATH`：需要包含Go的可执行文件路径

### 5.1. 查看当前Go环境变量

```bash
go env
```

### 5.2. 设置Go环境变量

```bash
## 设置GOPATH
go env -w GOPATH=$HOME/go

## 设置代理（国内用户推荐）
go env -w GOPROXY=https://goproxy.cn,direct
```

## 6. Go多版本管理

在实际开发中，可能需要在不同项目中使用不同版本的Go。以下是几种常用的Go多版本管理工具：

### 6.1. 使用gvm（Go Version Manager）

gvm是Linux和macOS平台下常用的Go版本管理工具。

#### 6.1.1. 安装gvm

```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

#### 6.1.2. 使用gvm

```bash
## 查看可安装的Go版本
gvm listall

## 安装指定版本的Go
gvm install go1.21.0

## 切换Go版本
gvm use go1.21.0

## 设置默认Go版本
gvm use go1.21.0 --default
```

### 6.2. 使用goenv

goenv是一个跨平台的Go版本管理工具，灵感来源于rbenv。

#### 6.2.1. 安装goenv

```bash
git clone https://github.com/syndbg/goenv.git ~/.goenv
```

添加到PATH：

```bash
echo 'export GOENV_ROOT="$HOME/.goenv"' >> ~/.bashrc
echo 'export PATH="$GOENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(goenv init -)"' >> ~/.bashrc
```

#### 6.2.2. 使用goenv

```bash
## 查看可用的Go版本
goenv install -l

## 安装指定版本
goenv install 1.21.0

## 设置全局版本
goenv global 1.21.0

## 设置项目版本
goenv local 1.20.0
```

### 6.3. 使用第三方包管理器

#### 6.3.1. macOS (使用Homebrew)

```bash
## 安装不同版本的Go
brew install go@1.20
brew install go@1.19

## 切换版本
brew unlink go
brew link go@1.20
```

## 7. Docker中使用Go

对于不想在本地安装Go或者需要隔离环境的开发者，可以使用Docker：

```bash
## 拉取Go官方镜像
docker pull golang:1.21

## 运行Go容器
docker run -it --rm golang:1.21 go version
```

## 8. 常见问题及解决方案

### 8.1. 网络问题

国内用户可能遇到下载包慢的问题，可以配置Go代理：

```bash
go env -w GOPROXY=https://goproxy.cn,direct
```
