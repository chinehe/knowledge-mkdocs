## 1. 为什么需要迁移工具

项目上线后，表结构会一直变：加表、加列、加索引、改默认值。如果靠手工在每个环境执行 SQL，很快会遇到下面的问题：

* 开发库、测试库、生产库各执行了哪些 SQL，没人说得清，某个环境漏执行一条就会出错。
* 新同事拿到代码，不知道该按什么顺序执行哪些 SQL 才能建出一个和线上一致的库。
* 改错了想回退，没有现成的反向 SQL。

**数据库迁移工具**把每次表结构变更写成一个脚本，按顺序管理起来。SQLAlchemy 生态里的迁移工具是 [Alembic](https://alembic.sqlalchemy.org/)，它和 SQLAlchemy 出自同一作者。

### 1.1. Alembic 的做法

| 概念 | 说明 |
|------|------|
| 迁移脚本 | 每次变更是 `versions/` 目录下的一个 Python 文件，包含 `upgrade()`（升级）和 `downgrade()`（回退），和代码一起提交到 git |
| 版本号（revision） | 每个脚本有一个唯一的版本号，如 `42c6de21e591` |
| `down_revision` | 指向上一个版本号，所有脚本由此串成一条链 |
| `alembic_version` 表 | Alembic 在数据库里建的一张表，只有一行，记录这个库当前处于哪个版本 |

执行 `alembic upgrade head` 时，Alembic 先读 `alembic_version` 得到当前版本，再沿着链依次执行之后的每个 `upgrade()`，最后把 `alembic_version` 更新为最新版本。已经执行过的脚本不会再执行。

### 1.2. 版本链

以一个有两个迁移的项目为例，迁移脚本头部是这样写的：

```python
# 20261118_1118_33a373c63bbf_init.py
revision: str = "33a373c63bbf"
down_revision: Union[str, Sequence[str], None] = None   # 第一个迁移，没有上一个版本

# 20261118_1351_42c6de21e591_add_users_and_refresh_tokens.py
revision: str = "42c6de21e591"
down_revision: Union[str, Sequence[str], None] = "33a373c63bbf"   # 指向 init
```

串起来就是：

```mermaid
graph LR
    B["base<br/>空库"] -->|upgrade| R1["33a373c63bbf<br/>init"]
    R1 -->|upgrade| R2["42c6de21e591<br/>add users and refresh_tokens<br/>(head)"]
    R2 -.->|downgrade| R1
    R1 -.->|downgrade| B
```

`alembic history` 输出的也是这条链：

```shell
$ uv run alembic history
33a373c63bbf -> 42c6de21e591 (head), add users and refresh_tokens
<base> -> 33a373c63bbf, init
```

* `<base>` 表示空库（还没有任何迁移）。
* `head` 表示链的最新一端。

## 2. 初始化

### 2.1. alembic init

Alembic 已经作为运行依赖安装（`uv add alembic`）。项目使用异步驱动 asyncpg，所以初始化时选择**异步模板**：

```shell
uv run alembic init -t async migrations
```

| 参数 | 含义 |
|------|------|
| `-t async` | 使用异步模板，生成的 `env.py` 用异步引擎连接数据库 |
| `migrations` | 迁移目录名，默认叫 `alembic`，这里改成更直白的 `migrations` |

生成的文件：

```
my-service/
├── alembic.ini              # Alembic 配置：脚本目录、文件名模板、钩子、日志
└── migrations/
    ├── env.py               # 迁移环境：怎么连数据库、拿哪份元数据做对比
    ├── README
    ├── script.py.mako       # 新迁移脚本的模板
    └── versions/            # 迁移脚本存放目录
```

生成的 `env.py` 只是一个通用模板，需要改造成从应用读取配置和模型，见下一节。

## 3. 改造 env.py

`env.py` 是每条 Alembic 命令都会执行的脚本，它决定了连哪个数据库、拿什么做对比。下面逐段讲解项目改造后的 `migrations/env.py`。

### 3.1. 数据库地址取自应用配置

```python
from alembic import context
from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import create_async_engine

from my_service.core.config import get_settings
from my_service.models import Base

config = context.config

# 与应用使用同一份配置，不在 alembic.ini 里重复写数据库地址和密码
DATABASE_URL = get_settings().database_url.get_secret_value()
```

默认模板从 `alembic.ini` 的 `sqlalchemy.url` 读数据库地址。这样有两个问题：

* 数据库地址要在应用配置（`.env` / 环境变量）和 `alembic.ini` 里各写一份，改了一处忘了另一处。
* `alembic.ini` 要提交到 git，密码写在里面就泄露了。

改成调用应用的 `get_settings()` 后，迁移和应用读的是同一个 `DATABASE_URL`：本地读 `.env`，容器里读环境变量。`database_url` 是 pydantic 的 `SecretStr` 类型，用 `get_secret_value()` 取出原文。

同时把 `alembic.ini` 里 `sqlalchemy.url` 的占位符注释掉（或删掉），避免有人误以为地址是在这里配置的：

```ini
# sqlalchemy.url = driver://user:pass@localhost/dbname
```

> `alembic.ini` 里的 `prepend_sys_path = .` 会把项目根目录加入 `sys.path`。项目是 src 布局，`my_service` 已经通过 `uv sync` 安装进虚拟环境，所以 `env.py` 里可以直接 `import my_service`。

### 3.2. target_metadata：告诉 Alembic 模型长什么样

```python
# autogenerate 会对比这份元数据和数据库的实际结构，生成差异。
# 所有模型都要在 my_service.models 包里导入，否则会被当成"不存在的表"。
target_metadata = Base.metadata
```

`Base.metadata` 收集了所有继承 `Base` 的模型的表结构。默认模板里这一行是 `target_metadata = None`，不改的话 autogenerate 什么都检测不到。

### 3.3. 在线模式：异步引擎 + NullPool

```python
def do_run_migrations(connection: Connection) -> None:
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        compare_server_default=True,  # 列的数据库默认值变化也生成迁移
    )
    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    # 迁移是一次性任务，用 NullPool：不保留连接，跑完即断开
    engine = create_async_engine(DATABASE_URL, poolclass=pool.NullPool)
    async with engine.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await engine.dispose()


def run_migrations_online() -> None:
    asyncio.run(run_async_migrations())
```

执行过程：

```mermaid
graph TD
    A["alembic upgrade head"] --> B["run_migrations_online()"]
    B --> C["asyncio.run(run_async_migrations())"]
    C --> D["create_async_engine(..., NullPool)"]
    D --> E["connection.run_sync(do_run_migrations)"]
    E --> F["context.configure + run_migrations<br/>（同步代码，在一个事务里执行）"]
    F --> G["engine.dispose() 断开连接"]
```

要点：

| 写法 | 原因 |
|------|------|
| `create_async_engine(DATABASE_URL, ...)` | 直接用连接串创建引擎。默认模板是 `async_engine_from_config(config.get_section(...))`，从 ini 里读 `sqlalchemy.url`，这里不再需要（原因见 3.6） |
| `poolclass=pool.NullPool` | 连接池是为长期运行、反复取连接的应用准备的。迁移跑一次就结束，NullPool 每次用完直接关闭连接，不保留空闲连接 |
| `connection.run_sync(...)` | Alembic 的迁移执行逻辑是同步代码，`run_sync` 把异步连接包装成同步连接交给它 |
| `asyncio.run(...)` | 命令行入口是同步的，在这里启动事件循环。这也决定了在代码里调用迁移的限制（见第 8 节） |

### 3.4. compare_server_default=True

`context.configure` 默认**不检测**列的 `server_default`（数据库层面的默认值）变化。比如把模型里的 `server_default=sa.text("true")` 改成 `sa.text("false")`，不开这个选项时 autogenerate 不会生成任何迁移，模型和数据库就悄悄不一致了。

项目的 `is_active`、`created_at` 等列都用了 `server_default`，所以打开这个选项。

### 3.5. 离线模式：只输出 SQL

```python
def run_migrations_offline() -> None:
    """离线模式：不连接数据库，只输出 SQL（alembic upgrade head --sql）。"""
    context.configure(
        url=DATABASE_URL,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

命令带 `--sql` 时进入离线模式：Alembic 不连接数据库，而是把要执行的 SQL 打印出来。`url` 在这里只用来确定数据库方言（PostgreSQL），生成对应语法的 SQL。`literal_binds=True` 把参数直接写进 SQL 文本，输出的脚本可以直接执行。

数据库没启动时也能运行，输出节选：

```shell
$ uv run alembic upgrade head --sql
BEGIN;
CREATE TABLE alembic_version (
    version_num VARCHAR(32) NOT NULL, 
    CONSTRAINT alembic_version_pkc PRIMARY KEY (version_num)
);
-- Running upgrade  -> 33a373c63bbf
INSERT INTO alembic_version (version_num) VALUES ('33a373c63bbf') RETURNING alembic_version.version_num;
-- Running upgrade 33a373c63bbf -> 42c6de21e591
CREATE TABLE users (
    id UUID NOT NULL, 
    username VARCHAR(50) NOT NULL, 
    ...
    CONSTRAINT pk_users PRIMARY KEY (id), 
    CONSTRAINT uq_users_username UNIQUE (username)
);
...
CREATE INDEX ix_refresh_tokens_user_id ON refresh_tokens (user_id);
UPDATE alembic_version SET version_num='42c6de21e591' WHERE alembic_version.version_num = '33a373c63bbf';
COMMIT;
```

生产库不允许程序直接改表结构时，可以把这份 SQL 交给 DBA 审核后执行。

### 3.6. ini 的 % 插值陷阱

有一种常见写法是在 `env.py` 里把应用配置的地址塞回 Alembic 配置，其余代码保持模板原样：

```python
# 不推荐
config.set_main_option("sqlalchemy.url", DATABASE_URL)
```

问题在于 `alembic.ini` 是用 Python 的 `configparser` 解析的，`%` 在里面有**插值**含义（例如 `%(here)s` 会被替换成 ini 文件所在目录）。`set_main_option` 写入的值同样会被当作插值模板处理。

数据库密码里含有特殊字符时，连接串里要做 URL 编码，例如 `@` 编码成 `%40`，这时就会报错：

```python
>>> config.set_main_option("sqlalchemy.url", "postgresql+asyncpg://u:p%40ss@h/db")
ValueError: invalid interpolation syntax in 'postgresql+asyncpg://u:p%40ss@h/db' at position 24
```

解决办法：

| 做法 | 说明 |
|------|------|
| 直接用连接串创建引擎（3.3 节的做法） | `create_async_engine(DATABASE_URL, ...)`，地址不经过 configparser，没有插值问题 |
| 写入前转义 `%` | `config.set_main_option("sqlalchemy.url", url.replace("%", "%%"))`，能用但容易忘 |

同理，`alembic.ini` 里凡是要表示字面量 `%` 的地方都要写成 `%%`，下一节的 `file_template` 就是这样。

## 4. 配置 alembic.ini

### 4.1. file_template：文件名以时间开头

默认的迁移文件名是 `%%(rev)s_%%(slug)s`，即"版本号_描述"，例如 `42c6de21e591_add_users_and_refresh_tokens.py`。版本号是随机的哈希值，在文件管理器或 IDE 里按名字排序时顺序是乱的，看不出哪个先哪个后。

项目改成以时间开头：

```ini
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d_%%(rev)s_%%(slug)s
```

| 占位符 | 含义 |
|--------|------|
| `%%(year)d%%(month).2d%%(day).2d` | 日期，如 `20261118`（`.2d` 表示补齐两位） |
| `%%(hour).2d%%(minute).2d` | 时分，如 `1351` |
| `%%(rev)s` | 版本号 |
| `%%(slug)s` | `-m` 描述转换成的文件名片段 |

`%%` 是 configparser 里字面量 `%` 的写法（见 3.6）。对比效果：

```
# 默认：按名字排序，顺序和创建顺序无关
33a373c63bbf_init.py
42c6de21e591_add_users_and_refresh_tokens.py

# 项目：按名字排序就是时间顺序
20261118_1118_33a373c63bbf_init.py
20261118_1351_42c6de21e591_add_users_and_refresh_tokens.py
```

> 文件名只影响阅读。Alembic 判断执行顺序始终看脚本里的 `revision` / `down_revision`，不看文件名。

### 4.2. post_write_hooks：生成后自动格式化

autogenerate 生成的代码风格不一定符合项目规范（引号、行宽等）。`[post_write_hooks]` 可以在每次生成迁移脚本后自动执行一个命令：

```ini
[post_write_hooks]
hooks = ruff_format
ruff_format.type = module
ruff_format.module = ruff
ruff_format.options = format REVISION_SCRIPT_FILENAME
```

| 配置项 | 含义 |
|--------|------|
| `hooks = ruff_format` | 要执行的钩子名称，可以写多个 |
| `ruff_format.type = module` | 以 `python -m <模块>` 的方式运行，用的是当前虚拟环境里的 ruff |
| `ruff_format.module = ruff` | 模块名 |
| `ruff_format.options = format REVISION_SCRIPT_FILENAME` | 参数，`REVISION_SCRIPT_FILENAME` 会被替换成新生成的文件路径，整体相当于 `python -m ruff format <新文件>` |

生成迁移时会看到：

```
Running post write hook 'ruff_format' ...
1 file reformatted
```

> 注意和第 7 节 ruff 排除配置的冲突：
>
> * **现象**：`pyproject.toml` 的 `[tool.ruff]` 设置了 `extend-exclude = ["migrations/versions"]` 和 `force-exclude = true` 后，生成迁移时钩子照常运行，但新文件没有被格式化，ruff 输出 `warning: No Python files found under the given path(s)`。
> * **原因**：`force-exclude` 会让 ruff 即使在命令行显式传入文件路径，也跳过被排除的文件。钩子传入的新迁移文件位于 `migrations/versions/` 下，因此被直接跳过，钩子等于不生效。
> * **解决办法**：在钩子参数里加 `--no-force-exclude`，ruff 就会正常格式化传入的文件：
>
> ```ini
> ruff_format.options = format --no-force-exclude REVISION_SCRIPT_FILENAME
> ```

## 5. 生成迁移：autogenerate

### 5.1. 工作方式

```shell
uv run alembic revision --autogenerate -m "add users and refresh_tokens"
```

Alembic 连接数据库，读取实际的表结构，和 `target_metadata`（模型）做对比，把差异写成 `upgrade()` / `downgrade()` 里的 `op.xxx` 调用：

```mermaid
graph LR
    M["target_metadata<br/>（ORM 模型）"] --> C{"对比"}
    D["数据库实际结构<br/>（连接后读取）"] --> C
    C --> F["新迁移脚本<br/>upgrade() / downgrade()"]
```

输出节选（省略了日志前缀）：

```
Detected added table 'users'
Detected added table 'refresh_tokens'
Detected added index 'ix_refresh_tokens_family_id' on '('family_id',)'
Detected added index 'ix_refresh_tokens_user_id' on '('user_id',)'
Generating .../migrations/versions/20261118_1351_42c6de21e591_add_users_and_refresh_tokens.py ...  done
Running post write hook 'ruff_format' ...
1 file reformatted
```

生成的脚本（节选）：

```python
def upgrade() -> None:
    """Upgrade schema."""
    # ### commands auto generated by Alembic - please adjust! ###
    op.create_table(
        "users",
        sa.Column("id", sa.Uuid(), nullable=False),
        sa.Column("username", sa.String(length=50), nullable=False),
        sa.Column("is_active", sa.Boolean(), server_default=sa.text("true"), nullable=False),
        ...
        sa.PrimaryKeyConstraint("id", name=op.f("pk_users")),
        sa.UniqueConstraint("username", name=op.f("uq_users_username")),
    )
    op.create_table(
        "refresh_tokens",
        ...
        sa.ForeignKeyConstraint(
            ["user_id"],
            ["users.id"],
            name=op.f("fk_refresh_tokens_user_id_users"),
            ondelete="CASCADE",
        ),
        ...
    )
    op.create_index(op.f("ix_refresh_tokens_user_id"), "refresh_tokens", ["user_id"], unique=False)
    # ### end Alembic commands ###


def downgrade() -> None:
    """Downgrade schema."""
    # ### commands auto generated by Alembic - please adjust! ###
    op.drop_index(op.f("ix_refresh_tokens_user_id"), table_name="refresh_tokens")
    op.drop_index(op.f("ix_refresh_tokens_family_id"), table_name="refresh_tokens")
    op.drop_table("refresh_tokens")
    op.drop_table("users")
    # ### end Alembic commands ###
```

注释 `please adjust!` 的意思是：这是自动生成的草稿，提交前要人工检查。

### 5.2. 空的基线迁移

项目的第一个迁移是在还没有任何模型时生成的，`upgrade()` 和 `downgrade()` 都是 `pass`：

```python
"""init

Revision ID: 33a373c63bbf
Revises:
Create Date: 2026-11-18 11:18:42.738457
"""

revision: str = "33a373c63bbf"
down_revision: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    """Upgrade schema."""
    pass


def downgrade() -> None:
    """Downgrade schema."""
    pass
```

它不改任何表，只作为版本链的起点：配好 `env.py` 后先生成它并执行一次，可以验证 Alembic 能连上数据库、`alembic_version` 表能正常创建。之后每次加模型，都在它后面接新的迁移。

### 5.3. 局限：生成后必须人工检查

autogenerate 只是对比"现在长什么样"，不知道"你想做什么"。常见的坑：

| 情况 | autogenerate 的结果 | 处理 |
|------|--------------------|------|
| 列或表改名 | 识别成"删掉旧列 + 新增新列"，执行后**旧列的数据会丢失** | 手工改成 `op.alter_column("users", "name", new_column_name="username")` |
| 新模型没在 `models/__init__.py` 导入 | 模型所在模块没被加载，表不在 `Base.metadata` 里，检测不到；如果数据库里已有这张表，甚至会生成删表语句 | 新增模型后在 `models/__init__.py` 里导入 |
| 约束没有固定名字 | 数据库自动起名，之后修改或删除约束时迁移里写不出名字 | 在 `Base` 上配置 `naming_convention`（项目已配置，所以生成了 `pk_users`、`fk_refresh_tokens_user_id_users` 这类名字） |
| 外键的 `ondelete` | 只会照抄模型，模型里漏写就没有 | 对照业务确认，例如 `refresh_tokens.user_id` 需要 `ondelete="CASCADE"` |
| `downgrade()` 的删除顺序 | 通常正确，但手工改过 `upgrade()` 后要同步调整 | 有外键依赖时先删子表再删父表：先 `refresh_tokens`，后 `users` |

项目的 `models/__init__.py` 统一导入所有模型，`env.py` 只需要 `from my_service.models import Base`：

```python
"""ORM 模型。新增模型后在这里导入，Alembic 才能发现它。"""

from my_service.models.base import Base
from my_service.models.refresh_token import RefreshToken
from my_service.models.user import User

__all__ = ["Base", "RefreshToken", "User"]
```

## 6. 常用命令

| 命令 | 作用 |
|------|------|
| `uv run alembic revision --autogenerate -m "描述"` | 对比模型和数据库，生成新迁移脚本 |
| `uv run alembic upgrade head` | 升级到最新版本 |
| `uv run alembic downgrade -1` | 回退一个版本（执行当前版本的 `downgrade()`） |
| `uv run alembic current` | 查看数据库当前所处的版本 |
| `uv run alembic history` | 查看版本链 |
| `uv run alembic check` | 检查模型和数据库是否一致，有未生成的变更时返回非零退出码 |
| `uv run alembic upgrade head --sql` | 不连数据库，只输出要执行的 SQL |

`alembic check` 在模型和数据库一致时输出：

```
No new upgrade operations detected.
```

适合放进 CI：数据库先 `upgrade head`，再 `check`。如果有人改了模型却忘了生成迁移，CI 就会失败。

> 除 `history` 和 `upgrade head --sql` 外，这些命令都要连接数据库。数据库没启动时会报 `Connect call failed ('127.0.0.1', 5432)`。

## 7. 迁移文件的维护规则

### 7.1. 已执行过的迁移不再修改

迁移脚本一旦提交并在别的环境（同事的库、测试库、生产库）执行过，就**不要再改**它的内容：

* 那些环境的 `alembic_version` 已经记录了这个版本，Alembic 不会重新执行它，改动永远不会生效。
* 结果是新环境（按改后的脚本建库）和老环境（按改前的脚本建库）结构不一致。

需要调整时，新建一个迁移来修正。只有在本地生成、还没提交也没在别处执行过的迁移，才可以直接改或删掉重新生成。

### 7.2. 让格式工具不碰旧迁移

格式化工具升级或规则调整后，可能会批量改写所有 `.py` 文件，旧迁移也会被"顺手"改掉，产生没有意义的 diff。项目在 `pyproject.toml` 里把迁移目录排除：

```toml
[tool.ruff]
extend-exclude = ["migrations/versions"]   # 自动生成的迁移文件不检查
force-exclude = true   # 即使显式传入文件路径（如 pre-commit），也遵守上面的排除规则
```

| 配置 | 作用 |
|------|------|
| `extend-exclude` | 在 ruff 默认排除列表基础上追加目录，`ruff check .` / `ruff format .` 扫描时跳过 |
| `force-exclude` | pre-commit 会把改动的文件路径逐个传给 ruff，默认情况下显式传入的文件不受 exclude 约束；开启后同样跳过 |

这个配置对 4.2 的 post write hook 也生效，需要的话按 4.2 的说明加 `--no-force-exclude`。

## 8. 在代码里调用迁移

除了命令行，Alembic 也可以在 Python 代码里调用。项目的测试用它来建表：

```python
# tests/conftest.py
from alembic import command
from alembic.config import Config


@pytest.fixture(scope="session", autouse=True)
def _test_database() -> None:
    """整个测试会话只执行一次：重建测试库，用 Alembic 迁移建表。

    用迁移而不是 Base.metadata.create_all 建表，顺便验证了迁移脚本本身。
    """
    asyncio.run(_recreate_test_database())
    command.upgrade(Config("alembic.ini"), "head")
```

* `Config("alembic.ini")` 用的是相对路径，相对于当前工作目录。在项目根目录执行 `uv run pytest` 时能找到。
* `command.upgrade(config, "head")` 等价于命令行的 `alembic upgrade head`。`command` 模块里还有 `downgrade`、`current`、`check` 等，与命令行一一对应。
* 测试库地址在 conftest 开头通过环境变量 `DATABASE_URL` 覆盖，`env.py` 里的 `get_settings()` 读到的就是测试库（conftest 里调用了 `get_settings.cache_clear()` 清掉缓存）。

**必须在同步上下文中调用。** 这个 fixture 是普通的 `def`，不是 `async def`，原因是 `env.py` 内部会执行 `asyncio.run(...)`（见 3.3），而 `asyncio.run` 不能在已经运行的事件循环里调用。如果把它写进 `async def` 的 fixture 或测试里，会报错：

```
RuntimeError: asyncio.run() cannot be called from a running event loop
```

## 9. 部署

### 9.1. 迁移作为一次性任务执行

不要在应用启动时（例如 FastAPI 的 lifespan 里）执行迁移：

* 多副本部署时，几个实例同时启动，会同时去执行同一批迁移，可能互相冲突。
* 迁移失败时应用也起不来，而且迁移日志和应用日志混在一起，不好排查。

正确的做法是把迁移拆成一个独立的、执行完就退出的任务，成功后再启动应用。

### 9.2. docker compose：migrate 服务

```yaml
services:
  migrate:
    profiles: ["app"]
    build: .
    image: my-service:local
    command: ["alembic", "upgrade", "head"]
    environment: &app-env
      APP_ENV: prod
      # 容器之间用服务名 postgres 访问，不是 localhost
      DATABASE_URL: postgresql+asyncpg://postgres:postgres@postgres:5432/my_service
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:?请在 .env 中设置 JWT_SECRET_KEY}
    depends_on:
      postgres:
        condition: service_healthy

  app:
    profiles: ["app"]
    image: my-service:local
    environment: *app-env
    depends_on:
      migrate:
        condition: service_completed_successfully   # 迁移成功后才启动应用
