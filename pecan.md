# Pecan框架

---

上一篇文章我们了解了一个框架Paste。Pecan框架相比上一篇文章的啰嗦框架有如下好处：

* 不用自己写WSGI application了

* 请求路由很容易就可以实现了

总的来说，用上Pecan框架以后，很多重复的代码不用写了，开发人员可以专注于业务，也就是实现每个API的功能。

## 目录结构

使用Pecan框架时，OpenStack项目一般会把API服务的实现都放在一个api目录下，比如OpenStack中Ceilometer项目是这样的：

```
api/
├── app.py
├── app.wsgi
├── controllers
│   ├── __init__.py
│   ├── root.py
│   └── v2
│       ├── base.py
│       ├── capabilities.py
│       ├── events.py
│       ├── __init__.py
│       ├── meters.py
│       ├── query.py
│       ├── resources.py
│       ├── root.py
│       ├── samples.py
│       └── utils.py
├── hooks.py
├── __init__.py
├── middleware.py
└── rbac.py
```

我们最经典的用法就是ceilometer这个项目了。介绍一下几个主要的文件，这样你以后看到一个使用Pecan的OpenStack项目时就会比较容易找到入口。

* app.py 一般包含了Pecan应用的入口，包含应用初始化代码

* controllers/ 这个目录会包含所有的控制器，也就是API具体逻辑的地方

* controllers/root.py 这个包含根路径对应的控制器

* controllers/v1/ 这个目录对应v1版本的API的控制器。如果有多个版本的API，你一般能看到v2等目录。

## 初始化app

Pecan的配置很容易，通过一个Python源码式的配置文件就可以完成基本的配置。这个配置的主要目的是指定应用程序的root，然后用于生成WSGI application。在Ceilometer中，这部分是写在app.py中的：

```
def setup_app(pecan_config=None):
    # FIXME: Replace DBHook with a hooks.TransactionHook
    app_hooks = [hooks.ConfigHook(),
                 hooks.DBHook(),
                 hooks.NotifierHook(),
                 hooks.TranslationHook()]

    pecan_config = pecan_config or {
        "app": {
            'root': 'ceilometer.api.controllers.root.RootController', # 这里就定义了控制器的入口
            'modules': ['ceilometer.api'],
        }
    }

    pecan.configuration.set_config(dict(pecan_config), overwrite=True)

    app = pecan.make_app(
        pecan_config['app']['root'],
        debug=CONF.api.pecan_debug,
        hooks=app_hooks,
        wrap_app=middleware.ParsableErrorMiddleware,
        guess_content_type_from_ext=False
    )

    return app

...

def app_factory(global_config, **local_conf):
    return setup_app()
```





