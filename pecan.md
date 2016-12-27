# Pecan框架

---

上一篇文章我们了解了一个框架Paste。Pecan框架相比上一篇文章的啰嗦框架有如下好处：

* 不用自己写WSGI application了

* 请求路由很容易就可以实现了

总的来说，用上Pecan框架以后，很多重复的代码不用写了，开发人员可以专注于业务，也就是实现每个API的功能。

## 目录结构

使用Pecan框架时，OpenStack项目一般会把API服务的实现都放在一个api目录下，比如OpenStack中Ceilometer项目是这样的：

```
# cd pecan_demo
# tree  .
.
├── app.py
├── controllers
│   ├── __init__.py
│   ├── root.py
│   └── v1
│       ├── base.py
│       ├── __init__.py
│       ├── root.py
│       └── server.py
├── expose.py
```

我们最经典的用法就是ceilometer这个项目了。介绍一下几个主要的文件，这样你以后看到一个使用Pecan的OpenStack项目时就会比较容易找到入口。

* app.py 一般包含了Pecan应用的入口，包含应用初始化代码

* config.py 包含Pecan的应用配置，会被app.py使用

* controllers/ 这个目录会包含所有的控制器，也就是API具体逻辑的地方

* controllers/root.py 这个包含根路径对应的控制器

* controllers/v1/ 这个目录对应v1版本的API的控制器。如果有多个版本的API，你一般能看到v2等目录。



