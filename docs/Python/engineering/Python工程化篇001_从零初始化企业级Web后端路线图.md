## 1. 路线总览

本文给出一个企业级 Python Web 后端从空目录到可部署的推荐初始化顺序。示例项目名为 `my-service`（Python 包名 `my_service`），技术选型如下：

| 类别 | 选型 |
|------|------|
| 语言 / 包管理 | Python 3.12 / uv |
| Web 框架 | FastAPI（Uvicorn） |
| 数据库 | PostgreSQL 17、SQLAlchemy 2.0（异步，asyncpg 驱动）、Alembic 迁移 |
| 配置 | pydantic-settings（环境变量 / `.env`） |
| 日志 | structlog（开发环境彩色输出，生产环境 JSON） |
| 认证 | JWT access token + 存库的 refresh token（Argon2id 密码哈希） |
| 质量 | ruff（lint + 格式化）、mypy（strict）、pytest、pre-commit |
| 交付 | Docker 多阶段构建、docker compose |

本文是路线图，每一步只讲目的、产出文件和验证方式，具体技术细节放在同系列的详细篇里，各步骤末尾附有链接。

### 1.1. 13 步路线

```mermaid
flowchart TD
    subgraph A[阶段一：工程骨架]
        S1[1. .gitignore / .env.example / 首次提交] --> S2[2. pyproject 配置 ruff / mypy / pytest]
    end
    subgraph B[阶段二：应用基础设施]
        S3[3. 配置模块 core/config.py] --> S4[4. 结构化日志 core/logging.py] --> S5[5. main.py 与健康检查]
    end
    subgraph C[阶段三：数据库]
        S6[6. Docker 起 PostgreSQL<br/>引擎、会话、ORM 基类] --> S7[7. Alembic 迁移]
    end
    subgraph D[阶段四：请求链路与业务]
        S8[8. 统一异常与 request-id 中间件] --> S9[9. 用户表与 create-user 命令] --> S10[10. JWT 登录 / 刷新 / 注销]
    end
    subgraph E[阶段五：质量与交付]
        S11[11. pytest 测试] --> S12[12. pre-commit] --> S13[13. Dockerfile 与 compose]
    end
    S2 --> S3
    S5 --> S6
    S7 --> S8
    S10 --> S11
```

排列顺序遵循一个原则：**先有地基，再有业务**。配置和日志是后面所有模块都要用的，所以排在最前；数据库会话是业务的前提；统一异常要在写业务接口之前定好，否则每个接口都会各写一套错误格式。

### 1.2. 每步一次提交

每完成一步就提交一次，一次提交只包含这一步的产出；步骤 9 和步骤 10 联系紧密，也可以合成一次提交。提交信息建议从第一次提交起就使用约定式前缀：`feat:`（新功能）、`chore:`（工程配置）、`test:`（测试）、`build:`（构建与交付）。

| 步骤 | 建议的提交信息 | 主要产出 |
|------|---------|---------|
| 1 | `chore: init project skeleton` | `.gitignore`、`.env.example`、`pyproject.toml`、`uv.lock`、`alembic.ini`、`migrations/`、`tests/__init__.py` |
| 2 | `chore: configure ruff, mypy and pytest` | `pyproject.toml` 的 `[tool.*]` 配置 |
| 3 | `feat: add settings module` | `core/config.py` |
| 4 | `feat: add structured logging` | `core/logging.py` |
| 5 | `feat: add app entry and health endpoints` | `main.py`、`api/health.py`、`api/v1/router.py` |
| 6 | `feat: add database engine, session and readiness check` | `docker-compose.yml`、`db/session.py`、`models/base.py`、`api/deps.py` |
| 7 | `feat: configure alembic with app settings and models` | `alembic.ini`、`migrations/env.py`、第一个迁移 |
| 8 | `feat: add unified error handling and request context middleware` | `core/exceptions.py`、`core/middleware.py` |
| 9、10 | `feat: add users, JWT login, refresh token rotation and logout` | 模型、迁移、repositories、schemas、services、`cli.py`、`core/security.py`、`api/v1/auth.py` |
| 11 | `test: add tests for security, health and auth` | `tests/conftest.py`、`tests/test_*.py` |
| 12 | `chore: add pre-commit hooks` | `.pre-commit-config.yaml` |
| 13 | `build: add Dockerfile and compose app/migrate services` | `Dockerfile`、`.dockerignore`、`docker-compose.yml` |

### 1.3. 每一步都要能验证

每一步结束时都要跑一条命令，确认这一步真的做对了，再提交。这样出问题时只需要排查最近一步的改动。下文每一步都列出了**验证命令**和**预期结果**。

## 2. 阶段一：工程骨架

### 2.1. 步骤 1：.gitignore、.env.example 与首次提交

**目的**：建立 uv 项目结构，确定哪些文件进仓库、哪些不进，完成第一次提交。

**主要命令**：

