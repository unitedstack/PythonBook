# IPython

IPython 是一个交互式的 python shell，性能与可用性优于 python shell

## 安装

```shell
$ sudo yum install ipython
$ sudo apt-get install ipython
$ sudo pip install ipython
```

## 使用

### 帮助命令

| 命令 | 描述 |
| :--- | :--- |
| ? | 简介及 IPython 特性描述 |
| %quickref | 快速入门 |
| help | Python 帮助内容 |
| object? | 有关 ‘object’ 的帮助内容，object ??提供更加精确的内容 |

### 自动补齐

使用 Tab 可以将待输入内容进行自动补齐，同时可以通过 object.&lt;tab&gt; 支持对象中的属性

### 命令支持

支持直接在 ipython 命令行中输入 ls 与 cd 等命令，执行其他 shell 命令可通过在命令前加!

### 配置

通过 ipython profile create 命令生成 IPython 默认配置文件，位于 ~/.ipython/profile_default，可对其进行修改，更多细节内容参考 Profiles [http://ipython.readthedocs.io/en/stable/config/intro.html#profiles](http://ipython.readthedocs.io/en/stable/config/intro.html#profiles)



