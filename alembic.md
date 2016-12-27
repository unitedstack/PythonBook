## 数据库版本管理的概念 {#articleHeader8}

上面我们在`get_engine()`函数中使用了内存数据库，并且创建了所有的表。在实际项目中，这么做肯定是不行的：

1. 实际项目中不会使用内存数据库，这种数据库一般只是在单元测试中使用。

2. 如果每次`create_engine`都把数据库的表重新创建一次，那么数据库中的数据就丢失了，绝对不可容忍。

解决这个问题的办法也很简单：不使用内存数据库，并且在运行项目代码前先把数据库中的表都建好。这么做确实是解决了问题，但是看起来有点麻烦：

1. 如果每次都手动写SQL语句来创建数据库中的表，会很容易出错，而且很麻烦。

2. 如果项目修改了数据模型，那么不能简单的修改建表的SQL语句，因为重新建表会让数据丢失。我们只能增加新的SQL语句来修改现有的数据库。

3. 最关键的是：我们怎么知道一个正在生产运行的数据库是要执行那些SQL语句？如果数据库第一次使用，那么执行全部的语句是正确的；如果数据库已经在使用，里面有数据，那么我们只能执行那些修改表定义的SQL语句，而不能执行那些重新建表的SQL语句。

为了解决这种问题，就有人发明了数据库版本管理的概念，也称为Database Migration。基本原理是：在我们要使用的数据库中建立一张表，里面保存了数据库的当前版本，然后我们在代码中为每个数据库版本写好所需的SQL语句。当对一个数据库执行migration操作时，会执行从当前版本到目标版本之间的所有SQL语句。举个例子：

1. 在_Version 1_时，我们在数据库中建立一个user表。

2. 在_Version 2_时，我们在数据库中建立一个project表。

3. 在_Version 3_时，我们修改user表，增加一个age列。

那么在我们对一个数据库执行migration操作，数据库的当前版本_Version 1_，我们设定的目标版本是_Version 3_，那么操作就是：建立一个project表，修改user表，增加一个age列，并且把数据库当前版本设置为_Version 3_。

数据库的版本管理是所有大型数据库项目的需求，每种语言都有自己的解决方案。OpenStack中主要使用SQLAlchemy的两种解决方案：[sqlalchemy-migrate](https://github.com/openstack/sqlalchemy-migrate)和[Alembic](https://alembic.readthedocs.org/en/latest/)。早期的OpenStack项目使用了sqlalchemy-migrate，后来换成了Alembic。做出这个切换的主要原因是Alembic对数据库版本的设计和管理更灵活，可以支持分支，而sqlalchemy-migrate只能支持直线的版本管理，具体可以看OpenStack的WiKi文档[Alembic](https://wiki.openstack.org/wiki/Obsolete:Alembic)。

接下来，我们就在我们的webdemo项目中引入Alembic来进行版本管理。

## Alembic {#articleHeader9}

要使用Alembic，大概需要以下步骤：

1. 安装Alembic

2. 在项目中创建Alembic的migration环境

3. 修改Alembic配置文件

4. 创建migration脚本

5. 执行迁移动作

看起来步骤很复杂，其实搭建好环境后，新增数据库版本只需要执行最后两个步骤。

### 安装Alembic {#articleHeader10}

在_webdemo/requirements.txt_中加入：alembic&gt;=0.8.0。然后在virtualenv中安装即可。

### 在项目中创建Alembic的migration环境 {#articleHeader11}

一般OpenStack项目中，Alembic的环境都是放在_db/sqlalchemy/_目录下，因此，我们先建立目录_webdemo/db/sqlalchemy/_，然后在这个目录下初始化Alembic环境：

```

```

现在，我们就在_webdemo/db/sqlalchemy/alembic/_目录下建立了一个Alembic migration环境：

![](https://segmentfault.com/img/bVsT0t)

### 修改Alembic配置文件 {#articleHeader12}

_webdemo/db/sqlalchemy/alembic.ini_文件是Alembic的配置文件，我们现在需要修改文件中的sqlalchemy.url这个配置项，用来指向我们的数据库。这里，我们使用SQLite数据库，数据库文件存放在webdemo项目的根目录下，名称是webdemo.db：

```
# sqlalchemy.url = driver://user:pass
@localhost
/dbname

sqlalchemy.url = 
sqlite:
/
//
../../../webdemo.db
```

注意：实际项目中，数据库的URL信息是从项目配置文件中读取，然后通过动态的方式传递给Alembic的。具体的做法，读者可以参考Magnum项目的实现：[https://github.com/openstack/magnum/blob/master/magnum/db/sqlalchemy/migration.py](https://github.com/openstack/magnum/blob/master/magnum/db/sqlalchemy/migration.py)。

### 创建migration脚本 {#articleHeader13}

现在，我们可以创建第一个迁移脚本了，我们的第一个数据库版本就是创建我们的user表：

```
(.venv)➜ ~
/programming/python
/webdemo/webdemo
/db/sqlalchemy
git:
(master) ✗ 
$ 
alembic revision -m 
"Create user table"
Generating
 /home/diabloneo/programming/python/webdemo/webdemo/db/sqlalchemy/alembic/versions/
4
bafdb464737_create_user_table.py ... done
```

现在脚本已经帮我们生成好了，不过这个只是一个空的脚本，我们需要自己实现里面的具体操作，补充完整后的脚本如下：

```

```

其实就是把`User`类的定义再写了一遍，使用了Alembic提供的接口来方便的创建和删除表。

### 执行迁移操作 {#articleHeader14}

我们需要在_webdemo/db/sqlalchemy/_目录下执行迁移操作，可能需要手动指定PYTHONPATH：

```

```

`alembic upgrade head`会把数据库升级到最新的版本。这个时候，在webdemo的根目录下会出现webdemo.db这个文件，可以使用sqlite3命令查看内容：

```

```

### 测试新的数据库 {#articleHeader15}

现在我们可以把之前使用的内存数据库换掉，使用我们的文件数据库，修改`get_engine()`函数：

```

```

现在你可以手动往webdemo.db中添加数据，然后测试下API：

```
 
```

### 小结 {#articleHeader16}

现在，我们就已经完成了database migration代码框架的搭建，可以成功执行了第一个版本的数据库迁移。OpenStack项目中也是这么来做数据库迁移的。后续，一旦修改了项目，需要修改数据模型时，只要新增migration脚本即可。这部分代码也可以在[https://github.com/diabloneo/webdemo](https://github.com/diabloneo/webdemo)中看到。

在实际生产环境中，当我们发布了一个项目的新版本后，在上线的时候，都会自动执行数据库迁移操作，升级数据库版本到最新的版本。如果线上的数据库版本已经是最新的，那么这个操作没有任何影响；如果不是最新的，那么会把数据库升级到最新的版本。

关于Alembic的更多使用方法，请阅读官方文档[Alembic](https://alembic.readthedocs.org/en/latest/)。

