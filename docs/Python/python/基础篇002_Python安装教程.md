## 1. 安装前检查
在很多系统上，Python都是预装的，因此在安装前，可以先执行`python`（或者`py`）命令看看python是否已经预装了。

如果已经安装了，可以看到包含版本号的响应，例如：
```shell
Python 3.9.6 (tags/v3.9.6:db3ff76, Jun 28 2021, 15:26:21) [MSC v.1929 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
```

如果没有看到这些信息，则需要安装。
> 如果版本号是Python 2.x.x，则说明预装的是已经不受支持的Python 2，它已经不适合开发。你可以尝试运行`python3`命令，看看是否也已经安装Python 3.x.x。如果没有，则说明需要安装最新版本的Python。

## 2. 安装

### 2.1 Windows系统安装

步骤1：下载安装包
1. 访问 [Python官方下载页面](https://www.python.org/downloads/)
2. 点击 **Download Python 3.x.x**（当前最新稳定版）

步骤2：运行安装程序
1. 双击下载的 `.exe` 文件
2. **关键操作**：
   - ✅ 勾选 `Add Python to PATH`（非常重要！）
   - 选择 `Install Now`（默认安装所有组件）


> ⚠️ **常见错误**：忘记勾选 `Add Python to PATH` 会导致命令行无法识别 `python` 命令

步骤3：验证安装
```shell
## 打开命令提示符（cmd）
python --version
## 应显示：Python 3.x.x

## 测试运行
python -c "print('Hello Python!')"
```

### 2.2 macOS系统安装

#### 2.2.1. 方法一：使用官方安装包（推荐）
1. 从 [Python官网](https://www.python.org/downloads/macos/) 下载 `.pkg` 文件
2. 双击安装，按提示完成
3. 验证安装：
```shell
python3 --version
```

#### 2.2.2. 方法二：使用Homebrew（适合开发者）
```shell
## 1. 安装Homebrew（如未安装）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

## 2. 安装Python
brew install python

## 3. 验证
python3 --version
```

### 2.3. Linux系统安装

#### 2.3.1. Ubuntu/Debian系统
```shell
## 1. 更新软件包
sudo apt update

## 2. 安装Python3和pip
sudo apt install python3 python3-pip

## 3. 验证
python3 --version
pip3 --version
```

#### 2.3.2. CentOS/RHEL系统
```shell
## 1. 安装EPEL仓库
sudo yum install epel-release

## 2. 安装Python
sudo yum install python3 python3-pip

## 3. 验证
python3 --version
```

## 3. 推荐IDE
| 工具 | 用途 | 安装方式 |
|------|------|----------|
| VS Code | 轻量级编辑器 | [官网下载](https://code.visualstudio.com/) |
| PyCharm Community | 专业Python IDE | [官网下载](https://www.jetbrains.com/pycharm/) |
| Jupyter Notebook | 数据分析环境 | `pip install notebook` |