# Paste实战

花了好大的篇幅讲 paste 的那一套组件，现在我们通过一个小程序来增强我们的认知。

## 构建我们的工程结构

```shell
# mkdir demo_session1/
# tree demo_session1/
demo_session1/
├── config.ini
└── paste_deploy.py
```

首先是，我们需要一个定义我们 API 的配置文件:

### config.ini

```shell
[composite:main]
use = egg:paste#urlmap
/hello = hello

[app:hello]
version = 1.0.0
paste.app_factory = paste_deploy:SayHello.factory
```

使用 Paste 和 PasteDeploy 模块来实现 WSGI 服务时，都需要一个 config.ini 文件。这个文件也是 Paste 框架的精髓，这里需要重点说明一下这个文件如何阅读。

paste.ini 文件的格式类似于 INI 格式，每个 section 的格式为 \[type:name\]。这里重要的是理解几种不同 type 的 section 的作用。

* composite
    **这种 section 用于将 HTTP 请求分发到指定的 app。**

* app
    **这种 section 表示具体的 app。**

* filter
    **实现一个过滤器中间件。**

* pipeline
    **用来把把一系列的 filter 串起来。**

上面这些 section 是在 keystone的paste.ini 中用到的，下面详细介绍一下如何使用。这里需要用到 WSGIMiddleware\(WSGI中间件\) 的知识，可以在 WSGI 简介[http://segmentfault.com/a/1190000003069785](http://segmentfault.com/a/1190000003069785)这篇文章中找到。

我们先来分层来讲解这个文件的意思：

```
[composite:main]
use = egg:paste#urlmap
/hello = hello
```

所有在 composite:main 里面定义的 URL 就会去找对应的 app。如上面的 /hello = hello 就定义了 URL 为 [http://your\_ip/hello](http://your\_ip/hello) 的对应响应 app 为 hello 模块，也就是:

```
[app:hello]
version = 1.0.0
paste.app_factory = paste_deploy:SayHello.factor
```

这里面默认找的 Python 模块就是 paste\_deploy.SayHello 下面的 factor 函数。

### paste\_deploy.py

我们现在来看看 paste\_deploy.py 的主程序：

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

我们现在就可以运行了 paste\_deploy 了。

```
# python paste_deploy.py
version 1.0
```

### 访问API

```
# curl http://127.0.0.1:9000/hello
Hello World!
```

---

## 编写v1 版本API

因为在 OpenStack 中，又或者其他大型项目中，我们通常都是要根据版本来区分 API 的。那么问题来了，怎么用 paste 实现呢？同样的，我们以代码示例：

### 拓展我们的工程结构

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

pipeline 关键字指定了很多个名字，这些名字也是 paste.ini 文件中其他section的名字。请求会从最前面的 section 开始处理，一直向后传递。pipeline 指定的 section 有如下要求：

* 最后一个名字对应的 section 一定要是一个 app

* 非最后一个名字对应的 section 一定要是一个 filter

设计这个 V1 版的 API 其实理念很简单，访问/v1 的时候会去找 app:apiv1app 这个模块，这个模块的工厂函数又指向了 v1.router.MyRouterApp.factory 这个工厂函数。

### manage.py

```
'''''
Created on 2016-12-27

@author: zhaozhilong
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

    # 加载 api 的配置文件
    app = loadapp('config:%s' % config_path)

    # 启动API程序
    server = make_server('localhost', 9000, app)
    server.serve_forever()
```

这个主程序文件其实并没有什么大的变化。接下来我们就应该定义 v1 函数的路由函数了：

### 实现v1版本代码的路由功能

#### router.py

然后书写 router.py 的主代码：

```
'''''
Created on 2016-12-27

@author: zhaozhilong
'''

from wsgiref.simple_server import make_server
from paste.deploy import loadapp
import sys
import os
import wsgi

class Controller(object):
    def __init__(self):
        print "Controller"

    def test(self, req):
        print "req",req
        return {
            "name":"test",
            "properties":"test"
        }

class MyRouterApp(wsgi.Router):
    def __init__(self, mapper):
        controller = Controller() # 实例化Controller这个类
        mapper.connect(
            '/test',
            controller=wsgi.Resource(controller), # 注册这个类
            action='test', # 定义了这个类下面的test这个方法
            conditions={'method':['GET']} # 匹配了哪一个HTTP的方法
        )
        super(MyRouterApp, self).__init__(mapper)
```

我们这里定义了 router 的规则，设置了 /test 这个 URL 的匹配规则，我们在 config.ini 中匹配到 v1 之后，就会被转发到 MyRouterApp, 这个类中，然后在这里匹配到了二级 URL ：/test，定义了当 HTTP 的方法为 GET 的时候去找 Controller 函数的 test 方法。

#### wsgi.py

wsgi 重在做路由，利用到了 routes 这个 Python 库。

```
import datetime
import json
import routes
import routes.middleware
import webob
import webob.dec
import webob.exc

class APIMapper(routes.Mapper):
   """
   Handle route matching when url is '' because routes.Mapper returns
   an error in this case.
   """

   def routematch(self, url=None, environ=None):
       if url is "":
           result = self._match("", environ)
           return result[0], result[1]
       return routes.Mapper.routematch(self, url, environ)

class Router(object):
   def __init__(self, mapper):
       mapper.redirect("", "/")
       self.map = mapper
       self._router = routes.middleware.RoutesMiddleware(self._dispatch,
                                                         self.map)

   @classmethod
   def factory(cls, global_conf, **local_conf):
       return cls(APIMapper())

   @webob.dec.wsgify
   def __call__(self, req):
       """
       Route the incoming request to a controller based on self.map.
       If no match, return a 404.
       """
       return self._router

   @staticmethod
   @webob.dec.wsgify
   def _dispatch(req):
       """
       Called by self._router after matching the incoming request to a route
       and putting the information into req.environ.  Either returns 404
       or the routed WSGI app's response.
       """
       match = req.environ['wsgiorg.routing_args'][1]
       if not match:
           return webob.exc.HTTPNotFound()
       app = match['controller']
       return app

class Request(webob.Request):
   """Add some Openstack API-specific logic to the base webob.Request."""

   def best_match_content_type(self):
       """Determine the requested response content-type."""
       supported = ('application/json',)
       bm = self.accept.best_match(supported)
       return bm or 'application/json'

   def get_content_type(self, allowed_content_types):
       """Determine content type of the request body."""
       if "Content-Type" not in self.headers:
           return

       content_type = self.content_type

       if content_type not in allowed_content_types:
           return
       else:
           return content_type

class JSONRequestDeserializer(object):
   def has_body(self, request):
       """
       Returns whether a Webob.Request object will possess an entity body.

       :param request:  Webob.Request object
       """
       if 'transfer-encoding' in request.headers:
           return True
       elif request.content_length > 0:
           return True

       return False

   def _sanitizer(self, obj):
       """Sanitizer method that will be passed to json.loads."""
       return obj

   def from_json(self, datastring):
       try:
           return json.loads(datastring, object_hook=self._sanitizer)
       except ValueError:
           msg = ('Malformed JSON in request body.')
           raise webob.exc.HTTPBadRequest(explanation=msg)

   def default(self, request):
        if self.has_body(request):
            return {'body': self.from_json(request.body)}
        else:
            return {}

class JSONResponseSerializer(object):

    def _sanitizer(self, obj):
        """Sanitizer method that will be passed to json.dumps."""
        if isinstance(obj, datetime.datetime):
            return obj.isoformat()
        if hasattr(obj, "to_dict"):
            return obj.to_dict()
        return obj

    def to_json(self, data):
        return json.dumps(data, default=self._sanitizer)

    def default(self, response, result):
        response.content_type = 'application/json'
        response.body = self.to_json(result)

class Resource(object):
    def __init__(self, controller, deserializer=None, serializer=None):
        self.controller = controller
        self.serializer = serializer or JSONResponseSerializer()
        self.deserializer = deserializer or JSONRequestDeserializer()

    @webob.dec.wsgify(RequestClass=Request)
    def __call__(self, request):
        """WSGI method that controls (de)serialization and method dispatch."""
        action_args = self.get_action_args(request.environ)
        action = action_args.pop('action', None)

        deserialized_request = self.dispatch(self.deserializer,
                                             action, request)
        action_args.update(deserialized_request)

        action_result = self.dispatch(self.controller, action,
                                      request, **action_args)
        try:
            response = webob.Response(request=request)
            self.dispatch(self.serializer, action, response, action_result)
            return response

        except webob.exc.HTTPException as e:
            return e
        # return unserializable result (typically a webob exc)
        except Exception:
            return action_result

    def dispatch(self, obj, action, *args, **kwargs):
        """Find action-specific method on self and call it."""
        try:
            method = getattr(obj, action)
        except AttributeError:
            method = getattr(obj, 'default')

        return method(*args, **kwargs)

    def get_action_args(self, request_environment):
        """Parse dictionary created by routes library."""
        try:
            args = request_environment['wsgiorg.routing_args'][1].copy()
        except Exception:
            return {}

        try:
            del args['controller']
        except KeyError:
            pass

        try:
            del args['format']
        except KeyError:
            pass

        return args
```

### 运行程序

```
# python manage.py
Controller
ShowVersion
```

### 调用API

```
# curl http://127.0.0.1:9000/v1/test
{"name": "test", "properties": "test"}
```

## 总结:

总的来说 paste 的框架巨烦巨复杂，如果你对这个框架不熟悉用起来特别难上手。所以，社区后来的项目就慢慢进化到 pecan 这个框架上面了。本文的代码都被托管在了[ https://github.com/zhaozhilong1993/paste ](https://github.com/zhaozhilong1993/webdemo)上面了。可以下载下来好好分析。

