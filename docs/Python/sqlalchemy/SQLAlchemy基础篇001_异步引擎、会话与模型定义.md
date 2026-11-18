## 1. 引擎与会话

SQLAlchemy 2.0 的异步用法里有两个核心对象：**引擎（AsyncEngine）** 和 **会话（AsyncSession）**。分清它们的职责和生命周期，是写好数据库代码的第一步。

### 1.1. 两者的关系

| 对象 | 内部持有 | 数量 | 生命周期 |
|------|---------|------|---------|
| `AsyncEngine` | 连接池（多条数据库连接） | 整个进程**一个** | 应用启动时创建，关闭时 `dispose()` |
| `AsyncSession` | 一次**工作单元**：本次要读写的对象、当前事务 | **每个请求一个** | 请求开始时创建，请求结束时关闭 |

会话本身不是连接。会话第一次需要执行 SQL 时，才从引擎的连接池里借一条连接，事务结束后把连接还回池里。

```mermaid
flowchart LR
    subgraph 进程
        E["AsyncEngine<br/>（全局唯一）"]
        subgraph P["连接池"]
            C1["连接 1"]
            C2["连接 2"]
            C3["连接 ..."]
        end
        E --- P
        S1["AsyncSession<br/>请求 A"] -- 借用 / 归还 --> P
        S2["AsyncSession<br/>请求 B"] -- 借用 / 归还 --> P
    end
    P <--> DB[("PostgreSQL")]
```

### 1.2. 驱动与连接串

异步模式需要异步驱动，PostgreSQL 用 `asyncpg`。连接串里用 `+驱动名` 指定驱动：

```shell
# 安装（sqlalchemy 的 asyncio 扩展 + asyncpg 驱动）
uv add "sqlalchemy[asyncio]" asyncpg
```

```shell
# .env
DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/my_service
```

格式为 `方言+驱动://用户名:密码@主机:端口/数据库名`。只写 `postgresql://...` 时 SQLAlchemy 会选用该方言的默认驱动（SQLAlchemy 2.1.8 中是 psycopg），而不是 asyncpg，所以异步项目要显式写上 `+asyncpg`。

### 1.3. 在 lifespan 中创建和释放引擎

FastAPI 的 `lifespan` 在应用启动时执行 `yield` 之前的代码，关闭时执行之后的代码，正好对应引擎的生命周期：

```python
# src/my_service/main.py
@asynccontextmanager
async def lifespan(app: FastAPI) -> AsyncGenerator[None]:
    settings = get_settings()

    # 启动：创建数据库引擎（连接池）。这里不会立即建立连接，第一次使用时才连接
    engine = create_engine(settings)
    app.state.db_engine = engine
    app.state.session_factory = create_session_factory(engine)

    yield

    # 关闭：释放连接池中的所有连接
    await engine.dispose()
```

引擎和会话工厂挂在 `app.state` 上，后面的会话依赖从这里取。

## 2. 创建引擎：create_async_engine

### 2.1. 连接池参数

```python
# src/my_service/db/session.py
from sqlalchemy.ext.asyncio import AsyncEngine, create_async_engine


def create_engine(settings: Settings) -> AsyncEngine:
    return create_async_engine(
        settings.database_url.get_secret_value(),
        pool_size=settings.database_pool_size,  # 常驻连接数
        max_overflow=settings.database_max_overflow,  # 高峰期允许额外创建的连接数
        pool_pre_ping=True,  # 取出连接前先探活，避免拿到已被数据库断开的连接
        pool_recycle=1800,  # 连接最长使用 30 分钟后重建，避免被防火墙或代理静默断开
    )
```

| 参数 | 示例取值 | 含义 |
|------|---------|------|
| `pool_size` | 10（配置项默认值） | 池里常驻的连接数 |
| `max_overflow` | 20（配置项默认值） | 常驻连接都被占用时，最多再临时创建多少条；用完即关。同时最多 `pool_size + max_overflow` = 30 条连接 |
| `pool_pre_ping` | `True` | 每次从池里取连接前先发一个轻量探测，连接已断就丢弃并换一条新的 |
| `pool_recycle` | `1800` | 连接从创建起超过 1800 秒（30 分钟），下次取出时先关掉重建 |