```shell
uv init --app --package --python 3.12 .

# 运行依赖与开发依赖
uv add "fastapi[standard]" pydantic-settings "sqlalchemy[asyncio]" asyncpg alembic structlog
uv add --dev ruff mypy pytest pytest-asyncio pytest-cov pre-commit httpx

# 用 async 模板生成 alembic.ini 和 migrations/（步骤 7 再接入应用配置）
uv run alembic init -t async migrations
```

**产出文件**：

| 文件 | 说明 |
|------|------|
| `.python-version` | `3.12`，uv 按它自动下载 Python |
| `pyproject.toml` / `uv.lock` | 依赖声明与锁文件，都要提交 |
| `.gitignore` | 排除虚拟环境、缓存、`.env` |
| `.env.example` | 配置模板，只写变量名和示例值 |
| `src/my_service/__init__.py` | 包目录（src 布局） |
| `tests/__init__.py` | 空文件，让 git 能跟踪 `tests/` 目录（见第 10 章） |
| `alembic.ini`、`migrations/` | Alembic 模板生成的文件 |

`.gitignore` 的核心部分：

```gitignore
# 虚拟环境
.venv/

# 环境变量
.env
.env.*
!.env.example

# 工具缓存
.ruff_cache/
.mypy_cache/
.pytest_cache/
.coverage
htmlcov/
```

`.env.*` 加 `!.env.example` 的写法：先排除所有 `.env.xxx` 文件（如 `.env.local`、`.env.prod`），再单独放行模板文件。

**验证**：

```shell
git check-ignore -v .env
# .gitignore:12:.env	.env        ← 说明 .env 被第 12 行规则忽略

git ls-files tests
# tests/__init__.py              ← tests 目录已被跟踪
```

详细篇：[Python 虚拟环境和包](../python/基础篇010_Python虚拟环境和包.md)

### 2.2. 步骤 2：在 pyproject.toml 中配置 ruff、mypy、pytest

**目的**：在写业务代码之前定好代码规范和类型检查标准，之后每一行代码都按同一标准检查。

**产出文件**：`pyproject.toml` 新增 `[tool.ruff]`、`[tool.ruff.lint]`、`[tool.mypy]`、`[tool.pytest.ini_options]`、`[tool.coverage.run]` 几节，各节作用见第 7 章。

**验证**：

```shell
uv run ruff check --fix     # 预期：All checks passed!
uv run ruff format --check  # 预期：所有文件 already formatted
uv run mypy src             # 预期：Success: no issues found in N source files
uv run pytest               # 此时还没有测试，预期：no tests ran
```

首次运行 `ruff check --fix` 时，它可能会调整 Alembic 模板生成的 `migrations/env.py` 的 import 顺序（把 `from alembic import context` 移到第三方库分组里）。这说明 lint 配置已经生效。

详细篇：[代码质量工具 ruff 与 mypy](./Python工程化篇002_代码质量工具ruff与mypy.md)

## 3. 阶段二：应用基础设施

### 3.1. 步骤 3：配置模块 core/config.py

**目的**：所有配置从环境变量和 `.env` 读取，启动时统一校验类型和取值范围，缺项或格式错误直接启动失败。

**产出文件**：`src/my_service/core/__init__.py`、`src/my_service/core/config.py`，`.env.example` 补充 `LOG_LEVEL`、`DATABASE_POOL_SIZE`。

下面是完整版本（JWT 相关字段在步骤 10 加入）：

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8", extra="ignore")

    app_name: str = "my-service"
    app_env: Literal["dev", "test", "prod"] = "dev"
    app_debug: bool = False
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"

    # 用 SecretStr 包装：打印配置或写日志时密码会显示为 **********
    database_url: SecretStr
    database_pool_size: int = Field(default=10, ge=1)
    database_max_overflow: int = Field(default=20, ge=0)

    jwt_secret_key: SecretStr = Field(min_length=32)
    access_token_expire_minutes: int = Field(default=15, ge=1)
    refresh_token_expire_days: int = Field(default=7, ge=1)

    @field_validator("database_url")
    @classmethod
    def _check_database_url(cls, v: SecretStr) -> SecretStr:
        PostgresDsn(v.get_secret_value())
        return v

    @property
    def is_prod(self) -> bool:
        return self.app_env == "prod"


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

**验证**：

```shell
uv run python -c "from my_service.core.config import get_settings; print(get_settings())"
```

| 场景 | 预期结果 |
|------|---------|
| 配置齐全 | 打印全部字段，`database_url=SecretStr('**********')`，密码不会明文显示 |
| 缺少 `DATABASE_URL` | `database_url  Field required` |
| `JWT_SECRET_KEY` 太短 | `Value should have at least 32 items after validation` |
| `DATABASE_URL` 用了 `mysql://` | `URL scheme should be 'postgres', 'postgresql', 'postgresql+asyncpg', ...` |

详细篇：[配置管理 pydantic-settings](./Python工程化篇005_配置管理pydantic-settings.md)

### 3.2. 步骤 4：结构化日志 core/logging.py

**目的**：让 structlog 和标准库 logging（uvicorn、sqlalchemy 等第三方库）走同一条处理链，开发环境输出彩色可读格式，生产环境每行一条 JSON。

