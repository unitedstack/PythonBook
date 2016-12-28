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

---

## 设计我们的数据库 {#articleHeader1}

我们这边的数据库设计是为了上面讲的WSGI对应起来的，毕竟这个是两个最重要的部分。在开发数据库应用前，需要先定义好数据模型。因为本文只是要演示SQLAlchemy的应用，所以我们定义个最简单的数据模型。user表的定义如下：

* id: 主键，一般由数据库的自增类型实现。

* user\_id: user id，是一个UUID字符串，是OpenStack中最常用来标记资源的方式，全局唯一，并且为该字段建立索引。

* name: user的名称，允许修改，全局唯一，不能为空。

* email: user的email，允许修改，可以为空。

## 搭建数据库层的代码框架 {#articleHeader2}

OpenStack项目中我见过两种数据库的代码框架分隔，一种是Keystone的风格，它把一组API的API代码和数据库代码都放在同一个目录下，如下所示：

由于webdemo采用的是Pecan框架，而且把数据库操作的代码放到同一个目录下也会比较清晰，所以我们采用和Magnum项目相同的方式来编写数据库相关的代码，创建webdemo/db目录，然后把数据库操作的相关代码都放在这个目录下，如下所示：

由于webdemo项目还没有使用oslo\_db库，所以代码看起来比较直观，没有Magnum项目复杂。接下来，我们就要开始写数据库操作的相关代码，分为两个步骤：

1. 在_db/models.py_中定义`User`类，对应数据库的user表。

2. 在_db/api.py_中实现一个`Connection`类，这个类封装了所有的数据库操作接口。我们会在这个类中实现对user表的CRUD等操作。

### 定义User数据模型映射类 {#articleHeader3}

_db/models.py_中的代码如下：

```
from
 sqlalchemy 
import
 Column, Integer, String

from
 sqlalchemy.ext 
import
 declarative

from
 sqlalchemy 
import
 Index


Base = declarative.declarative_base()



class
User
(Base)
:
"""User table"""


    __tablename__ = 
'user'

    __table_args__ = (
        Index(
'ix_user_user_id'
, 
'user_id'
),
    )
    id = Column(Integer, primary_key=
True
)
    user_id = Column(String(
255
), nullable=
False
)
    name = Column(String(
64
), nullable=
False
, unique=
True
)
    email = Column(String(
255
))
```

我们按照我们之前定义的数据模型，实现了映射类。

### 实现DB API {#articleHeader4}

#### DB通用函数

在_db/api.py_中，我们先定义了一些通用函数，代码如下：

```
from
 sqlalchemy 
import
 create_engine

import
 sqlalchemy.orm

from
 sqlalchemy.orm 
import
 exc


from
 webdemo.db 
import
 models 
as
 db_models


_ENGINE = 
None

_SESSION_MAKER = 
None
def
get_engine
()
:
global
 _ENGINE

if
 _ENGINE 
is
not
None
:

return
 _ENGINE

    _ENGINE = create_engine(
'sqlite://'
)
    db_models.Base.metadata.create_all(_ENGINE)

return
 _ENGINE



def
get_session_maker
(engine)
:
global
 _SESSION_MAKER

if
 _SESSION_MAKER 
is
not
None
:

return
 _SESSION_MAKER

    _SESSION_MAKER = sqlalchemy.orm.sessionmaker(bind=engine)

return
 _SESSION_MAKER



def
get_session
()
:

    engine = get_engine()
    maker = get_session_maker(engine)
    session = maker()


return
 session
```

上面的代码中，我们定义了三个函数：

* `get_engine`：返回全局唯一的engine，不需要重复分配。

* `get_session_maker`：返回全局唯一的session maker，不需要重复分配。

* `get_session`：每次返回一个新的session，因为一个session不能同时被两个数据库客户端使用。

这三个函数是使用SQLAlchemy中经常会封装的，所以OpenStack的oslo\_db项目就封装了这些函数，供所有的OpenStack项目使用。

这里需要注意一个地方，在`get_engine()`中：

```
_ENGINE
 = create_engine(
'sqlite://'
)
    db_models.Base.metadata.create_all(_ENGINE)
```

我们使用了sqlite内存数据库，并且立刻创建了所有的表。这么做只是为了演示方便。在实际的项目中，`create_engine()`的数据库URL参数应该是从配置文件中读取的，而且也不能在创建engine后就创建所有的表（这样数据库的数据都丢了）。要解决在数据库中建表的问题，就要先了解数据库版本管理的知识，也就是database migration，我们在下文中会说明。

