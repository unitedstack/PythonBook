# paste

---

paste是OpenStack一些比较悠久的项目都会使用paste这个模块，例如:nova, neutron, cinder, heat这类项目。我们现在就主要是从简单的例子开始学起，了解OpenStack的API是怎么设计与实现的：

## 基础练习

### 建立项目

```shell
# mkdir demo_session1/
# tree demo_session1/
demo_session1/
├── config.ini
└── paste_deploy.py
```

首先是，我们需要一个定义我们API的配置文件:

### config.ini

```shell
[composite:main]
use = egg:paste#urlmap
/hello = hello

[app:hello]
version = 1.0.0
paste.app_factory = paste_deploy:SayHello.factory
```

我们分层来讲解这个文件的意思：

```
[composite:main]
use = egg:paste#urlmap
/hello = hello
```

所有在composite:main里面定义的URL就会去找对应的app。如上面的/hello = hello 就定义了URL为 [http://your\_ip/hello](http://your\_ip/hello) 的对应响应app为hello模块，也就是:

```
[app:hello]
version = 1.0.0
paste.app_factory = paste_deploy:SayHello.factor
```

这里面默认找的Python模块就是paste\_deploy.SayHello下面的factor函数。

### paste\_deploy.py

我们现在来看看paste\_deploy.py的主程序：

```
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

from paste.deploy import loadapp
from wsgiref.simple_server import make_server
import os
import sys
import webob


class SayHello(object):
    def __init__(self, version):
        self.version = version

    def __call__(self, environ, start_response):
        res = webob.Response("Hello World!")
        res.status = '200 OK'
        return res(environ, start_response)

    @classmethod
    def factory(cls, global_conf, **kwargs):
        print 'version 1.0'
        return SayHello(kwargs['version'])


if __name__ == "__main__":
    # 获取配置文件的路径
    path = os.path.abspath('.') + '/'
    config_name = 'config.ini'
    config_path = path + config_name

    # 把当前的模块导入系统库
    sys.path.append(path)

    # 加载api的配置文件
    app = loadapp('config:%s' % config_path)

    # 启动API程序
    server = make_server('localhost', 9000, app)
    server.serve_forever()
```

### 运行程序

我们现在就可以运行了paste\_deploy了。

```
# python paste_deploy.py
version 1.0
```

### 访问API

```
# curl http://127.0.0.1:9000/hello
Hello World!
```

## 设置API版本

因为在OpenStack中，又或者其他大型项目中，我们通常都是要根据版本来区分API的。那么问题来了，怎么用paste实现呢？同样的，我们以代码示例：

### 建立工程

```
# mkdir demo_session2/
# tree demo_session2/
demo_session2/
├── config.ini
├── manage.py
└── v1
    ├── __init__.py
    ├── router.py
    └── wsgi.py
```

同样也要编辑我们的配置文件：

### config.ini

```
[composite:common]
use = egg:Paste#urlmap
/: showversion
/log: showversion_log # 这个访问了下面的pipeline
/v1: apiv1app # 这个就是你v1版本的API的主入口

[pipeline:showversion_log]
pipeline = filter_log showversion # 就会按照这个顺序去调用你定义的app

[filter:filter_log]
#filter2 deal with time,read args belowmanage
paste.filter_factory = manage:LogFilter.factory

[app:apiv1app]
paste.app_factory = v1.router:MyRouterApp.factory # 这个是你v1版本的路由函数

[app:showversion]
version = 1.0.0
paste.app_factory = manage:ShowVersion.factory
```

设计这个V1版的API其实理念很简单，访问/v1的时候会去找app:apiv1app这个模块，这个模块的工厂函数又指向了v1.router.MyRouterApp.factory这个工厂函数。

### manage.py

```
''''' 
Created on 2013-6-2 

@author: spch2008 
'''  

from wsgiref.simple_server import make_server
import os
import webob
from paste.deploy import loadapp
import sys


class ShowVersion(object):
    def __init__(self, version):
        self.version = version

    def __call__(self, environ, start_response):
        res = webob.Response("version: %s" % self.version)
        res.status = '200 OK'
        return res(environ, start_response)

    @classmethod
    def factory(cls, global_conf, **kwargs):
        print 'ShowVersion'
        return ShowVersion(kwargs['version'])


class LogFilter(object):
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        print 'you can write your log'
        return self.app(environ, start_response)

    @classmethod
    def factory(cls, global_conf, **kwargs):
        print 'LogFilter'
        return LogFilter


if __name__ == "__main__":
    # 获取配置文件的路径
    path = os.path.abspath('.') + '/'
    config_name = 'config.ini'
    config_path = path + config_name

    # 把当前的模块导入系统库
    sys.path.append(path)

    # 加载api的配置文件
    app = loadapp('config:%s' % config_path)

    # 启动API程序
    server = make_server('localhost', 9000, app)
    server.serve_forever()
```

这个主程序文件其实并没有什么大的变化。接下来我们就应该定义v1函数的路由函数了：



