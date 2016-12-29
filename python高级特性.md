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





