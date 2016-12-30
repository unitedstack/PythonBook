#PDB

当需要开始对代码进行 debug 的时候，Python 提供了一个整洁有效的调试模块 PDB。

## 简单使用

通过两行代码即可以开始 python 的代码调试：
```python
import pdb
pdb.set_trace()
```

一个在程序中进行调试的示例：
```python
import pdb
import sys
def add(num1=0, num2=0):
    return int(num1) + int(num2)
def sub(num1=0, num2=0):
    return int(num1) - int(num2)
def main():
    #Assuming our inputs are valid numbers
    print sys.argv
    pdb.set_trace() # <-- Break point added here
    addition = add(sys.argv[1], sys.argv[2])
    print addition
    subtraction = sub(sys.argv[1], sys.argv[2])
    print subtraction
if __name__ == '__main__':
    main()
```

设置好断点后，我们可以按照平时一样的的方式去执行程序，程序将会在第一个断点处停止执行：
```shell
$ python script.py 1 2
['script.py', '1', '2']
> /root/script.py(11)main()
-> addition = add(sys.argv[1], sys.argv[2])
(Pdb)
```

## 具体操作

输入 n 可以进入下一行，但并不会进入到函数内部，同时回车则表示执行上一条调试命令：
```shell
(Pdb) n
> /root/script.py(12)main()
-> print addition
(Pdb)
```
单击 c 将进入到下一个断点或直接跳到末尾，取决于是否程序下放还是否存在断点。
如果我们希望知道某个变量的内容，可以通过 p 来显示：
```shell
$ python script.py 1 2
['script.py', '1', '2']
> /root/script.py(11)main()
-> addition = add(sys.argv[1], sys.argv[2])
(Pdb) p sys.argv[1]
'1'
(Pdb)
```
如果我们希望能够进入到一个函数内部，则需要使用到单步操作 s，进入到函数内部后，n 和 p 等操作是可以继续使用的，l 则可以显示当前所在位置，而在函数内部时通过 r 则可以快速跳转到函数的返回行：
```shell
(Pdb) s
--Call--
> /root/script.py(3)add()
-> def add(num1=0, num2=0):
(Pdb) n
> /root/script.py(4)add()
-> return int(num1) + int(num2)
(Pdb) p num1
'1'
(Pdb) l
  1  	import pdb
  2  	import sys
  3  	def add(num1=0, num2=0):
  4  ->		return int(num1) + int(num2)
  5  	def sub(num1=0, num2=0):
  6  		return int(num1) - int(num2)
  7  	def main():
  8  	#Assuming our inputs are valid numbers
  9  		print sys.argv
 10  		pdb.set_trace() # <-- Break point added here
 11  		addition = add(sys.argv[1], sys.argv[2])
(Pdb) r
--Return--
> /root/script.py(4)add()->3
-> return int(num1) + int(num2)
(Pdb)
```
当我们在调试已经开始之后想要在程序中特定的地方增加断点的时候，可以通过 b 来实现：
```shell
$ python script.py 1 2
['script.py', '1', '2']
> /root/script.py(11)main()
-> addition = add(sys.argv[1], sys.argv[2])
(Pdb) b 6
Breakpoint 1 at /root/script.py:6
(Pdb) c
3
> /root/script.py(6)sub()
-> return int(num1) - int(num2)
(Pdb) l
  1  	import pdb
  2  	import sys
  3  	def add(num1=0, num2=0):
  4  		return int(num1) + int(num2)
  5  	def sub(num1=0, num2=0):
  6 B->		return int(num1) - int(num2)
  7  	def main():
  8  	#Assuming our inputs are valid numbers
  9  		print sys.argv
 10  		pdb.set_trace() # <-- Break point added here
 11  		addition = add(sys.argv[1], sys.argv[2])
(Pdb)
```
在调试期间，我们可以动态分配变量的值以协助调试：
```shell
$ python script.py 1 2
['script.py', '1', '2']
> /root/script.py(11)main()
-> addition = add(sys.argv[1], sys.argv[2])
(Pdb)  sys.argv[1] = 3
(Pdb) p sys.argv[1]
3
(Pdb) c
5
1
```
如果你想设置一些如 n（即 PDB 指令）这样的变量，你应该使用这种指令：
```shell
(Pdb) !n=5
(Pdb) p n
5
```
最后，在代码的任何地方如果你想结束调试，可以使用“q”，那么正在执行的程序将会终止。

更多内容可参考官方文档[https://docs.python.org/2/library/pdb.html](https://docs.python.org/2/library/pdb.html)。
