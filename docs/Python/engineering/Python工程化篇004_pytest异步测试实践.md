## 1. 测试相关依赖与配置

本文以一个 FastAPI + SQLAlchemy 异步 + PostgreSQL 的 Web 服务为例，说明异步 Web 服务的测试怎么组织：事件循环怎么配、测试数据库怎么隔离、怎么不启动服务就调用接口、怎么看覆盖率，以及怎么确认测试真的有效。

### 1.1. 依赖

测试用到的包都放在开发依赖组里，不会进入生产镜像：

```toml
[dependency-groups]
dev = [
    "httpx>=0.28.1",          # 测试客户端：AsyncClient + ASGITransport
    "pytest>=9.2.0",          # 测试框架
    "pytest-asyncio>=1.5.0",  # 让 pytest 能运行 async def 测试和 fixture
    "pytest-cov>=7.1.0",      # 覆盖率（基于 coverage.py）
    # ...
]
```

| 包 | 作用 |
|----|------|
| `pytest` | 发现、收集、运行测试，提供 fixture 机制 |
| `pytest-asyncio` | pytest 本身不支持 `async def` 测试，这个插件负责创建事件循环并运行协程 |
| `httpx` | 异步 HTTP 客户端，配合 `ASGITransport` 直接调用 ASGI 应用 |
| `pytest-cov` | 在 pytest 运行时统计代码覆盖率 |

### 1.2. pytest 配置

pytest 会读取 `pyproject.toml` 里的 `[tool.pytest.ini_options]`：

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"                          # async def 测试函数无需逐个加装饰器
asyncio_default_fixture_loop_scope = "function"
addopts = "-ra --strict-markers"
```

| 配置项 | 含义 |
|--------|------|
| `testpaths` | 只在 `tests/` 下找测试，直接执行 `uv run pytest` 即可 |
| `asyncio_mode` | pytest-asyncio 的工作模式，见 2.1 |
| `asyncio_default_fixture_loop_scope` | 异步 fixture 默认使用哪个范围的事件循环，见 2.2 |
| `addopts` | 每次运行都自动带上的命令行参数 |
| `-ra` | 结尾的 short test summary 里列出所有非通过的结果（失败、错误、跳过等） |
| `--strict-markers` | 使用未注册的标记（如把 `@pytest.mark.asyncio` 拼错）直接报错，而不是悄悄忽略 |

运行时，pytest 的输出头部会显示这些配置是否生效：

```
platform darwin -- Python 3.12.13, pytest-9.2.0, pluggy-1.7.0
rootdir: ~/projects/my-service
configfile: pyproject.toml
testpaths: tests
plugins: platformdirs-4.14.5, cov-7.1.0, anyio-4.16.0, asyncio-1.5.0
asyncio: mode=Mode.AUTO, debug=False, asyncio_default_fixture_loop_scope=function, asyncio_default_test_loop_scope=function
```

## 2. pytest-asyncio

### 2.1. asyncio_mode = "auto"

pytest-asyncio 有两种模式：

| 模式 | 行为 |
|------|------|
| `strict`（默认） | 只有加了 `@pytest.mark.asyncio` 的测试、用 `@pytest_asyncio.fixture` 定义的 fixture 才由插件处理 |
| `auto` | 所有 `async def` 测试和 `async def` fixture 都自动交给插件处理，不需要任何装饰器 |

strict 模式下忘记加标记的 `async def` 测试不会运行，会直接失败：

```python
async def test_plain():          # 没有加 @pytest.mark.asyncio
    assert False
```

```
async def functions are not natively supported.
You need to install a suitable plugin for your async framework, for example:
  - anyio
  - pytest-asyncio
  ...
```

项目里几乎所有测试和 fixture 都是异步的，用 `auto` 模式可以省掉大量重复的装饰器：

```python
# auto 模式：直接写 async def 即可
async def test_healthz(client: AsyncClient) -> None:
    resp = await client.get("/healthz")
    assert resp.status_code == 200


# strict 模式下要写成这样
@pytest.mark.asyncio
async def test_healthz(client: AsyncClient) -> None:
    ...
