# Git

Git是目前世界上最先进的分布式版本控制系统，同时支持极其强大的分支管理。

## 安装

```shell
$ sudo yum install git
$ sudo apt-get install git
```

## 操作

### 创建版本库
版本库又名仓库，英文名 repository，你可以简单的理解一个目录，这个目录里面的所有文件都可以被 Git 管理起来，每个文件的修改，删除，Git 都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻还可以将文件”还原”。
创建版本库的操作非常简单，新建一个空文件夹，并在该目录下执行 git init 命令就可以将该目录变成 git 可管理的仓库。

```shell
$ mkdir newrepo
$ cd newrepo
$ git init
$ ls -a
.  ..  .git
```

### 将文件添加到版本库中
首先该文件一定要位于此版本库文件夹中，通过 git add 命令就可以将文件添加到暂存区中。

```shell
$ echo "this is a new file" > readme.txt
$ git add readme.txt    #此处不会有任何显示，则表示执行成功
```
通过命令 git commit 则可以将文件提交到仓库

```shell
$ git commit -m "add readme"
[master (root-commit)] add readme
 1 file changed, 1 insertion
 create mode 100644 readme.txt
```
则此时已提交了一个 readme.txt 文件

### 查看当前状态
可以通过 git status 查看当前目录是否有文件未提交

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
此时则可以通过 git diff 来显示未提交的内容。

### 版本回退
当我们对文件进行了多次修改后，可以通过 git log 显示修改记录，我们可以看到最近的三次提交：

```shell
$ git log
commit b7323c15962b16266e581893b32936a2af5a77e2

    modify2

commit 4a8bfa9e8677d03b0ce5aa96983279bfb49f8015

    modify

commit a46b3bac345837f1632e48b2cdec30a6ecdd4e12

    add readme
```

如果我们希望将文件回退到之前的版本的时候，则可以使用 git reset 命令

```shell
$ git reset --hard HEAD^    #HEAD^ 表示回退到上一版本,HEAD^^ 则表示上两个版本,以此类推
HEAD is now at 4a8bfa9 modify
```
如果我们希望将文件回退到之前的最新版本，但现在已经无法通过 git log 显示相应的内容，则可以通过 git reflog 查看相应版本号

```shell
$ git reflog
4a8bfa9 HEAD@{0}: reset: moving to HEAD^
b7323c1 HEAD@{1}: commit: modify2
4a8bfa9 HEAD@{2}: commit: modify
a46b3ba HEAD@{3}: commit (initial): add readme
$ git reset --hard b7323c1
HEAD is now at b7323c1 modify2
```

### 管理修改

当我们在工作过程中发现当前的修改内容有误，则可以通过 git checkout 丢弃掉当前的修改内容

```shell
$ git checkout -- readme.txt
```

### 远程仓库

通常与远程仓库的通信过程是通过 ssh 加密的，所以需要我们将公钥上传到相应的远程仓库服务器上。以 GitHub 为例，登录后在右上角的 settings 中可以将我们本地服务器私钥对应的公钥进行导入。
之后我们就可以为我们的本地仓库创建对应的远程仓库，以 GitHub 为例，登录后在右上角找到 “create a new repo”，在 repo name 中填入希望对该仓库的命名，则成功的在 GitHub 上创建了一个仓库。
之后我们则可以通过 git remote add 命令将本地仓库与远程仓库进行关联，通过 git push 将本地仓库内容进行推送。

