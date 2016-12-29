## oslo\_config

---

oslo\_config主要是用于服务端程序启动的时候需要从配置文件里面读取一些参数而设定的，  
如:heat-engine,heat-api等  
，注册后的配置文件的参数可以在通过类似cfg.CONF.&lt;register\_group\_name&gt;.&lt;cmd&gt;调用  
。下面我们就书写一个小程序，把oslo\_config用起来。

## 目录结构

```
[root@devstack oslo_cfg]# tree .
.
├── service.conf
└── service.py
```

我们的目录结构很简单，因为让大家熟悉这个库的使用，并不需要做一个特别复杂的目录结构。

## 书写配置文件



