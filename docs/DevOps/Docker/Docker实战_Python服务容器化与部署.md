《Docker入门指南》讲了镜像、容器、Compose 这些基本概念。本文以一个 FastAPI 示例项目 my-service 为例，讲解如何把一个 Python 后端服务做成可以上生产的镜像，并用 Docker Compose 在本地以生产形态跑起来。

文章主要内容包括：

1. 项目与目标：项目涉及哪些文件，`.dockerignore` 的作用。
2. Dockerfile 逐行解析：uv 官方推荐的多阶段构建写法。
3. 运行时细节：非 root 运行、优雅停机、日志输出、健康检查，以及常见的坑。
4. Compose 编排逐段解析：健康检查、启动顺序、profiles、必填变量、YAML 锚点。
5. 运行与验证：启动、检查、执行管理命令、停止。
6. 数据库迁移作为一次性任务。
7. 部署到 Kubernetes 的要点。
8. 镜像体积与优化方向。



关于本文：

* 文中的 Dockerfile、docker-compose.yml 是一套完整可用的配置，示例项目名为 `my-service`，Python 包名为 `my_service`。
* 文中给出的命令输出、构建耗时和镜像体积，是按这套配置构建、运行后得到的典型结果，具体数值会随依赖数量、网络和镜像版本变化。

## 1. 项目与目标

### 1.1. 项目背景

示例项目 my-service 是一个 FastAPI 后端服务，使用 uv 管理依赖，PostgreSQL 存储数据，Alembic 做数据库迁移。

和容器化相关的文件如下：

| 文件 | 作用 |
| --- | --- |
| `Dockerfile` | 构建应用镜像（多阶段） |
| `.dockerignore` | 排除不进入构建上下文的文件 |
| `docker-compose.yml` | 编排数据库、迁移任务、应用三个服务 |
| `pyproject.toml` / `uv.lock` | 依赖声明与锁定版本 |
| `alembic.ini` / `migrations/` | 数据库迁移配置与脚本 |



容器化要达到的目标：

* **镜像小而干净：** 运行镜像里只有运行所需的文件，没有构建工具、源码目录、测试代码。
* **构建快：** 只改业务代码时，不重新安装依赖。
* **安全：** 不以 root 运行，敏感文件不进镜像。
* **可运维：** 能被健康检查探测，能优雅停机，日志实时输出。
* **启动顺序可靠：** 数据库就绪 → 迁移成功 → 启动应用。

### 1.2. .dockerignore

`docker build .` 时，`.` 目录下的文件会作为**构建上下文**发送给构建器。`.dockerignore` 用来排除不需要的文件，作用有两个：

* 构建上下文更小，构建更快。
* 防止敏感文件（如 `.env`）被 `COPY . .` 打进镜像。

项目的 `.dockerignore`：

```text
# 不进入镜像构建上下文的文件：既让构建更快，也防止敏感文件被打进镜像
.git
.venv
.env
.env.*
!.env.example
**/__pycache__
.mypy_cache
.ruff_cache
.pytest_cache
.coverage
htmlcov
.idea
.vscode
.DS_Store
tests
```

| 条目 | 排除原因 |
| --- | --- |
| `.git` | 版本历史体积大，镜像里用不到 |
| `.venv` | 本地虚拟环境（macOS 上的二进制）不能在 Linux 容器里用；不排除的话，`COPY . /app` 还会把它复制到 `/app/.venv`，和构建阶段生成的虚拟环境混在一起 |
| `.env`、`.env.*` | 含真实密钥，绝不能进镜像 |
| `!.env.example` | `!` 表示例外：示例配置不含密钥，可以保留 |
| 各类缓存、`htmlcov`、IDE 目录 | 开发工具的产物，与运行无关 |
| `tests` | 测试代码不进生产镜像 |

## 2. Dockerfile 逐行解析

### 2.1. 完整 Dockerfile

项目的 Dockerfile 采用 uv 官方推荐的多阶段写法：