```

> `auto` 模式适合项目只用 asyncio 的情况。如果同一个项目里还要测试 trio 等其他异步框架，应使用 `strict` 模式，避免插件之间抢着处理同一个测试。

### 2.2. asyncio_default_fixture_loop_scope = "function"

pytest-asyncio 需要为异步 fixture 和异步测试提供一个事件循环。这个配置决定**异步 fixture 默认运行在哪个范围的事件循环里**：

| 取值 | 含义 |
|------|------|
| `function` | 每个测试函数一个新的事件循环，测试结束后关闭 |
| `class` / `module` / `package` / `session` | 同一个类 / 模块 / 包 / 整个测试会话共用一个事件循环 |

设为 `function` 的效果：

* 每个测试都在全新的事件循环里运行，上一个测试遗留的任务、连接不会影响下一个测试。
* 异步 fixture 和测试在同一个循环里，fixture 里创建的数据库连接池可以在测试里直接使用。

不设置这个选项时，pytest-asyncio 1.5 会给出弃用警告：

```
PytestDeprecationWarning: The configuration option "asyncio_default_fixture_loop_scope" is unset.
The event loop scope for asynchronous fixtures will default to the fixture scope. Future versions
of pytest-asyncio will default the loop scope for asynchronous fixtures to "function" scope. ...
```

所以显式写上 `function`，既消除警告，又和插件未来的默认值保持一致。输出头部的 `asyncio_default_test_loop_scope=function` 是测试函数本身的循环范围，默认就是 `function`。

> 默认循环范围是 `function` 时，`scope="session"` 的**异步** fixture 需要一个跨测试存活的事件循环，得单独为它指定循环范围（`@pytest_asyncio.fixture(scope="session", loop_scope="session")`），测试也要用同一个循环才能共享它创建的连接。项目回避了这个问题：会话级的建库 fixture 写成同步函数，见 5.3。

## 3. conftest.py 与 fixture

### 3.1. fixture 通过同名参数注入

fixture 是 pytest 提供"测试前准备、测试后清理"的机制。用 `@pytest.fixture` 定义一个函数，测试函数只要**声明一个同名参数**，pytest 就会先执行这个 fixture，把它的返回值（或 `yield` 出来的值）传进来：

```python
@pytest.fixture
async def client(app: FastAPI) -> AsyncGenerator[AsyncClient]:
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as c:
        yield c


# 参数名 client 和 fixture 名一致，pytest 自动注入
async def test_healthz(client: AsyncClient) -> None:
    resp = await client.get("/healthz")
    assert resp.json() == {"status": "ok"}
```

`yield` 把一个 fixture 分成两段：

| 位置 | 执行时机 |
|------|---------|
| `yield` 之前 | 测试开始前：创建资源 |
| `yield` 的值 | 注入给测试函数 |
| `yield` 之后 | 测试结束后（无论成功失败）：释放资源 |

常用参数：

| 参数 | 含义 |
|------|------|
| `scope="function"`（默认） | 每个测试执行一次 |
| `scope="session"` | 整个测试会话只执行一次 |
| `autouse=True` | 不需要测试声明参数，自动对范围内的所有测试生效 |

### 3.2. fixture 之间相互依赖

fixture 自己也可以声明参数来依赖其他 fixture。项目 `tests/conftest.py` 里的依赖关系如下：

```python
@pytest.fixture
async def app() -> AsyncGenerator[FastAPI]: ...

@pytest.fixture
async def client(app: FastAPI) -> AsyncGenerator[AsyncClient]: ...      # 依赖 app

@pytest.fixture
async def db_session(app: FastAPI) -> AsyncGenerator[AsyncSession]:    # 依赖 app
    async with app.state.session_factory() as session:
        yield session

@pytest.fixture
async def user(db_session: AsyncSession) -> User:                      # 依赖 db_session
    """一个可以登录的测试用户。"""
    return await UserService(db_session).create_user(
        UserCreate(username=TEST_USERNAME, password=TEST_PASSWORD)
    )
```

```mermaid
graph TD
    S["_test_database<br/>session 级 · autouse<br/>重建测试库 + 迁移"] -.整个会话一次.-> A
    A["app<br/>创建应用 + 触发 lifespan<br/>结束后 TRUNCATE"] --> C["client<br/>AsyncClient + ASGITransport"]
    A --> D["db_session<br/>直接操作数据库"]
    D --> U["user<br/>创建测试用户 Alice"]
```

几个要点：

* 测试只声明自己直接需要的 fixture，pytest 会沿着依赖链把上游的都准备好。例如测试只写了 `user`，pytest 会依次执行 `app → db_session → user`。
* **同一个测试里，同一个 fixture 只执行一次**。`test_inactive_user_cannot_login(client, user, db_session)` 里的 `client`、`db_session` 拿到的是同一个 `app`，所以通过 `db_session` 修改的数据，`client` 发出的请求能读到。
* 清理顺序与创建顺序相反：先关闭 `client`、`db_session`，最后才执行 `app` 中 `yield` 之后的清表和关闭连接池。

例如下面这个测试用 `db_session` 停用用户，再用 `client` 验证登录被拒绝：

```python
async def test_inactive_user_cannot_login(
    client: AsyncClient, user: User, db_session: AsyncSession
) -> None:
    user.is_active = False
    await db_session.commit()

    resp = await client.post(
        f"{AUTH}/login", json={"username": TEST_USERNAME, "password": TEST_PASSWORD}
    )
    assert resp.status_code == 401
```

再如 `test_unhandled_exception_returns_500_without_details` 同时声明了 `app` 和 `client`，先在 `app` 上临时注册一个会抛异常的路由，再用 `client` 请求它：

```python
async def test_unhandled_exception_returns_500_without_details(
    app: FastAPI, client: AsyncClient
) -> None:
    @app.get("/boom")
    async def boom() -> None:
        raise RuntimeError("secret internal detail")

    resp = await client.get("/boom")
    assert resp.status_code == 500
    assert resp.json()["code"] == "internal_error"
    assert "secret internal detail" not in resp.text
