# IPython

---

IPython是一个交互式的python shell，性能与可用性优于python shell

# 安装

```shell
$ sudo yum install ipython
$ sudo apt-get install ipython
$ sudo pip install ipython
```

# 使用

## 帮助命令

| 命令 | 描述 |
| :--- | :--- |
| ? | 简介及IPython特性描述 |
| %quickref | 快速入门 |
| help | Python帮助内容 |
| object? | 有关‘object’的帮助内容，object??提供更加精确的内容 |

## 自动补齐

使用Tab可以将待输入内容进行自动补齐，同时可以通过object.&lt;tab&gt;支持对象中的属性

## 命令支持

支持直接在ipython命令行中输入ls与cd等命令，执行其他shell命令可通过在命令前加!

## 配置

通过ipython profile create命令生成IPython默认配置文件，位于~/.ipython/profile_default，可对其进行修改，更多细节内容参考[Profiles](http://ipython.readthedocs.io/en/stable/config/intro.html#profiles)



