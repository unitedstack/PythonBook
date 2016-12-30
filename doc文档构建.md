# Sphinx 文档构建

OpenStack 所有的项目都要求有对应的文档，如果你正在书写一个 OpenStack 项目那么你就需要构建你的一个文档工程。默认的 OpenStack 构建文档都是使用了 Sphinx 这个模块，主要是用于生成一个文档目录。用 Sphinx 构建的文档，格式都是 rst 格式的，如: http://docs.openstack.org/developer/nova/ ，这种格式的写法和传统的 markdown 文档不太一样。下面我们为我们的 webdemo 项目来构建一个 OpenStack 文档系统。
## Sphinx 安装

```
pip install sphinx
```

pip 源里面就自带了 sphinx 这个包，所以可以直接安装实现。

## 生成文档

```
# cd webdemo
# mkdir doc
# sphinx-quickstart
# ls
_build  conf.py  index.rst  make.bat  Makefile  _static  _templates
```

默认情况下我们生成的是 rst 格式的。要把 rst 格式的文件构建成一个可视化的模板，我们可以 sphinx-build -b html . output 或者 make html 命令生成 html 文档，生成的文档位于 output 目录 。你只需要把这个 output 目录发布出来，那么你的文档就可以在网站上打开了。

## 小结

Sphinx 的使用其实很简单，但是构建文档的时候要注意 rst 格式的语法规范。详情可以参照：rst 文件格式[http://ju.outofmemory.cn/entry/105733](http://ju.outofmemory.cn/entry/105733)，这里面讲解了很多的关于 rst 格式的规范和样式。
