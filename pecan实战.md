# Pecan 实战


## 使用 Pecan 搭建 API 服务的框架

既然花了大量的篇幅介绍 Pecan 的使用，接下来就要开始编码工作了。首先要把整个服务的框架搭建起来。

## 代码目录结构

一般来说，OpenStack 项目中，使用 Pecan 来开发 API 服务时，都会在代码目录下有一个专门的 API 目录，用来保存 API 相关的代码。比如 Magnum 项目的 _magnum/api_，或者 Ceilometer 项目的 _ceilometer/api_ 等。我们的代码也遵守这个规范，让我们直接来看下我们的代码目录结构（\#后面的表示注释）：

```
[webdemo] $ tree .
.
├── app.py           # 这个文件存放 WSGI application 的入口
├── config.py        # 这个文件存放 Pecan 的配置
├── controllers/     # 这个目录用来存放 Pecan 控制器的代码
├── hooks.py         # 这个文件存放 Pecan 的 hooks 代码（本文中用不到）
└── __init__.py
```

我们有这样的目录，接下来就是具体的代码实现了。

---

## 先让我们的服务跑起来

为了后面更好的开发，我们需要先让我们的服务在本地跑起来，这样可以方便自己做测试，看到代码的效果。不过要做到这点，还是有些复杂的。

### 配置环境

首先，先创建 config.py 文件的内容：

```
app = {
    'root': 'webdemo.api.controllers.root.RootController',
    'modules': ['webdemo.api'],
    'debug': False,
}
```

就是包含了 Pecan 的最基本配置，其中指定了 root controller 的位置。然后看下 app.py 文件的内容，主要就是读取 config.py 中的配置，然后创建一个 WSGI application：

```
import pecan

from webdemo.api import config as api_config


def get_pecan_config():
    filename = api_config.__file__.replace('.pyc', '.py')
    return pecan.configuration.conf_from_file(filename)


def setup_app():
    config = get_pecan_config()

    app_conf = dict(config.app)
    app = pecan.make_app(
        app_conf.pop('root'),
        logging=getattr(config, 'logging', {}),
        **app_conf
    )

    return app
```

然后，我们至少还需要实现一下 root controller，也就是 _webdemo/api/controllers/root.py_ 这个文件中的 `RootController` 类：

```
from pecan import rest
from wsme import types as wtypes
import wsmeext.pecan as wsme_pecan


class RootController(rest.RestController):

    @wsme_pecan.wsexpose(wtypes.text)
    def get(self):
        return "webdemo"
```

### 本地测试服务器

为了继续开放的方便，我们要先创建一个 Python 脚本，可以启动一个单进程的 API 服务。这个脚本会放在 _webdemo/cmd/_ 目录下，名称是 api.py（这目录和脚本名称也是惯例），来看看我们的 api.py 吧：

```
from wsgiref import simple_server

from webdemo.api import app


def main():
    host = '0.0.0.0'
    port = 9000

    application = app.setup_app()
    srv = simple_server.make_server(host, port, application)

    srv.serve_forever()


if __name__ == '__main__':
    main()
```

### 运行测试服务器的环境

要运行这个测试服务器，首先需要安装必要的包，并且设置正确的路径。在后面的文章中，我们将会知道，这个可以通过 tox 这个工具来实现。现在，我们先做个简单版本的，就是手动创建这个运行环境。

首先，完善一下 requirements.txt 这个文件，包含我们需要的包：

```
pbr<2.0,>=0.11
pecan
WSME
```

### 启动我们的服务

启动服务需要技巧，因为我们的 webdemo 还没有安装到系统的 Python 路径中，也不在上面创建 virtualenv 环境中，所以我们需要通过指定 PYTHONPATH 这个环境变量来为 Python 程序增加库的查找路径：

```
$ PYTHONPATH=. python webdemo/cmd/api.py
```

