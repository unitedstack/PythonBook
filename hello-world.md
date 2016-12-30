# Python 基础

本教程适合想从零开始学习 Python 编程语言的开发人员。当然本教程也会对一些模块进行深入，让你更好的了解 Python 的应用。

## 构建环境

Python 已经经历过很长的时间的磨炼，现在有2.6，2.7，3.0+这3种大版本。
默认情况下，CentOS 的发行版都默认安装了 python。查看本地的 python 版本可以使用：

```shell
python -V
```

如果你的 python 工具没有安装，那么你可以通过：

```shell
# Centos/RHEL/Fedorq
yum install -y python

# Ubuntu/Debian
sudo apt-get install python
```

## Hello World

有了上面的Python环境，你可以执行你的第一个 Python 程序，跟大家打声招呼：

```python
#!/usr/bin/python

print "Hello, World!";
```

至此，你开始成为 Python 世界的一员。

前面章节中我们已经学会了如何用 Python 输出 "Hello, World!"，英文没有问题，但是如果你输出中文字符"你好，世界"就有可能会碰到中文编码问题。

## Python中文支持

Python 文件中如果未指定编码，在执行过程会出现报错：

```
#!/usr/bin/python

print "你好，世界";
```

以上程序执行输出结果为：

```
File "test.py", line 2
SyntaxError: Non-ASCII character '\xe4' in file test.py on line 2, but no encoding declared; see http://www.python.org/peps/pep-0263.html for details
```

Python 中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。

解决方法为只要在文件开头加入\# -\*- coding: UTF-8 -\*-或者\#coding=utf-8就行了。

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-

print "你好，世界";
```

## 简单快速入门

### 判断

```python
#!/usr/bin/python

flag = False
name = 'luren'
if name == 'python':
    flag = True
    print 'welcome boss'
else:
    print name
```

### for 循环

```python
#!/usr/bin/python

for letter in 'Python':
   print 'Now :', letter

fruits = ['banana', 'apple',  'mango']
for fruit in fruits:
   print 'Now :', fruit

print "Good bye!"
```

### while 循环

```python
#!/usr/bin/python

count = 0
while (count < 9):
   print 'The count is:', count
   count = count + 1

print "Good bye!"

while (count < 9):
   print 'The count is:', count
   count = count + 1
   break
```

## Python 数据类型

Python 主要支持的数据类型有 List\(列表\), Map\(字典\), String\(字符串\)。

```Python
#!/usr/bin/python

a = ['a', 'b']
print type(a)

b = {
    "key1": 123;
    "key2": 123;
}
print type(b)

c = "String"
print type(c)
```

## Python 模块引用

模块是一个保存了 Python 代码的文件。模块能定义函数，类和变量。模块里也能包含可执行的代码。
下例是个简单的模块 support.py

```Python
def print_func( par ):
   print "Hello : ", par
   return
```

想使用 Python 源文件，只需在另一个源文件里执行 import 语句，如下：

```Python
#!/usr/bin/python

# 导入模块
import support

# 现在可以调用模块里包含的函数了
support.print_func("Zara")
```

Python的from 语句让你从模块中导入一个指定的部分到当前命名空间中，如下:

```Python
#!/usr/bin/python

# 导入模块
from support import print_func

# 现在可以调用模块里包含的函数了
support.print_func("Zara")
```

把一个模块的所有内容全都导入到当前的命名空间也是可行的，只需使用如下声明：

```Python
from modname import *
```

当你导入一个模块，Python解析器对模块位置的搜索顺序是：

* 当前目录
* 如果不在当前目录，Python 则搜索在 shell 变量 PYTHONPATH 下的每个目录
* 如果都找不到，Python 会察看默认路径。Linux 下，默认路径一般为/usr/lib/python/

模块搜索路径存储在 system 模块的 sys.path 变量中。变量里包含当前目录，PYTHONPATH 和由安装过程决定的默认目录。

Ref: [http://www.runoob.com/python/python-tutorial.html](http://www.runoob.com/python/python-tutorial.html)

