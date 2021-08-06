## 提交到review审核

### 第一次提交

```bash
# 提交之前先同步网站最新代码，避免出现冲突
git fetch
# git rebase origin/master
git add .
git commit -m "提交信息"
# git commit --amend  # 可修改提交信息，添加Jira等
# git review -t UOSP-2332
# 这样可以把提交的topic设置上，便于后续跟踪
# 提交平台审核
git review
```

### 修改后提交

```bash
git fetch
git add .
git commit --amend  # 可修改提交信息，添加Jira等
git review
```

### 第一次提交多余文件

```bash
# 上次提交了多余的test.txt文件，下一次提交需要删除该文件
git rm test.txt
```

### rebase

```bash
git fetch --all
git rebase gerrit/master
git review
```

### 撤销工作空间改动代码，撤销commit，撤销git add . 

```bash
git reset --hard
```



## 创建版本库

### 把目录变成Git可以管理的仓库

```git
$ git init
Initialized empty Git repository in F:/Git/learngit/.git/
```

### 把文件添加到仓库

`git add`命令

```git
$ git add readme.txt
```

### 把文件提交到仓库

`git commit -m "xxx"`

```git
$ git commit -m "wrote a readme file"
[master (root-commit) e86b00a] wrote a readme file
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt
```

添加文件到Git仓库，分两步：

1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
2. 使用命令`git commit -m <message>`，完成。

### 掌握仓库当前的状态

`git status`

```git
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

### 查看具体修改了什么内容

`git diff <file>`

```git
$ git diff readme.txt
diff --git a/readme.txt b/readme.txt
index d8036c1..013b5bc 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.
\ No newline at end of file
```

### 查看历史记录

````
git log
显示最近到最远的提交日志(详细)
````

```git
git log --pretty=oneline
显示大致信息

$ git log --pretty=oneline
9df3bf644a179d2b9aa7efc0abdb211ec7a9b9af (HEAD -> master) append GPL
180912b6d8f074d001cf9b7083e6a5578e3aa916 add distributed
e86b00a4ac3d2c302a5cb7ff69bc482f2ce92df0 wrote a readme file
```

## 时光回溯

### 版本回退

查看记录表示:`HEAD`表示当前版本,上个版本是`HEAD^`,上上个版本就是`HEAD^^`,往上一百个版本就是`HEAD~100`

把当前版本`append GPL`回退到上一个版本`add distributed`，就可以使用`git reset`命令

```git
$ git reset --hard HEAD^
HEAD is now at 180912b add distributed
```

### 查看内容

```git
$ cat readme.txt
Git is a distributed version control system.
Git is free software.
```

### 回退之后又想回去

命令窗口还没关闭,找到开始版本的hash值(不必写全,能区分就行),使用这个命令

```git
$ git reset --hard 9df3b
HEAD is now at 9df3bf6 append GPL
```

`git reflog`用来记录你的每一次命令,可以找到对应的hash版本恢复

```git
$ git reflog
9df3bf6 (HEAD -> master) HEAD@{0}: reset: moving to 9df3b
180912b HEAD@{1}: reset: moving to HEAD^
9df3bf6 (HEAD -> master) HEAD@{2}: commit: append GPL
180912b HEAD@{3}: commit: add distributed
e86b00a HEAD@{4}: commit (initial): wrote a readme file
```

### 查看工作区和版本库里面最新版本的区别

```git
git diff HEAD -- readme.txt
```

多次修改进行提交可以使用多次`git add`

### 丢弃工作区的修改

**撤销修改**

```git
$ git restore readme.txt
或,--重要不能丢
$ git checkout -- readme.txt
```

这个命令是让这个文件回到最近一次`git commit`或`git add`时的状态。

万一已经`git add readme.txt`:

```git
git restore --staged readme.txt
该命令将add操作撤销,将改动放回工作区
然后再使用上面的命令丢弃工作区
git restore readme.txt
```

### 删除误删

手动删除工作区文件,但是git还没删除,要用`git rm test.txt`删除,然后`git commit`提交,版本库才能完全删除.

如果手动误删,可以使用`git restore test.txt`从版本库恢复.

## 远程仓库

[连接远程仓库](https://www.liaoxuefeng.com/wiki/896043488029600/898732864121440)

### 关联远程仓库

```git
$  git remote add origin git@github.com:guogander/learngit.git
```

在本地仓库运行:

```git
$ git push -u origin master
```

将本地库的所有内容推送到远程库(origin远程库默认叫法)

### 本地库的内容推送到远程

用`git push`命令,实际上是把当前分支`master`推送到远程。

由于远程库是空的，我们第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

从现在起，只要本地作了提交，就可以通过命令：

```git
git push origin master
```

把本地`master`分支的最新修改推送至`GitHub`，现在，你就拥有了真正的分布式版本库！

### 查看远程库信息

```git
$ git remote -v
origin  git@github.com:guogander/learngit.git (fetch)
origin  git@github.com:guogander/learngit.git (push)
```

### 删除远程库

可以用`git remote rm <name>`命令

```git
$ git remote rm origin
```

此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到`GitHub`，在后台页面找到删除按钮再删除。

### 克隆远程库

`GitHub`上先创建一个仓库,然后克隆到本地:

```git
$ git clone git@github.com:guogander/gitskills.git
```

## 分支管理

### 创建分支

创建`dev`分支:

```git
$ git checkout -b dev
Switched to a new branch 'dev'
```

`-b`表示创建并切换,相当于:

```git
$ git branch dev
$ git checkout dev  # 切换分支
Switched to branch 'dev'
```



### 切换分支

```git
$ git checkout dev  # 切换分支
Switched to branch 'dev'

#切换远程分支
git checkout -b dev

# 切换到远程分支
gitcheckout -b <本地分支> origin/<远程分支>
```

### 查看分支

`git branch`命令查看本地分支,列出所有分支,并在当前分支前面添加`*`号

```git
$ git branch
* dev
  master

# 查看远程分支
git branch -a
```

### 合并dev分支到master上

先切换到`master`分支再合并:

```git
$ git merge dev
Updating 7b88926..37f5b56
Fast-forward
 readme.txt | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

### 删除dev分支

```git
$ git branch -d dev
Deleted branch dev (was 37f5b56).
```

### 新的切换分支switch

创建并切换到新的`dev`分支，可以使用：

```git
$ git switch -c dev
```

直接切换到已有的`master`分支，可以使用：

```git
$ git switch master
```

### 冲突解决

如果分支有冲突,先解决冲突然后再提交,合并完成

### 查看分支合并图

```git
$ git log --graph --pretty=oneline
```

![image-20210531141016698](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20210531141016698.png)