**产出文件**：`src/my_service/core/logging.py`，对外提供 `setup_logging(settings)` 和 `get_logger(name)`。

**验证**：

```shell
uv run python -c "
from my_service.core.config import get_settings
from my_service.core.logging import setup_logging, get_logger
setup_logging(get_settings())
get_logger('demo').info('user_login', user_id=1)
"
```

预期输出（`APP_ENV=dev`）：

```
2026-11-18T07:07:05.842827Z [info     ] user_login                     [demo] user_id=1
```

在命令前加 `APP_ENV=prod` 再执行一次，预期输出 JSON：

```json
{"user_id": 1, "event": "user_login", "level": "info", "logger": "demo", "timestamp": "2026-11-18T07:07:05.987676Z"}
```

详细篇：[结构化日志 structlog](./Python工程化篇006_结构化日志structlog.md)

### 3.3. 步骤 5：main.py 与健康检查

**目的**：创建 FastAPI 应用入口，挂载路由，提供 `/healthz`（存活检查）和 `/readyz`（就绪检查，这一步先占位，步骤 6 接入数据库检查）。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `src/my_service/main.py` | `create_app()` 工厂函数、`lifespan`、模块级 `app = create_app()` |
| `src/my_service/api/health.py` | `/healthz`、`/readyz` |
| `src/my_service/api/v1/router.py` | v1 路由汇总 |
| `src/my_service/__init__.py` | 删掉 `uv init` 生成的 `main()`，改为从包元数据读取 `__version__` |
| `pyproject.toml` | 新增 `[tool.fastapi] entrypoint`；删掉指向旧 `main()` 的 `[project.scripts]`（步骤 9 再加回来） |

**验证**：

```shell
uv run fastapi dev

# 另开一个终端
curl -i localhost:8000/healthz
# HTTP/1.1 200 OK
# ...
# {"status":"ok"}
```

浏览器打开 <http://127.0.0.1:8000/docs> 能看到 Swagger 文档。

详细篇：[应用入口、路由与项目分层](../fastapi/FastAPI基础篇001_应用入口、路由与项目分层.md)、[健康检查](../fastapi/FastAPI基础篇004_健康检查.md)

## 4. 阶段三：数据库

### 4.1. 步骤 6：Docker 起 PostgreSQL、数据库引擎与会话、ORM 基类

**目的**：本地用 Docker 运行 PostgreSQL；应用启动时创建引擎（连接池），每个请求一个会话；定义所有模型共用的基类；`/readyz` 真正检查数据库。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `docker-compose.yml` | `postgres:17-alpine`，数据卷持久化，`pg_isready` 健康检查 |
| `src/my_service/db/session.py` | `create_engine()`、`create_session_factory()` |
| `src/my_service/models/base.py` | `Base`（带约束命名规则）、`TimestampMixin` |
| `src/my_service/models/__init__.py` | 汇总导入所有模型，供 Alembic 发现 |
| `src/my_service/api/deps.py` | `DbSession` 依赖：每个请求一个会话 |
| `main.py` / `api/health.py` | `lifespan` 中创建和释放引擎；`/readyz` 执行 `SELECT 1`，超时 2 秒 |

compose 里的健康检查要加 `-h 127.0.0.1`：

```yaml
healthcheck:
  # 不加 -h 时检查走 socket，容器首次初始化期间会过早显示 healthy
  test: ["CMD-SHELL", "pg_isready -h 127.0.0.1 -U postgres -d my_service"]
  interval: 5s
  timeout: 3s
  retries: 10
```

**验证**：

```shell
docker compose up -d
docker compose ps                    # 预期：STATUS 显示 (healthy)

uv run fastapi dev
curl -s -o /dev/null -w "%{http_code}\n" localhost:8000/readyz
# 200
```

数据库停掉后再请求 `/readyz`，预期返回 503，服务日志中出现 `readiness_check_failed`。

详细篇：[异步引擎、会话与模型定义](../sqlalchemy/SQLAlchemy基础篇001_异步引擎、会话与模型定义.md)、[健康检查](../fastapi/FastAPI基础篇004_健康检查.md)

### 4.2. 步骤 7：Alembic 迁移

**目的**：表结构的每次变化都用迁移脚本记录。迁移读取应用同一份配置，不在 `alembic.ini` 里重复写数据库地址。

**产出文件**：

| 文件 | 改动 |
|------|------|
| `alembic.ini` | 设置 `file_template`（迁移文件名以时间开头）；注释掉 `sqlalchemy.url`；配置 `post_write_hooks` 在生成后自动 `ruff format` |
| `migrations/env.py` | 数据库地址取自 `get_settings()`，`target_metadata = Base.metadata`，迁移用 `NullPool` |
| `migrations/versions/<日期>_<时间>_<rev>_init.py` | 第一个迁移，此时还没有模型，`upgrade()` 里只有 `pass` |

`alembic.ini` 中的关键配置：

```ini
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d_%%(rev)s_%%(slug)s

# sqlalchemy.url = driver://user:pass@localhost/dbname

[post_write_hooks]
hooks = ruff_format
ruff_format.type = module
ruff_format.module = ruff
ruff_format.options = format REVISION_SCRIPT_FILENAME
```

