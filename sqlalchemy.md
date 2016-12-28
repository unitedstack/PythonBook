# SQLAlchemy

---

## SQLAlchemy简介 {#articleHeader6}

SQLAlchemy项目是Python中最著名的ORM实现，不仅在Python项目中也得到了广泛的应用，而且对其他语言的ORM有很大的影响。OpenStack一开始选择这个库，也是看中了它足够稳定、足够强大的特点。SQLAlchemy项目的官网是 [http://www.sqlalchemy.org/](http://www.sqlalchemy.org/)。

## SQLAlchemy的基本概念和使用 {#articleHeader10}

使用SQLAlchemy大体上分为三个步骤：连接到数据库，定义数据模型，执行数据操作。

### 基本概念

首先这个是一张SQLAlchemy官网的图。

![](/assets/import.png)

我们发现，Database对外的接口DBAPI而调用DBAPI的时候就要经过Pool或者Dialect模块，但在这个两个模块之前，你必须先声明一个Engine，然后通过连接这个Engine来间接操作DB。

### 连接到数据库-Engine {#articleHeader11}

在你的应用可以使用数据库前，你要先定义好数据库的连接，包括数据库在哪里，用什么账号访问等。所有的这些工作都是通过Engine对象来进行的，所以我们的操作顺序就是数据库URL到创建 Engine 对象，然后调用 Engine对象操作 DB。

#### 数据库URL

SQLAlchemy使用URL的方式来指定要访问的数据库，整个URL的具体格式如下：

```
dialect+driver://username:password@host:port/database
```

其中，dialect就是指DBMS的名称，一般可选的值有：_postgresql_,_mysql_,_sqlite_等。driver就是指驱动的名称，如果不指定，SQLAlchemy会使用默认值。_database_就是指DBMS中的一个数据库，一般是指通过_CREATE DATABASE_语句创建的数据库。我们可以在很多OpenStack组件的配置文件里面找到这样的定义，如nova：

```
connection=mysql+pymysql://nova_api:416801e9ced4496f@192.168.176.254/nova_api
```

这个就是定义了数据库连接的URL。

#### 创建Engine对象

确定了要连接的数据库信息后，就可以通过`create_engine`函数来创建一个Engine对象了。

```
from sqlalchemy import create_engine

engine = create_engine( 'sqlite://:memory:' )
```

`create_engine`函数还支持以下几个参数：

* _connect\_args_：一个字典，用来自定义数据库连接的参数，比如指定客户端使用的字符编码。

* _pool\_size_和_max\_overflow_：指定连接池的大小。

* _echo_：一个布尔值，用来指定是否打印执行的SQL语句到日志中。

具体的参数就不祥系展开了，可以参考官方文档：[Engine Configuration](http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html)。有了Engine之后我们就可以获取数据库的连接了：

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

然后对这个module和你的table做映射，怎么做呢？其实很简单，让module继承Declarative 就好了。如：

```
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

base = declarative_base()


class Stack(base):
    __tablename__ = 'stack'

    id = Column(Interger, primary_key=True)
    name = Column(String(250), nullable=False)
```

这边的column对应数据库的字段，注意的是需要保持字段类型一致。那么我们要怎么创建这个数据库呢?很简单：

### metadata

SQLAlchemy中通过 schema metadata 来实现上面说的Schema。Schema metadata，官方文档中也称为 database metadata，简称为metadata，是一个容器，其中包含了和DDL相关的所有信息，包括Table, Column等对象。当SQLAlchemy要根据映射类生成SQL语句时，它会查询metadata中的信息，根据信息来生成SQL语句。例如，利用metadata来创建表：

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

metadata会自动去生成所有的注册了module的create tables的SQL。现在我们已经学会怎么创建一个表了，那么如何对一个表进行基本的操作呢？如增删改查。首先，我们就需要获取一个数据库连接的session.

### 会话\( session \) {#articleHeader14}

会话\(session\)是我们通过SQLAlchemy来操作数据库的入口。我们前面有介绍过SQLAlchemy的架构，session是属于ORM层的。Session的功能是管理我们的程序和数据库之间的会话，它利用Engine的连接管理功能来实现会话。我们在上文有提到，我们创建了Engine对象，但是一般不直接使用它，而是把它交给ORM去使用。其中，通过session来使用Engine就是一个常用的方式。

要是用session，我们需要先通过`sessionmaker`函数创建一个session类，然后通过这个类的实例来使用会话，如下所示：

```
from sqlalchemy.orm import sessionmaker

DBSession = sessionmaker(bind=engine)
session = DBSession()
```

我们通过`sessionmaker`的 _bind _参数把 Engine 对象传递给`DBSession`去管理。然后，`DBSession`实例化的对象`session`就能被我们使用了。

### 增删改查\(CRUD\)

CRUD就是CREATE, READ, UPDATE, DELETE，增删改查。这个也是SQLAlchemy中最常用的功能，而且都是通过上一小节中的`session`对象来使用的。我们这简单的介绍一下这四个操作，后面会给出官方文档的位置。

#### Create

在数据库中插入一条记录，是通过session的`add()`方法来实现的，你需要先创建一个映射类的实例，然后调用`session.add()`方法，然后调用`session.commit()`方法提交你的事务：

```
new_stack = Stack(name= 
    'new_stack'
)
session.add(new_stack)
session.commit()
```

#### Read

查询操作，一般称为query，在SQLAlchemy中一般是通过Query对象来完成的。我们可以通过`session.query()`方法来创建一个Query对象，然后调用Query对象的众多方法来完成查询操作。

```
get_stack = session.query(models.Stacks)
```

#### Update

更新一条记录需要先使用查询操作获得一条记录对应的对象，然后修改对象的属性，再通过`session.update()`方法来完成更新操作。

```
get_stack = session.query(models.Stacks)
values.update({'updated_at': timeutils.utcnow()})
get_stack.update(value)
```

#### Delete

删除操作和创建操作差不多，是把一个映射类实例传递给`session.delete()`方法。

```
get_stack = session.query(models.Stacks)
session.delete(get_stack)
```

### 事务 {#articleHeader16}

使用session，就会涉及到事务，我们的应用程序也会有很多事务操作的要求。当你调用一个session的方法，导致session执行一条SQL语句时，它会自动开始一个事务，直到你下次调用`session.commit()`或者`session.rollback()`，它就会结束这个事务。你也可以显示的调用`session.begin()`来开始一个事务，并且`session.begin()`还可以配合Python的with来使用。



