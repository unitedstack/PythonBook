# Git

---

Git是目前世界上最先进的分布式版本控制系统，同时支持极其强大的分支管理。

# 安装

```shell
$ sudo yum install git
$ sudo apt-get install git
```

# 操作

## 创建版本库
版本库又名仓库，英文名repository,你可以简单的理解一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改，删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻还可以将文件”还原”。
创建版本库的操作非常简单，新建一个空文件夹，并在该目录下执行git init命令就可以将该目录变成git可管理的仓库。
```shell
$ mkdir newrepo
$ cd newrepo
$ git init
$ ls -a
.  ..  .git    
```

## 将文件添加到版本库中
首先该文件一定要位于此版本库文件夹中，通过git add命令就可以将文件添加到暂存区中。
```shell
$ echo "this is a new file" > readme.txt
$ git add readme.txt    #此处不会有任何显示，则表示执行成功
```
通过命令git commit则可以将文件提交到仓库
```shell
$ git commit -m "add readme"
[master (root-commit)] add readme
 1 file changed, 1 insertion
 create mode 100644 readme.txt
```
则此时已提交了一个readme.txt文件

## 查看当前状态
可以通过git status查看当前目录是否有文件未提交

无未提交示例：
```shell
$ git status
# On branch master
nothing to commit, working directory clean
```
存在未提交示例：
```shell
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#	modified:   readme.txt
#
no changes added to commit (use "git add" and/or "git commit -a")
```
此时则可以通过git diff来显示未提交的内容。

## 版本回退
当我们对文件进行了多次修改后，可以通过git log显示修改记录,我们可以看到最近的三次提交：
```shell
$ git log
commit b7323c15962b16266e581893b32936a2af5a77e2
Author: 
Date:   

    modify2

commit 4a8bfa9e8677d03b0ce5aa96983279bfb49f8015
Author: 
Date:   

    modify

commit a46b3bac345837f1632e48b2cdec30a6ecdd4e12
Author: 
Date:   

    add readme
```
如果我们希望将文件回退到之前的版本的时候,则可以使用git reset命令
```shell

```