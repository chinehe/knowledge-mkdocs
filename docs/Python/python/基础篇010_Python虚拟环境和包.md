## 1. 为什么需要虚拟环境

Python 的第三方包默认安装在解释器的全局目录里。只要一台机器上有两个项目，就会遇到下面的问题：

* 项目 A 需要 `sqlalchemy 1.4`，项目 B 需要 `sqlalchemy 2.0`，全局只能装一个版本。
* 升级一个包，可能悄悄把另一个项目弄坏。
* 新同事拿到代码，不知道要装哪些包、装什么版本。

**虚拟环境（Virtual Environment）** 就是为每个项目单独准备一份 Python 解释器和包目录，项目之间互不影响。

### 1.1. 传统做法：venv + pip

Python 3.3 起自带 `venv` 模块，配合 `pip` 就能完成最基础的隔离：

```shell
# 在项目目录下创建虚拟环境（生成 .venv 目录）
python3 -m venv .venv

# 激活虚拟环境（macOS / Linux）
source .venv/bin/activate

# 安装依赖
pip install fastapi sqlalchemy

# 把当前环境的依赖导出到文件，供别人安装
pip freeze > requirements.txt

# 退出虚拟环境
deactivate
```

这套做法能用，但在团队协作中有几个明显的短板：

| 问题 | 说明 |
|------|------|
| Python 版本不受管理 | `venv` 只能使用机器上已有的 Python，团队成员的版本可能不一致，需要另外装 pyenv |
| 没有锁文件 | `requirements.txt` 通常只写直接依赖，间接依赖的版本每次安装都可能不同 |
| 依赖分组不方便 | 运行依赖和开发依赖（pytest、mypy 等）要拆成多个 requirements 文件 |
| 需要手动激活 | 忘记 `activate` 就会装到全局环境里 |
| 速度慢 | pip 解析和安装大型依赖树比较慢 |

因此社区陆续出现了 pipenv、poetry、pdm 等工具。目前最受关注的是 **uv**。

## 2. uv 简介