`pool_recycle` 针对的是这种情况：数据库前面有防火墙、NAT 或代理，空闲太久的 TCP 连接会被它们**静默断开**，客户端并不知道。定期重建可以避开这个时间窗口，`pool_pre_ping` 再兜住剩下的意外断开。

> 连接总数要算上所有进程：例如 4 个 worker 进程，每个最多 30 条，合计 120 条，不能超过数据库的 `max_connections`。

### 2.2. 引擎是懒连接的

`create_async_engine` 只是创建引擎对象和一个空的连接池，**不会连接数据库**。即使地址根本连不上，创建也会成功：

```python
eng = create_async_engine("postgresql+asyncpg://u:p@10.255.255.1:5999/nodb", pool_size=1)
print(eng.pool.status())
# Pool size: 1  Connections in pool: 0 Current Overflow: -1 Current Checked out connections: 0
```

第一次执行 SQL 时才真正建立连接。所以数据库地址或密码写错，应用照样能启动，要等到第一个访问数据库的请求才报错。需要启动时就发现问题，可以加一个健康检查接口或在启动时主动执行一次查询。

## 3. 会话工厂：async_sessionmaker

`async_sessionmaker` 是创建会话的工厂，把引擎和会话的公共配置绑定在一起，之后每次调用 `factory()` 得到一个新会话。

```python
# src/my_service/db/session.py
from sqlalchemy.ext.asyncio import AsyncSession, async_sessionmaker


def create_session_factory(engine: AsyncEngine) -> async_sessionmaker[AsyncSession]:
    # expire_on_commit=False：commit 后对象属性仍可直接读取。
    # 异步模式下若为 True，commit 后访问属性会触发隐式查询并报错。
    return async_sessionmaker(engine, expire_on_commit=False)
```

### 3.1. expire_on_commit=False

`expire_on_commit` 默认为 `True`：`commit()` 之后，会话里所有对象的属性都被标记为"过期"，下次访问属性时 SQLAlchemy 自动发一条 SELECT 重新加载。

| 模式 | `expire_on_commit=True`（默认）时 commit 后访问 `user.id` |
|------|------|
| 同步 | 自动执行 SELECT 重新加载，能正常拿到值 |
| 异步 | 属性访问是普通的同步操作，没有地方 `await`，这种**隐式 IO** 无法执行，直接报错（常见报错为 `MissingGreenlet`） |

典型场景是创建用户后返回给接口：

```python
await session.commit()
return user  # FastAPI 序列化时读取 user.id、user.created_at
```

设成 `False` 后，commit 后对象保持 commit 前的属性值，可以直接读取。代价是对象不会自动刷新为数据库的最新值，需要时显式 `await session.refresh(obj)`。

## 4. 会话依赖：每个请求一个会话

在 FastAPI 中用 `yield` 依赖提供会话：

```python
# src/my_service/api/deps.py
async def get_db_session(request: Request) -> AsyncGenerator[AsyncSession]:
    """每个请求一个会话；请求结束时自动关闭，未提交的事务自动回滚。"""
    session_factory: async_sessionmaker[AsyncSession] = request.app.state.session_factory
    async with session_factory() as session:
        yield session


# 路由函数里这样用：async def handler(session: DbSession): ...
DbSession = Annotated[AsyncSession, Depends(get_db_session)]
```

`async with` 退出时会关闭会话：还没提交的事务被回滚，连接还回连接池。不管请求是正常返回还是抛了异常，都会走到这一步。

```mermaid
sequenceDiagram
    participant R as 请求
    participant D as get_db_session
    participant S as Service
    participant DB as PostgreSQL
    R->>D: 进入依赖
    D->>D: session_factory() 创建会话
    D->>S: yield session
    S->>DB: 执行 SQL（首次执行时借连接并开启事务）
    S->>DB: session.commit()（由 service 显式调用）
    S-->>D: 处理结束 / 抛出异常
    D->>DB: 关闭会话：未提交的事务回滚，连接归还连接池
```

注意这个依赖**不会自动 commit**。是否提交、什么时候提交是业务决定，由 service 层显式调用 `session.commit()`（见第 11 节）。这样做的好处是：只读请求不会产生无意义的提交；业务中途失败抛出异常时，已执行的修改自然被回滚。

## 5. 模型声明

### 5.1. DeclarativeBase