```dockerfile
# syntax=docker/dockerfile:1

# ============ 构建阶段：安装依赖，生成虚拟环境 ============
FROM ghcr.io/astral-sh/uv:0.11-python3.12-trixie-slim AS builder

ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=0

WORKDIR /app

# 先只装第三方依赖：只要 pyproject.toml 和 uv.lock 不变，这一层就走缓存，改业务代码不会重装依赖
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project --no-dev

# 再装项目本身。--no-editable：代码直接装进虚拟环境，运行阶段不需要 src 目录
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev --no-editable

# ============ 运行阶段：只包含运行所需的文件 ============
FROM python:3.12-slim-trixie

# 用非 root 用户运行：即使应用被攻破，攻击者也拿不到容器内的 root 权限
RUN groupadd --system app && useradd --system --gid app --no-create-home app

WORKDIR /app
COPY --from=builder --chown=app:app /app/.venv /app/.venv
# 数据库迁移需要的文件（迁移作为单独的任务执行，见 docker-compose.yml）
COPY --chown=app:app alembic.ini ./
COPY --chown=app:app migrations ./migrations

ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1

USER app
EXPOSE 8000

# 运行阶段没有 pyproject.toml，读不到 [tool.fastapi] entrypoint，所以这里显式指定
CMD ["fastapi", "run", "--entrypoint", "my_service.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

第一行 `# syntax=docker/dockerfile:1` 指定使用最新的 Dockerfile 1.x 语法，后面用到的 `RUN --mount` 依赖它。

### 2.2. 两阶段产物流向

构建阶段负责"装"，运行阶段只负责"跑"。运行镜像从构建阶段拿走的只有 `.venv` 一个目录，另外从构建上下文直接复制迁移文件：

```mermaid
graph LR
    subgraph ctx["构建上下文"]
        lock["uv.lock + pyproject.toml"]
        src["src/ 等全部代码"]
        mig["alembic.ini + migrations/"]
    end

    subgraph builder["构建阶段 uv:0.11-python3.12-trixie-slim"]
        dep["依赖层: uv sync --no-install-project"]
        proj["项目层: uv sync --no-editable"]
        venv["/app/.venv"]
        dep --> proj --> venv
    end

    subgraph runtime["运行阶段 python:3.12-slim-trixie"]
        rvenv["/app/.venv"]
        rmig["/app/alembic.ini + /app/migrations"]
    end

    lock -->|"bind mount"| dep
    src -->|"COPY"| proj
    venv -->|"COPY --from=builder"| rvenv
    mig -->|"COPY"| rmig
```

构建阶段里的 uv 工具、构建缓存、`src/` 源码目录都留在构建阶段，不会进入最终镜像。

### 2.3. 构建阶段：基础镜像与环境变量

```dockerfile
FROM ghcr.io/astral-sh/uv:0.11-python3.12-trixie-slim AS builder
```

这是 uv 官方发布的镜像：在 Debian trixie slim 上预装了 Python 3.12 和 uv 0.11。`AS builder` 给这个阶段命名，运行阶段用 `--from=builder` 引用它。

```dockerfile
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=0
```

| 环境变量 | 含义 | 为什么这样设 |
| --- | --- | --- |
| `UV_COMPILE_BYTECODE=1` | 安装时把 `.py` 编译成 `.pyc` 字节码 | 容器启动时不用再编译，启动更快 |
| `UV_LINK_MODE=copy` | 从缓存安装包时用复制，不用硬链接 | 缓存在 cache mount 里，和 `/app/.venv` 不在同一个文件系统，无法硬链接；而且 cache mount 不会进入镜像，必须把文件真正复制到虚拟环境里 |
| `UV_PYTHON_DOWNLOADS=0` | 禁止 uv 自动下载 Python | 只用镜像自带的 Python，保证虚拟环境链接的是 `/usr/local/bin/python3`（见 2.7） |

### 2.4. 依赖层：bind mount + cache mount

```dockerfile
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project --no-dev
```

这一步只安装第三方依赖，不安装项目本身。用到了两种 `RUN --mount`：

| 挂载 | 作用 |
| --- | --- |
| `type=bind`（uv.lock、pyproject.toml） | 把这两个文件临时挂进来，只在这条 `RUN` 执行期间可见，不会写入镜像层。只要这两个文件内容不变，这一层就命中缓存 |
| `type=cache`（/root/.cache/uv） | uv 的下载缓存在多次构建之间保留，不进入镜像。即使 `uv.lock` 变了需要重建这一层，已下载过的包也不用重新下载 |



`uv sync` 参数：

| 参数 | 含义 |
| --- | --- |
| `--locked` | 严格按 `uv.lock` 安装；`uv.lock` 和 `pyproject.toml` 不一致时直接失败，保证镜像里的依赖版本和本地一致 |
| `--no-install-project` | 只装依赖，不装项目本身（此时还没有复制代码） |
| `--no-dev` | 不装 `[dependency-groups] dev` 里的 pytest、ruff、mypy 等开发依赖 |



**为什么要把依赖单独成层？**

《Docker入门指南》讲过：一旦某层缓存失效，后面所有层都要重建。如果直接 `COPY . .` 再安装依赖，任何一行业务代码的修改都会让依赖安装重跑一遍。

