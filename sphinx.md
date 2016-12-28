# Sphinx文档构建

---

OpenStack所有的项目都要求有对应的文档，如果你正在书写一个OpenStack项目那么你就需要构建你的一个文档工程。默认的OpenStack构建文档都是使用了Sphinx这个模块，主要是用于生成一个文档目录。用Sphinx构建的文档，格式都是rst格式的，这种格式的写法和传统的markdown文档不太一样。下面我们为我们的webdemo项目来构建一个OpenStack文档系统。



## Sphinx安装

```
pip install sphinx
```

pip源里面就自带了sphinx这个包，所以可以直接安装实现