```

### 3.3. conftest.py 是固定文件名

`conftest.py` 是 pytest 约定的文件名。pytest 收集测试时会自动加载测试目录（及其上级目录）里的 `conftest.py`，其中定义的 fixture 对该目录下所有测试文件可见，**测试文件不需要 import 就能使用**。

文件名写错时，pytest 不会把它当作 conftest 加载。例如把它误写成 `configtest.py`，运行后会输出：

```
collected 13 items / 1 error

ImportError while importing test module '.../tests/test_auth.py'.
tests/test_auth.py:7: in <module>
    from tests.conftest import TEST_PASSWORD, TEST_USERNAME
E   ModuleNotFoundError: No module named 'tests.conftest'
!!!!!!!!!!!!!!!!!!!!! Interrupted: 1 error during collection !!!!!!!!!!!!!!!!!!!!!
```

| 现象 | 原因 |
|------|------|
| `ModuleNotFoundError: No module named 'tests.conftest'` | `test_auth.py` 里写了 `from tests.conftest import ...`，而 `tests/conftest.py` 不存在 |
| 只收集到 13 个用例，而不是 28 个 | `test_auth.py` 整个导入失败，它的 15 个用例都没有被收集；剩下的是 `test_health.py`（6 个）和 `test_security.py`（7 个） |
| `fixture 'client' not found` | 如果测试文件里没有这句 import，就不会在导入阶段失败，而是在运行时找不到 conftest 里定义的 fixture |

> conftest.py 里的 fixture 不需要 import；但普通常量（如 `TEST_USERNAME`）不是 fixture，要用的话仍然需要显式 import，这就依赖下一节的导入路径。

## 4. tests/__init__.py 与导入路径

`test_auth.py` 里有这样一句：

```python
from tests.conftest import TEST_PASSWORD, TEST_USERNAME
```

要让 `tests.conftest` 能被导入，项目根目录必须在 `sys.path` 里。这由 `tests/__init__.py` 决定。

pytest 默认使用 `prepend` 导入模式：导入一个测试文件时，从它所在目录往上找，找到**第一个不含 `__init__.py` 的目录**，把这个目录插入 `sys.path` 最前面，再按包路径导入。

```
my-service/                  ← 不含 __init__.py，被插入 sys.path
├── pyproject.toml
├── src/my_service/...
└── tests/
    ├── __init__.py         ← 有这个文件，tests 被当作一个包
    ├── conftest.py
    ├── test_auth.py        ← 以 tests.test_auth 的名字导入
    ├── test_health.py
    └── test_security.py
```

| tests/ 下有没有 `__init__.py` | 插入 sys.path 的目录 | 测试模块名 | `from tests.conftest import ...` |
|------|------|------|------|
| 有 | 项目根目录 `my-service/` | `tests.test_auth` | 可以 |
| 没有 | `tests/` 自己 | `test_auth` | 找不到 `tests` 包 |

> 业务代码 `my_service` 能被导入，靠的是 `uv sync` 把项目以可编辑模式装进了 `.venv`，与这里的 `sys.path` 无关。

## 5. 测试数据库：用真实 PostgreSQL

### 5.1. 为什么不用 SQLite

很多教程用 SQLite 内存库做测试，启动快、不需要外部服务。但测试用的数据库应该和生产一致，否则测试通过不代表线上能跑。项目依赖了这些 PostgreSQL 特性：

| 特性 | 项目中的用法 | SQLite 的情况 |
|------|-------------|--------------|
| `SELECT ... FOR UPDATE` 行锁 | 刷新令牌时加锁，防止同一个令牌并发刷新换出两组新令牌 | 不支持行锁，SQLAlchemy 生成 SQL 时会直接忽略 `with_for_update()` |
| `timestamptz` | 所有时间字段 `DateTime(timezone=True)`，比较过期时间 | 没有带时区的时间类型 |
| `UUID` | 主键、`user_id`、`family_id` | 没有原生 UUID 类型 |

例如刷新令牌的查询：

```python
async def get_by_hash_for_update(self, token_hash: str) -> RefreshToken | None:
    """查询并加行锁（SELECT ... FOR UPDATE）。"""
    result = await self.session.execute(
        select(RefreshToken).where(RefreshToken.token_hash == token_hash).with_for_update()
    )
    return result.scalar_one_or_none()
```

在 SQLite 上这段代码也能"通过"，但行锁根本没有生效，测试就失去了意义。

### 5.2. 隔离方案

用真实数据库时，要解决两个问题：不能弄脏开发库，测试之间不能互相影响。项目的做法：

| 层面 | 做法 |
|------|------|
| 与开发库隔离 | 使用独立的测试库，名字是开发库名加 `_test`（如 `my_service_test`） |
| 每次测试会话 | 开始时删除并重建测试库，用 Alembic 迁移建表（顺便测试了迁移脚本） |
| 每个测试之间 | 测试结束后 `TRUNCATE` 所有表，下一个测试从空库开始 |

```mermaid
sequenceDiagram
    participant P as pytest
    participant PG as PostgreSQL
    P->>PG: 连接 postgres 库
    P->>PG: DROP DATABASE IF EXISTS my_service_test WITH (FORCE)
    P->>PG: CREATE DATABASE my_service_test
    P->>PG: alembic upgrade head（建表）
    loop 每个测试
        P->>PG: 创建连接池（lifespan 启动）
        P->>PG: 执行测试
        P->>PG: TRUNCATE 所有表 CASCADE
        P->>PG: 释放连接池（lifespan 关闭）
    end