```shell
$ git remote add origin https://github.com/username/remoterepo.git
$ git push -u origin master    # -u 参数的使用可以将本地的 master 分支和远程 master 分支进行关联，之后的操作就可以省略掉 origin master 参数
Username for 'https://github.com': username
Password for 'https://username@github.com':
Counting objects: 9, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (9/9), 652 bytes | 0 bytes/s, done.
Total 9 (delta 0), reused 0 (delta 0)
To https://github.com/huangmu1987/remoterepo.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

而常见的使用过程中，远程库通常是先于本地库而存在的，那么我们则需要从远程库克隆一个本地库，可以通过 git clone 完成相应的工作，假设我们已存在一个 remotefirst 的远程库。

```shell
$ git clone https://github.com/username/remotefirst.git
Cloning into 'remotefirst'...
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
```

### 分支管理

在 Git 中，除了主分支即 master 分支外，可以通过相应的命令创建其他的分支，常见命令如下：

```shell
$ git checkout -b dev    # -b 参数表示创建并切换，可以理解为是 git branch dev 与 git checkout dev 两条命令的集合
Switched to a new branch 'dev'
$ git branch
* dev
  master
$ echo "add for dev branch" >> README.md
$ git add README.md
$ git commit -m "add for dev branch"
[dev 6844da0] add for dev branch
 1 file changed, 1 insertion(+), 1 deletion(-)
```
对分支的修改不会影响 master 分支中的内容：

```shell
$ git checkout master
Switched to branch 'master'
$ cat README.md
# remotefirst
```

通过命令可以将分支的内容 merge 回 master 分支中：

```shell
$ git merge dev
Updating 379a420..6844da0
Fast-forward
 README.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git branch
* master
dev
$ cat README.md
# remotefirst
add for dev branch
```

当分支的内容已经 merge 回 master 中后，就可以将分支删除：

```shell
$ git branch -d dev
Deleted branch dev (was 6844da0).
$ git branch
* master
```
当有冲突的修改被 merge 到 master 中的时候，会被提示有冲突存在：

```shell
$ git merge dev
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
$ git status
# On branch master
# Your branch is ahead of 'origin/master' by 2 commits.
#   (use "git push" to publish your local commits)
#
# You have unmerged paths.
#   (fix conflicts and run "git commit")
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)
#
#	both modified:      README.md
#
no changes added to commit (use "git add" and/or "git commit -a")
$ cat README.md
# remotefirst
add for dev branch
<<<<<<< HEAD
to conflict from master
=======
to conflict
>>>>>>> dev
```

冲突的解决办法只有修改相应的内容后再次提交。

```shell
$ git checkout dev
README.md: needs merge
error: you need to resolve your current index first
...# 修改相应内容后再次提交
$ git add README.md
$ git commit -m "to merge"
[dev b838ac8] to merge
 1 file changed, 1 insertion(+), 1 deletion(-)
$ git checkout master
$ git merge dev
Already up-to-date.
```

### 标签管理

在 Git 中打标签非常简单，通过 git tag 命令即可完成：

```shell
$ git tag v1.0
$ git tag
v1.0
```

可以通过携带具体的版本号实现给之前的版本添加 tag：

```shell
$ git log --pretty=oneline --abbrev-commit
6a5819e merged bug fix 101
cc17032 fix bug 101
7825a50 merge with no-ff
6224937 add merge
59bc1cb conflict fixed
400b400 & simple
75a857c AND simple
fec145a branch test
d17efd8 remove test.txt
$ git tag v0.9 6224937
$ git tag
v0.9
v1.0
$ git show v0.9
commit 622493706ab447b6bb37e4e2a2f276a20fed2ab4

    add merge
```

tag 中也可以添加详细的描述：

```shell
$ git tag -a v0.1 -m "version 0.1 released" 3628164
$ git show v0.1
tag v0.1

version 0.1 released
```

可以通过 -d 参数删除 tag：

```shell
$ git tag -d v0.1
Deleted tag 'v0.1' (was e078af9)
```

当有远程库的时候，也可以将 tag 同步或删除：

```shell
$ git push origin v1.0
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:username/learngit.git
 * [new tag]         v1.0 -> v1.0
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 554 bytes, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:username/learngit.git
 * [new tag]         v0.2 -> v0.2
 * [new tag]         v0.9 -> v0.9
$ git tag -d v0.9
Deleted tag 'v0.9' (was 6224937)
$ git push origin :refs/tags/v0.9
To git@github.com:username/learngit.git
 - [deleted]         v0.9
```