所有模型继承同一个基类，基类持有一个 `MetaData`，里面登记了所有表的结构，Alembic 生成迁移时就是读它。

```python
# src/my_service/models/base.py
from sqlalchemy import MetaData
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=NAMING_CONVENTION)  # 命名规则见第 6 节
```

### 5.2. Mapped + mapped_column：类型化写法

2.0 风格用类型注解 `Mapped[...]` 声明列的 Python 类型，`mapped_column(...)` 补充数据库层面的细节：

```python
# src/my_service/models/user.py
import uuid

from sqlalchemy import String, true
from sqlalchemy.orm import Mapped, mapped_column

from my_service.models.base import Base, TimestampMixin


class User(TimestampMixin, Base):
    __tablename__ = "users"

    id: Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
    username: Mapped[str] = mapped_column(String(50), unique=True)  # 区分大小写
    password_hash: Mapped[str] = mapped_column(String(255))
    is_active: Mapped[bool] = mapped_column(default=True, server_default=true())
```

SQLAlchemy 会根据注解推断列类型：

| 注解 | 推断出的 PostgreSQL 类型 |
|------|------------------------|
| `Mapped[uuid.UUID]` | `UUID` |
| `Mapped[bool]` | `BOOLEAN` |
| `Mapped[str]` | `VARCHAR`（不带长度）；需要长度时写 `String(50)` |
| `Mapped[datetime]` | `TIMESTAMP WITHOUT TIME ZONE`；需要带时区时写 `DateTime(timezone=True)` |

类型注解的好处是 mypy 和编辑器能知道 `user.username` 是 `str`，写错类型会被提前发现。

### 5.3. 可空与 NOT NULL

是否可空也由注解决定：

```python
# src/my_service/models/refresh_token.py
expires_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))           # NOT NULL
revoked_at: Mapped[datetime | None] = mapped_column(DateTime(timezone=True))    # 可空
```

| 注解 | 生成的列 |
|------|---------|
| `Mapped[datetime]` | `expires_at TIMESTAMP WITH TIME ZONE NOT NULL` |
| `Mapped[datetime \| None]` | `revoked_at TIMESTAMP WITH TIME ZONE`（可空） |

不是 `Optional`（`X | None`）的列**默认就是 NOT NULL**，不需要再写 `nullable=False`。

## 6. 约束命名规则

主键、唯一约束、外键、索引在数据库里都有名字。不指定名字时由数据库自动生成，不同数据库的规则还不一样。问题出在以后：Alembic 修改或删除约束时要按名字操作，名字不可预测，生成的迁移就找不到它。

解决办法是在 `MetaData` 上配置命名规则：

```python
# src/my_service/models/base.py
NAMING_CONVENTION = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=NAMING_CONVENTION)
```

| 键 | 约束类型 | 示例模型生成的名字 |
|----|---------|----------------|
| `pk` | 主键 | `pk_users`、`pk_refresh_tokens` |
| `uq` | 唯一约束 | `uq_users_username`、`uq_refresh_tokens_token_hash` |
| `fk` | 外键 | `fk_refresh_tokens_user_id_users` |
| `ix` | 索引 | `ix_refresh_tokens_user_id`、`ix_refresh_tokens_family_id` |
| `ck` | 检查约束 | 示例模型未使用 |

`ix` 里的 `column_0_label` 是"表名_列名"，所以得到 `ix_refresh_tokens_user_id`。Alembic 自动生成的迁移里，这些名字用 `op.f()` 包裹，表示"名字已经按规则生成好，不要再套一次规则"：

```python
# migrations/versions/<revision>_add_users_and_refresh_tokens.py（节选）
sa.PrimaryKeyConstraint("id", name=op.f("pk_users")),
sa.UniqueConstraint("username", name=op.f("uq_users_username")),
...
sa.ForeignKeyConstraint(
    ["user_id"], ["users.id"],
    name=op.f("fk_refresh_tokens_user_id_users"),
    ondelete="CASCADE",
),
...
op.create_index(op.f("ix_refresh_tokens_user_id"), "refresh_tokens", ["user_id"], unique=False)
```

> 命名规则**必须在建第一张表之前定好**。已经用数据库自动命名建好的约束，事后加规则不会改名，只能写迁移逐个重命名。

## 7. 用 Mixin 复用字段

