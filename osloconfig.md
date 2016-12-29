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

接下来我们开始书写我们的配置文件，也就是service.conf

```
[DEFAULT]
roll_back=True
collect_worker=2
host=192.168.122.1

[Auth]
auth_url=192.168.122.1:5000
```

可以看到我们的配置文件很简单，就几个非常简单的参数，接下来我们需要做的就是把这个配置文件里面的参数，全部注册到oslo\_config里面。

## 实现主程序

我们的主程序其实就需要找到配置文件，然后把对应的配置项给读取进来。

```
#!/usr/bin/env python

from oslo_config import cfg

import socket

opt_group = cfg.OptGroup(
    name='DEFAULT',
    title='default setting'
)

opts = [
    cfg.StrOpt(
        'host',
        default=socket.gethostname(),
        help='Name of host'
    ),
    cfg.IntOpt(
        'collect_worker',
        default=1,
        help='Number of worker for collector service.'
    ),
    cfg.BoolOpt(
        'roll_back',
        default=False,
        help='roll_back options'
    )
]

CONF.register_group(opt_group)
CONF.register_opts(opts)

CONF = cfg.CONF

default_config_files = cfg.find_config_files('service')
CONF(default_config_files=['service.conf'])

print CONF.roll_back
print CONF.collect_worker
print CONF.host
```

我们先看看这个程序的输出：

```
[root@devstack oslo_cfg]# python service.py 
True
2
192.168.122.1
```

基本上调用oslo\_config的时候就需要做两件事：

* 注册group，就像上面的注册了一个opt\_group，CONF.register\_group\(opt\_group\)
* 注册group的opts，就是像CONF.register\_opts\(opts\)
* 定义配置文件的名称，去找配置文件就是cfg.find\_configfile\('service'\)



