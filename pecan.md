# Pecan框架

---

上一篇文章我们了解了一个框架Paste。Pecan框架相比上一篇文章的啰嗦框架有如下好处：

* 不用自己写WSGI application了

* 请求路由很容易就可以实现了

总的来说，用上Pecan框架以后，很多重复的代码不用写了，开发人员可以专注于业务，也就是实现每个API的功能。

## OpenStack中的使用场景

### 目录结构

使用Pecan框架时，OpenStack项目一般会把API服务的实现都放在一个api目录下，比如OpenStack中aodh项目是这样的：

```
api/
├── app.py
├── app.wsgi
├── controllers
│   ├── __init__.py
│   ├── root.py
│   └── v2
│       ├── base.py
│       ├── capabilities.py
│       ├── events.py
│       ├── __init__.py
│       ├── meters.py
│       ├── query.py
│       ├── resources.py
│       ├── root.py
│       ├── samples.py
│       └── utils.py
├── hooks.py
├── __init__.py
├── middleware.py
└── rbac.py
```

介绍一下几个主要的文件，这样你以后看到一个使用Pecan的OpenStack项目时就会比较容易找到入口。

* app.py 一般包含了Pecan应用的入口，包含应用初始化代码

* controllers/ 这个目录会包含所有的控制器，也就是API具体逻辑的地方

* controllers/root.py 这个包含根路径对应的控制器

* controllers/v1/ 这个目录对应v1版本的API的控制器。如果有多个版本的API，你一般能看到v2等目录。

### 初始化app

Pecan的配置很容易，通过一个Python源码式的配置文件就可以完成基本的配置。这个配置的主要目的是指定应用程序的root，然后用于生成WSGI application。在aodh中，这部分是写在app.py中的：

```
PECAN_CONFIG = {
    'app': {
        'root': 'aodh.api.controllers.root.RootController',
        'modules': ['aodh.api'],
    },
}


def setup_app(pecan_config=PECAN_CONFIG, conf=None):
    if conf is None:
        # NOTE(jd) That sucks but pecan forces us to use kwargs :(
        raise RuntimeError("Config is actually mandatory")
    # FIXME: Replace DBHook with a hooks.TransactionHook
    app_hooks = [hooks.ConfigHook(conf),
                 hooks.DBHook(
                     storage.get_connection_from_config(conf)),
                 hooks.TranslationHook()]

    pecan.configuration.set_config(dict(pecan_config), overwrite=True)

    app = pecan.make_app(
        pecan_config['app']['root'],
        hooks=app_hooks,
        wrap_app=middleware.ParsableErrorMiddleware,
        guess_content_type_from_ext=False
    )

    return app

...

def app_factory(global_config, **local_conf): # 这边是工厂函数初始化之后调用的函数
    return setup_app() # 完成API部分的初始化
```

Ceilometer在初始化API的时候就设置了root也就是Http中的: / 目录的位置是从模块aodh.api.controllers.root.RootController中开始的。

app\_factory是Ceilometer的api\_paste.ini这个文件指定的调用的工厂函数，我们可以看到在api\_paste.ini中：

```
# path: /etc/aodh/api_paste.ini

[pipeline:main]
pipeline = cors request_id authtoken api-server

[app:api-server]
paste.app_factory = aodh.api.app:app_factory
```

经过一系列的中间件之后就会进入这个app\_factory,然后由setup\_app\(\)完成API的Pecan框架的初始化。pecan.make\_app\(\)会自动读取上面设置的pecan\_config 和 app\_hooks 。hooks对应的配置是一些Pecan的hook，作用类似于WSGI Middleware，也就是中间件。使用了Pecan之后，我们不再需要自己写那些冗余的WSGI application代码了，直接调用Pecan的`make_app()`函数就能完成这些工作。另外，对于之前使用PasteDeploy时用到的很多WSGI中间件，可以选择使用Pecan的hooks机制来实现，也选择使用WSGI中间件的方式来实现。

## 分发式路由 {#articleHeader3}

Pecan不仅缩减了生成WSGI application的代码，而且也让开发人员更容易的指定一个application的路由。Pecan采用了一种对象分发风格（object-dispatch style）的路由模式。在Magnum中也有相关的提现。

```
# path : api/controllers/root.py


from magnum.api.controllers import v1

class RootController(rest.RestController):
...
    v1 = v1.Controller()

    @expose.expose(Root)
    def get(self):
        # NOTE: The reason why convert() it's being called for every
        #       request is because we need to get the host url from
        #       the request object to make the links.
        return Root.convert()
```

* 我们要记得RootController是我们访问URL时的根路径，也就是 : /
* _RootController _继承自 _rest.RestController_，是Pecan实现的RESTful控制器。这里的`get()`函数表示，当访问的是_GET / _时，由该函数处理。`get()`函数会返回一个WSME对象，表示一个形式化的HTTP Response，这个下面再讲。`get()`函数上面的`expose`装饰器是Pecan实现路由控制的一个方式，被expose的函数才会被路由处理。

* v1 = v1.Controller\(\)表示访问 /v1的时候找下一个控制器，也就是magnum.api.controllers.v1下面的\_\_init\_\_.py。

我们看下v1.controller\(\)又对应了什么：

```
# path : api/controllers/v1/__init__.py

class Controller(controllers_base.Controller):
    """Version 1 API controller root."""

    bays = bay.BaysController()
    baymodels = baymodel.BayModelsController()
    clusters = cluster.ClustersController()
    clustertemplates = cluster_template.ClusterTemplatesController()
    certificates = certificate.CertificateController()
    mservices = magnum_services.MagnumServiceController()

    @expose.expose(V1)
    def get(self):
        # NOTE: The reason why convert() it's being called for every
        #       request is because we need to get the host url from
        #       the request object to make the links.
        return V1.convert()
```

这边就是如果GET /v1的时候就直接从get\(\)这个返回了。同样的如果访问了DELETE /v1/clusters这个函数，就直接从cluster.ClustersController\(\)下面的delete\(\)回了。在代码中体现就是：

```
# path : api/controllers/v1/cluster.py
..
class ClustersController(base.Controller):
    @expose.expose(ClusterID, body=Cluster, status_code=202)
    def post(self, cluster):
    ...
    
    @expose.expose(None, types.uuid_or_name, status_code=204)
    def delete(self, cluster_ident):
    ...
```