**验证**：

```shell
uv run alembic revision --autogenerate -m "init"   # 预期：生成 migrations/versions/<时间>_<rev>_init.py
uv run alembic upgrade head
uv run alembic current                             # 预期：显示最新 revision，带 (head)
uv run alembic check                               # 预期：模型与数据库一致，没有待生成的迁移
```

详细篇：[Alembic 数据库迁移](../sqlalchemy/SQLAlchemy基础篇002_Alembic数据库迁移.md)

## 5. 阶段四：请求链路与业务

### 5.1. 步骤 8：统一异常与 request-id 中间件

**目的**：所有错误（业务异常、参数校验失败、路由不存在、未处理异常）返回同一种 JSON 结构；每个请求分配 `request_id`，写入响应头和该请求的所有日志。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `src/my_service/core/exceptions.py` | `ErrorResponse`、`AppError` 及其子类、`register_exception_handlers()` |
| `src/my_service/core/middleware.py` | 纯 ASGI 写法的 `RequestContextMiddleware`：分配 request_id、访问日志、兜底 500 |
| `src/my_service/core/logging.py` | 关闭 uvicorn 自带的访问日志，改由中间件输出 |
| `src/my_service/main.py` | 注册异常处理器和中间件 |

**验证**：

```shell
curl -i localhost:8000/nope
# HTTP/1.1 404 Not Found
# x-request-id: 7b578c08cb064b61ac6bceb4b4577534
# ...
# {"code":"not_found","message":"Not Found","request_id":"7b578c08cb064b61ac6bceb4b4577534","details":null}

curl -i -H "X-Request-ID: abc-123" localhost:8000/healthz
# 响应头中 x-request-id: abc-123     ← 沿用上游传来的值
```

服务日志里同时出现一条 `http_request`，带 `method`、`path`、`status`、`duration_ms` 和同一个 `request_id`。`/healthz`、`/readyz` 不记访问日志。

详细篇：[统一异常处理与请求追踪中间件](../fastapi/FastAPI基础篇003_统一异常处理与请求追踪中间件.md)

### 5.2. 步骤 9：用户表与 create-user 命令

**目的**：建用户表；示例服务不开放注册接口，由管理员在命令行创建用户。这一步同时建立分层结构：`api → services → repositories → models`。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `models/user.py`、`models/refresh_token.py` | ORM 模型，并在 `models/__init__.py` 中导入 |
| `migrations/versions/..._add_users_and_refresh_tokens.py` | 自动生成后人工检查的迁移 |
| `repositories/user_repository.py` | 只执行 SQL，不 `commit` |
| `schemas/user.py` | `UserCreate`（用户名 3 到 50 位，只允许字母、数字和 `_ . -`；密码至少 8 位）、`UserResponse` |
| `services/user_service.py` | 用户名重复时抛 `ConflictError("用户名已存在")` |
| `src/my_service/cli.py` | `my-service create-user <用户名>`，用 `getpass` 输入两次密码 |
| `pyproject.toml` | `[project.scripts] my-service = "my_service.cli:main"` |

**验证**：

```shell
uv run alembic revision --autogenerate -m "add users and refresh tokens"
# 打开新生成的迁移文件人工检查后再执行
uv run alembic upgrade head

uv sync                                # 修改 [project.scripts] 后需要重新 sync
uv run my-service create-user Alice
# 密码：
# 确认密码：
# 已创建用户：Alice（id=...）

uv run my-service create-user Alice    # 再执行一次，预期：用户名已存在
```

详细篇：[Python 虚拟环境和包](../python/基础篇010_Python虚拟环境和包.md)（`[project.scripts]` 一节）、[Alembic 数据库迁移](../sqlalchemy/SQLAlchemy基础篇002_Alembic数据库迁移.md)

### 5.3. 步骤 10：JWT 登录、刷新、注销

**目的**：短期 access token（JWT，15 分钟）+ 长期 refresh token（随机串，7 天，库里只存 SHA-256 哈希）。刷新时轮换，旧 refresh token 被复用时作废整次登录。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `core/security.py` | Argon2id 密码哈希、JWT 签发与校验、refresh token 生成 |
| `services/auth_service.py` | 登录、刷新（轮换 + 复用检测）、注销、校验 access token |
| `repositories/refresh_token_repository.py`、`schemas/auth.py` | refresh token 的读写、请求与响应模型 |
| `api/v1/auth.py` | `/login`、`/refresh`、`/logout`、`/me` |
| `api/deps.py` | `AuthServiceDep`、`CurrentUser` |
| `core/config.py`、`.env.example` | `JWT_SECRET_KEY`、`ACCESS_TOKEN_EXPIRE_MINUTES`、`REFRESH_TOKEN_EXPIRE_DAYS` |
| `pyproject.toml` | 新增依赖 `pyjwt`、`argon2-cffi` |

**验证**：用第 9 章的 curl 脚本把登录、刷新、复用检测、注销完整走一遍，预期结果：