```

### 5.3. session 级同步 fixture：重建测试库并迁移

```python
async def _recreate_test_database() -> None:
    # CREATE DATABASE 不能在目标库里执行，所以先连到同一实例的 postgres 库
    conn = await asyncpg.connect(
        host=TEST_DATABASE_URL.host,
        port=TEST_DATABASE_URL.port,
        user=TEST_DATABASE_URL.username,
        password=TEST_DATABASE_URL.password,
        database="postgres",
    )
    try:
        await conn.execute(f'DROP DATABASE IF EXISTS "{TEST_DATABASE_URL.database}" WITH (FORCE)')
        await conn.execute(f'CREATE DATABASE "{TEST_DATABASE_URL.database}"')
    finally:
        await conn.close()


@pytest.fixture(scope="session", autouse=True)
def _test_database() -> None:
    """整个测试会话只执行一次：重建测试库，用 Alembic 迁移建表。

    用迁移而不是 Base.metadata.create_all 建表，顺便验证了迁移脚本本身。
    """
    asyncio.run(_recreate_test_database())
    command.upgrade(Config("alembic.ini"), "head")
```

逐条说明：

| 写法 | 原因 |
|------|------|
| `scope="session", autouse=True` | 整个测试会话只执行一次，且所有测试自动依赖它，不需要声明参数 |
| fixture 写成普通 `def`，而不是 `async def` | 同步 fixture 运行时**没有正在运行的事件循环**，可以直接用 `asyncio.run(...)` 执行异步建库代码；同时也不需要像 2.2 中那样为 session 级异步 fixture 单独配置循环范围 |
| 先连 `postgres` 库 | `CREATE DATABASE` / `DROP DATABASE` 不能在目标库自己的连接里执行，要连到同一实例的另一个库 |
| `DROP DATABASE ... WITH (FORCE)` | PostgreSQL 13+ 支持，会先断开还连着测试库的会话（例如上次测试中断残留的连接），否则 DROP 会失败 |
| `IF EXISTS` | 第一次运行时测试库还不存在 |
| `command.upgrade(Config("alembic.ini"), "head")` | 用 Python API 执行等同于 `alembic upgrade head` 的迁移 |
| 迁移必须在同步上下文调用 | 项目的 `migrations/env.py` 里用 `asyncio.run(run_async_migrations())` 执行迁移；如果在 `async def` 里调用，就会在已有事件循环中再次 `asyncio.run`，抛出 `RuntimeError` |

`migrations/env.py` 的相关部分：

```python
# 与应用使用同一份配置，不在 alembic.ini 里重复写数据库地址和密码
DATABASE_URL = get_settings().database_url.get_secret_value()

def run_migrations_online() -> None:
    asyncio.run(run_async_migrations())
```

`env.py` 从应用配置读取数据库地址，而 conftest 顶部已经把 `DATABASE_URL` 环境变量改成了测试库（见第 6 节），所以迁移会执行到测试库上。

> 因为这个 fixture 是 `autouse` 的，即使只运行不需要数据库的 `test_security.py`，也会先连接数据库。PostgreSQL 没启动时，所有用例都会报错：
> `OSError: Multiple exceptions: [Errno 61] Connect call failed ('::1', 5432, 0, 0), [Errno 61] Connect call failed ('127.0.0.1', 5432)`

### 5.4. 派生测试库地址

测试库地址不单独配置，而是从开发库地址自动派生：

```python
from sqlalchemy.engine import make_url

_dev_url = make_url(Settings().database_url.get_secret_value())
TEST_DATABASE_URL = _dev_url.set(database=f"{_dev_url.database}_test")

os.environ["DATABASE_URL"] = TEST_DATABASE_URL.render_as_string(hide_password=False)
```

| 方法 | 作用 |
|------|------|
| `make_url(str)` | 把连接串解析成 `URL` 对象，可以读取 `.host`、`.port`、`.username`、`.password`、`.database` |
| `url.set(database=...)` | `URL` 是不可变对象，`set` 返回一个修改了指定部分的新对象，其他部分（驱动、账号、主机）保持不变 |
| `render_as_string(hide_password=False)` | 转回字符串。默认会把密码显示为 `***`，写入环境变量供应用连接时必须输出完整密码 |

```python
url = make_url("postgresql+asyncpg://app:secret@localhost:5432/my_service")
test_url = url.set(database="my_service_test")

str(test_url)
# postgresql+asyncpg://app:***@localhost:5432/my_service_test

test_url.render_as_string(hide_password=False)
# postgresql+asyncpg://app:secret@localhost:5432/my_service_test
```

这样不需要在 `.env` 里额外维护一个测试库地址，开发库在哪，测试库就建在同一个实例上。

### 5.5. 每个测试结束后清空所有表

```python
from sqlalchemy import text
from my_service.models import Base

