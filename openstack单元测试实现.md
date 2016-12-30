# tox单元测试实现

首先，我们需要先书写我们 tox.ini：

## 编写配置文件：

tox.ini 文件是我们所有测试的主要的配置文件，开始的时候我们只需要：

```
[root@devstack webdemo]# tree  .
.
├── LICENSE
├── README.md
├── requirements.txt
├── setup.cfg
├── setup.py
├── tox.ini # 在当前目录下面添加了一个tox配置文件
```

## PEP8静态语法测试

我们在 tox 里面加 pep8 的测试，主要是用 flake8 这个工具。

```
[tox]
envlist = pep8
minversion = 1.6
skipsdist = True

[testenv]
setenv = VIRTUAL_ENV={envdir}
usedevelop = True
deps = -r{toxinidir}/requirements.txt
       -r{toxinidir}/test-requirements.txt
install_command = pip install -U {opts} {packages}
commands = find . -type f -name "*.py[c|o]" -delete
           python setup.py testr --slowest --testr-args='{posargs}'
whitelist_externals = find

[testenv:pep8]
deps = flake8
commands =
    flake8 ./webdemo
```

我们大概来讲解下，这个文件的含义：

```
[testenv]
setenv = VIRTUAL_ENV={envdir}
usedevelop = True
deps = -r{toxinidir}/test-requirements.txt
install_command = pip install -U {opts} {packages}
commands = find . -type f -name "*.py[c|o]" -delete
           python setup.py testr --slowest --testr-args='{posargs}'
whitelist_externals = find
```

* deps
    **是指第一次运行这个的时候需要安装的依赖。这里指定的是 tox.ini 当前目录下面的 test-requirements.txt 这个文件，就是说运行 tox 之前会自动安装这里面的所有的依赖，这里可以先不写这个文件。**

* install\_command
    **是指你用什么命令去安装依赖，这里指的是使用 pip**

* commands
    **指的是你运行 tox 之前会干什么，这里是删除所有的 python 编译文件还有运行 testr 的测试**

```
[tox]
envlist = pep8
minversion = 1.6
skipsdist = True
```

* envlist 表示你会找到下面哪一个模块进行测试。这里指定了 pep8 这个模块

在下面的模块中：

```
[testenv:pep8]
deps = flake8
commands =
    flake8 ./webdemo
```

* deps 是指依赖什么库，运行的时候会自动使用 pip 安装
* commands 是指 tox -e pep8 的时候运行什么命令，这边使用 flake8 去检测当前目录下面的 webdemo 的语法规范

### 运行测试

```
# tox -e pep8
pep8 develop-inst-nodeps: /root/webdemo
pep8 installed: alembic==0.8.9,beautifulsoup4==4.5.1,configparser==3.5.0,enum34==1.1.6,flake8==3.2.1,logutils==0.3.3,Mako==1.0.6,MarkupSafe==0.23,mccabe==0.5.3,netaddr==0.7.18,pbr==1.10.0,pecan==1.2.1,pycodestyle==2.2.0,pyflakes==1.3.0,python-editor==1.0.3,pytz==2016.10,simplegeneric==0.8.1,singledispatch==3.4.0.3,six==1.10.0,SQLAlchemy==1.0.16,waitress==1.0.1,-e git+https://github.com/diabloneo/webdemo@bd89a5407bac92020aa152f1b1c497fa92b814cc#egg=webdemo,WebOb==1.7.0,WebTest==2.0.24,WSME==0.8.0
pep8 runtests: PYTHONHASHSEED='1631962962'
pep8 runtests: commands[0] | flake8 ./webdemo
./webdemo/db/sqlalchemy/alembic/env.py:67:1: E305 expected 2 blank lines after class or function definition, found 1
./webdemo/db/sqlalchemy/alembic/versions/4bafdb464737_create_user_table.py:15:1: E402 module level import not at top of file
./webdemo/db/sqlalchemy/alembic/versions/4bafdb464737_create_user_table.py:16:1: E402 module level import not at top of file
ERROR: InvocationError: '/root/webdemo/.tox/pep8/bin/flake8 ./webdemo'
```


## Python2.7 单元测试

### Py27 环境配置

在上面的例子中我们已经加入了 pep8 的静态语法测试，那么下面我们就可以加入我们的 Python2.7 的单元测试了，首先我们要吧这个模块添加到我们的 tox 配置文件里面：

```
[tox]
envlist = pep8,py27
minversion = 1.6
skipsdist = True

...
[testenv:debug-py27]
deps = test
basepython = python2.7
commands =
   oslo_debug_helper -t webdemo/tests {posargs}
...
```

这个测试需要我们在当前目录下建立一个测试文件：

```
# cat .testr.conf 
[DEFAULT]
test_command=
    PYTHON=$(echo ${PYTHON:-python} | sed 's/--source webdemo//g')
    START_AT=${TESTR_START_DIR:-.}
    ${PYTHON} -m subunit.run discover -s $START_AT -t . $LISTOPT $IDOPTION
test_id_option=--load-list $IDFILE
test_list_option=--list
```

然后书写我们的依赖,也就是我们的 test-requirements.txt 文件

```
coverage>=4.0 # Apache-2.0
fixtures>=3.0.0 # Apache-2.0/BSD
requests-mock>=1.1 # Apache-2.0
mock>=2.0 # BSD
mox3>=0.7.0 # Apache-2.0
oslosphinx>=4.7.0 # Apache-2.0
oslotest>=1.10.0 # Apache-2.0
python-openstackclient>=3.3.0 # Apache-2.0
sphinx!=1.3b1,<1.4,>=1.2.1 # BSD
tempest>=12.1.0 # Apache-2.0
testrepository>=0.0.18 # Apache-2.0/BSD
testscenarios>=0.4 # Apache-2.0/BSD
testtools>=1.4.0 # MIT
mox>=0.5
```

尝试运行:

在运行测试之前，需要一些 RPM 包的支持:

```
$ yum install -y openssl-devel postgresql-devel* libffi-devel libxslt-devel

# 运行测试
$ tox -e py27
...
____________________________________________________________________ summary ____________________________________________________________________
  py27: commands succeeded
  congratulations :)
```

因为我们并没有书写任何的单元测试函数，所以说运行测试肯定会通过的。我们在下节就会讲到利用mock做单元测试。

## 小结

tox 的使用方法还有很多，具体的其他功能可以参照官网的介绍：[http://tox.readthedocs.io/en/latest/](http://tox.readthedocs.io/en/latest/) 。这里的简单介绍已经足够你了解一个 OpenStack 工程中的 tox 要怎么写。

