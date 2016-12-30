# Keystone 的单元测试框架

现在，我们以 Keystone 项目为例，来看下真实项目中的单元测试是如何架构的。我们采用自顶向下的方式，先从最上层的部分介绍起。

## 使用 tox 进行测试环境管理

大部分情况下，我们都是通过 tox 命令来执行单元测试的，并且传递环境名称给 tox 命令：

```
$ tox -e py27
```

tox命令首先会读取项目根目录下的 _tox.ini_ 文件，获取相关的信息，然后根据配置构建 virtualenv，保存在 _.tox/_ 目录下，以环境名称命名：

```
ls .tox
```

除了 _log_ 目录，其他的都是普通的 virtualenv 环境，你可以自己查看一下内容。我们来看下 _py27_ 这个环境的相关配置（在 tox.ini）中，我直接在内容上注释一些配置的用途：

```
[tox]
minversion = 1.6
skipsdist = True
# envlist 表示本文件中配置的环境都有哪些
envlist = py34,py27,pep8,docs,genconfig,releasenotes

# testenv 是默认配置，如果某个配置在环境专属的 section 中没有，就从这个 section 中读取
[testenv]
# usedevelop 表示安装 virtualenv 的时候，本项目自己的代码采用开发模式安装，也就是不会拷贝代码到 virtualenv 目录中，只是做个链接
usedevelop = True
# install_command 表示构建环境的时候要执行的命令，一般是使用 pip 安装
install_command = pip install -U {opts} {packages}
setenv = VIRTUAL_ENV={envdir}
# deps 指定构建环境的时候需要安装的依赖包，这个就是作为 pip 命令的参数
# keystone 这里使用的写法比较特殊一点，第二行的.[ldap,memcache,mongodb]是两个依赖，第一个点'.'表示当前项目的依赖，也就是 requirements.txt，第二个部分[ldap,memcache,mongodb]表示 extra，是在 setup.cfg 文件中定义的一个段的名称，该段下定义了额外的依赖，这些可以查看 PEP0508
# 一般的项目这里会采用更简单的方式来书写，直接安装两个文件中的依赖：
#    -r{toxinidir}/requirements.txt
#    -r{toxinidir}/test-requirements.txt
deps = -r{toxinidir}/test-requirements.txt
       .[ldap,memcache,mongodb]
# commands 表示构建好 virtualenv 之后要执行的命令，这里调用了 tools/pretty_tox.sh 来执行测试
commands =
  find keystone -type f -name "*.pyc" -delete
  bash tools/pretty_tox.sh '{posargs}'
whitelist_externals =
  bash
  find
passenv = http_proxy HTTP_PROXY https_proxy HTTPS_PROXY no_proxy NO_PROXY PBR_VERSION

# 这个 section 是为 py34 环境定制某些配置的，没有定制的配置，从[testenv]读取
[testenv:py34]
commands =
  find keystone -type f -name "*.pyc" -delete
  bash tools/pretty_tox_py3.sh
```

上面提到的 PEP-0508 [https://www.python.org/dev/peps/pep-0508/](https://www.python.org/dev/peps/pep-0508/)是依赖格式的完整说明。setup.cfg的_extra_部分如下：

```
[extras]
ldap =
  python-ldap>=2.4:python_version=='2.7' # PSF
  ldappool>=1.0:python_version=='2.7' # MPL
memcache =
  python-memcached>=1.56 # PSF
mongodb =
  pymongo!=3.1,>=3.0.2 # Apache-2.0
bandit =
  bandit>=0.17.3 # Apache-2.0
```

## 使用testrepository管理测试的运行 

上面我们看到_tox.ini_文件中的 `commands` 参数中执行的是_tools/pretty\_tox.sh_命令。这个脚本的内容如下：

```
#!/usr/bin/env bash

set -o pipefail

TESTRARGS=$1
# testr 和 setuptools 已经集成，所以可以通过 setup.py testr 命令来执行
# --testr-args 表示传递给 testr 命令的参数，告诉 testr 要传递给 subunit 的参数
# subunit-trace 是 os-testr 包中的命令（os-testr 是 OpenStack 的一个项目），用来解析 subunit 的输出的。
python setup.py testr --testr-args="--subunit $TESTRARGS" | subunit-trace -f
retval=$?
# NOTE(mtreinish) The pipe above would eat the slowest display from pbr's testr
# wrapper so just manually print the slowest tests.
echo -e "\nSlowest Tests:\n"
# 测试结束后，让 testr 显示出执行时间最长的那些测试用例
testr slowest
exit $retval
```

tox 就是从 _tools/pretty\_tox.sh_ 这个命令开始调用 testr 来执行单元测试的。testr 本身的配置是放在项目根目录下的_.testr.conf_文件：
```
    [DEFAULT]
    test_command=
        ${PYTHON:-python} -m subunit.run discover -t ./ ${OS_TEST_PATH:-./keystone/tests/unit} $LISTOPT $IDOPTION

    test_id_option=--load-list $IDFILE
    test_list_option=--list
    group_regex=.*(test_cert_setup)


    # NOTE(morganfainberg): If single-worker mode is wanted (e.g. for live tests)
    # the environment variable ``TEST_RUN_CONCURRENCY`` should be set to ``1``. If
    # a non-default (1 worker per available core) concurrency is desired, set
    # environment variable ``TEST_RUN_CONCURRENCY`` to the desired number of
    # workers.
    test_run_concurrency=echo ${TEST_RUN_CONCURRENCY:-0}
```
这个文件中的配置项可以从 testr 官方文档[http://testrepository.readthedocs.org/en/latest/](http://testrepository.readthedocs.org/en/latest/)中找到。其中 `test_command` 命令表示要执行什么命令来运行测试用例，这里使用的是 `subunit.run`，这个我们在上面提到过了。

到目前为止的流程就是：

1. tox 建好 virtualenv

2. tox 调用testr

3. testr 调用 subunit 来执行测试用例

每个 OpenStack 项目基本上也都是这样。如果你自己在开发一个 Python 项目，你也可以参考这个架构。

从上面的层次结构可以看出，OpenStack 中的大项目，由于单元测试用例很多（Keystone 现在有超过6200个单元测试用例），所以其单元测试架构也会比较复杂。要写好单元测试，需要先了解一下整个测试代码的架构。

## 总结

本文我们了解了 Python 中的单元测试的概念和工具，并且通过 Keystone 项目了解了实际项目中的单元测试的架构，希望有助于各位读者更好的掌握 OpenStack 项目的单元测试基础。webdemo 项目[https://github.com/diabloneo/webdemo](https://github.com/diabloneo/webdemo) 目前没有单元测试的代码，接下来我们就为它添加单元测试框架。

