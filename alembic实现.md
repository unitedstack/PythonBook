# Alembic 实现


## 安装 Alembic

接下来，我们就在我们的 webdemo 项目中引入 Alembic 来进行版本管理。首先先写依赖，在 _webdemo/requirements.txt_ 中加入：alembic&gt;=0.8.0。

### 在项目中创建 Alembic 的 migration 环境

一般 OpenStack 项目中，Alembic 的环境都是放在 _db/sqlalchemy/_ 目录下，因此，我们先建立目录 _webdemo/db/sqlalchemy/_，然后在这个目录下初始化 Alembic 环境：

```
# mkdir db/sqlalchemy/
# cd db/sqlalchemy/
# alembic init alembic

# tree  .
.
├── alembic
│   ├── env.py
│   ├── README
│   ├── script.py.mako
│   └── versions
│       └── 001_create_user_table.py
└── alembic.ini
```

### 修改 Alembic 配置文件

_webdemo/db/sqlalchemy/alembic.ini_ 文件是 Alembic 的配置文件，我们现在需要修改文件中的 sqlalchemy.url 这个配置项，用来指向我们的数据库。这里，我们使用 SQLite 数据库，数据库文件存放在 webdemo 项目的根目录下，名称是 webdemo.db：

```
# sqlalchemy.url = driver://user:pass@localhost/dbname
sqlalchemy.url = sqlite:///../../../webdemo.db
```

注意：实际项目中，数据库的URL信息是从项目配置文件中读取，然后通过动态的方式传递给 Alembic 的。

### 创建 migration 脚本

现在，我们可以创建第一个迁移脚本了，我们的第一个数据库版本就是创建我们的 user 表：

```
$ alembic revision -m "Create user table"
```

现在脚本已经帮我们生成好了，不过这个只是一个空的脚本，我们需要自己实现里面的具体操作，补充完整后的脚本如下：

```
"""Create user table

Revision ID: 4bafdb464737
Revises:
Create Date: 2016-02-21 12:24:46.640894

"""

# revision identifiers, used by Alembic.
revision = '4bafdb464737'
down_revision = None
branch_labels = None
depends_on = None

from alembic import op
import sqlalchemy as sa


def upgrade():
    op.create_table(
        'user',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('user_id', sa.String(255), nullable=False),
        sa.Column('name', sa.String(64), nullable=False, unique=True),
        sa.Column('email', sa.String(255))
    )


def downgrade():
    op.drop_table('user')
```

其实就是把 `User` 类的定义再写了一遍，使用了 Alembic 提供的接口来方便的创建和删除表。

### 执行迁移操作

我们需要在 _webdemo/db/sqlalchemy/_ 目录下执行迁移操作，可能需要手动指定 PYTHONPATH：

```
$ PYTHONPATH=../../../ alembic upgrade head
INFO  [alembic.migration] Context impl SQLiteImpl.
INFO  [alembic.migration] Will assume non-transactional DDL.
INFO  [alembic.migration] Running upgrade  -> 4bafdb464737, Create user table
```

`alembic upgrade head` 会把数据库升级到最新的版本。这个时候，在 webdemo 的根目录下会出现 webdemo.db 这个文件，可以使用 sqlite3 命令查看内容：

```
sqlite3 webdemo.db
SQLite version 3.8.11.1 2015-07-29 20:00:57
Enter ".help" for usage hints.
sqlite> .tables
alembic_version  user
sqlite> .schema alembic_version
CREATE TABLE alembic_version (
        version_num VARCHAR(32) NOT NULL
);
sqlite> .schema user
CREATE TABLE user (
        id INTEGER NOT NULL,
        user_id VARCHAR(255) NOT NULL,
        name VARCHAR(64) NOT NULL,
        email VARCHAR(255),
        PRIMARY KEY (id),
        UNIQUE (name)
);
sqlite> .header on
sqlite> select * from alembic_version;
version_num
4bafdb464737
```

### 小结 

现在，我们就已经完成了 database migration 代码框架的搭建，可以成功执行了第一个版本的数据库迁移。OpenStack 项目中也是这么来做数据库迁移的。后续，一旦修改了项目，需要修改数据模型时，只要新增 migration 脚本即可。这部分代码也可以在 [https://github.com/zhaozhilong1993/webdemo](https://github.com/zhaozhilong1993/webdemo) 中看到。

在实际生产环境中，当我们发布了一个项目的新版本后，在上线的时候，都会自动执行数据库迁移操作，升级数据库版本到最新的版本。如果线上的数据库版本已经是最新的，那么这个操作没有任何影响；如果不是最新的，那么会把数据库升级到最新的版本。

关于 Alembic 的更多使用方法，请阅读官方文档 Alembic [https://alembic.readthedocs.org/en/latest/](https://alembic.readthedocs.org/en/latest/)。