tables = ", ".join(t.name for t in Base.metadata.sorted_tables)
async with application.state.db_engine.begin() as conn:
    await conn.execute(text(f"TRUNCATE {tables} CASCADE"))
```

| 写法 | 原因 |
|------|------|
| `Base.metadata.sorted_tables` | 所有 ORM 模型对应的表，按外键依赖排序；新增模型后自动包含，不需要手动维护表名列表 |
| 一条 `TRUNCATE` 列出所有表 | 一次清空，比逐表 `DELETE` 快；`refresh_tokens.user_id` 外键引用了 `users`，两张表放在同一条语句里才允许清空 `users` |
| `CASCADE` | 兜底：如果还有不在列表里的表通过外键引用了这些表，一并清空，而不是报错 |
| `engine.begin()` | 开启事务，代码块结束时自动提交 |

> Alembic 的版本表 `alembic_version` 不属于 ORM 模型，不在 `sorted_tables` 里，所以不会被清空，测试库始终保持"已迁移到 head"的状态。

> 另一种常见方案是"每个测试包在一个事务里，结束时回滚"。但项目的业务代码自己会 `commit`（例如刷新令牌时检测到复用要立即提交作废操作），回滚方案需要额外处理嵌套事务；TRUNCATE 方案更直观，代价是每个测试多一条 SQL。

## 6. 环境变量要在导入 app 之前设置

`main.py` 在模块级创建了应用实例：

```python
# src/my_service/main.py
def create_app() -> FastAPI:
    settings = get_settings()
    setup_logging(settings)
    ...

app = create_app()   # 模块被导入时就会执行
```

而 `get_settings()` 用 `@lru_cache` 缓存了配置：

```python
@lru_cache
def get_settings() -> Settings:
    return Settings()
```

这意味着：**只要 `my_service.main` 被导入，就会读取一次配置并缓存下来**。如果此时环境变量还指向开发库，测试就会连到开发库上。

所以 conftest 有两处安排：

**1. 文件顶部先改环境变量，再清缓存**

```python
# ---------- 测试环境配置：必须在创建 app 之前完成 ----------
# 环境变量优先级高于 .env，这里覆盖的值会生效
_dev_url = make_url(Settings().database_url.get_secret_value())
TEST_DATABASE_URL = _dev_url.set(database=f"{_dev_url.database}_test")

os.environ["APP_ENV"] = "test"
os.environ["DATABASE_URL"] = TEST_DATABASE_URL.render_as_string(hide_password=False)
os.environ["JWT_SECRET_KEY"] = "test-secret-key-only-for-tests-0123456789"
get_settings.cache_clear()
```

* pydantic-settings 中**环境变量的优先级高于 `.env` 文件**，所以在 `os.environ` 里覆盖就能生效。
* 注意这里用 `Settings()` 直接创建一个新实例读取开发库地址，而不是 `get_settings()`，避免把开发配置写进缓存。
* `get_settings.cache_clear()` 清掉可能已经缓存的旧配置，下次调用时按新的环境变量重新读取。

**2. `create_app` 的 import 放在 fixture 函数内**

```python
@pytest.fixture
async def app() -> AsyncGenerator[FastAPI]:
    from my_service.main import create_app   # 不放在文件顶部

    application = create_app()
    ...
```

如果把 `from my_service.main import create_app` 写在 conftest 顶部，导入发生在修改环境变量之前，模块级的 `app = create_app()` 就会用开发配置执行一遍。放进 fixture 里，可以保证导入时环境变量已经改好。

```mermaid
graph LR
    A["conftest 顶部<br/>修改 os.environ"] --> B["get_settings.cache_clear()"]
    B --> C["fixture 内<br/>import my_service.main"]
    C --> D["create_app()<br/>读取测试配置"]
```

## 7. httpx.AsyncClient + ASGITransport

### 7.1. 不启动服务直接调用应用

FastAPI 应用本质上是一个 ASGI 可调用对象。`httpx.ASGITransport` 把 HTTP 请求直接转换成对这个对象的函数调用，不需要启动 uvicorn、不占用端口、不走网络：

```python
from httpx import ASGITransport, AsyncClient

@pytest.fixture
async def client(app: FastAPI) -> AsyncGenerator[AsyncClient]:
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as c:
        yield c
```

| 参数 | 说明 |
|------|------|
| `transport=ASGITransport(app=app)` | 请求交给 ASGI 应用处理，而不是发到网络 |
| `base_url="http://test"` | 请求必须是完整 URL，这里随便给一个主机名，测试里就可以只写路径 `client.get("/healthz")` |

和真实请求一样，中间件、异常处理器、依赖注入都会完整执行，所以 `test_health.py` 能测到中间件生成的 `X-Request-ID`：

```python
async def test_response_has_request_id(client: AsyncClient) -> None:
    resp = await client.get("/healthz")
    assert len(resp.headers["x-request-id"]) == 32


async def test_upstream_request_id_is_kept(client: AsyncClient) -> None:
    resp = await client.get("/healthz", headers={"X-Request-ID": "from-gateway-1"})
    assert resp.headers["x-request-id"] == "from-gateway-1"