| 检查项 | 预期状态码 |
|--------|-----------|
| 用户名大小写不对 | 401 |
| 正确登录 | 200，返回 `access_token`、`refresh_token` 及过期时间 |
| 带 access token 访问 `/me` | 200 |
| 不带令牌访问 `/me` | 401 |
| 用旧的 refresh token 再次刷新（模拟被盗用） | 401 |
| 刷新得到的新 refresh token 也被连带作废 | 401 |
| 注销 | 204 |
| 注销后用 refresh token 刷新 | 401 |
| 注销后用 access token 访问 `/me` | **200**：access token 无法撤回，过期前仍可用 |

详细篇：[依赖注入与 Bearer 认证](../fastapi/FastAPI基础篇002_依赖注入与Bearer认证.md)

## 6. 阶段五：质量与交付

### 6.1. 步骤 11：pytest 测试

**目的**：用真实 PostgreSQL 跑测试。测试库独立（开发库名加 `_test`），会话开始时重建并通过 Alembic 迁移建表，每个测试后清空所有表。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `tests/conftest.py` | 覆盖 `APP_ENV`、`DATABASE_URL`、`JWT_SECRET_KEY`；fixture：`app`、`client`、`db_session`、`user` |
| `tests/test_security.py`、`tests/test_health.py`、`tests/test_auth.py` | 测试用例 |
| `pyproject.toml` | 测试目录忽略 `S105`、`S106`（允许写死测试密码）；`[tool.coverage.run]` 增加 `concurrency` |

**验证**：

```shell
docker compose up -d
uv run pytest --collect-only -q    # 预期：N tests collected，没有收集错误
uv run pytest                      # 预期：全部 passed
uv run pytest --cov --cov-report=term-missing
```

详细篇：[pytest 异步测试实践](./Python工程化篇004_pytest异步测试实践.md)

### 6.2. 步骤 12：pre-commit

**目的**：每次 `git commit` 前自动检查行尾空格、文件末尾换行、YAML/TOML 语法、私钥，以及 ruff 和 mypy，不通过就不能提交。

**产出文件**：`.pre-commit-config.yaml`；`pyproject.toml` 的 `[tool.ruff]` 增加 `force-exclude = true`（pre-commit 会把文件路径显式传给 ruff，不加这项时 `extend-exclude` 对显式传入的文件不生效）。

**验证**：

```shell
uv run pre-commit install           # 预期：pre-commit installed at .git/hooks/pre-commit
uv run pre-commit run --all-files   # 预期：每个钩子都显示 Passed
```

第一次对全部文件运行时，`end-of-file-fixer` 常会修复一批末尾缺少换行的文件（如 `.env.example`、`.gitignore`、Alembic 模板生成的 `migrations/README`），这些修复应包含在这一步的提交里。被钩子自动修改的文件需要重新 `git add` 后再提交。

详细篇：[pre-commit 提交前自动检查](./Python工程化篇003_pre-commit提交前自动检查.md)

### 6.3. 步骤 13：Dockerfile 与 compose

**目的**：多阶段构建镜像，以非 root 用户运行；compose 增加 `migrate`（执行迁移后退出）和 `app` 两个服务，放在 `app` profile 下，日常 `docker compose up -d` 仍只启动数据库。

**产出文件**：

| 文件 | 说明 |
|------|------|
| `Dockerfile` | 构建阶段 `uv sync --locked --no-dev`，依赖层单独缓存；运行阶段只复制 `.venv`、`alembic.ini`、`migrations/` |
| `.dockerignore` | 排除 `.git`、`.venv`、`.env`、缓存和 `tests` |
| `docker-compose.yml` | `migrate` 依赖数据库 healthy；`app` 依赖 `migrate` 成功完成，映射到宿主机 8080 |

**验证**：

```shell
docker compose --profile app up -d --build
docker compose --profile app ps -a
# migrate 显示 Exited (0)，app 显示 (healthy)

curl -s -o /dev/null -w "%{http_code}\n" localhost:8080/healthz   # 200
curl -s -o /dev/null -w "%{http_code}\n" localhost:8080/docs      # 404：APP_ENV=prod 时关闭接口文档

docker compose exec app my-service create-user <用户名>          # 在容器中创建用户
```

详细篇：[Docker 实战：Python 服务容器化与部署](../../DevOps/Docker/Docker实战_Python服务容器化与部署.md)

## 7. pyproject.toml 集中配置各工具

项目的依赖、命令行入口和所有开发工具的配置都写在 `pyproject.toml` 里，不再单独维护 `setup.cfg`、`.flake8`、`mypy.ini`、`pytest.ini` 等文件。

### 7.1. 各节的作用