[uv](https://docs.astral.sh/uv/) 是 Astral 公司（ruff 的作者）用 Rust 编写的 Python 包与项目管理工具，一个工具就替代了下面这些：

| 原来的工具 | uv 中的对应功能 |
|-----------|----------------|
| pyenv | `uv python install`，按 `.python-version` 自动下载 Python |
| venv / virtualenv | 自动创建并管理项目的 `.venv` |
| pip / pip-tools | `uv add`、`uv sync`、`uv pip` |
| poetry / pdm | `pyproject.toml` + `uv.lock` 的项目管理 |
| pipx | `uv tool install` |

它的特点：

* **快**：安装速度通常是 pip 的 10 到 100 倍。
* **有锁文件**：`uv.lock` 记录完整依赖树的精确版本，所有人、所有环境装出来的东西一模一样。
* **不需要激活**：用 `uv run <命令>` 就能在项目环境中执行，不存在"忘记激活"的问题。
* **遵循标准**：项目信息写在标准的 `pyproject.toml` 里，不是私有格式。

安装 uv：

```shell
curl -LsSf https://astral.sh/uv/install.sh | sh
uv --version
```

## 3. 用 uv 初始化项目

### 3.1. uv init

```shell
mkdir my-service && cd my-service
uv init --app --package --python 3.12 .
```

参数说明：

| 参数 | 含义 |
|------|------|
| `--app` | 这是一个应用（Web 服务、命令行工具），而不是要发布到 PyPI 的库 |
| `--package` | 生成可安装的包结构，代码放在 `src/<包名>/` 下（src 布局） |
| `--python 3.12` | 指定 Python 版本，写入 `.python-version` |

生成的文件：

```
my-service/
├── .python-version         # 3.12，uv 会按这个版本自动下载 Python
├── pyproject.toml          # 项目信息与依赖
├── README.md
└── src/
    └── my_service/
        └── __init__.py
```

> 项目名里的 `-` 在包名里会变成 `_`：项目叫 `my-service`，导入时写 `import my_service`。

生成的 `pyproject.toml` 大致如下：

```toml
[project]
name = "my-service"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []

[project.scripts]
my-service = "my_service:main"

[build-system]
requires = ["uv_build>=0.11.1,<0.12.0"]
build-backend = "uv_build"
```

`[build-system]` 声明用 `uv_build` 把项目构建成可安装的包，`--package` 模式下 uv 会把项目本身也安装进虚拟环境。

### 3.2. Python 版本自动管理

`.python-version` 里写的是 `3.12`。执行任何 uv 命令时，如果本机没有这个版本，uv 会自动下载一份独立的 Python，不需要事先安装，也不会影响系统自带的 Python。

```shell
uv python list        # 查看可用和已安装的 Python 版本
uv python install 3.13
```

## 4. 管理依赖

### 4.1. uv add：添加依赖

```shell
# 运行依赖：写入 [project] dependencies
uv add "fastapi[standard]" pydantic-settings "sqlalchemy[asyncio]" asyncpg alembic

# 开发依赖：写入 [dependency-groups] dev，不会打进生产镜像
uv add --dev ruff mypy pytest pytest-asyncio pytest-cov pre-commit

# 删除依赖
uv remove asyncpg
```

执行后 `pyproject.toml` 会变成：

```toml
[project]
dependencies = [
    "alembic>=1.20.1",
    "asyncpg>=0.32.0",
    "fastapi[standard]>=0.163.1",
    "pydantic-settings>=2.16.3",
    "sqlalchemy[asyncio]>=2.1.8",
]

[dependency-groups]
dev = [
    "mypy>=2.4.1",
    "pre-commit>=4.7.0",
    "pytest>=9.2.0",
    "pytest-asyncio>=1.5.0",
    "pytest-cov>=7.1.0",
    "ruff>=0.17.5",
]
```

* `pyproject.toml` 里写的是**版本范围**（`>=`），表达"至少需要哪个版本"。
* 真正安装的**精确版本**记录在 `uv.lock` 里。
* `[dependency-groups]` 是 PEP 735 定义的标准写法，`dev` 组默认会被安装，部署时用 `--no-dev` 排除。

### 4.2. uv.lock 与 uv sync

`uv.lock` 记录了整棵依赖树（包括间接依赖）每个包的精确版本和哈希值，**必须提交到仓库**。

```shell
# 按 uv.lock 创建或更新 .venv，让环境和锁文件完全一致
uv sync

# 只装运行依赖（部署时用）
uv sync --no-dev

# 要求锁文件必须是最新的：pyproject.toml 和 uv.lock 不一致时直接报错，不会偷偷更新锁文件
uv sync --locked
```

`--locked` 常用在 CI 和 Dockerfile 里：如果有人改了 `pyproject.toml` 却忘了提交更新后的 `uv.lock`，构建会立即失败，避免"线上装的版本和本地测试的不一样"。

新同事克隆项目后只需要一条命令：

```shell
git clone <仓库地址> && cd my-service
uv sync
```

### 4.3. uv run：在项目环境中执行命令

```shell
uv run python -c "import fastapi; print(fastapi.__version__)"
uv run pytest
uv run ruff check
uv run alembic upgrade head
```

`uv run` 会先确认环境和锁文件同步（必要时自动 sync），再用 `.venv` 里的解释器执行命令。因此**不需要 `source .venv/bin/activate`**，团队文档里所有命令都统一写成 `uv run xxx` 即可。

## 5. 项目本身的安装方式

### 5.1. 可编辑安装

`--package` 模式下，`uv sync` 会把项目本身以**可编辑模式（editable）**安装进 `.venv`：虚拟环境里放的不是代码的拷贝，而是一个指向 `src/` 目录的链接。

| 改动 | 是否需要重新 `uv sync` |
|------|----------------------|
| 修改 `src/` 下已有的 `.py` 文件 | 不需要，立即生效 |
| 在 `src/my_service/` 下新增模块 | 不需要，立即生效 |
| 修改 `pyproject.toml`（依赖、`[project.scripts]` 等） | **需要** |

> 部署时则相反：Docker 镜像里用 `uv sync --no-editable`，把代码直接装进虚拟环境，运行阶段就不再需要 `src/` 目录。

### 5.2. [project.scripts]：定义命令行入口

```toml
[project.scripts]
my-service = "my_service.cli:main"
```

含义是：安装后生成一个叫 `my-service` 的命令，执行它就相当于调用 `my_service.cli` 模块里的 `main()` 函数。

```shell
uv sync                                  # 修改 scripts 后需要重新 sync
uv run my-service create-user Alice
```

常见错误：

| 现象 | 原因 |
|------|------|
| `ImportError: cannot import name 'main'` | 入口指向的函数被删除或改名了（例如删掉了 `uv init` 生成的 Hello 函数，但没改 scripts） |
| `ModuleNotFoundError: No module named 'my_service.cli'` | `cli.py` 没放在包目录 `src/my_service/` 下，而是放到了项目根目录 |
| 命令不存在 | 修改 `[project.scripts]` 后没有重新 `uv sync` |

### 5.3. 版本号只维护一份

版本号写在 `pyproject.toml` 的 `version` 里。代码中需要版本号时（例如接口文档、启动日志），不要再写一遍，而是从已安装包的元数据里读取：

```python
# src/my_service/__init__.py
from importlib.metadata import version

__version__ = version("my-service")
```

`importlib.metadata` 读取的是"已安装的包"的信息，所以前提是项目已经通过 `uv sync` 安装。这样发布新版本只需要改 `pyproject.toml` 一处。

## 6. 常用命令速查

| 命令 | 作用 |
|------|------|
| `uv init --app --package` | 初始化 src 布局的应用项目 |
| `uv add <包>` / `uv add --dev <包>` | 添加运行依赖 / 开发依赖 |
| `uv remove <包>` | 删除依赖 |
| `uv sync` | 按锁文件同步环境 |
| `uv sync --locked --no-dev` | CI / 部署：校验锁文件，只装运行依赖 |
| `uv lock --upgrade-package <包>` | 升级某个包在锁文件中的版本 |
| `uv run <命令>` | 在项目环境中执行命令 |
| `uv tree` | 查看依赖树 |
| `uv python install <版本>` | 安装指定版本的 Python |

## 7. 小结

* 虚拟环境解决了"多个项目依赖冲突"的问题；`venv + pip` 是最基础的方案。
* uv 用一个工具统一了 Python 版本、虚拟环境、依赖和锁文件的管理，速度快且遵循 `pyproject.toml` 标准。
* `pyproject.toml` 写版本范围，`uv.lock` 记录精确版本，两者都要提交；CI 和镜像构建用 `--locked` 保证一致。
* 命令统一用 `uv run` 执行，不需要手动激活环境。
* 项目本身以可编辑方式安装：改代码立即生效，改 `pyproject.toml` 需要重新 `uv sync`。
