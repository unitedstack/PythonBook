# SQLAlchemy

## SQLAlchemy 简介

SQLAlchemy 项目是 Python 中最著名的 ORM 实现，不仅在 Python 项目中也得到了广泛的应用，而且对其他语言的 ORM 有很大的影响。OpenStack 一开始选择这个库，也是看中了它足够稳定、足够强大的特点。SQLAlchemy 项目的官网是 [http://www.sqlalchemy.org/](http://www.sqlalchemy.org/)。

## SQLAlchemy 的基本概念和使用

使用 SQLAlchemy 大体上分为三个步骤：连接到数据库，定义数据模型，执行数据操作。

### 基本概念

首先这个是一张 SQLAlchemy 官网的图。

![](/assets/import.png)

我们发现，Database 对外的接口 DBAPI 而调用 DBAPI 的时候就要经过 Pool 或者 Dialect 模块，但在这个两个模块之前，你必须先声明一个 Engine，然后通过连接这个 Engine 来间接操作 DB。

### 连接到数据库-Engine

在你的应用可以使用数据库前，你要先定义好数据库的连接，包括数据库在哪里，用什么账号访问等。所有的这些工作都是通过 Engine 对象来进行的，所以我们的操作顺序就是数据库URL到创建 Engine 对象，然后调用 Engine 对象操作 DB。

#### 数据库URL

SQLAlchemy 使用URL的方式来指定要访问的数据库，整个URL的具体格式如下：

```
dialect+driver://username:password@host:port/database
```

其中，dialect 就是指 DBMS 的名称，一般可选的值有：_postgresql_, _mysql_, _sqlite_ 等。driver 就是指驱动的名称，如果不指定，SQLAlchemy 会使用默认值。_database_ 就是指 DBMS 中的一个数据库，一般是指通过 _CREATE DATABASE_ 语句创建的数据库。我们可以在很多 OpenStack 组件的配置文件里面找到这样的定义，如 nova：

```
connection=mysql+pymysql://nova_api:416801e9ced4496f@192.168.176.254/nova_api
```

这个就是定义了数据库连接的URL。

#### 创建Engine 对象

确定了要连接的数据库信息后，就可以通过 `create_engine` 函数来创建一个 Engine 对象了。

```
from sqlalchemy import create_engine

engine = create_engine( 'sqlite://:memory:' )
```

`create_engine`函数还支持以下几个参数：

* _connect\_args_
    **一个字典，用来自定义数据库连接的参数，比如指定客户端使用的字符编码。**

* _pool\_size_和_max\_overflow_
    **指定连接池的大小。**

* _echo_
    **一个布尔值，用来指定是否打印执行的SQL语句到日志中。**

具体的参数就不祥系展开了，可以参考官方文档：Engine Configuration [http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html)。有了 Engine 之后我们就可以获取数据库的连接了：

```
connect = engine.connect()
```

#### Declarative 系统

我们在上面获取了一个数据库的连接，但是我们到现在为止，并没有告诉我们的数据库我们的模型怎么映射我们数据库中的表结构。所以，我们需要定义这些映射结构，相比很久之前的复杂的定义流程，Declarative 要简单许多。首先我们要明白，我们现在是需要把我们的module和我们的数据库的table结构做映射，所以说，你要先定义一个module,如：

```
from sqlalchemy import Column, Integer, String

class Person():
    __tablename__ = 'person'

    id = Column(Interger, primary_key=True)
    name = Column(String(250), nullable=False)
```

然后对这个 module 和你的 table 做映射，怎么做呢？其实很简单，让 module 继承 Declarative 就好了。如：

```
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

base = declarative_base()


class Stack(base):
    __tablename__ = 'stack'

    id = Column(Interger, primary_key=True)
    name = Column(String(250), nullable=False)
```

这边的 column 对应数据库的字段，注意的是需要保持字段类型一致。那么我们要怎么创建这个数据库呢?很简单：

### metadata

SQLAlchemy中通过 schema metadata 来实现上面说的 Schema。Schema metadata，官方文档中也称为 database metadata，简称为 metadata，是一个容器，其中包含了和 DDL 相关的所有信息，包括 Table, Column 等对象。当 SQLAlchemy 要根据映射类生成 SQL 语句时，它会查询 metadata 中的信息，根据信息来生成 SQL 语句。例如，利用 metadata 来创建表：

```
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

base = declarative_base()


class Stack(base):
    __tablename__ = 'stack'

    id = Column(Interger, primary_key=True)
    name = Column(String(250), nullable=False)

engine = create_engine('sqlite:///:memory:', echo=True)
Base.metadata.create_all(engine)
```

metadata 会自动去生成所有的注册了module 的 create tables 的 SQL。现在我们已经学会怎么创建一个表了，那么如何对一个表进行基本的操作呢？如增删改查。首先，我们就需要获取一个数据库连接的 session.

### 会话 \( session \) 

会话 \(session\) 是我们通过 SQLAlchemy 来操作数据库的入口。我们前面有介绍过 SQLAlchemy 的架构，session 是属于 ORM 层的。Session 的功能是管理我们的程序和数据库之间的会话，它利用 Engine 的连接管理功能来实现会话。我们在上文有提到，我们创建了 Engine 对象，但是一般不直接使用它，而是把它交给 ORM 去使用。其中，通过 session 来使用 Engine 就是一个常用的方式。

要是用 session，我们需要先通过 `sessionmaker` 函数创建一个 session 类，然后通过这个类的实例来使用会话，如下所示：

```
from sqlalchemy.orm import sessionmaker

DBSession = sessionmaker(bind=engine)
session = DBSession()
```

我们通过 `sessionmaker` 的  _bind_ 参数把 Engine 对象传递给 `DBSession` 去管理。然后，`DBSession` 实例化的对象 `session` 就能被我们使用了。

### 增删改查\(CRUD\)

CRUD 就是 CREATE, READ, UPDATE, DELETE，增删改查。这个也是 SQLAlchemy 中最常用的功能，而且都是通过上一小节中的 `session` 对象来使用的。我们这简单的介绍一下这四个操作，后面会给出官方文档的位置。

#### Create

在数据库中插入一条记录，是通过 session 的 `add()` 方法来实现的，你需要先创建一个映射类的实例，然后调用 `session.add()` 方法，然后调用 `session.commit()` 方法提交你的事务：

```
new_stack = Stack(name=
    'new_stack'
)
session.add(new_stack)
session.commit()
```

#### Read

查询操作，一般称为 query，在 SQLAlchemy 中一般是通过 Query 对象来完成的。我们可以通过 `session.query()` 方法来创建一个 Query 对象，然后调用 Query 对象的众多方法来完成查询操作。

```
get_stack = session.query(models.Stacks)
```

#### Update

更新一条记录需要先使用查询操作获得一条记录对应的对象，然后修改对象的属性，再通过 `session.update()` 方法来完成更新操作。

```
get_stack = session.query(models.Stacks)
values.update({'updated_at': timeutils.utcnow()})
get_stack.update(value)
```

#### Delete

删除操作和创建操作差不多，是把一个映射类实例传递给 `session.delete()` 方法。

```
get_stack = session.query(models.Stacks)
session.delete(get_stack)
```

### 事务

使用 session，就会涉及到事务，我们的应用程序也会有很多事务操作的要求。当你调用一个 session 的方法，导致 session 执行一条SQL语句时，它会自动开始一个事务，直到你下次调用 `session.commit()` 或者 `session.rollback()`，它就会结束这个事务。你也可以显示的调用 `session.begin()` 来开始一个事务，并且 `session.begin()` 还可以配合 Python 的 with 来使用。