多张表都有的字段（如创建时间、更新时间）放进一个普通类里，模型通过多重继承复用：

```python
# src/my_service/models/base.py
class TimestampMixin:
    """创建时间、更新时间。时间一律带时区存储（timestamptz）。"""

    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), server_default=func.now())
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True), server_default=func.now(), onupdate=func.now()
    )


class User(TimestampMixin, Base):  # Mixin 写在 Base 前面
    ...
```

| 写法 | 作用 |
|------|------|
| `DateTime(timezone=True)` | PostgreSQL 的 `TIMESTAMP WITH TIME ZONE`（即 `timestamptz`），按 UTC 存储，读出时带时区信息 |
| `server_default=func.now()` | 表结构里写入 `DEFAULT now()`，插入时由数据库填当前时间 |
| `onupdate=func.now()` | 通过 ORM 执行 UPDATE 时，SQLAlchemy 在 SET 子句里加上 `updated_at=now()` |

注意 `onupdate` 是 SQLAlchemy 层面的行为，不会写进表结构。直接用 SQL 执行 UPDATE 时 `updated_at` 不会变；需要数据库层面保证就得用触发器。

## 8. default 与 server_default

两者都是"插入时没给值就用默认值"，区别在于**由谁来填**：

| | `default` | `server_default` |
|---|-----------|------------------|
| 谁填值 | SQLAlchemy 在 Python 里生成值，放进 INSERT 语句 | 数据库按表结构里的 `DEFAULT` 填值 |
| 是否写进表结构 | 否 | 是（出现在 DDL 和迁移文件里） |
| 绕过 ORM 直接写 SQL 插入 | **不生效** | 生效 |
| 典型用法 | `default=uuid.uuid4`（传函数，每行生成一次） | `server_default=func.now()`、`server_default=true()` |

`User.is_active` 两个都写了：

```python
is_active: Mapped[bool] = mapped_column(default=True, server_default=true())
```

生成的表结构：

```sql
CREATE TABLE users (
	id UUID NOT NULL,
	username VARCHAR(50) NOT NULL,
	password_hash VARCHAR(255) NOT NULL,
	is_active BOOLEAN DEFAULT true NOT NULL,
	created_at TIMESTAMP WITH TIME ZONE DEFAULT now() NOT NULL,
	updated_at TIMESTAMP WITH TIME ZONE DEFAULT now() NOT NULL,
	CONSTRAINT pk_users PRIMARY KEY (id),
	CONSTRAINT uq_users_username UNIQUE (username)
)
```

`server_default` 的值必须是 SQL 表达式或字符串。布尔值用 `sqlalchemy.true()`，生成的是 `DEFAULT true`；直接写 Python 的 `True` 是不行的。

> 上面的 DDL 可以不连数据库直接打印出来，用来检查模型写得对不对：
>
> ```python
> from sqlalchemy.dialects import postgresql
> from sqlalchemy.schema import CreateTable
> from my_service.models import User
>
> print(CreateTable(User.__table__).compile(dialect=postgresql.dialect()))
> ```

## 9. 主键与外键

### 9.1. UUID 主键

```python
id: Mapped[uuid.UUID] = mapped_column(primary_key=True, default=uuid.uuid4)
```

* `Mapped[uuid.UUID]` 在 PostgreSQL 上对应原生的 `UUID` 类型（16 字节），不是字符串。
* `default=uuid.uuid4` 传的是函数本身（不带括号），每插入一行调用一次，由 Python 生成 ID。

为什么不用自增整数 ID：

| 问题 | 说明 |
|------|------|
| 暴露数据量 | 用户 ID 出现在 URL 里（如 `/users/1024`），别人从数字就能估出有多少用户 |
| 可被遍历 | 把 ID 从 1 依次加 1 去请求，就能逐个探测资源；权限校验稍有疏漏就会大面积泄露 |

UUID 随机且不连续，这两个问题都不存在。

### 9.2. 外键

```python
# src/my_service/models/refresh_token.py
# 用户被删除时，其令牌一并删除
user_id: Mapped[uuid.UUID] = mapped_column(
    ForeignKey("users.id", ondelete="CASCADE"), index=True
)
```

| 写法 | 作用 |
|------|------|
| `ForeignKey("users.id")` | 引用 `users` 表的 `id` 列，用的是**表名.列名**，不是类名 |
| `ondelete="CASCADE"` | 数据库层面的级联：删除用户时，数据库自动删除他的所有令牌 |
| `index=True` | 给外键列建索引 |

