## 1. 为什么要集中管理配置

一个服务总有一批随环境变化的参数：数据库地址、密钥、日志级别、连接池大小等。最直接的写法是在用到的地方随手 `os.getenv`：

```python
import os

DEBUG = os.getenv("APP_DEBUG")
POOL_SIZE = int(os.getenv("DATABASE_POOL_SIZE", "10"))
DATABASE_URL = os.getenv("DATABASE_URL")
```

这种写法项目一大就会出问题：

| 问题 | 说明 |
|------|------|
| 拼错变量名运行时才发现 | `os.getenv("APP_DEBG")` 不会报错，只会安静地返回 `None`，直到某个分支执行到才暴露 |
| 读到的全是字符串 | `APP_DEBUG=false` 读出来是字符串 `"false"`，`bool("false")` 为 `True`，"关掉"调试反而打开了 |
| 缺少必填项照样启动 | 没配 `DATABASE_URL` 服务也能起来，第一次访问数据库才报错，问题从部署阶段拖到了线上请求 |
| 散落在各处 | 有哪些配置、默认值是什么、哪些是必填，只能全局搜索 `getenv` 才知道 |

前两个问题可以直接验证：

```shell
$ APP_DEBUG=false python3 -c "import os; print(bool(os.getenv('APP_DEBUG')), repr(os.getenv('APP_DEBG')))"
True None
```

目标是：**所有配置集中定义在一个类里，带类型和约束；服务启动时一次性读取并校验，有任何不合法的配置直接启动失败。** `pydantic-settings` 就是做这件事的库，它是 Pydantic 官方拆出来的配置管理包。

```shell
uv add pydantic-settings
```

## 2. BaseSettings 基本用法

### 2.1. 定义配置类

继承 `BaseSettings`，像写 Pydantic 模型一样声明字段。例如 `src/my_service/core/config.py`（简化版）：

```python
from typing import Literal

from pydantic import Field, SecretStr
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8", extra="ignore")

    # 应用
    app_name: str = "my-service"
    app_env: Literal["dev", "test", "prod"] = "dev"
    app_debug: bool = False

    # 数据库
    database_url: SecretStr
    database_pool_size: int = Field(default=10, ge=1)
```

`SettingsConfigDict` 中各项的含义：

| 配置项 | 含义 |
|--------|------|
| `env_file=".env"` | 除了系统环境变量，还从 `.env` 文件读取。路径相对于**当前工作目录**，所以要在项目根目录下启动 |
| `env_file_encoding="utf-8"` | `.env` 的文件编码，注释里写中文时需要 |
| `extra="ignore"` | `.env` 里出现类中没有声明的变量时忽略。`BaseSettings` 默认是 `forbid`，会报 `Extra inputs are not permitted` |

`extra="ignore"` 的实际意义：`.env` 往往会被 docker-compose 等其他工具共用，里面可能有不属于本应用的变量，不应该因此启动失败。

### 2.2. 字段名与环境变量的对应

字段名就是环境变量名，**默认不区分大小写**：

| 字段 | 对应的环境变量 |
|------|---------------|
| `app_env` | `APP_ENV`（写成 `app_env` 也能识别） |
| `database_url` | `DATABASE_URL` |
| `database_pool_size` | `DATABASE_POOL_SIZE` |

环境变量的值都是字符串，pydantic-settings 会按字段类型转换：`"false"` 转成 `False`，`"10"` 转成 `10`；转换不了的值直接报错，例如 `APP_DEBUG=maybe`：

```
app_debug
  Input should be a valid boolean, unable to interpret input [type=bool_parsing, input_value='maybe', input_type=str]
```

### 2.3. 读取优先级

同一个配置可能同时出现在多个地方，优先级从高到低是：

```mermaid
flowchart LR
    A[系统环境变量] -->|没有时| B[.env 文件] -->|没有时| C[字段默认值]
```

在一个临时目录里放一份 `.env`（`DATABASE_POOL_SIZE=5`、`DATABASE_URL` 主机名为 `dotenv`），再分别用系统环境变量覆盖，结果如下：