```

### 7.2. 手动触发 lifespan

项目在 `lifespan` 里创建数据库连接池：

```python
@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None]:
    settings = get_settings()
    engine = create_engine(settings)
    app.state.db_engine = engine
    app.state.session_factory = create_session_factory(engine)
    yield
    await engine.dispose()
```

`ASGITransport` 只处理 HTTP 请求，**不会执行 lifespan**。如果直接用，`app.state.db_engine` 根本不存在，第一个访问数据库的请求就会报错。

所以 `app` fixture 用 `app.router.lifespan_context` 手动进入 lifespan：

```python
@pytest.fixture
async def app() -> AsyncGenerator[FastAPI]:
    from my_service.main import create_app

    application = create_app()
    # 手动触发 lifespan：创建数据库连接池，测试结束后释放
    async with application.router.lifespan_context(application):
        yield application
        # 清空所有表，保证下一个测试从空库开始
        tables = ", ".join(t.name for t in Base.metadata.sorted_tables)
        async with application.state.db_engine.begin() as conn:
            await conn.execute(text(f"TRUNCATE {tables} CASCADE"))
```

* 进入 `async with` 时执行 lifespan 中 `yield` 之前的代码（创建连接池）。
* 测试结束后，先在连接池还在的时候清表，再退出 `async with`，执行 `engine.dispose()` 释放连接。
* 每个测试都 `create_app()` 一个新实例，测试里临时注册的路由（如 3.2 中的 `/boom`）不会泄漏到其他测试。

## 8. 测试写法示例

### 8.1. 单元测试：不需要数据库

`test_security.py` 测试密码哈希与 JWT 工具函数，都是纯函数，不需要任何 fixture，写成普通的同步测试：

```python
import uuid
from datetime import timedelta

import jwt

from my_service.core.security import (
    create_access_token,
    decode_access_token,
    hash_password,
    verify_password,
)

SECRET = "x" * 40


def test_password_hash_and_verify() -> None:
    hashed = hash_password("Passw0rd!")
    assert hashed.startswith("$argon2id$")
    assert hashed != "Passw0rd!"
    assert verify_password(hashed, "Passw0rd!")
    assert not verify_password(hashed, "wrong")


def test_expired_access_token_is_rejected() -> None:
    # 有效期设为负数，签发出来就是过期的
    token, _ = create_access_token(uuid.uuid4(), SECRET, timedelta(seconds=-1))
    assert decode_access_token(token, SECRET) is None


def test_unsigned_token_is_rejected() -> None:
    # alg=none 攻击：不带签名的令牌必须被拒绝
    token = jwt.encode({"sub": str(uuid.uuid4()), "type": "access"}, None, algorithm="none")
    assert decode_access_token(token, SECRET) is None
```

单元测试的重点是边界和攻击场景：过期、密钥不对、不带签名（`alg=none`）、用 refresh 类型的令牌冒充 access 令牌。

### 8.2. 接口测试：认证流程

`test_auth.py` 通过 `client` 走完整的 HTTP 接口，先抽出两个辅助函数：

```python
AUTH = "/api/v1/auth"


async def _login(client: AsyncClient, username: str = TEST_USERNAME) -> dict[str, str]:
    resp = await client.post(
        f"{AUTH}/login", json={"username": username, "password": TEST_PASSWORD}
    )
    assert resp.status_code == 200, resp.text
    tokens: dict[str, str] = resp.json()
    return tokens


def _bearer(access_token: str) -> dict[str, str]:
    return {"Authorization": f"Bearer {access_token}"}
```

> `assert resp.status_code == 200, resp.text`：断言失败时把响应体打印出来，排查时能直接看到接口返回的错误信息。

**登录**：声明 `user` fixture 就有了一个可以登录的用户。

```python
async def test_login_success(client: AsyncClient, user: User) -> None:
    tokens = await _login(client)
    assert tokens["token_type"] == "bearer"
    assert tokens["access_token"]
    assert tokens["refresh_token"]


async def test_login_unknown_user_has_same_message(client: AsyncClient) -> None:
    # 不声明 user：库里没有任何用户
    resp = await client.post(
        f"{AUTH}/login", json={"username": "nobody", "password": TEST_PASSWORD}
    )
    assert resp.status_code == 401
    assert resp.json()["message"] == "用户名或密码错误"
```

**刷新轮换**：刷新后拿到的是新的 refresh token，新的 access token 可以正常使用。

```python
async def test_refresh_rotates_tokens(client: AsyncClient, user: User) -> None:
    old = await _login(client)
    resp = await client.post(f"{AUTH}/refresh", json={"refresh_token": old["refresh_token"]})
    assert resp.status_code == 200
    new = resp.json()
    assert new["refresh_token"] != old["refresh_token"]

    me = await client.get(f"{AUTH}/me", headers=_bearer(new["access_token"]))
    assert me.status_code == 200