| 配置节 | 作用 |
|--------|------|
| `[project]` | 项目名、版本、Python 版本要求、运行依赖 |
| `[project.scripts]` | 定义命令行入口：`my-service = "my_service.cli:main"` |
| `[build-system]` | 用 `uv_build` 构建，项目本身会被安装进虚拟环境 |
| `[tool.fastapi]` | `entrypoint = "my_service.main:app"`，`fastapi dev` / `fastapi run` 据此找到应用，无需在命令里写路径 |
| `[dependency-groups]` | 开发依赖（`dev` 组），部署时用 `--no-dev` 排除 |
| `[tool.ruff]` | 行宽、目标版本、源码目录、排除目录 |
| `[tool.ruff.lint]` | 启用哪些规则、忽略哪些规则 |
| `[tool.ruff.lint.per-file-ignores]` | 针对部分文件放宽规则（如测试） |
| `[tool.mypy]` | 类型检查的严格程度和插件 |
| `[tool.pytest.ini_options]` | 测试目录、异步模式、默认参数 |
| `[tool.coverage.run]` | 覆盖率统计范围和方式 |

### 7.2. 完整配置示例

```toml
[project.scripts]
my-service = "my_service.cli:main"

[tool.fastapi]
entrypoint = "my_service.main:app"

# ---------- Ruff：lint + 格式化 ----------
[tool.ruff]
line-length = 100
target-version = "py312"
src = ["src", "tests"]
extend-exclude = ["migrations/versions"]   # 自动生成的迁移文件不检查
force-exclude = true   # 即使显式传入文件路径（如 pre-commit），也遵守上面的排除规则

[tool.ruff.lint]
select = [
    "E", "W",   # pycodestyle 基础风格
    "F",        # pyflakes：未使用的变量、导入等
    "I",        # import 排序
    "B",        # bugbear：常见 bug 写法
    "UP",       # 使用新版 Python 语法
    "N",        # 命名规范
    "S",        # bandit：安全问题
    "ASYNC",    # 异步代码中的阻塞调用
    "SIM",      # 可简化的写法
    "RUF",      # ruff 自带规则
]
ignore = [
    "RUF001","RUF002","RUF003" # 允许中文全角标点出现在字符串、文档字符、注释中
]

[tool.ruff.lint.per-file-ignores]
"tests/**/*.py" = [
    "S101",            # 测试里允许使用 assert
    "S105", "S106",    # 测试里允许写死测试用的密码和密钥
]

# ---------- Mypy：静态类型检查 ----------
[tool.mypy]
python_version = "3.12"
strict = true
plugins = ["pydantic.mypy"]   # 让 mypy 理解 Pydantic 模型
mypy_path = "src"

# ---------- Pytest ----------
[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"                          # async def 测试函数无需逐个加装饰器
asyncio_default_fixture_loop_scope = "function"
addopts = "-ra --strict-markers"

# ---------- 覆盖率 ----------
[tool.coverage.run]
source = ["src/my_service"]
branch = true
concurrency = ["greenlet", "thread"]
```

几个容易忽略的点：

* `RUF001`~`RUF003` 会把中文全角标点（如 `，`、`：`）报成"易混淆字符"。中文注释多的项目建议直接忽略。
* `[tool.fastapi]` 只在有 `pyproject.toml` 的目录下生效。Docker 运行阶段如果只复制 `.venv`、`alembic.ini`、`migrations/`，就没有这个文件，所以 `Dockerfile` 的 `CMD` 里要显式写 `--entrypoint my_service.main:app`。
* `addopts = "--strict-markers"`：使用未注册的 `@pytest.mark.xxx` 时直接报错，拼错的标记名不会只留下一条容易被忽略的警告。

## 8. .env 与 .env.example 的分工

### 8.1. 两个文件各管什么

| 文件 | 是否提交 | 内容 |
|------|---------|------|
| `.env.example` | 提交 | 所有变量名、本地开发可用的示例值；密钥类变量留空，并注明生成方式 |
| `.env` | **不提交** | 真实值，被 `.gitignore` 和 `.dockerignore` 同时排除 |

`.env.example` 示例：

```shell
APP_ENV=dev
APP_DEBUG=true
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/my_service

LOG_LEVEL=INFO
DATABASE_POOL_SIZE=10

# 生成方式：uv run python -c 'import secrets; print(secrets.token_urlsafe(48))'
JWT_SECRET_KEY=
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7
```

新成员的使用方式：

```shell
cp .env.example .env
uv run python -c 'import secrets; print(secrets.token_urlsafe(48))'
# 把输出填到 .env 的 JWT_SECRET_KEY= 后面
```

`JWT_SECRET_KEY` 留空有两个好处：模板里不会出现一个"能用"的密钥被所有人复制；忘记填写时，`Field(min_length=32)` 会让应用直接启动失败，而不是带着弱密钥运行。

`.dockerignore` 中同样排除了 `.env`，这样即使 `Dockerfile` 里有 `COPY . /app`，密钥也不会被打进镜像：

```gitignore
.env
.env.*
!.env.example
```

### 8.2. 坑：修改 .env 后不生效

现象：修改 `.env` 后重启应用，读到的配置仍是旧值。

原因：pydantic-settings 的读取优先级是：**系统环境变量 > `.env` 文件 > 默认值**。部分 IDE 的集成终端会把项目里的 `.env` 自动注入为环境变量，新打开的 IDE 终端里就已经存在 `DATABASE_URL`、`DATABASE_POOL_SIZE` 等变量。这时再修改 `.env`，应用读到的仍然是终端里那份旧的环境变量。