| 场景 | 读到的 `database_url` 主机 | `database_pool_size` |
|------|---------------------------|---------------------|
| 只有 `.env` | `dotenv` | `5`（来自 `.env`） |
| 额外导出 `DATABASE_URL=...@sysenv...`、`DATABASE_POOL_SIZE=7` | `sysenv` | `7`（系统环境变量胜出） |
| `.env` 和环境变量都不设 `DATABASE_POOL_SIZE` | — | `10`（默认值） |

这个优先级带来两个常见用法：

* **生产环境不放 `.env`**：配置和密钥由容器（`docker run -e`、compose 的 `environment`）或 K8s（ConfigMap / Secret）注入为环境变量，镜像里不带任何密钥。`.env` 只是本地开发的便利。
* **测试里覆盖配置**：测试代码直接改 `os.environ`，就能盖过开发者本地 `.env` 里的值（见第 7.2 节）。

## 3. 用类型做校验

配置类的类型注解本身就是校验规则。常用的几种写法：

| 写法 | 作用 | 示例 |
|------|------|------|
| `Literal[...]` | 限定只能取几个固定值 | `app_env: Literal["dev", "test", "prod"] = "dev"` |
| 不写默认值 | 必填，缺了启动失败 | `database_url: SecretStr` |
| `Field(ge=..., le=...)` | 约束数值范围 | `database_pool_size: int = Field(default=10, ge=1)` |
| `Field(min_length=...)` | 约束长度 | `jwt_secret_key: SecretStr = Field(min_length=32)` |

完整的配置类字段：

```python
class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8", extra="ignore")

    # 应用
    app_name: str = "my-service"
    app_env: Literal["dev", "test", "prod"] = "dev"
    app_debug: bool = False

    # 日志
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"

    # 数据库
    database_url: SecretStr
    database_pool_size: int = Field(default=10, ge=1)
    database_max_overflow: int = Field(default=20, ge=0)

    # 认证
    jwt_secret_key: SecretStr = Field(min_length=32)
    access_token_expire_minutes: int = Field(default=15, ge=1)
    refresh_token_expire_days: int = Field(default=7, ge=1)
```

### 3.1. 校验失败时的报错

不合法的配置会在 `Settings()` 实例化时抛出 `ValidationError`，错误信息会指出具体是哪个字段、为什么不合法：

| 错误配置 | 报错信息 |
|----------|----------|
| `APP_ENV=staging` | `Input should be 'dev', 'test' or 'prod'` |
| 没有设置 `DATABASE_URL` | `database_url Field required` |
| `DATABASE_POOL_SIZE=0` | `Input should be greater than or equal to 1` |

完整的报错大致是这样：

```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Settings
database_pool_size
  Input should be greater than or equal to 1 [type=greater_than_equal, input_value='0', input_type=str]
```

一次实例化会把**所有**不合法的字段一起列出来，不用改一个、启动一次、再发现下一个。

## 4. SecretStr：保护敏感配置

### 4.1. 打印时自动打码

数据库连接串里有密码，JWT 密钥泄露后任何人都能伪造令牌。这类字段用 `SecretStr` 包装后，打印配置、写日志、出现在异常信息里时都只显示星号：

```python
>>> print(Settings())
app_name='my-service' app_env='dev' app_debug=True log_level='INFO'
database_url=SecretStr('**********') database_pool_size=5 database_max_overflow=20
jwt_secret_key=SecretStr('**********') access_token_expire_minutes=15 refresh_token_expire_days=7
```

真正需要明文的地方显式调用 `.get_secret_value()`：

```python
# src/my_service/db/session.py
def create_engine(settings: Settings) -> AsyncEngine:
    return create_async_engine(
        settings.database_url.get_secret_value(),
        pool_size=settings.database_pool_size,
        max_overflow=settings.database_max_overflow,
        ...
    )
```

`.get_secret_value()` 调用点就是明文出现的全部位置，代码审查时搜这个方法名即可。

### 4.2. 对 SecretStr 约束长度

`Field(min_length=32)` 对 `SecretStr` 同样生效，校验的是明文的长度。设置 `JWT_SECRET_KEY=short` 时：