生成的 DDL：

```sql
CONSTRAINT fk_refresh_tokens_user_id_users FOREIGN KEY(user_id) REFERENCES users (id) ON DELETE CASCADE
CREATE INDEX ix_refresh_tokens_user_id ON refresh_tokens (user_id)
```

PostgreSQL **不会**自动给外键列建索引。按用户查令牌、删除用户时级联删除令牌，都要按 `user_id` 查找，没有索引就是全表扫描，所以外键列通常都加 `index=True`。

## 10. 2.0 风格查询

2.0 风格的统一写法是：用 `select()` / `update()` 构造语句，交给 `session.execute()` 执行，再从结果里取数据。

### 10.1. select + where：按条件查一个

```python
# src/my_service/repositories/user_repository.py
async def get_by_username(self, username: str) -> User | None:
    result = await self.session.execute(select(User).where(User.username == username))
    return result.scalar_one_or_none()
```

`User.username == username` 不是普通的布尔比较，而是生成一个 SQL 条件表达式。生成的 SQL：

```sql
SELECT users.id, users.username, users.password_hash, users.is_active, users.created_at, users.updated_at
FROM users
WHERE users.username = %(username_1)s::VARCHAR
```

参数以占位符传给驱动，不会拼接进 SQL 字符串，不存在 SQL 注入问题。

`execute()` 返回的结果对象有几种取法：

| 方法 | 返回 | 结果行数不符时 |
|------|------|-------------|
| `scalar_one_or_none()` | 一个对象或 `None` | 多于 1 行时报错 |
| `scalar_one()` | 一个对象 | 0 行或多于 1 行都报错 |
| `scalars().all()` | 对象列表 | 不报错 |
| `scalars().first()` | 第一个对象或 `None` | 不报错 |

带 `scalar` 的方法取每行的第一列。`select(User)` 每行只有一个 `User` 对象，所以用 `scalar*` 直接拿到对象；不带 `scalar` 时拿到的是 `Row` 元组。

### 10.2. session.get：按主键查

```python
async def get_by_id(self, user_id: uuid.UUID) -> User | None:
    return await self.session.get(User, user_id)
```

按主键查询时用 `session.get()` 更简洁。它会先在当前会话已加载的对象里找，找到就直接返回，找不到才发 SELECT。

### 10.3. update：批量更新

```python
# src/my_service/repositories/refresh_token_repository.py
async def revoke_family(self, family_id: uuid.UUID, now: datetime) -> None:
    await self.session.execute(
        update(RefreshToken)
        .where(RefreshToken.family_id == family_id, RefreshToken.revoked_at.is_(None))
        .values(revoked_at=now)
    )
```

生成的 SQL：

```sql
UPDATE refresh_tokens SET revoked_at=%(revoked_at)s::TIMESTAMP WITH TIME ZONE
WHERE refresh_tokens.family_id = %(family_id_1)s::UUID AND refresh_tokens.revoked_at IS NULL
```

要点：

* `where()` 传多个条件时用 `AND` 连接。
* 判断 NULL 要写 `.is_(None)`，生成 `IS NULL`。写成 `== None` 也能工作，但 ruff 等工具会报警告。
* 单个对象的修改不需要 `update()`：直接给属性赋值（如 `stored.revoked_at = now`），commit 时 SQLAlchemy 自动生成 UPDATE。`update()` 适合"一条语句改一批行"。

### 10.4. join：多表查询

下面基于前文的 `User` 和 `RefreshToken` 模型举例，并附上语句编译后的 SQL。

```python
# 查出所有未作废的令牌，以及所属用户的用户名
stmt = (
    select(RefreshToken, User.username)
    .join(User, RefreshToken.user_id == User.id)
    .where(RefreshToken.revoked_at.is_(None))
)
result = await session.execute(stmt)
for token, username in result.all():  # 选了多列，每行是一个元组
    ...
```

```sql
SELECT refresh_tokens.id, ..., refresh_tokens.created_at, users.username
FROM refresh_tokens JOIN users ON refresh_tokens.user_id = users.id
WHERE refresh_tokens.revoked_at IS NULL
```

