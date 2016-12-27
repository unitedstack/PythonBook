# Pip

---

## 前言

pip 是一个Python包管理工具，主要是用于安装 PyPI 上的软件包，可以替代 easy\_install 工具。、
* GitHub: https://github.com/pypa/pip
* Doc: https://pip.pypa.io/en/latest/

## 安装

脚本安装pip
```shell
$ curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
$ python get-pip.py
```
工具安装pip
```shell
$ sudo yum install python-pip
$ sudo apt-get install python-pip
```
更新pip
```shell
$ pip install -U pip
```

##使用

安装软件
```shell
$ pip install somepackage
```
查看具体安装文件
```shell
$ pip show --files somepackage

  Name: SomePackage
  Version: 1.0
  Location: /my/env/lib/pythonx.x/site-packages
  Files:
   ../somepackage/__init__.py
   [...]
```
查看哪些软件需要更新
```shell
$ pip list --outdated

  somepackage (Current: 1.0 Latest: 2.0)
```
升级软件包
```shell
$ pip install --upgrade SomePackage

  [...]
  Found existing installation: SomePackage 1.0
  Uninstalling SomePackage:
    Successfully uninstalled SomePackage
  Running setup.py install for SomePackage
  Successfully installed SomePackage
```
卸载软件包
```shell
$ pip uninstall SomePackage

  Uninstalling SomePackage:
    /my/env/lib/pythonx.x/site-packages/somepackage
  Proceed (y/n)? y
  Successfully uninstalled SomePackage
```
搜索软件包
```shell
$ pip search pycuda

pycuda                    - Python wrapper for Nvidia CUDA
pyfft                     - FFT library for PyCuda and PyOpenCL
cudatree                  - Random Forests for the GPU using PyCUDA
reikna                    - GPGPU algorithms for PyCUDA and PyOpenCL
compyte                   - A common set of compute primitives for PyCUDA and PyOpenCL (to be created)
```
配置pip
配置文件: $HOME/.pip/pip.conf, 举例:
```shell
[global]
timeout = 60
index-url = http://download.zope.org/ppix

[install]
ignore-installed = true
no-dependencies = yes
```
