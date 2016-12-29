# Python高级特性

---

我们在上章的Python基础中讲到了Python的高级特性，之所以叫做Python的高级特性，是因为Python因为这些特性而区别于其他语言，就是说并不是每个语言都有的特性，或者说只有Python自带了这种特性。

当然，Python的高级特性有很多，我们就讲一些我们经常会被大家忽略的特性就好，提炼重点。

## 装饰器
相信很多资料都有讲什么事装饰器，装饰器是干什么的。所以我就简单的提一下，对装饰器的很多使用方法可以参考：

装饰器，顾名思义就是对函数或者类进行装饰用的。语法如下：


``` python
@decorator             
def function():        
    pass
```
这边的这种写法就等于

``` python
def function():                  
    pass
function = decorator(function)   
```
function做为decorator的入参，然后又把处理完后的值赋给function.往往在decorator里面做完处理之后又返回了一个function.如：


``` python
def decorator(funtion):
    print "doing decoration"
    return funtion
```
这样在调用function的时候都会先进入decorator也就是装饰器里执行装饰器的具体实现，最后再返回原来的function。这是基础用法，下面我想讲的是装饰器的一些特性，在很多资料中都没有提到，然后再OpenStack中处处可见的。就是一写标准库里的一些装饰器

### 装饰器classmethod
classmethod让一个方法变成“类方法”，即它能够无需创建实例调用。当一个常规方法被调用时，解释器插入实例对象作为第一个参数self。当类方法被调用时，类本身被给做第一个参数，一般叫cls。
类方法也能通过类命名空间读取，所以它们不必污染模块命名空间。类方法可用来提供替代的构建器(constructor):

``` python
class Kls(object):

    def __init__(self, data):
        self.data = data

    def printd(self):
        print(self.data)
    
    @classmethod
    def cls_print(cls):
        print('Class:', cls)    

    @classmethod
    def cmethod(*arg):
        print('Class:', arg)

kls.cmethod()
```
这比用一大堆标记的__init__简单多了。这个代码的输出：


``` python
>>> Kls.cmethod()
Class: (<class '__main__.Kls'>,)
>>> Kls.cls_print()
Class: (<class '__main__.Kls'>,)
```

### 装饰器staticmethod
应用到方法上让它们“静态”，例如，本来一个常规函数，但通过类命名空间存取。这在函数仅在类中需要时有用(它的名字应该以_为前缀)，或者当我们想要用户以为方法连接到类时也有用——虽然对实现本身不必要。
``` python

class Kls(object):

    def __init__(self, data):
        self.data = data

    def printd(self):
        print(self.data)

    @staticmethod
    def cmethod(*arg):
        print('Class:', arg)

kls.cmethod()
```
输出：
``` python
>>> Kls.smethod()
Static: ()
```
我们可以看到这里的 `classmethod` 和 `staticmethod` 只是输入参数不一样而已，都是声明了一个类方法，可以在外部直接调用而不需要去声明这个类。唯一不同的是`classmethod`会自动传入当前类的值，也就是self.

### 装饰器property
`property`是对`getter`和`setter`问题Python风格的答案。通过`property`装饰的方法变成在属性存取时自动调用的`getter`。


```python
>>> class D(object):
...    @property
...    def a(self):
...      print "getting", 1
...      return 1
...    @a.setter
...    def a(self, value):
...      print "setting", value
...    @a.deleter
...    def a(self):
...      print "deleting"
>>> D.a                                    
<property object at 0x...>
>>> D.a.fget                               
<function a at 0x...>
>>> D.a.fset                               
<function a at 0x...>
>>> D.a.fdel                               
<function a at 0x...>
>>> d = D()               # ... varies, this is not the same `a` function
>>> d.a
getting 1
1
>>> d.a = 2
setting 2
>>> del d.a
deleting
>>> d.a
getting 1
1
```
通过`property`装饰器取代带一个属性(`property`)对象的`getter`方法，以上代码起作用。这个对象反过来有三个可用于装饰器的方法`getter`、`setter`和`deleter`。它们的作用就是设定属性对象的`getter`、`setter`和`deleter`(被存储为`fget`、`fset`和`fdel`属性(`attributes`))。当创建对象时，`getter`可以像上例一样设定。当定义`setter`时，我们已经在`area`中有`property`对象，可以通过`setter`方法向它添加`setter`，一切都在创建类时完成。其实上面的代码就是重写了类的定义，删除的类方法。


### 装饰器contextlib
上下文管理器是可以在with语句中使用,拥有__enter__和__exit__方法的对象。
``` python
>>> class closing(object):
...   def __init__(self, obj):
...     self.obj = obj
...   def __enter__(self):
...     return self.obj
...   def __exit__(self, *args):
...     self.obj.close()
>>> with closing(open('/tmp/file', 'w')) as f:
...   f.write('the contents\n')
```
这就是上下文管理工具的正常用法，其实就是在执行一件事之前要做什么，执行完之后又做什么的一个封装。但是在OpenStack或者其他大型项目里面，有contextlib这个库可以直接使用，如：

``` python
import contextlib

@contextlib.contextmanager
def test():
    print 'perpare test'
    yield
    print 'after test'


with test():
    print 'hello world'
```
contextlib.contextmanager装饰器提供了一个上下文的管理器,
但是你还是需要指定上下文，使用yield：
如下面:
   yield 前面的部分就相当于 __enter__
   yield 后面就相当于 __exit__

## 小结