排查和解决：

```shell
echo $DATABASE_URL           # 有输出，说明当前终端已存在同名环境变量
env | grep DATABASE          # 查看所有相关变量

unset DATABASE_URL DATABASE_POOL_SIZE   # 方式一：清除后应用会改读 .env
# 方式二：新开一个终端（如果是 IDE 注入的，需要先关掉 IDE 的这项功能）
```

测试也利用了这条优先级：`tests/conftest.py` 在创建应用之前写 `os.environ["DATABASE_URL"] = ...`，覆盖 `.env` 里的开发库地址，再调用 `get_settings.cache_clear()` 清掉已缓存的配置。

详细篇：[配置管理 pydantic-settings](./Python工程化篇005_配置管理pydantic-settings.md)

## 9. 调试常用命令

### 9.1. curl

| 命令 | 用途 |
|------|------|
| `curl -i URL` | 同时显示响应头和响应体，用来看 `X-Request-ID`、状态码 |
| `curl -s -o /dev/null -w "%{http_code}\n" URL` | 只输出状态码，适合批量检查 |
| `curl -s -w " %{http_code}\n" URL` | 输出响应体，末尾追加状态码 |
| `curl -s -X POST URL -H 'Content-Type: application/json' -d '{...}'` | 发送 JSON 请求 |

`-s` 关闭进度条，`-o /dev/null` 丢弃响应体，`-w` 按格式输出请求信息，`%{http_code}` 是状态码。

### 9.2. 在 shell 里解析 JSON

不想额外安装 `jq` 时，可以用 Python 自带的 `json` 模块定义一个小函数：

```shell
jget() { python3 -c "import sys,json;print(json.load(sys.stdin)['$1'])"; }

echo '{"access_token":"abc","token_type":"bearer"}' | jget access_token
# abc
```

### 9.3. 完整示例：验证认证流程

下面这段脚本（zsh）可以把步骤 10 的认证流程完整验证一遍：

```shell
U='Alice'; PW='<密码>'
B=localhost:8000/api/v1/auth; J='Content-Type: application/json'
jget() { python3 -c "import sys,json;print(json.load(sys.stdin)['$1'])"; }
login() { curl -s -X POST $B/login -H "$J" -d "{\"username\":\"$U\",\"password\":\"$PW\"}"; }

echo "1. 用户名大小写不对："
curl -s -o /dev/null -w "%{http_code}\n" -X POST $B/login -H "$J" \
  -d "{\"username\":\"${U:l}\",\"password\":\"$PW\"}"

echo "2. 登录："; R1=$(login); echo $R1
echo $R1 | grep -q access_token || { echo ">>> 登录失败，请检查 U 和 PW"; return 2>/dev/null || exit; }
A1=$(echo $R1 | jget access_token); F1=$(echo $R1 | jget refresh_token)

echo "3. 带 access token 访问 /me："; curl -s -w " %{http_code}\n" $B/me -H "Authorization: Bearer $A1"
echo "4. 不带令牌访问 /me："; curl -s -o /dev/null -w "%{http_code}\n" $B/me

echo "5. 刷新："
R2=$(curl -s -X POST $B/refresh -H "$J" -d "{\"refresh_token\":\"$F1\"}"); F2=$(echo $R2 | jget refresh_token)

echo "6. 再次使用旧的 F1（模拟被盗用）："
curl -s -o /dev/null -w "%{http_code}\n" -X POST $B/refresh -H "$J" -d "{\"refresh_token\":\"$F1\"}"
echo "7. 新的 F2 也被连带作废："
curl -s -o /dev/null -w "%{http_code}\n" -X POST $B/refresh -H "$J" -d "{\"refresh_token\":\"$F2\"}"

echo "8. 重新登录后注销："
R3=$(login); A3=$(echo $R3 | jget access_token); F3=$(echo $R3 | jget refresh_token)
curl -s -o /dev/null -w "   注销：%{http_code}\n" -X POST $B/logout -H "$J" -d "{\"refresh_token\":\"$F3\"}"
curl -s -o /dev/null -w "   注销后刷新：%{http_code}\n" -X POST $B/refresh -H "$J" -d "{\"refresh_token\":\"$F3\"}"
curl -s -o /dev/null -w "   注销后用 access token：%{http_code}\n" $B/me -H "Authorization: Bearer $A3"
```

脚本里用到的几个 shell 技巧：

| 写法 | 作用 |
|------|------|
| `${U:l}` | zsh 的参数修饰符，把变量转成小写（`Alice` → `alice`），用来验证用户名区分大小写；`${U:u}` 转大写 |
| `R1=$(login)` | 把命令输出保存到变量，后面多次使用 |
| `grep -q ... \|\| { ...; }` | 登录失败时提前退出，避免后面的步骤拿空令牌产生误导性的结果 |

### 9.4. 端口占用与残留进程

```shell
lsof -nP -i :8000
# COMMAND   PID  USER   FD   TYPE ... NAME
# Python  12345 alice  ...  TCP 127.0.0.1:8000 (LISTEN)
```