```

**复用检测**：已经用过的旧令牌再次被使用，视为被盗用，同一家族的令牌全部作废。

```python
async def test_reused_refresh_token_revokes_whole_family(client: AsyncClient, user: User) -> None:
    first = await _login(client)
    second = (
        await client.post(f"{AUTH}/refresh", json={"refresh_token": first["refresh_token"]})
    ).json()

    # 旧令牌被再次使用（模拟被盗用）
    reuse = await client.post(f"{AUTH}/refresh", json={"refresh_token": first["refresh_token"]})
    assert reuse.status_code == 401

    # 同一家族里最新的令牌也一并失效
    latest = await client.post(f"{AUTH}/refresh", json={"refresh_token": second["refresh_token"]})
    assert latest.status_code == 401
```

**注销只影响当前会话**：同一用户登录两次，注销其中一个，另一个仍然可以刷新。

```python
async def test_logout_only_affects_current_session(client: AsyncClient, user: User) -> None:
    session_a = await _login(client)
    session_b = await _login(client)
    await client.post(f"{AUTH}/logout", json={"refresh_token": session_a["refresh_token"]})

    resp = await client.post(f"{AUTH}/refresh", json={"refresh_token": session_b["refresh_token"]})
    assert resp.status_code == 200
```

这些用例覆盖的就是手工用 curl 调接口时要走的流程，区别是每次提交都能自动重复执行。

### 8.3. 运行结果

```shell
uv run pytest
```

正常情况下 28 个用例全部通过（`test_auth.py` 15 个、`test_health.py` 6 个、`test_security.py` 7 个），耗时约 2 秒：

```
28 passed in ...s
```

其他常用参数：

```shell
uv run pytest tests/test_auth.py                          # 只跑一个文件
uv run pytest tests/test_auth.py::test_refresh_rotates_tokens   # 只跑一个用例
uv run pytest -k logout                                   # 只跑名字包含 logout 的用例
uv run pytest -x                                          # 遇到第一个失败就停止
```

## 9. 覆盖率

### 9.1. 运行与配置

```shell
uv run pytest --cov --cov-report=term-missing
```

| 参数 | 含义 |
|------|------|
| `--cov` | 开启覆盖率统计，统计范围取 `[tool.coverage.run] source` |
| `--cov-report=term-missing` | 在终端输出报告，并列出未覆盖的行号 |

```toml
[tool.coverage.run]
source = ["src/my_service"]
branch = true
concurrency = ["greenlet", "thread"]
```

| 配置 | 含义 |
|------|------|
| `source` | 只统计业务代码，不统计测试代码和第三方库 |
| `branch = true` | 开启**分支覆盖**：不仅看每行是否执行过，还看每个 `if` 的"真"和"假"两条路径是否都走过 |
| `concurrency` | 告诉 coverage 代码里用了哪些并发方式，见 9.3 |

### 9.2. 读懂报告

报告的表头：

```
Name                                  Stmts   Miss Branch BrPart  Cover   Missing
```

| 列 | 含义 |
|----|------|
| `Stmts` | 可执行语句数 |
| `Miss` | 没有执行到的语句数 |
| `Branch` | 分支数（开启 `branch = true` 后才有） |
| `BrPart` | 部分覆盖的分支数：某个判断只走过其中一条路径 |
| `Cover` | 覆盖率 |
| `Missing` | 未覆盖的位置。`18` 表示第 18 行没执行，`41-49` 表示一段连续的行，`23->25` 表示从第 23 行跳到第 25 行的那条分支没走过 |

几点经验：

* `cli.py` 显示 0% 是正常的：它是命令行入口（`my-service create-user`），项目没有为它写测试。
* 覆盖率是"哪里一定没测到"的提示，而不是"测得好不好"的证明。代码被执行了不代表断言检查了它的结果，这一点要靠第 10 节的变异检查来确认。
* 重点看 `Missing` 列里业务分支（如 `if stored.revoked_at is not None`）有没有被走到，而不是追求 100%。

### 9.3. 坑：SQLAlchemy 异步模式与 greenlet

现象：不做额外配置直接统计覆盖率时，`services/auth_service.py` 只显示约 30%，但登录、刷新、注销明明都有测试覆盖。

原因：SQLAlchemy 的异步模式内部用 **greenlet** 在同步的 ORM 代码和异步驱动之间切换。coverage 默认只按线程跟踪执行，识别不了 greenlet 的切换，于是 **`await` 数据库调用之后执行的代码都被算作未覆盖**。

```python
async def refresh(self, refresh_token: str) -> TokenResponse:
    now = datetime.now(UTC)
    stored = await self.refresh_tokens.get_by_hash_for_update(...)   # 数据库调用
    if stored is None or stored.expires_at <= now:                   # ← 之后的行都被误判为未执行
        raise UnauthorizedError("登录已失效，请重新登录")
    ...
