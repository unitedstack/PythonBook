# Paste

paste 是 OpenStack 一些比较悠久的项目都会使用 paste 这个模块，例如:nova, neutron, cinder, heat这类项目。我们首先了解 OpenStack 的 API 是怎么设计与实现的,然后再使用 paste 实现我们的小程序：

## OpenStack 古老的 API 实现

可以稍微讲一下 OpenStack API 的发展史，原先很 OpenStack 的古老的 API 基本都是用：Paste + PasteDeploy + Routes + WebOb 来实现的，发展到后面，社区的人开始厌倦了这一套东西，觉得这套逻辑很烦人，之后兴起的一些 OpenStack 的项目都采用了新的 WSGI 框架 Pecan 。但是 Paste + PasteDeploy + Routes + WebOb 这套东西既然存在了，我们就有学习的必要。

首先我们来看看，这套 PPRW 组合的在 Heat 中的实现：

### WSGI 程序入口

```
# path : cmd/heat-api
def launch_api(setup_logging=True):
    ...

    # common/config.py
    app = config.load_paste_app()

    # common/wsgi.py
    server = wsgi.Server('heat-api', cfg.CONF.heat_api)
    server.start(app, default_port=port)
    return server
```

启动 heat-api 的时候会先用 paste\_deploy 这去加载你的 /etc/heat/api-paste.ini 这个文件，然后实例化一个 paste\_deploy 的对象。

```python
# path: common/config.py
def load_paste_app(app_name=None):
    try:
        # 这里有一个很重要的变量 conf_file，它定义了我们 api-paste.ini 这个文件的位置
        app = wsgi.paste_deploy_app(conf_file, app_name, cfg.CONF)
        return app
    except:
        ...
```

在上面的 paste\_deploy\_app 方法中有一个 app\_name 是指定了你整个 paste 流程从哪里开始。下面会再次提到。

然后调用 evenlet 来启动我们的 wsgi 服务。

```python
# path: common/wsgi.py
class Server(object):
    """Server class to manage multiple WSGI sockets and applications."""

    def run_server(self):
        try:
            eventlet.wsgi.server(
                self.sock,
                self.application,
                custom_pool=self.pool,
                url_length_limit=URL_LENGTH_LIMIT,
                log=self._logger,
                debug=cfg.CONF.debug,
                keepalive=cfg.CONF.eventlet_opts.wsgi_keep_alive,
                socket_timeout=socket_timeout)
```

这边启动的时候其实 paste\_deploy 已经加载完了 API 的配置。

## api-paste.ini

使用 Paste 和 PasteDeploy 模块来实现 WSGI 服务时，都需要一个 api-paste.ini 文件。这个文件也是 Paste 框架的精髓，这里需要重点说明一下这个文件如何阅读。
api-paste.ini 文件的格式类似于 INI 格式，每个 section 的格式为 \[type:name\]。这里重要的是理解几种不同 type 的 section 的作用。

* composite 
    **这种 section 用于将 HTTP 请求分发到指定的 app。**
* app
    **这种 section 表示具体的 app。**
* filter 
    **实现一个过滤器中间件。**
* pipeline
    **用来把把一系列的filter串起来。**

上面这些 section 是在 Heat 的 api-paste.ini 中用到的，下面详细介绍一下如何使用。
这种 section 用来决定如何分发 HTTP 请求。Heat 的 api-paste.ini 文件中有都是用了 pipeline 管理的：

```
# heat-api pipeline
[pipeline:heat-api]
pipeline = cors request_id faultwrap http_proxy_to_wsgi versionnegotiation osprofiler authurl authtoken context apiv1app

[app:apiv1app]
paste.app_factory = heat.common.wsgi:app_factory
heat.app_factory = heat.api.openstack.v1:API
```

就是最后 api 调用最终还是落到了 heat.api.openstack.v1 下面的 API 类上。你可能会有疑问为什么 API 调用会了 pipeline:heat-api 这个模块。还记得我们在上面讲的 app\_name 的事么？

```python
# path: common/config.py
def load_paste_app(app_name=None):
    try:
        # app_name 为 heat-api
        app = wsgi.paste_deploy_app(conf_file, app_name, cfg.CONF)
        return app
    except:
        ...
```

在这里其实 app\_name 的值为 heat-api，这样 paste 的流程就知道要从 api-paste.ini 中 name 为 heat-api 的 type 开始了。

## 最后的 Application

上面我们已经看到了，对于/ 开头的请求，在 paste.ini 中会被路由到 \[pipeline:heat-api\] 这个 section，会交给这个函数生成的 heat.api.openstack.v1:API application处理。最后这个 application 需要根据 URL path 中剩下的部分，/stacks，来实现URL路由。从这里开始，就需要用到 Routes 模块了。

同样由于篇幅限制，我们只能展示 Routes 模块的大概用法。Routes 模块是用 Python 实现的类似 Rails 的 URL 路由系统，它的主要功能就是把 path 映射到对应的动作。

Routes 模块的一般用法是创建一个 `Mapper` 对象，然后调用该对象的 `connect()` 方法把 path 和 method 映射到一个 controller 的某个 action 上，这里 controller 是一个自定义的类实例，action 是表示 controller 对象的方法的字符串。一般调用的时候还会指定映射哪些方法，比如 GET 或者 POST 之类的。

```
from heat.api.openstack.v1 import stacks

class API(wsgi.Router):

    """WSGI router for Heat v1 REST API requests."""

    def __init__(self, conf, **local_conf):
        self.conf = conf
        mapper = routes.Mapper()
        default_resource = wsgi.Resource(wsgi.DefaultMethodController(),
                                         wsgi.JSONRequestDeserializer())

        def connect(controller, path_prefix, routes):
            ...
                mapper.connect(r['name'], url, controller=controller,
                               action=r['action'],
                               conditions={'method': methods_str})
            ...

        ...

        # Stacks
        stacks_resource = stacks.create_resource(conf)
        connect(controller=stacks_resource,
                path_prefix='/{tenant_id}',
                routes=[
                    {
                        'name': 'stack_index',
                        'url': '/stacks',
                        'action': 'index',
                        'method': 'GET'
                    }
                ])
```

在上面 connect 中调用了 router 的 mapper.connect 来对我们设置的规则进行注册。

我们定义了，访问 /stacks 这个 URL 的时候，就会去调用 stacks 这个对象下面的 index 方法。