```
jwt_secret_key
  Value should have at least 32 items after validation, not 5
```

这样能在启动时拦住"随手写了个 `secret`"之类的弱密钥。

## 5. field_validator：校验 SecretStr 的内容

`Literal`、`Field` 能表达的规则有限。像"数据库连接串必须是合法的 PostgreSQL URL"这种规则，用 `field_validator` 自己写：

```python
from pydantic import PostgresDsn, SecretStr, field_validator


class Settings(BaseSettings):
    database_url: SecretStr

    @field_validator("database_url")
    @classmethod
    def _check_database_url(cls, v: SecretStr) -> SecretStr:
        PostgresDsn(v.get_secret_value())
        return v
```

要点：

| 要点 | 说明 |
|------|------|
| 为什么不直接把类型写成 `PostgresDsn` | 那样打印时会露出密码；保留 `SecretStr` 类型，只借用 `PostgresDsn` 做校验 |
| 校验逻辑 | 先取明文，构造一次 `PostgresDsn(...)`，不合法时它会抛出校验错误，Pydantic 会把它当作这个字段的错误报告 |
| 返回值 | 原样返回 `v`，字段的值和类型都不变 |
| `@classmethod` | `field_validator` 要求写成类方法，装饰器顺序是 `field_validator` 在外 |

设置 `DATABASE_URL=mysql://x` 时：

```
URL scheme should be 'postgres', 'postgresql', 'postgresql+asyncpg', ...
```

`postgresql+asyncpg` 这种带驱动名的写法在允许列表里，所以 SQLAlchemy 异步引擎用的连接串可以直接通过校验。

## 6. 派生属性

有些值是由其他配置推导出来的，不需要单独配置。用 `@property` 定义即可：

```python
class Settings(BaseSettings):
    app_env: Literal["dev", "test", "prod"] = "dev"

    @property
    def is_prod(self) -> bool:
        return self.app_env == "prod"
```

使用时不再到处写 `settings.app_env == "prod"`：

```python
# src/my_service/main.py
app = FastAPI(
    title=settings.app_name,
    debug=settings.app_debug,
    # 生产环境不暴露接口文档
    docs_url=None if settings.is_prod else "/docs",
    redoc_url=None if settings.is_prod else "/redoc",
    openapi_url=None if settings.is_prod else "/openapi.json",
)
```

`@property` 不是字段，不会被当成环境变量去读取，也不会出现在打印结果里。

## 7. get_settings()：单例与测试覆盖

### 7.1. 用 lru_cache 做单例

每次 `Settings()` 都会重新读环境变量和 `.env` 并校验一遍。整个进程里只需要一份，用 `functools.lru_cache` 包一个函数：

```python
from functools import lru_cache


@lru_cache
def get_settings() -> Settings:
    return Settings()
```

* 第一次调用时创建并缓存，之后每次返回同一个对象。
* 各模块统一调用 `get_settings()`，不在模块顶层写 `settings = Settings()`。顶层实例化会在 `import` 时就读取配置，测试来不及覆盖。

### 7.2. 测试中覆盖配置

测试需要连独立的测试库、使用固定的密钥。做法是：先改 `os.environ`（系统环境变量优先级高于 `.env`），再调用 `get_settings.cache_clear()` 清掉缓存，下次调用就会按新的环境变量重新加载。`tests/conftest.py` 中的写法：

```python
import os

from sqlalchemy.engine import make_url

from my_service.core.config import Settings, get_settings

# ---------- 测试环境配置：必须在创建 app 之前完成 ----------
# 环境变量优先级高于 .env，这里覆盖的值会生效
_dev_url = make_url(Settings().database_url.get_secret_value())
TEST_DATABASE_URL = _dev_url.set(database=f"{_dev_url.database}_test")

os.environ["APP_ENV"] = "test"
os.environ["DATABASE_URL"] = TEST_DATABASE_URL.render_as_string(hide_password=False)
os.environ["JWT_SECRET_KEY"] = "test-secret-key-only-for-tests-0123456789"
get_settings.cache_clear()
```