把依赖安装放在 `COPY . /app` 之前，并且只依赖 `uv.lock` 和 `pyproject.toml`，日常改代码就只会让后面的项目层失效。

对构建耗时的影响（示例项目的典型数据）：

| 场景 | 耗时 |
| --- | --- |
| 首次构建（无缓存） | 约 2.5 分钟 |
| 只改业务代码后重建 | 约 7 秒 |

### 2.5. 项目层：COPY 与 --no-editable

```dockerfile
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked --no-dev --no-editable
```

先把代码复制进来（`.dockerignore` 已经排除了 `.venv`、`.env`、`tests` 等），再执行一次 `uv sync`。此时依赖已经装好，只会安装项目本身。

关键是 `--no-editable`：

| 安装方式 | 虚拟环境里装的是什么 | 运行时是否需要 `src/` |
| --- | --- | --- |
| 默认（editable，可编辑安装） | 一个指向 `src/` 目录的链接 | 需要 |
| `--no-editable` | 把项目打包后完整装进 `.venv/lib/python3.12/site-packages/` | 不需要 |

本地开发用可编辑安装，改代码立即生效；镜像里用 `--no-editable`，项目代码成为虚拟环境的一部分，运行阶段只复制 `.venv` 就够了。

`[project.scripts]` 里声明的命令行入口（如 `my-service`）也会生成在 `.venv/bin/` 下。

### 2.6. 运行阶段：只复制运行所需

```dockerfile
FROM python:3.12-slim-trixie

RUN groupadd --system app && useradd --system --gid app --no-create-home app

WORKDIR /app
COPY --from=builder --chown=app:app /app/.venv /app/.venv
COPY --chown=app:app alembic.ini ./
COPY --chown=app:app migrations ./migrations

ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONUNBUFFERED=1

USER app
EXPOSE 8000
```

| 指令 | 作用 |
| --- | --- |
| `FROM python:3.12-slim-trixie` | Docker 官方 Python 精简镜像，不含 uv 和构建工具 |
| `COPY --from=builder ... /app/.venv` | 从构建阶段只拿虚拟环境 |
| `COPY alembic.ini`、`COPY migrations` | 迁移任务需要这两样（迁移不是 Python 包的一部分，不在 `.venv` 里） |
| `ENV PATH="/app/.venv/bin:$PATH"` | 让 `fastapi`、`alembic`、`my-service`、`python` 直接指向虚拟环境里的版本，相当于"激活"虚拟环境 |
| `EXPOSE 8000` | 声明容器监听 8000 端口（只是声明，发布端口靠 `-p` 或 compose 的 `ports`） |

非 root 和 `PYTHONUNBUFFERED` 见第 3 章。

### 2.7. 两阶段的 Python 路径必须一致

多阶段构建复制虚拟环境，有一个容易忽略的前提：**虚拟环境不是自包含的**。

`.venv/bin/python` 只是一个指向基础解释器的链接，`.venv/pyvenv.cfg` 里也记录了基础解释器所在的目录。虚拟环境被复制到运行阶段后，链接指向的路径必须在运行镜像里同样存在，否则 `python` 根本启动不了。

两个镜像的解释器位置如下（补丁版本会随镜像更新而变化）：

| 阶段 | 镜像 | `python3` 路径 | 补丁版本 |
| --- | --- | --- | --- |
| 构建 | `ghcr.io/astral-sh/uv:0.11-python3.12-trixie-slim` | `/usr/local/bin/python3` | 3.12.13 |
| 运行 | `python:3.12-slim-trixie` | `/usr/local/bin/python3` | 3.12.16 |

路径一致，虚拟环境可以直接使用。补丁版本不同不影响：同为 3.12，安装的包对应的是 `python3.12` 目录和 3.12 的 ABI。



选基础镜像时要注意：

* 两个阶段用**相同的 Python 小版本**（这里都是 3.12）和**相同的发行版**（这里都是 Debian trixie），避免 C 扩展依赖的系统库不一致。
* 两个阶段的虚拟环境路径也要一致（都在 `/app/.venv`）。`.venv/bin/` 下的命令行脚本，第一行写的是虚拟环境 Python 的绝对路径。
* 这也是设置 `UV_PYTHON_DOWNLOADS=0` 的原因：如果 uv 自己下载了一份 Python 放在别的目录，虚拟环境就会链接到那里，运行镜像里没有这份 Python。

## 3. 运行时细节

### 3.1. 非 root 运行