现在测试服务器已经起来了，可以通过浏览器访问 http://localhost:9000/ 这个地址来查看结果。（你可能会发现，返回的是 XML 格式的结果，而我们想要的是 JSON 格式的。这个是 WSME 的问题，我们后面再来处理）。

到这里，我们的 REST API 服务的框架已经搭建完成，并且测试服务器也跑起来了。


## 用户管理API的实现

现在我们来实现我们在第一章设计的 API。这里先说明一下：

我们会直接使用 Pecan 的 RestController 来实现REST API，这样可以不用为每个接口指定接受的 method。

### 让 API 返回 JSON 格式的数据

现在，所有的 OpenStack 项目的 REST API 的返回格式都是使用 JSON 标准，所以我们也要这么做。那么有什么办法能够让 WSME 框架返回 JSON 数据呢？可以通过设置 `wsmeext.pecan.wsexpose()` 的 _rest\_content\_types_ 参数来是先。这里，我们借鉴一段 Magnum 项目中的代码，把这段代码存放在文件 _webdemo/api/expose.py_ 中：

```
import wsmeext.pecan as wsme_pecan


def expose(*args, **kwargs):
    """Ensure that only JSON, and not XML, is supported."""
    if 'rest_content_types' not in kwargs:
        kwargs['rest_content_types'] = ('json',)

    return wsme_pecan.wsexpose(*args, **kwargs)
```

这样我们就封装了自己的 `expose` 装饰器，每次都会设置响应的 content-type 为 JSON。上面的 root controller 代码也就可以修改为：

```
from pecan import rest
from wsme import types as wtypes

from webdemo.api import expose


class RootController(rest.RestController):

    @expose.expose(wtypes.text)
    def get(self):
        return "webdemo"
```

再次运行我们的测试服务器，就可以返现返回值为JSON格式了。


## 实现 GET /v1

这个其实就是实现 v1 这个版本的API的路径前缀。在 Pecan 的帮助下，我们很容易实现这个，只要按照如下两步做即可：

* 先实现 v1 这个 controller

* 把 v1 controller 加入到 root controller 中

按照 OpenStack 项目的规范，我们会先建立一个 _webdemo/api/controllers/v1/_ 目录，然后将 v1 controller 放在这个目录下的一个文件中，假设我们就放在 _v1/controller.py_ 文件中，效果如下:

```
from pecan import rest
from wsme import types as wtypes

from webdemo.api import expose


class V1Controller(rest.RestController):

    @expose.expose(wtypes.text)
    def get(self):
        return 'webdemo v1controller'
```

然后把这个 controller 加入到 root controller 中：

```
...
from webdemo.api.controllers.v1 import controller as v1_controller
from webdemo.api import expose


class RootController(rest.RestController):
    v1 = v1_controller.V1Controller()

    @expose.expose(wtypes.text)
    def get(self):
        return "webdemo"
```