* 测试库地址由开发库地址推导：库名加 `_test` 后缀，避免测试清表时误伤开发数据。
* 这段代码放在 `conftest.py` 顶层，在任何 fixture 创建 app 之前执行。
* 测试用的 `JWT_SECRET_KEY` 同样要满足 `min_length=32`。

### 7.3. 在 FastAPI 依赖中注入

路由层不直接调用 `get_settings()`，而是把它作为依赖注入，例如 `src/my_service/api/deps.py`：

```python
from typing import Annotated

from fastapi import Depends

from my_service.core.config import Settings, get_settings


def get_auth_service(
    session: DbSession, settings: Annotated[Settings, Depends(get_settings)]
) -> AuthService:
    return AuthService(session, settings)
```

这样写有两个好处：

| 好处 | 说明 |
|------|------|
| 依赖显式 | `AuthService` 通过参数拿到配置，不在内部偷偷读全局状态，单元测试时可以直接传入一个 `Settings` 实例 |
| 便于覆盖 | FastAPI 支持 `app.dependency_overrides[get_settings] = lambda: Settings(...)`，可以只为某个测试替换配置 |

`get_settings` 已经被 `lru_cache` 缓存，作为依赖每个请求调用一次也只是取缓存，没有额外开销。

## 8. .env.example 与常见坑

### 8.1. .env.example 的写法

`.env` 里有本地密码和密钥，必须加入 `.gitignore`。仓库里提交一份 `.env.example` 作为模板，列出所有配置项和示例值：

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

新同事的使用方式：

```shell
cp .env.example .env
uv run python -c 'import secrets; print(secrets.token_urlsafe(48))'   # 把输出填入 JWT_SECRET_KEY
```

写法上的约定：

* **密钥留空**，并在上方注释写明生成命令。不要放一个"看起来能用"的示例密钥，否则很容易被原样带到生产。
* 留空的 `JWT_SECRET_KEY=` 会被读成空字符串，触发 `min_length=32` 校验失败。忘记生成密钥时服务起不来，而不是带着弱密钥运行。
* 新增配置项时同步更新 `.env.example`，它就是配置项的文档。

### 8.2. 坑：改了 .env 却不生效

现象：在 `.env` 里改了 `DATABASE_URL`，重启服务后连的还是旧地址。

原因：系统环境变量优先级高于 `.env`。如果当前终端里已经存在 `DATABASE_URL`（例如 IDE 按项目配置给集成终端注入了环境变量，或者之前手动 `export` 过），`.env` 里的值就会被盖住。只要终端里有这个变量，改 `.env` 就没有效果。

排查和解决：

```shell
# 1. 看看当前终端里是否已经有这个变量
echo $DATABASE_URL
env | grep -E '^(APP_|DATABASE_|JWT_)'

# 2. 临时去掉它，让 .env 生效
unset DATABASE_URL

# 3. 或者只在单条命令里去掉
env -u DATABASE_URL uv run my-service ...
```

也可以新开一个不带注入变量的终端。如果是 IDE 注入的，去 IDE 的终端 / 运行配置里删掉对应的环境变量，否则每次新开集成终端都会再注入一次。

## 9. 小结

* 到处 `os.getenv` 的问题是：拼错名字不报错、值全是字符串、缺必填项也能启动。集中到一个 `BaseSettings` 类里，启动时一次性校验。
* 字段名对应同名环境变量，不区分大小写；读取优先级是 **系统环境变量 > `.env` > 默认值**。生产靠容器 / K8s 注入环境变量，测试靠改 `os.environ` 覆盖。
* 用类型表达规则：`Literal` 限定取值，不写默认值即必填，`Field(ge=..., min_length=...)` 约束范围和长度；复杂规则用 `field_validator`。
* 敏感配置用 `SecretStr`，打印时显示 `**********`，需要明文时显式 `.get_secret_value()`。
* `@lru_cache` 的 `get_settings()` 提供单例；测试中改环境变量后 `cache_clear()` 重新加载；FastAPI 里用 `Depends(get_settings)` 注入。
* 提交 `.env.example`，密钥留空并注明生成命令；改 `.env` 不生效时，先 `echo $变量名` 检查终端里是否已有同名环境变量。