#### Connection实现

`Connection`的实现就简单得多了，直接看代码。这里只实现了`get_user()`和`list_users()`方法。

```
class
Connection
(object)
:
def
__init__
(self)
:
pass
def
get_user
(self, user_id)
:

        query = get_session().query(db_models.User).filter_by(user_id=user_id)

try
:
            user = query.one()

except
 exc.NoResultFound:

# TODO(developer): process this situation
pass
return
 user


def
list_users
(self)
:

        session = get_session()
        query = session.query(db_models.User)
        users = query.all()


return
 users


def
update_user
(self, user)
:
pass
def
delete_user
(self, user)
:
pass
```

### 在API Controller中使用DB API {#articleHeader5}

现在我们有了DB API，接下来就是要在Controller中使用它。对于使用Pecan框架的应用来说，我们定义一个Pecan hook，这个hook在每个请求进来的时候实例化一个db的`Connection`对象，然后在controller代码中我们可以直接使用这个`Connection`实例。关于Pecan hook的相关信息，请查看[Pecan官方文档](http://pecan.readthedocs.org/en/latest/hooks.html)。

首先，我们要实现这个hook，并且加入到app中。hook的实现代码在_webdemo/api/hooks.py_中：

```
from
 pecan 
import
 hooks


from
 webdemo.db 
import
 api 
as
 db_api



class
DBHook
(hooks.PecanHook)
:
"""Create a db connection instance."""
def
before
(self, state)
:

        state.request.db_conn = db_api.Connection()
```

然后，修改_webdemo/api/app.py_中的`setup_app()`方法：

```
def
setup_app
()
:

    config = get_pecan_config()

    app_hooks = [hooks.DBHook()]
    app_conf = dict(config.app)
    app = pecan.make_app(
        app_conf.pop(
'root'
),
        logging=getattr(config, 
'logging'
, {}),
        hooks=app_hooks,
        **app_conf
    )


return
 app
```

现在，我们就可以在controller使用DB API了。我们这里要重新实现[API服务\(4\)](https://segmentfault.com/a/1190000004004179)实现的_GET /v1/users_这个接口：

    现在，我们就已经完整的实现了这个API，客户端访问API时是从数据库拿数据，而不是返回一个模拟的数据。读者可以使用[API服务\(4\)](https://segmentfault.com/a/1190000004004179)中的方法运行测试服务器来测试这个API。注意：由于数据库操作依赖于SQLAlchemy库，所以需要把它添加到_requirement.txt_中：SQLAlchemy&lt;1.1.0,&gt;=0.9.9。

        ## 小结 {#articleHeader6}

        现在我们已经完成了由于webdemo采用的是Pecan框架，而且把数据库操作的代码放到同一个目录下也会比较清晰，所以我们采用和Magnum项目相同的方式来编写数据库相关的代码，创建webdemo/db目录，然后把数据库操作的相关代码都放在这个目录下，如下所示：

        ![](https://sf-static.b0.upaiyun.com/v-5860a40c/global/img/squares.svg)

        由于webdemo项目还没有使用oslo\_db库，所以代码看起来比较直观，没有Magnum项目复杂。接下来，我们就要开始写数据库操作的相关代码，分为两个步骤：

        1. 在_db/models.py_中定义`User`类，对应数据库的user表。

        2. 在_db/api.py_中实现一个`Connection`类，这个类封装了所有的数据库操作接口。我们会在这个类中实现对user表的CRUD等操作。

        ### 定义User数据模型映射类 {#articleHeader3}

        _db/models.py_中的代码如下：

我们按照我们之前定义的数据模型，实现了映射类。

### 实现DB API {#articleHeader4}

#### DB通用函数

在_db/api.py_中，我们先定义了一些通用函数，代码如下：

    上面的代码中，我们定义了三个函数：

        * `get_engine`：返回全局唯一的engine，不需要重复分配。

        * `get_session_maker`：返回全局唯一的session maker，不需要重复分配。

        * `get_session`：每次返回一个新的session，因为一个session不能同时被两个数据库客户端使用。

        这三个函数是使用SQLAlchemy中经常会封装的，所以OpenStack的oslo\_db项目就封装了这些函数，供所有的OpenStack项目使用。

        这里需要注意一个地方，在`get_engine()`中：

    \_ENGINE  
     = create\_engine\(  
    'sqlite://'  
    \)  
        db\_models.Base.metadata.create\_all\(\_ENGINE\)

        我们使用了sqlite内存数据库，并且立刻创建了所有的表。这么做只是为了演示方便。在实际的项目中，`create_engine()`的数据库URL参数应该是从配置文件中读取的，而且也不能在创建engine后就创建所有的表（这样数据库的数据都丢了）。要解决在数据库中建表的问题，就要先了解数据库版本管理的知识，也就是database migration，我们在下文中会说明。

        #### Connection实现

        `Connection`的实现就简单得多了，直接看代码。这里只实现了`get_user()`和`list_users()`方法。

        ### 在API Controller中使用DB API {#articleHeader5}

        现在我们有了DB API，接下来就是要在Controller中使用它。对于使用Pecan框架的应用来说，我们定义一个Pecan hook，这个hook在每个请求进来的时候实例化一个db的`Connection`对象，然后在controller代码中我们可以直接使用这个`Connection`实例。关于Pecan hook的相关信息，请查看[Pecan官方文档](http://pecan.readthedocs.org/en/latest/hooks.html)。

        首先，我们要实现这个hook，并且加入到app中。hook的实现代码在_webdemo/api/hooks.py_中：

    from  
     pecan   
    import  
     hooks

    from  
     webdemo.db   
    import  
     api   
    as  
     db\_api

    class  
    DBHook  
    \(hooks.PecanHook\)  
    :  
    """Create a db connection instance."""  
    def  
    before  
    \(self, state\)  
    :

```
state.request.db_conn = db_api.Connection()
```

    然后，修改_webdemo/api/app.py_中的`setup_app()`方法：

    def setup\_app\(\):  
        config = get\_pecan\_config\(\)

app\_hooks = \[hooks.DBHook\(\)\]  
app\_conf = dict\(config.app\)  
app = pecan.make\_app\(  
    app\_conf.pop\('root'\),  
    logging=getattr\(config, 'logging', {}\),  
    hooks=app\_hooks,  
    \*\*app\_conf  
\)

return app

```
现在，我们就可以在controller使用DB API了。我们这里要重新实现[API服务\(4\)](https://segmentfault.com/a/1190000004004179)实现的_GET /v1/users_这个接口：

...  
class User\(wtypes.Base\):  
    id = int  
    user\_id = wtypes.text  
    name = wtypes.text  
    email = wtypes.text

class Users\(wtypes.Base\):  
    users = \[User\]  
...  
class UsersController\(rest.RestController\):
```

@pecan.expose\(\)  
def \_lookup\(self, user\_id, \*remainder\):  
    return UserController\(user\_id\), remainder

@expose.expose\(Users\)  
def get\(self\):  
    db\_conn = request.db\_conn     \# 获取DBHook中创建的Connection实例  
    users = db\_conn.list\_users\(\)  \# 调用所需的DB API  
    users\_list = \[\]  
    for user in users:  
        u = User\(\)  
        u.id = user.id  
        u.user\_id = user.user\_id  
        u.name = user.name  
        u.email = user.email  
        users\_list.append\(u\)  
    return Users\(users=users\_list\)

@expose.expose\(None, body=User, status\_code=201\)  
def post\(self, user\):  
    print user  
\`\`\`

\`\`\`

现在，我们就已经完整的实现了这个API，客户端访问API时是从数据库拿数据，而不是返回一个模拟的数据。读者可以使用[API服务\(4\)](https://segmentfault.com/a/1190000004004179)中的方法运行测试服务器来测试这个API。注意：由于数据库操作依赖于SQLAlchemy库，所以需要把它添加到_requirement.txt_中：SQLAlchemy&lt;1.1.0,&gt;=0.9.9。

## 小结 {#articleHeader6}

现在我们已经完成了数据库层的代码框架搭建，读者可以大概了解到一个OpenStack项目中是如何进行数据库操作的。上面的代码可以到[https://github.com/diabloneo/webdemo](https://github.com/diabloneo/webdemo)下载。数据库层的代码框架搭建，读者可以大概了解到一个OpenStack项目中是如何进行数据库操作的。上面的代码可以到[https://github.com/diabloneo/webdemo](https://github.com/diabloneo/webdemo)下载。

