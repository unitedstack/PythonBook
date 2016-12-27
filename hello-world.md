# Python基础

---

本教程适合想从零开始学习Python编程语言的开发人员。当然本教程也会对一些模块进行深入，让你更好的了解Python的应用。

## 构建环境

Python已经经历过很长的时间的磨炼，现在有2.6，2.7，3.0+这3种大版本。  
默认情况下，Centos的发行版都默认安装了python。查看本地的python版本可以使用：

```shell
python -V
```

如果你的python工具没有安装，那么你可以通过：

```shell
# Centos/RHEL/Fedorq
yum install -y python

# ubuntu/debian
sudo apt-get install python
```

## Hello World

有了上面的Python环境，你可以执行你的第一个Python程序，跟大家打声招呼：

```python
#!/usr/bin/python

print "Hello, World!";
```

至此，你开始成为Python世界的一员。

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

Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。

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

## Python数据类型

Python主要支持的数据类型有List\(列表\),Map\(字典\),String\(字符串\)。

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