`-n` 不把 IP 解析成主机名，`-P` 不把端口号解析成服务名，输出更快也更直观。

`fastapi dev` 实际由两个进程组成：

```mermaid
flowchart LR
    R[reloader 进程<br/>监听文件变化] -->|代码改动时重启| W[worker 进程<br/>监听 8000 端口、处理请求]
```

用 `kill -INT` 结束进程后，worker 可能没有跟着退出，继续占着 8000 端口，**用的还是启动时的旧配置**。表现是：改了 `.env` 或代码，重启服务，请求结果却完全没变；或者新启动的服务报 `address already in use`。

```shell
lsof -nP -i :8000      # 找到仍在监听的 PID
kill <PID>             # 结束残留的 worker
lsof -nP -i :8000      # 再查一次，没有输出说明端口已释放
```

## 10. git 不跟踪空目录

git 只跟踪文件，不跟踪目录。一个空的 `tests/` 目录执行 `git add` 后不会出现在提交里，别人克隆下来也没有这个目录。

常见做法是在 `tests/` 下放一个空的 `__init__.py`，第一次提交时就带上：

```shell
touch tests/__init__.py
git add tests/__init__.py
git ls-files tests            # 能看到 tests/__init__.py
```

这个 `__init__.py` 还有实际用途：它让 `tests` 成为一个包，测试文件可以写 `from tests.conftest import TEST_PASSWORD, TEST_USERNAME`。

> 其他语言或非 Python 目录常用 `.gitkeep` 占位，这只是社区约定，git 本身对这个文件名没有特殊处理。

## 11. 分步教程的写法经验

按"每步给出说明，再由读者照着写入文件"的方式搭建项目时，下面这些错误很常见：

| 错误 | 后果 | 典型表现 |
|------|------|---------|
| 整步跳过 | 后续步骤依赖的模块不存在，报错位置和真正原因隔了好几步 | 后面某一步突然报错，要逐步核对产出文件才能找到原因 |
| 说明写的是"新增一个函数"，只把这个函数片段写进了文件 | 文件原有内容全部丢失 | import 报错、功能消失 |
| 模块文件放在包目录之外，如 `cli.py` 放在项目根目录，没有放进 `src/my_service/` | 包里没有这个模块 | `ModuleNotFoundError: No module named 'my_service.cli'` |
| 固定文件名拼错，如 `tests/conftest.py` 写成 `configtest.py` | pytest 不会自动加载它，fixture 和测试环境配置都不生效 | `ModuleNotFoundError: No module named 'tests.conftest'` |
| 提交里带行尾空格 | diff 里出现无意义的改动 | 引入 pre-commit 后被 `trailing-whitespace` 自动清理 |

把 `conftest.py` 拼错后，测试文件里的 `from tests.conftest import ...` 会在收集阶段报错：

```
ERROR collecting tests/test_auth.py
tests/test_auth.py:7: in <module>
    from tests.conftest import TEST_PASSWORD, TEST_USERNAME
E   ModuleNotFoundError: No module named 'tests.conftest'
```

预防办法：

1. **给完整文件，而不是"在某处插入几行"。** 完整文件可以直接覆盖，读者不需要判断插入位置，也不会误删原有内容。只改一两行时，至少给出改动前后的上下文。
2. **每个文件都写明完整路径**，如 `src/my_service/cli.py`、`tests/conftest.py`。路径和文件名写错是最常见、也最难从报错里看出来的问题。
3. **每一步都附带验证命令和预期结果**，照着操作的人能当场发现自己漏了或做错了，不会把问题带到后面。
4. **尽早引入 pre-commit 兜底**。行尾空格、文件末尾换行、import 顺序、格式、类型错误这类问题交给工具自动修复或拦截，不依赖人工检查。pre-commit 也可以提前到步骤 2，和 ruff、mypy 一起配置。

## 12. 小结

* 初始化顺序是**工程骨架 → 配置与日志 → 应用入口 → 数据库与迁移 → 统一异常 → 业务 → 测试 → 提交检查 → 容器化**，先做后面所有模块都依赖的部分。
* 每一步都有一条验证命令和明确的预期结果，验证通过再提交，出问题时只需排查最近一步。
* 依赖和所有工具配置集中在 `pyproject.toml`：`[tool.ruff]`、`[tool.mypy]`、`[tool.pytest.ini_options]`、`[tool.coverage.run]`、`[tool.fastapi]`、`[project.scripts]` 各管一件事。
* `.env.example` 提交，只写变量名、示例值和密钥的生成方式；`.env` 存真实值，同时被 `.gitignore` 和 `.dockerignore` 排除。系统环境变量优先于 `.env`，改了 `.env` 不生效时先 `echo $变量名`。
* git 不跟踪空目录，`tests/` 要放 `__init__.py` 才能提交。
* 调试时常用 `curl -i`、`curl -s -o /dev/null -w "%{http_code}\n"`、`lsof -nP -i :8000`；`fastapi dev` 有 reloader 和 worker 两个进程，结束后要确认端口已释放。
* 写分步教程时给完整文件和完整路径，每步附验证命令，再用 pre-commit 自动兜底。