此时，你访问 [http://localhost:9000/v1](http://localhost:9000/v1) 就可以看到结果了。


## 实现 GET /v1/users

### 添加 users controller

这个 API 就是返回所有的用户信息，功能很简单。首先要添加 users controller 到上面的 v1 controller 中。为了不影响阅读体验，这里就不贴代码了，请看 github 上的示例代码。

### 使用 WSME 来规范 API 的响应值

上篇文章中，我们已经提到了 WSME 可以用来规范 API 的请求和响应的值，这里我们就要用上它。首先，我们要参考 OpenStack 的惯例来设计这个 API 的返回值：

```
{
  "users": [
    {
      "name": "Alice",
      "age": 30
    },
    {
      "name": "Bob",
      "age": 40
    }
  ]
}
```

其中 _users_ 是一个列表，列表中的每个元素都是一个 user。那么，我们要如何使用 WSME 来规范我们的响应值呢？答案就是使用 WSME 的自定义类型。我们可以利用 WSME 的类型功能定义出一个 user 类型，然后再定义一个 user 的列表类型。最后，我们就可以使用上面的 expose 方法来规定这个 API 返回的是一个 user 的列表类型。

### 定义 user 类型和 user 列表类型

这里我们需要用到 WSME 的 Complex types 的功能，请先看一下文档 Types [http://wsme.readthedocs.org/en/latest/types.html](http://wsme.readthedocs.org/en/latest/types.html)。简单说，就是我们可以把 WSME 的基本类型组合成一个复杂的类型。我们的类型需要继承自 _wsme.types.Base_ 这个类。因为我们在本文只会实现一个 user 相关的 API，所以这里我们把所有的代码都放在 _webdemo/api/controllers/v1/users.py_ 文件中。来看下和 user 类型定义相关的部分：

```
from wsme import types as wtypes


class User(wtypes.Base):
    name = wtypes.text
    age = int


class Users(wtypes.Base):
    users = [User]
```

这里我们定义了 `class User`，表示一个用户信息，包含两个字段，name 是一个文本，age 是一个整型。`class Users` 表示一组用户信息，包含一个字段 users，是一个列表，列表的元素是上面定义的 `class User`。完成这些定义后，我们就使用 WSME 来检查我们的 API 是否返回了合格的值；另一方面，只要我们的 API 返回了这些类型，那么就能通过 WSME 的检查。我们先来完成利用 WSME 来检查 API 返回值的代码：

```
class UsersController(rest.RestController):

    # expose方法的第一个参数表示返回值的类型
    @expose.expose(Users)
    def get(self):
        pass
```

这样就完成了 API 的返回值检查了。

### 实现 API 逻辑

我们现在来完成API的逻辑部分。不过为了方便大家理解，我们直接返回一个写好的数据，就是上面贴出来的那个。

```
class UsersController(rest.RestController):

    @expose.expose(Users)
    def get(self):
        user_info_list = [
            {
                'name': 'zhao',
                'age': 23,
            },
            {
                'name': 'xu',
                'age': 21,
            }
        ]
        users_list = [User(**user_info) for user_info in user_info_list]
        return Users(users=users_list)
```

代码中，会先根据 user 信息生成 User 实例的列表 `users_list`，然后再生成 Users 实例。此时，重启测试服务器后，你就可以从浏览器访问 http://localhost:9000/v1/users ，就能看到结果了。


## 实现 POST /v1/users

这个 API 会接收用户上传的一个 JSON 格式的数据，然后打印出来（实际中一般是存到数据库之类的），要求用户上传的数据符合 User 类型的规范，并且返回的状态码为201。代码如下：

```
class UsersController(rest.RestController):

    @expose.expose(None, body=User, status_code=201)
    def post(self, user):
        print user
```

可以使用 curl 程序来测试：

```
$ curl -X POST http://localhost:8080/v1/users -H "Content-Type: application/json" -d '{"name": "zhao", "age": 23}'
```

服务器日志输出：

```
127.0.0.1 - - [16/Nov/2015 23:16:28] "POST /v1/users HTTP/1.1" 201 0
```

我们用3行代码就实现了这个 POST 的逻辑。现在来说明一下这里的秘密。`expose` 装饰器的第一个参数表示这个方法没有返回值；第三个参数表示这个 API 的响应状态码是201，如果不加这个参数，在没有返回值的情况下，默认会返回204。第二个参数要说明一下，这里用的是 `body=User`，你也可以直接写 `User`。使用 `body=User` 这种形式，你可以直接发送符合 User 规范的 JSON 字符串；如果是用 `expose(None, User, status_code=201)` 那么你需要发送下面这样的数据：

```
{ "user": {"name": "zhao", "age": 23} }
```

你可以自己测试一下区别。要更多的了解本节提到的 expose 参数，请参考 WSM 文档Functions [http://wsme.readthedocs.org/en/latest/functions.html](http://wsme.readthedocs.org/en/latest/functions.html)。最后，你接收到一个创建用户请求时，一般会为这个用户分配一个 id。本文前面已经提到了 OpenStack 项目中一般使用UUID。你可以修改一下上面的逻辑，为每个用户分配一个 UUID。


## 实现 GET /v1/users/&lt;UUID&gt;

要实现这个 API，需要两个步骤：

1. 在 UsersController 中解析出 &lt;UUID&gt; 的部分，然后把请求传递给这个一个新的 UserController。从命名可以看出，UsersController 是针对多个用户的，UserController 是针对一个用户的。

2. 在 UserController 中实现 `get()` 方法。

### 使用 \_lookup\(\) 方法

Pecan 的 `_lookup()` 方法是 controller 中的一个特殊方法，Pecan 会在特定的时候调用这个方法来实现更灵活的 URL 路由。Pecan 还支持用户实现 `_default()` 和 `_route()` 方法。这些方法的具体说明，请阅读Pecan的文档：routing [https://pecan.readthedocs.org/en/latest/routing.html](https://pecan.readthedocs.org/en/latest/routing.html)。

我们这里只用到 `_lookup()` 方法，这个方法会在 controller 中没有其他方法可以执行且没有 `_default()` 方法的时候执行。比如上面的 UsersController 中，没有定义 _/v1/users/&lt;UUID&gt;_ 如何处理，它只能返回404；如果你定义了 `_lookup()` 方法，那么它就会调用该方法。

`_lookup()` 方法需要返回一个元组，元组的第一个元素是下一个 controller 的实例，第二个元素是 URL path 中剩余的部分。

在这里，我们就需要在 `_lookup()` 方法中解析出 UUID 的部分并传递给新的 controller 作为新的参数，并且返回剩余的 URL path。来看下代码：

```
class UserController(rest.RestController):

    def __init__(self, user_id):
        self.user_id = user_id


class UsersController(rest.RestController):

    @pecan.expose()
    def _lookup(self, user_id, *remainder):
        return UserController(user_id), remainder
```

`_lookup()` 方法的形式为 `_lookup(self, user_id, *remainder)`，意思就是会把 /v1/users/&lt;UUID&gt; 中的 &lt;UUID&gt; 部分作为 user\_id 这个参数，剩余的按照"/"分割为一个数组参数（这里 remainder 为空）。然后，`_lookup()` 方法里会初始化一个 UserController 实例，使用 user\_id 作为初始化参数。这么做之后，这个初始化的控制器就能知道是要查找哪个用户了。然后这个控制器会被返回，作为下一个控制被调用。请求的处理流程就这么转移到 UserController 中了。

### 实现API逻辑

实现前，我们要先修改一下我们返回的数据，里面需要增加一个id字段。对应的 User 定义如下：

```
class User(wtypes.Base):
    id = wtypes.text
    name = wtypes.text
    age = int
```

现在，完整的 UserController 代码如下：

```
class UserController(rest.RestController):

    def __init__(self, user_id):
        self.user_id = user_id

    @expose.expose(User)
    def get(self):
        user_info = {
            'id': self.user_id,
            'name': 'Mr_zhao',
            'age': 23,
        }
        return User(**user_info)
```

使用 curl 来检查一下效果：

```
$ curl http://localhost:9000/v1/users/521cf58808494f09a6f951c115640daa
{"age": 30, "id": "521cf58808494f09a6f951c115640daa", "name": "Mr_zhao"}
```

# 总结

到此为止，我们已经完成了我们的 API 服务了，虽然没有实际的逻辑，但是本文搭建起来的框架也是 OpenStack 中 API 服务的一个常用框架，很多大项目的 API 服务代码都和我们的 webdemo 长得差不多。最后再说一下，本文的代码在 github 上托管着：

https://github.com/zhaozhilong1993/webdemo 。

现在我们已经了解了包管理和 API 服务了，那么接下来就要开始数据库相关的操作了。大部分 OpenStack 的项目都是使用非常著名的 sqlalchemy 库来实现数据库操作的，本系列接下来的文章就是要来说明数据库的相关知识和应用。

