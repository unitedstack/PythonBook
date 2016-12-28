#PDB

---

当需要开始对代码进行debug的时候，Python提供了一个整洁有效的调试模块PDB。

## 简单使用

通过两行代码即可以开始python的代码调试：
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

## 开始调试

输入n可以进入下一行,：
```shell
(Pdb)n
> /root/script.py(12)main()
-> print addition
(Pdb) 
```