容器默认以 root 运行。应用一旦被攻破，攻击者就拿到容器内的 root 权限，后续利用内核或挂载漏洞的空间更大。

```dockerfile
RUN groupadd --system app && useradd --system --gid app --no-create-home app
...
COPY --from=builder --chown=app:app /app/.venv /app/.venv
COPY --chown=app:app alembic.ini ./
COPY --chown=app:app migrations ./migrations
...
USER app
```

| 写法 | 含义 |
| --- | --- |
| `groupadd --system app` | 创建系统组 `app`（系统组的 gid 在系统保留范围内） |
| `useradd --system --gid app` | 创建系统用户 `app`，归属 `app` 组 |
| `--no-create-home` | 不创建家目录，服务账号用不到 |
| `COPY --chown=app:app` | 复制时直接设置文件属主。比复制后再 `RUN chown -R` 好：后者会把所有文件再写一遍，多出一个同样大小的层 |
| `USER app` | 之后的指令和容器运行进程都以 `app` 身份执行 |

在容器内检查，运行后会输出：

```shell
docker compose exec app id
# uid=999(app) gid=999(app) ...

docker compose exec app ls -A /app
# .venv  alembic.ini  migrations
```

`/app` 下只有虚拟环境和迁移文件，没有 `src/`、`tests/`、`pyproject.toml`。

### 3.2. CMD 的 exec 形式与优雅停机

```dockerfile
CMD ["fastapi", "run", "--entrypoint", "my_service.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

CMD 有两种写法：

| 写法 | 示例 | 实际执行 | PID 1 是谁 |
| --- | --- | --- | --- |
| exec 形式（JSON 数组） | `CMD ["fastapi", "run", ...]` | 直接执行 `fastapi` | 应用进程 |
| shell 形式（字符串） | `CMD fastapi run ...` | `/bin/sh -c "fastapi run ..."` | `sh` |

`docker stop` 会向容器的 PID 1 发送 `SIGTERM`，等待一段时间（默认 10 秒）后仍未退出就发送 `SIGKILL` 强杀。

* exec 形式：应用就是 PID 1，直接收到 `SIGTERM`，执行 FastAPI lifespan 里的关闭逻辑（释放数据库连接池），然后正常退出。
* shell 形式：`sh` 是 PID 1，它不会把信号转发给子进程。应用收不到 `SIGTERM`，只能等 10 秒后被强杀，关闭逻辑不会执行。



项目的 lifespan 关闭逻辑：

```python
@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None]:
    ...
    yield

    # 关闭：释放连接池中的所有连接
    await engine.dispose()
    log.info("app_shutdown")