两表之间只有一个外键时，可以省略 ON 条件，SQLAlchemy 会按外键自动推断：

```python
# 只查已启用用户的令牌
stmt = select(RefreshToken).join(User).where(User.is_active.is_(True))
tokens = (await session.execute(stmt)).scalars().all()
```

```sql
SELECT refresh_tokens.id, ...
FROM refresh_tokens JOIN users ON users.id = refresh_tokens.user_id
WHERE users.is_active IS true
```

## 11. flush 与 commit：分层职责

### 11.1. flush 与 commit 的区别

| 操作 | 做了什么 | 其他连接能否看到 |
|------|---------|---------------|
| `session.add(obj)` | 只把对象登记进会话，不发 SQL | 否 |
| `await session.flush()` | 把待写入的修改以 INSERT / UPDATE 发给数据库，**事务不提交** | 否 |
| `await session.commit()` | 先 flush，再提交事务 | 是 |
| `await session.rollback()` | 回滚事务，撤销本事务的所有修改（包括已 flush 的） | — |

flush 的用处是**提前拿到数据库生成的值、提前暴露约束错误**：INSERT 执行后，`server_default` 填的 `created_at` 等值就有了；唯一约束冲突也会在 flush 时立刻抛出，而不是等到 commit。

### 11.2. repository 只 flush，service 决定何时 commit

```python
# src/my_service/repositories/user_repository.py
async def add(self, user: User) -> User:
    self.session.add(user)
    await self.session.flush()  # 发送 INSERT，拿到数据库生成的默认值；不提交事务
    return user
```

| 层 | 职责 | 是否 commit |
|------|------|-----------|
| 路由（api） | 解析请求、调用 service、返回响应 | 否 |
| service | 业务规则；决定哪些操作属于同一个事务 | **是**，业务完成时显式 commit |
| repository | 只负责读写数据库，不包含业务规则 | 否，最多 flush |

原因是一个业务操作往往要调用多个 repository。例如刷新令牌：把旧令牌标记为作废、插入新令牌，这两步必须同时成功或同时失败。如果每个 repository 方法都自己 commit，第二步失败时第一步已经提交，数据就不一致了。由 service 在最后统一 commit，整个业务就是一个事务。

```python
# src/my_service/services/auth_service.py（节选）
stored.revoked_at = now                 # 修改旧令牌（尚未写库）
await self.refresh_tokens.add(...)      # flush：发出 UPDATE 和 INSERT
await self.session.commit()             # 一起提交
```

## 12. 并发问题的处理

### 12.1. 唯一约束兜底：捕获 IntegrityError

创建用户时先查一次用户名是否存在，这能给出友好提示，但挡不住并发：

```python
# src/my_service/services/user_service.py
async def create_user(self, data: UserCreate) -> User:
    if await self.users.get_by_username(data.username) is not None:
        raise ConflictError("用户名已存在")

    user = User(username=data.username, password_hash=hash_password(data.password))
    try:
        await self.users.add(user)
        await self.session.commit()
    except IntegrityError as e:
        # 两个请求同时创建同一用户名时，上面的检查可能都通过，由数据库唯一约束兜底
        await self.session.rollback()
        raise ConflictError("用户名已存在") from e
    return user
```

```mermaid
sequenceDiagram
    participant A as 请求 A
    participant B as 请求 B
    participant DB as PostgreSQL
    A->>DB: SELECT username='alice' → 不存在
    B->>DB: SELECT username='alice' → 不存在
    A->>DB: INSERT alice → 成功，COMMIT
    B->>DB: INSERT alice → 违反 uq_users_username
    DB-->>B: IntegrityError
    B->>B: rollback()，转成 ConflictError（409）
```

要点：

* 先查询：覆盖绝大多数情况，返回明确的业务错误。
* 唯一约束：真正保证不重复，由数据库原子地判断。
* 捕获 `IntegrityError` 后要先 `rollback()`：出错后的会话处于失败状态，不回滚就不能继续使用。
* 再转成业务异常 `ConflictError`，接口返回 409，而不是把数据库异常变成 500。

### 12.2. with_for_update：行锁防止重复刷新

refresh token 每次刷新都会作废旧令牌、签发新令牌。如果同一个令牌被并发提交两次，两个请求都读到"未作废"，就会换出两组新令牌。解决办法是查询时加行锁：