```

修复方法是在 `[tool.coverage.run]` 中声明 greenlet：

```toml
[tool.coverage.run]
concurrency = ["greenlet", "thread"]
```

> `concurrency` 的默认值是 `["thread"]`。一旦手动设置就会覆盖默认值，所以要把 `thread` 一起写上。

| 现象 | 原因 | 处理 |
|------|------|------|
| 有测试覆盖的 service 层覆盖率很低，Missing 集中在数据库调用之后 | coverage 识别不了 SQLAlchemy 异步模式内部的 greenlet 切换 | `concurrency = ["greenlet", "thread"]` |
| `cli.py` 0% | 没有为命令行入口写测试 | 正常，可以接受 |

## 10. 验证测试有效：变异检查

覆盖率只能说明代码被执行过。要确认测试**真的能发现错误**，可以故意把业务代码改坏，看测试会不会失败。这种做法叫变异检查（mutation testing），手工做几处关键逻辑就很有价值。

以刷新令牌的轮换逻辑为例，`services/auth_service.py` 的 `refresh` 方法里有一行：

```python
# 轮换：作废旧令牌，在同一家族内签发新令牌
stored.revoked_at = now
return await self._issue_tokens(user.id, family_id=stored.family_id)
```

这一行把旧令牌标记为已作废。如果漏掉它，旧令牌就能被无限次使用，复用检测也就不会触发。

操作步骤：

```shell
# 1. 把 stored.revoked_at = now 这一行注释掉

# 2. 运行测试
uv run pytest
```

结果：

```
FAILED tests/test_auth.py::test_reused_refresh_token_revokes_whole_family
1 failed, 27 passed
```

旧令牌没有被作废，第二次使用它时接口返回了 200 而不是 401，`test_reused_refresh_token_revokes_whole_family` 的断言失败，说明这个测试确实守住了"轮换后旧令牌失效"这条规则。

```shell
# 3. 恢复代码
git checkout src/my_service/services/auth_service.py
```

```mermaid
graph LR
    A[故意改坏一处关键逻辑] --> B[运行测试]
    B -->|有测试失败| C[测试有效]
    B -->|全部通过| D[测试有漏洞，补用例]
    C --> E[git checkout 恢复代码]
    D --> E
```

> 如果改坏代码后测试依然全部通过，说明这段逻辑虽然被"执行"了（覆盖率里显示已覆盖），但没有任何断言检查它的结果。

## 11. 常见问题速查

| 现象 | 原因 | 处理 |
|------|------|------|
| `async def functions are not natively supported.` | 没装 pytest-asyncio，或 strict 模式下忘了加 `@pytest.mark.asyncio` | 安装插件并设置 `asyncio_mode = "auto"` |
| `PytestDeprecationWarning: The configuration option "asyncio_default_fixture_loop_scope" is unset.` | 没有配置异步 fixture 的循环范围 | 设置 `asyncio_default_fixture_loop_scope = "function"` |
| `ModuleNotFoundError: No module named 'tests.conftest'`，收集数变少 | conftest 文件名写错（如 `configtest.py`） | 改为 `conftest.py` |
| `fixture 'client' not found` | conftest 没被加载，或 fixture 名拼错 | 检查文件名和 fixture 名 |
| `from tests.xxx import` 找不到 `tests` | `tests/` 下缺少 `__init__.py` | 添加空的 `tests/__init__.py` |
| 测试数据写进了开发库 | 在修改环境变量之前就导入了 `my_service.main` | conftest 顶部先改 `os.environ` 并 `cache_clear()`，`create_app` 在 fixture 内 import |
| 请求时报 `app.state` 没有 `db_engine` | `ASGITransport` 不执行 lifespan | `async with app.router.lifespan_context(app)` |
| 所有用例（包括不需要数据库的）都报 `Connect call failed ('127.0.0.1', 5432)` | PostgreSQL 没启动，autouse 的建库 fixture 失败 | 先启动数据库容器 |
| service 层覆盖率异常低 | coverage 识别不了 greenlet | `concurrency = ["greenlet", "thread"]` |

## 12. 小结

* `asyncio_mode = "auto"` 让 `async def` 测试和 fixture 无需装饰器；`asyncio_default_fixture_loop_scope = "function"` 让每个测试使用独立的事件循环，并消除弃用警告。
* fixture 通过同名参数注入，可以相互依赖（`app → client`、`app → db_session → user`），同一测试内同一 fixture 只创建一次；`conftest.py` 是固定文件名，写错就不会被加载。
* `tests/__init__.py` 让 pytest 把项目根目录加入 `sys.path`，才能 `from tests.conftest import ...`。
* 测试用真实 PostgreSQL：独立的 `_test` 库，会话开始时重建并用 Alembic 迁移建表，每个测试结束 `TRUNCATE ... CASCADE`。建库和迁移放在 session 级**同步** fixture 里，用 `asyncio.run` 执行。
* `main.py` 导入即创建应用、读取配置，所以环境变量要在导入前设置，`create_app` 在 fixture 内 import。
* `httpx.AsyncClient + ASGITransport` 不启动服务直接调用应用，但不会执行 lifespan，要用 `app.router.lifespan_context(app)` 手动触发。
* 覆盖率用 `--cov --cov-report=term-missing` 查看，开启分支覆盖；SQLAlchemy 异步模式需要 `concurrency = ["greenlet", "thread"]`，否则统计不准。
* 覆盖率只说明代码被执行过，用变异检查（故意改坏代码看测试是否失败）确认测试真的有效。