```

停止容器时，日志会依次输出：

```text
app_shutdown
Application shutdown complete.
Finished server process [1]
```

`[1]` 说明服务进程的 PID 就是 1。

### 3.3. PYTHONUNBUFFERED=1

Python 的标准输出写到管道（而不是终端）时，默认是块缓冲：攒够一定数量才真正写出去。容器里的 stdout 正是管道。

不设置时会出现两个问题：

* `docker logs` 看到的日志有延迟。
* 进程崩溃时，缓冲区里还没写出的最后几条日志直接丢失，而这几条往往是排查问题最需要的。

设置 `PYTHONUNBUFFERED=1` 后，每条日志立即输出。

### 3.4. 坑：fastapi run 找不到入口

如果在 `pyproject.toml` 里配置了入口，本地直接 `fastapi run` 就能启动：

```toml
[tool.fastapi]
entrypoint = "my_service.main:app"
```

现象：Dockerfile 的 CMD 如果也只写 `fastapi run`，容器启动后报错：

```text
Could not find a default file to run, please provide an explicit path
```

原因：`fastapi run` 从当前目录的 `pyproject.toml` 读取 `[tool.fastapi] entrypoint`，而运行镜像里只有 `.venv`、`alembic.ini`、`migrations`，没有 `pyproject.toml`。

解决：在 CMD 里显式指定入口。

```dockerfile
CMD ["fastapi", "run", "--entrypoint", "my_service.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

`--host 0.0.0.0` 也是必须的：只监听 `127.0.0.1` 的话，容器外（包括端口映射）都访问不到。

> 这类问题的共同原因是：多阶段构建精简了运行镜像，本地开发时隐式依赖的文件在镜像里不存在。写完 Dockerfile 后，要以运行镜像的实际内容为准检查启动命令。

### 3.5. slim 镜像没有 curl：用 Python 做健康检查

很多健康检查示例写的是 `curl -f http://localhost:8000/...`。`python:3.12-slim-trixie` 里没有 `curl`，为了健康检查专门装一个也不划算。

镜像里一定有 Python，用标准库 `urllib` 就能发请求：

```yaml
healthcheck:
  test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/readyz', timeout=3)"]
```

`urlopen` 在连接失败、超时或返回 4xx/5xx（如 `/readyz` 在数据库不可用时返回的 503）时都会抛出异常，Python 以非 0 状态码退出，Docker 就判定该次检查失败。

项目提供两个健康检查接口：

| 接口 | 用途 | 检查内容 |
| --- | --- | --- |
| `/healthz` | 存活检查 | 进程能响应就返回 200，不检查外部依赖 |
| `/readyz` | 就绪检查 | 执行 `SELECT 1`，数据库可用返回 200，否则 503；内部超时 2 秒 |

## 4. Compose 编排逐段解析

### 4.1. 完整 docker-compose.yml

```yaml
services:
  postgres:
    image: postgres:17-alpine
    container_name: my-service-postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres   # 仅本地开发用，与 .env.example 一致
      POSTGRES_DB: my_service
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data   # 数据持久化，删掉容器数据也还在
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -h 127.0.0.1 -U postgres -d my_service"]
      interval: 5s
      timeout: 3s
      retries: 10

  # ---------- 以下服务只在 --profile app 时启动，日常开发 docker compose up -d 仍只起数据库 ----------
  migrate:
    profiles: ["app"]
    build: .
    image: my-service:local
    command: ["alembic", "upgrade", "head"]
    environment: &app-env
      APP_ENV: prod
      # 容器之间用服务名 postgres 访问，不是 localhost
      DATABASE_URL: postgresql+asyncpg://postgres:postgres@postgres:5432/my_service
      # 从宿主机的 .env 或环境变量读取；未设置时直接报错，不会用空密钥启动
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:?请在 .env 中设置 JWT_SECRET_KEY}
    depends_on:
      postgres:
        condition: service_healthy

  app:
    profiles: ["app"]
    image: my-service:local
    environment: *app-env
    ports:
      - "8080:8000"   # 宿主机用 8080，避免和本地 fastapi dev 的 8000 冲突
    depends_on:
      migrate:
        condition: service_completed_successfully   # 迁移成功后才启动应用
    healthcheck:
      # 镜像里没有 curl，用 Python 自带的 urllib 请求就绪接口
      test: ["CMD", "python", "-c", "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/readyz', timeout=3)"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 10s

volumes:
  pgdata:
```

三个服务：

| 服务 | 镜像 | 作用 | 运行方式 |
| --- | --- | --- | --- |
| `postgres` | `postgres:17-alpine` | 数据库 | 常驻 |
| `migrate` | `my-service:local`（由 `build: .` 构建） | 执行 `alembic upgrade head` | 执行完退出 |
| `app` | `my-service:local`（复用同一个镜像） | 应用服务 | 常驻 |

`migrate` 同时写了 `build: .` 和 `image: my-service:local`，表示构建当前目录的 Dockerfile，并把结果命名为 `my-service:local`。`app` 只写 `image`，直接复用这个镜像，只是用 `command` 换了启动命令。迁移和应用用同一个镜像，保证迁移脚本和代码版本一致。

### 4.2. healthcheck 参数

| 参数 | 含义 | postgres | app |
| --- | --- | --- | --- |
| `test` | 检查命令，退出码 0 为成功 | `pg_isready ...` | `python -c "...urlopen..."` |
| `interval` | 两次检查的间隔 | 5s | 10s |
| `timeout` | 单次检查超过这个时间算失败 | 3s | 5s |
| `retries` | 连续失败多少次判定为 unhealthy | 10 | 3 |
| `start_period` | 启动宽限期，这段时间内的失败不计入 retries | 未设置 | 10s |



`test` 的两种写法：

| 写法 | 执行方式 | 适用场景 |
| --- | --- | --- |
| `["CMD", "可执行文件", "参数", ...]` | 直接执行，不经过 shell | 命令和参数都固定（app 的写法） |
| `["CMD-SHELL", "命令字符串"]` | 交给容器内的 `/bin/sh -c` 执行 | 需要 shell 语法，或习惯写成一整行（postgres 的写法） |

app 的检查命令里 `timeout=3` 小于 healthcheck 的 `timeout: 5s`，请求超时会先在 Python 里报错退出，而不是被 Docker 强行中止。

### 4.3. postgres：pg_isready 必须加 -h 127.0.0.1

```yaml
test: ["CMD-SHELL", "pg_isready -h 127.0.0.1 -U postgres -d my_service"]
```

PostgreSQL 官方镜像在**首次初始化**数据目录时，会先启动一个只监听 Unix socket 的临时实例，用来创建用户、数据库、执行初始化脚本；完成后关闭临时实例，再启动监听 TCP 的正式实例。

| 写法 | 检查走的通道 | 首次初始化时的表现 |
| --- | --- | --- |
| `pg_isready -U postgres` | Unix socket | 临时实例一起来就成功，过早显示 healthy，但其他容器这时还连不上 |
| `pg_isready -h 127.0.0.1 -U postgres` | TCP | 正式实例开始监听 TCP 后才成功，和其他容器的连接方式一致 |

`migrate` 依赖 postgres 的 healthy 状态启动，如果 healthy 来得太早，迁移会因为连不上数据库而失败。

### 4.4. depends_on：数据库就绪 → 迁移成功 → 启动应用

默认的 `depends_on` 只保证启动顺序，不等待依赖真正可用。加上 `condition` 后可以指定等待条件：

| condition | 含义 |
| --- | --- |
| `service_started` | 依赖的容器已启动（默认行为） |
| `service_healthy` | 依赖的容器健康检查通过 |
| `service_completed_successfully` | 依赖的容器运行结束且退出码为 0 |

项目的配置：

```yaml
migrate:
  depends_on:
    postgres:
      condition: service_healthy

app:
  depends_on:
    migrate:
      condition: service_completed_successfully
```

启动顺序：

```mermaid
graph LR
    pg["postgres 启动"] --> pgh{"pg_isready 通过?"}
    pgh -->|"否，每 5s 重试"| pgh
    pgh -->|"是：service_healthy"| mig["migrate: alembic upgrade head"]
    mig --> migok{"退出码为 0?"}
    migok -->|"否"| stop["app 不启动"]
    migok -->|"是：service_completed_successfully"| app["app 启动"]
    app --> apph{"/readyz 返回 200?"}
    apph -->|"是"| healthy["app healthy"]
```

迁移失败时 `app` 不会启动，避免新代码跑在旧表结构上。

### 4.5. profiles：日常只起数据库

```yaml
migrate:
  profiles: ["app"]
app:
  profiles: ["app"]
```

带 `profiles` 的服务默认不启动，只有命令行指定了对应 profile 才启动；没写 `profiles` 的服务（postgres）总是启动。

| 命令 | 启动的服务 | 场景 |
| --- | --- | --- |
| `docker compose up -d` | postgres | 日常开发：数据库在容器里，应用在本地 `fastapi dev` 运行 |
| `docker compose --profile app up -d --build` | postgres、migrate、app | 本地以生产形态验证整套服务 |

一个 compose 文件同时满足两种用法，不用维护两份配置。

### 4.6. 必填插值与 .env

```yaml
JWT_SECRET_KEY: ${JWT_SECRET_KEY:?请在 .env 中设置 JWT_SECRET_KEY}
```

`${VAR:?提示信息}` 是必填插值：变量未设置或为空时，compose 直接报错退出并显示提示信息，不会用空密钥启动应用。

常用插值写法：

| 写法 | 变量未设置或为空时 |
| --- | --- |
| `${VAR}` | 替换为空字符串 |
| `${VAR:-默认值}` | 使用默认值 |
| `${VAR:?提示信息}` | 报错退出 |

变量的来源：compose 会自动读取**项目目录下的 `.env` 文件**用于插值，宿主机的环境变量也可以提供。如果本地开发已经在 `.env` 里配置了 `JWT_SECRET_KEY`，compose 可以直接复用，不需要再复制一份。

> 注意：项目目录的 `.env` 只用于 compose 文件里的 `${...}` 插值，不会自动注入到容器中。容器里能看到哪些变量，由 `environment` 或 `env_file` 决定。这里正是通过 `environment` 把插值结果传给容器。

### 4.7. YAML 锚点复用环境变量

`migrate` 和 `app` 需要完全相同的环境变量。项目用 YAML 锚点避免重复：

```yaml
migrate:
  environment: &app-env        # &app-env：给这个映射起名
    APP_ENV: prod
    DATABASE_URL: postgresql+asyncpg://postgres:postgres@postgres:5432/my_service
    JWT_SECRET_KEY: ${JWT_SECRET_KEY:?请在 .env 中设置 JWT_SECRET_KEY}

app:
  environment: *app-env        # *app-env：引用上面的内容
```

| 符号 | 含义 |
| --- | --- |
| `&名称` | 定义锚点 |
| `*名称` | 引用锚点，等同于把锚点处的内容原样复制过来 |

锚点是 YAML 本身的语法，不是 compose 特有的功能。改配置时只改一处，两个服务不会不一致。

### 4.8. 网络、端口与 APP_ENV

```yaml
DATABASE_URL: postgresql+asyncpg://postgres:postgres@postgres:5432/my_service
```

compose 会为项目创建一个默认网络，所有服务都在这个网络里，可以**用服务名互相访问**。容器里的 `localhost` 指容器自己，所以数据库地址要写服务名 `postgres`，不是 `localhost`。

| 访问方 | 数据库地址 |
| --- | --- |
| 宿主机上本地运行的应用 | `localhost:5432`（通过 postgres 的 `ports` 映射） |
| compose 里的 migrate、app 容器 | `postgres:5432`（服务名） |



```yaml
ports:
  - "8080:8000"
```

容器内应用监听 8000，映射到宿主机 8080。本地开发时 `fastapi dev` 已经占用了宿主机 8000，用 8080 可以让两者同时运行。



```yaml
APP_ENV: prod
```

容器以生产配置运行：日志为 JSON 格式，`/docs`、`/redoc`、`/openapi.json` 关闭。项目中的实现：

```python
app = FastAPI(
    ...
    # 生产环境不暴露接口文档
    docs_url=None if settings.is_prod else "/docs",
    redoc_url=None if settings.is_prod else "/redoc",
    openapi_url=None if settings.is_prod else "/openapi.json",
)
```

## 5. 运行与验证

### 5.1. 启动与查看状态

```shell
docker compose --profile app up -d --build
docker compose --profile app ps -a
```

`ps` 加 `-a` 才能看到已经退出的 migrate。正常启动后的状态：

| 服务 | 状态 | 说明 |
| --- | --- | --- |
| `app` | `Up (healthy)` | 健康检查通过 |
| `migrate` | `Exited (0)` | 迁移完成后正常退出，这是预期状态 |
| `postgres` | `Up (healthy)` | 健康检查通过 |

### 5.2. 接口验证

```shell
curl -s -o /dev/null -w "%{http_code}\n" localhost:8080/readyz
# 200

curl -s -o /dev/null -w "%{http_code}\n" localhost:8080/docs
# 404
```

`/readyz` 返回 200，说明应用和数据库都正常；`/docs` 返回 404，说明 `APP_ENV=prod` 生效，接口文档已关闭。

### 5.3. 在容器中执行管理命令

```shell
docker compose exec app my-service create-user <用户名>
```

`my-service` 是 `pyproject.toml` 里 `[project.scripts]` 声明的命令行入口，安装在 `/app/.venv/bin/` 下。`PATH` 已包含这个目录，可以直接执行，并且使用的是和应用相同的环境变量和数据库连接。

迁移可以重复执行：再次运行 migrate 时，Alembic 会跳过已经执行过的版本。所以每次 `up` 都运行一遍 migrate 是安全的。

### 5.4. 停止与数据卷

```shell
docker compose --profile app down
```

| 命令 | 删除的内容 |
| --- | --- |
| `docker compose --profile app down` | 容器、默认网络 |
| `docker compose --profile app down -v` | 容器、默认网络，**以及命名卷 `pgdata`**（数据库数据全部丢失） |

**不要加 `-v`**，除非确实想清空数据库。`down` 命令同样要加 `--profile app`，否则不会处理 migrate 和 app。

## 6. 数据库迁移作为一次性任务

项目没有在应用启动时（比如 lifespan 里或启动脚本里）执行迁移，而是单独作为一次性任务（compose 里的 `migrate` 服务）。

| 方式 | 问题 |
| --- | --- |
| 应用启动时执行迁移 | 多副本部署时，多个实例会同时执行迁移，可能互相冲突；迁移耗时会拖慢启动、导致健康检查超时；应用运行用户还需要有修改表结构的权限 |
| 一次性任务执行迁移 | 只执行一次；失败时应用不会启动（`service_completed_successfully`）；迁移和应用的职责分开 |

迁移任务和应用使用同一个镜像，只是启动命令不同：

```yaml
migrate:
  image: my-service:local
  command: ["alembic", "upgrade", "head"]
```

这也是运行镜像里要保留 `alembic.ini` 和 `migrations/` 的原因。

## 7. 部署到 Kubernetes 的要点

compose 里的做法可以直接对应到 Kubernetes：

| 事项 | 做法 | 原因 |
| --- | --- | --- |
| 数据库迁移 | 部署前用 Job 或 initContainer 执行 `alembic upgrade head` | 对应 compose 的 migrate 服务，不在应用启动时执行 |
| 密钥 | `JWT_SECRET_KEY`、`DATABASE_URL` 放在 Secret 中，以环境变量注入 | 密钥不写进镜像和部署清单 |
| 存活探针 | `livenessProbe` → `GET /healthz` | 只检查进程本身。如果检查数据库，数据库故障时所有 Pod 都会被重启，于事无补 |
| 就绪探针 | `readinessProbe` → `GET /readyz`，`timeoutSeconds` ≥ 3 | `/readyz` 内部数据库检查超时为 2 秒，探针默认超时只有 1 秒，不够用 |
| 扩容 | 每个 Pod 运行一个进程，通过增加副本数扩容 | 进程数交给 K8s 管理，资源限制和故障恢复都以 Pod 为单位 |
| 镜像标签 | 使用 git commit hash | 每个镜像对应唯一的代码版本，回滚时直接切回旧标签 |

两种探针的区别：

| 探针 | 失败后 K8s 的动作 | 对应接口 |
| --- | --- | --- |
| `livenessProbe` 存活探针 | 重启容器 | `/healthz` |
| `readinessProbe` 就绪探针 | 从 Service 中摘除，不再分配流量，但不重启 | `/readyz` |

探针配置示例（只列关键字段）：

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8000
readinessProbe:
  httpGet:
    path: /readyz
    port: 8000
  timeoutSeconds: 3
```

## 8. 镜像体积与优化方向

按本文 Dockerfile 构建出的镜像约 **394MB**。

用 `fastapi[standard]` 安装 FastAPI 时，除了 FastAPI 本身，还会附带 `fastapi-cli`、`fastapi-cloud-cli` 等额外依赖，这是体积的主要来源。

以下是可以尝试的优化思路（效果需结合项目实际评估）：

| 思路 | 说明 | 代价 |
| --- | --- | --- |
| 不用 `fastapi[standard]` | 改为 `fastapi` + `uvicorn[standard]` 等实际用到的依赖 | `fastapi run` 命令来自 `fastapi-cli`，CMD 需要改为直接用 `uvicorn` 启动 |
| 分析各层体积 | 用 `docker history <镜像>` 或 dive 等工具查看每层大小，找出最大的依赖 | 需要逐个评估依赖是否可以去掉 |
| 更换更小的基础镜像 | 如 distroless 类镜像 | 要保证两个阶段的 Python 路径一致（见 2.7）；没有 shell，排查问题和 `CMD-SHELL` 类检查都会受影响 |

> 不建议为了体积换成 alpine：alpine 使用 musl libc，很多 Python 包没有现成的 musl 二进制包，可能需要在构建时编译，构建更慢，问题也更难排查。

## 9. 小结

1. **多阶段构建：** 构建阶段用 uv 生成虚拟环境，运行阶段只复制 `.venv`、`alembic.ini`、`migrations`。
2. **依赖单独成层：** 用 bind mount 挂载 `uv.lock`、`pyproject.toml` 安装依赖，再复制代码安装项目。只改代码时重建约 7 秒，首次构建约 2.5 分钟。
3. **`--no-editable`：** 项目代码装进虚拟环境，运行镜像不需要 `src/`。
4. **Python 路径一致：** 虚拟环境链接基础解释器，两个阶段的 `python3` 都在 `/usr/local/bin/python3`。
5. **非 root 运行：** `useradd --system` + `COPY --chown` + `USER app`。
6. **exec 形式的 CMD：** 应用是 PID 1，能收到 `SIGTERM` 并执行关闭逻辑；`PYTHONUNBUFFERED=1` 保证日志不丢。
7. **运行镜像里没有的文件不能依赖：** 没有 `pyproject.toml`，所以要显式写 `--entrypoint`；没有 `curl`，所以用 Python 的 `urllib` 做健康检查。
8. **compose 启动顺序：** `service_healthy` + `service_completed_successfully` 实现数据库就绪 → 迁移成功 → 启动应用；`pg_isready` 要加 `-h 127.0.0.1`。
9. **compose 配置技巧：** `profiles` 区分日常开发和完整运行；`${VAR:?}` 让必填变量缺失时报错；YAML 锚点复用环境变量；容器间用服务名访问。
10. **迁移是一次性任务：** 不在应用启动时执行；K8s 中用 Job 或 initContainer。
11. **K8s 探针：** liveness → `/healthz`，readiness → `/readyz` 且 `timeoutSeconds` ≥ 3。
12. **停止时不要加 `-v`：** 否则数据卷会被删除。
