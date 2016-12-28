# Sphinx文档构建

---

OpenStack所有的项目都要求有对应的文档，如果你正在书写一个OpenStack项目那么你就需要构建你的一个文档工程。默认的OpenStack构建文档都是使用了Sphinx这个模块，主要是用于生成一个文档目录。用Sphinx构建的文档，格式都是rst格式的，如: http://docs.openstack.org/developer/nova/ ，这种格式的写法和传统的markdown文档不太一样。下面我们为我们的webdemo项目来构建一个OpenStack文档系统。
## Sphinx安装

```
pip install sphinx
```

pip源里面就自带了sphinx这个包，所以可以直接安装实现。

## 生成文档

```
# cd webdemo
# mkdir doc
# sphinx-quickstart
# ls
_build  conf.py  index.rst  make.bat  Makefile  _static  _templates
```

默认情况下我们生成的是 rst 格式的。要把rst格式的文件构建成一个可视化的模板，我们可以sphinx-build -b html . output 或者 make html命令生成html文档，生成的文档位于output目录 。你只需要把这个output目录发布出来，那么你的文档就可以在网站上打开了。

## 小结

Sphinx的使用其实很简单，但是构建文档的时候要注意rst格式的语法规范。详情可以参照：http://ju.outofmemory.cn/entry/105733 ，这里面讲解了很多的关于rst格式的规范和样式。