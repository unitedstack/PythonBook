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

# ubuntu/debian
sudo apt-get install python
```

## Hello World

有了上面的 Python 环境，你可以执行你的第一个 Python 程序，跟大家打声招呼：

```python
#!/usr/bin/python

print "Hello, World!";
```

至此，你开始成为 Python 世界的一员。

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