```python
# src/my_service/repositories/refresh_token_repository.py
async def get_by_hash_for_update(self, token_hash: str) -> RefreshToken | None:
    """查询并加行锁（SELECT ... FOR UPDATE）。

    同一个令牌被并发提交刷新时，第二个请求会等第一个提交后再读，
    读到的是已作废状态，不会出现一个令牌换出两组新令牌的情况。
    """
    result = await self.session.execute(
        select(RefreshToken).where(RefreshToken.token_hash == token_hash).with_for_update()
    )
    return result.scalar_one_or_none()
```

生成的 SQL 末尾多了 `FOR UPDATE`：

```sql
SELECT refresh_tokens.id, ...
FROM refresh_tokens
WHERE refresh_tokens.token_hash = %(token_hash_1)s::VARCHAR FOR UPDATE
```

```mermaid
sequenceDiagram
    participant A as 请求 A
    participant B as 请求 B
    participant DB as PostgreSQL
    A->>DB: SELECT ... FOR UPDATE（拿到行锁）
    B->>DB: SELECT ... FOR UPDATE（等待 A 释放锁）
    A->>DB: 作废旧令牌、插入新令牌、COMMIT（释放锁）
    DB-->>B: 返回这一行，revoked_at 已有值
    B->>B: 判定为已作废令牌被重用 → 401
```

* 行锁从 SELECT 开始持有，直到所在事务 commit 或 rollback 才释放。会话依赖在请求结束时会回滚未提交的事务，所以抛异常的路径也不会一直占着锁。
* 用同一个令牌并发发起 3 个刷新请求，结果是 1 个 200、2 个 401。
* 按 `AuthService.refresh` 的逻辑，后到的请求读到已作废状态，会被当作"已作废令牌被重用"，并作废整个令牌家族，先成功的那个请求拿到的新令牌也会一起失效。

| 方式 | 适用场景 |
|------|---------|
| 唯一约束 + 捕获 `IntegrityError` | 防止**插入**重复数据 |
| `with_for_update()` 行锁 | "先读后改"的操作，防止多个请求基于同一个旧状态各自修改 |

### 12.3. 字符串比较区分大小写

PostgreSQL 的字符串比较默认**区分大小写**，`WHERE username = 'alice'` 匹配不到 `Alice`。唯一约束也一样，`Alice` 和 `alice` 被视为两个不同的值。

示例中的用户名设计为区分大小写，所以 `User.username == username` 直接比较即可，不需要额外处理。例如用户 `Alice` 用小写的 `alice` 登录会返回 401。

> 如果需要不区分大小写，就要在比较和唯一约束两处都做处理（例如统一转小写存储，或使用 `citext` 类型），只改查询不改约束会导致仍能注册大小写不同的重复用户名。

## 13. 小结

* `AsyncEngine` 持有连接池，整个进程一个，在 lifespan 中创建、关闭时 `dispose()`；它是懒连接的，创建时不连数据库。`AsyncSession` 是一次工作单元，每个请求一个。
* 连接池参数：`pool_size` 常驻连接，`max_overflow` 高峰额外连接，`pool_pre_ping` 取连接前探活，`pool_recycle` 定期重建连接。
* 异步模式下 `async_sessionmaker` 要设 `expire_on_commit=False`，否则 commit 后访问属性会触发隐式 IO 而报错。
* 会话依赖用 `async with` + `yield`：请求结束自动关闭、未提交事务自动回滚；不自动 commit，由 service 显式提交。
* 模型用 `DeclarativeBase` + `Mapped[...]` + `mapped_column`；`X | None` 为可空列，其余默认 NOT NULL。
* 在建第一张表前配置 `naming_convention`，让约束名可预测，Alembic 才能修改和删除它们。
* `default` 由 Python 填值，`server_default` 写进表结构由数据库填值；公共字段用 Mixin 复用。
* 主键用 UUID，避免暴露数据量和被遍历；外键加 `ondelete` 和 `index=True`。
* 查询用 2.0 风格：`select().where()` + `scalar_one_or_none()`、`session.get()`、`update().where().values()`、`join()`。
* repository 只 flush，service 决定何时 commit。
* 并发下用唯一约束 + 捕获 `IntegrityError` 防重复插入，用 `with_for_update()` 行锁保护"先读后改"。
