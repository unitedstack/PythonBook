## Alembic版本控制 {#articleHeader9}

---

## 数据库版本管理的概念 {#articleHeader8}

上面我们在`get_engine()`函数中使用了内存数据库，并且创建了所有的表。在实际项目中，这么做肯定是不行的：

* 实际项目中不会使用内存数据库，这种数据库一般只是在单元测试中使用。

* 如果每次`create_engine`都把数据库的表重新创建一次，那么数据库中的数据就丢失了，绝对不可容忍。

解决这个问题的办法也很简单：不使用内存数据库，并且在运行项目代码前先把数据库中的表都建好。这么做确实是解决了问题，但是看起来有点麻烦：

* 如果每次都手动写SQL语句来创建数据库中的表，会很容易出错，而且很麻烦。

* 如果项目修改了数据模型，那么不能简单的修改建表的SQL语句，因为重新建表会让数据丢失。我们只能增加新的SQL语句来修改现有的数据库。

* 最关键的是：我们怎么知道一个正在生产运行的数据库是要执行那些SQL语句？如果数据库第一次使用，那么执行全部的语句是正确的；如果数据库已经在使用，里面有数据，那么我们只能执行那些修改表定义的SQL语句，而不能执行那些重新建表的SQL语句。

为了解决这种问题，就有人发明了数据库版本管理的概念，也称为Database Migration。基本原理是：在我们要使用的数据库中建立一张表，里面保存了数据库的当前版本，然后我们在代码中为每个数据库版本写好所需的SQL语句。当对一个数据库执行migration操作时，会执行从当前版本到目标版本之间的所有SQL语句。举个例子：

* 在_Version 1_时，我们在数据库中建立一个user表。

* 在_Version 2_时，我们在数据库中建立一个project表。

* 在_Version 3_时，我们修改user表，增加一个age列。

那么在我们对一个数据库执行migration操作，数据库的当前版本_Version 1_，我们设定的目标版本是_Version 3_，那么操作就是：建立一个project表，修改user表，增加一个age列，并且把数据库当前版本设置为_Version 3_。

数据库的版本管理是所有大型数据库项目的需求，每种语言都有自己的解决方案。OpenStack中主要使用SQLAlchemy的两种解决方案：[sqlalchemy-migrate](https://github.com/openstack/sqlalchemy-migrate)和[Alembic](https://alembic.readthedocs.org/en/latest/)。早期的OpenStack项目使用了sqlalchemy-migrate，后来换成了Alembic。做出这个切换的主要原因是Alembic对数据库版本的设计和管理更灵活，可以支持分支，而sqlalchemy-migrate只能支持直线的版本管理，具体可以看OpenStack的WiKi文档[Alembic](https://wiki.openstack.org/wiki/Obsolete:Alembic)。