```

```mermaid
graph LR
    P["postgres<br/>service_healthy"] --> M["migrate<br/>alembic upgrade head<br/>执行完退出"]
    M -->|service_completed_successfully| A["app<br/>启动应用"]
```

* `migrate` 和 `app` 用同一个镜像、同一份环境变量（YAML 锚点 `&app-env` / `*app-env`），迁移连的就是应用要用的库。
* 镜像里要包含迁移所需的文件。项目的 Dockerfile 在运行阶段单独复制了它们：

  ```dockerfile
  COPY --chown=app:app alembic.ini ./
  COPY --chown=app:app migrations ./migrations
  ```

* `migrate` 失败（非零退出码）时，`app` 不会启动。

### 9.3. Kubernetes

| 方式 | 说明 |
|------|------|
| Job | 部署前单独运行一个 Job 执行 `alembic upgrade head`，整个集群只执行一次，多副本时推荐 |
| initContainer | 在应用 Pod 里加一个先于应用运行的容器执行迁移。注意它是**每个 Pod 都会执行一次**，多副本同时启动时仍会并发迁移 |

### 9.4. 迁移可以重复执行

`alembic upgrade head` 是幂等的：Alembic 先读 `alembic_version`，只执行比当前版本新的迁移；数据库已经是最新版本时什么也不做，直接成功退出。所以每次部署都无条件执行一遍 `migrate` 是安全的。

## 10. 小结

* Alembic 把每次表结构变更写成带版本号的 Python 脚本，通过 `revision` / `down_revision` 串成链，数据库里的 `alembic_version` 表记录当前版本。
* 用 `alembic init -t async migrations` 初始化，然后改造 `env.py`：地址从 `get_settings()` 读取，`target_metadata = Base.metadata`，`create_async_engine` + `NullPool`，打开 `compare_server_default`。
* `alembic.ini` 由 configparser 解析，`%` 有插值含义，所以项目不用 `set_main_option` 传连接串，而是直接用连接串建引擎；ini 里的字面量 `%` 写成 `%%`。
* `file_template` 让文件名以时间开头；`post_write_hooks` 在生成后自动执行 `ruff format`，和 `force-exclude` 同时使用时要加 `--no-force-exclude`。
* autogenerate 只是对比模型和数据库，识别不了改名，检测不到没导入的模型，生成后必须人工检查约束名、`ondelete` 和 `downgrade()` 顺序。
* 已在别处执行过的迁移不再修改，用新迁移修正；ruff 用 `extend-exclude` + `force-exclude` 不碰旧迁移。
* 代码里用 `command.upgrade(Config("alembic.ini"), "head")` 调用，必须在同步上下文中执行。
* 部署时迁移作为一次性任务（compose 的 migrate 服务、K8s Job）先于应用执行；`upgrade head` 可以重复执行。
