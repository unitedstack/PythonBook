# paste

---

paste是OpenStack一些比较悠久的项目都会使用paste这个模块，例如:nova, neutron, cinder, heat这类项目。我们现在就主要是从简单的例子开始学起，了解OpenStack的API是怎么设计与实现的：

## 实例一

### 建立项目

```shell
# mkdir demo1
```

### 书写配置文件

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

\[composite:main\] 这个是程序的主入口,可以不写这个，但是必须在python调用的时候指定从哪里开始读，在下面会讲解到。所有在composite:main里面定义的URL就会去找对应的app。如上面的/hello = hello 就定义了URL为 http://your\_ip/hello

