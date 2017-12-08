---
title: 学习git
date: 2017-12-08
categories: Linux
tags:
- linux
- git
- github
---

纯属搬砖操作，资料来源《Github入门与实战》，这本书的重要信息也就这些了，需要的时候找一找。

书上提到的一个学习网站<https://learngitbranching.js.org/>非常棒，线上学习。

<!-- more -->

# Git基本操作

## git init——初始化仓库

```shell
$ mkdir git-tutorial
$ cd git-tutorial
$ git init
Initialized empty Git repository in /Users/hirocaster/github/github-book
/git-tutorial/.git/
```

如果初始化成功，执行了 git init命令的目录下就会生成 .git 目录。这个 .git 目录里存储着管理当前目录内容所需的仓库数据。 在 Git 中，我们将这个目录的内容称为“附属于该仓库的工作树”。文件的编辑等操作在工作树中进行，然后记录到仓库中，以此管理文件的历史快照。如果想将文件恢复到原先的状态，可以从仓库中调取之前的快照，在工作树中打开。 



## git status——查看仓库状态 

git status命令用于显示 Git 仓库的状态。这是一个十分常用的命令，请务必牢记。

工作树和仓库在被操作的过程中，状态会不断发生变化。在 Git 操作过程中时常用 git status命令查看当前状态，可谓基本中的基本。下面，就让我们来实际查看一下当前状态 ：

```shell
$ git status
# On branch master
#
# Initial commit
#
nothing to commit (create/copy files and use "git add" to track)
```

结果显示了我们当前正处于 master 分支下。关于分支我们会在不久后讲到，现在不必深究。接着还显示了没有可提交的内容。所谓提交（Commit），是指“记录工作树中所有文件的当前状态”。 



## git add——向暂存区中添加文件

要想让文件成为 Git 仓库的管理对象，就需要用 git add命令将其加入暂存区（Stage 或者 Index）中。暂存区是提交之前的一个临时区域。 

```shell
$ git add README.md
$ git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
# (use "git rm --cached <file>..." to unstage)
#
# new file: README.md
#
```

将 README.md 文件加入暂存区后， git status命令的显示结果发生了变化。可以看到， README.md 文件显示在 Changes to be committed 中了。

 

## git commit——保存仓库的历史记录 

git commit命令可以将当前暂存区中的文件实际保存到仓库的历史记录中。通过这些记录，我们就可以在工作树中复原文件。 

```shell
$ git commit -m "First commit"
[master (root-commit) 9f129ba] First commit
1 file changed, 0 insertions(+), 0 deletions(-)
create mode 100644 README.md
```

-m 参数后的 "First commit"称作提交信息，是对这个提交的概述。 



## git log——查看提交日志 

git log命令可以查看以往仓库中提交的日志。包括可以查看什 么人在什么时候进行了提交或合并，以及操作前后有怎样的差别。关于合并我们会在后面解说。 

我们先来看看刚才的 git commit命令是否被记录了。 

```shell
$ git log
commit 9f129bae19b2c82fb4e98cde5890e52a6c546922
Author: hirocaster <hohtsuka@gmail.com>
Date: Sun May 5 16:06:49 2013 +0900
First commit
```

如上图所示，屏幕显示了刚刚的提交操作。 commit 栏旁边显示的“9f129b……”是指向这个提交的哈希值。 Git 的其他命令中，在指向提交时会用到这个哈希值。

Author 栏中显示我们给 Git 设置的用户名和邮箱地址。 Date 栏中显示提交执行的日期和时间。再往下就是该提交的提交信息。 



### 只显示提交信息的第一行

如果只想让程序显示第一行简述信息，可以在 git log命令后加上 --pretty=short。这样一来开发人员就能够更轻松地把握多个提交。 



### 只显示指定目录、文件的日志

只要在 git log命令后加上目录名，便会只显示该目录下的日志。如果加的是文件名，就会只显示与该文件相关的日志。 



### 显示文件的改动

如果想查看提交所带来的改动，可以加上 -p参数，文件的前后差别就会显示在提交信息之后。 

```shell
$ git log -p
```

如上所述， git log命令可以利用多种参数帮助开发者把握以往提交的内容。不必勉强自己一次记下全部参数，每当有想查看的日志就积极去查，慢慢就能得心应手了。



## git diff——查看更改前后的差别 

git diff命令可以查看工作树、暂存区、最新提交之间的差别。单从字面上可能很难理解，各位不妨跟着笔者的解说亲手试一试。 



### 查看工作树和暂存区的差别 

```shell
$ git diff
diff --git a/README.md b/README.md
index e69de29..cb5dc9f 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1 @@
+# Git教程
```

这里解释一下显示的内容。“+”号标出的是新添加的行，被删除的行则用“-”号标出。我们可以看到，这次只添加了一行 。

### 查看工作树和最新提交的差别 

要查看与最新提交的差别，请执行以下命令。

```shell
$ git diff HEAD
diff --git a/README.md b/README.md
index e69de29..cb5dc9f 100644
--- a/README.md
+++ b/README.md
@@ -0,0 +1 @@
+# Git教程
```

 **不妨养成这样一个好习惯：在执行 git commit命令之前先执行git diff HEAD命令，查看本次提交与上次提交之间有什么差别，等确认完毕后再进行提交**。这里的 HEAD 是指向当前分支中最新一次提交的指针。



# 分支操作

通过灵活运用分支，可以让多人同时高效地进行并行开发。在这里，我们将带大家学习与分支相关的 Git 操作。 

## git branch——显示分支一览表 

git branch命令可以将分支名列表显示，同时可以确认当前所在分支。让我们来实际运行 git branch命令。 

```shell
$ git branch
* master
```

可以看到 master 分支左侧标有“*”（星号），表示这是我们当前所在的分支。也就是说，我们正在 master 分支下进行开发。结果中没有显示其他分支名，表示本地仓库中只存在 master 一个分支。 

## git checkout -b——创建、切换分支 

如果想以当前的 master 分支为基础创建新的分支，我们需要用到git checkout -b命令。 

### 切换到 feature-A 分支并进行提交 

执行下面的命令，创建名为 feature-A 的分支。 

```shell
$ git checkout -b feature-A
Switched to a new branch 'feature-A'
```

实际上，连续执行下面两条命令也能收到同样效果。 

```shell
$ git branch feature-A
$ git checkout feature-A
```

创建 feature-A 分支，并将当前分支切换为 feature-A 分支。这时再来查看分支列表，会显示我们处于 feature-A 分支下。 

```shell
$ git branch
* feature-A
master
```

feature-A 分支左侧标有“*”，表示当前分支为 feature-A。在这个状态下像正常开发那样修改代码、执行 git add命令并进行提交的话，代 码 就 会 提 交 至 feature-A 分 支。 像 这 样 不 断 对 一 个 分 支（例 如feature-A）进行提交的操作，我们称为“培育分支”。 



### 切换回上一个分支 

```shell
$ git checkout -
```

像上面这样用“-”（连字符）代替分支名，就可以切换至上一个分支。

 

## 特性分支 

Git 与 Subversion（SVN）等集中型版本管理系统不同，创建分支时不需要连接中央仓库，所以能够相对轻松地创建分支。因此，当今大部分工作流程中都用到了特性（Topic）分支。

特性分支顾名思义，是集中实现单一特性（主题），除此之外不进行任何作业的分支。在日常开发中，往往会创建数个特性分支，同时在此之外再保留一个随时可以发布软件的稳定分支。稳定分支的角色通常由 master 分支担当。

 基于特定主题的作业在特性分支中进行，主题完成后再与 master 分支合并。只要保持这样一个开发流程，就能保证 master 分支可以随时供人查看。这样一来，其他开发者也可以放心大胆地从 master 分支创建新的特性分支 。

## git merge——合并分支 

接下来，我们假设 feature-A 已经实现完毕，想要将它合并到主干分支 master 中。首先切换到 master 分支。 

```shell
$ git checkout master
Switched to branch 'master'
```

然后合并 feature-A 分支。为了在历史记录中明确记录下本次分支合并，我们需要创建合并提交。因此，在合并时加上 --no-ff参数。 

```shell
$ git merge --no-ff feature-A
```

随后编辑器会启动，用于录入合并提交的信息。 



## git log --graph——以图表形式查看分支 

用 git log --graph命令进行查看的话，能很清楚地看到特性分支（feature-A）提交的内容已被合并。除此以外，特性分支的创建以及合并也都清楚明了。 

```shell
$ git log --graph
* commit 83b0b94268675cb715ac6c8a5bc1965938c15f62
|\ Merge: fd0cbf0 8a6c8b9
| | Author: hirocaster <hohtsuka@gmail.com>
| | Date: Sun May 5 16:37:57 2013 +0900
| |
| | Merge branch 'feature-A'
| |
| * commit 8a6c8b97c8962cd44afb69c65f26d6e1a6c088d8
|/ Author: hirocaster <hohtsuka@gmail.com>
| Date: Sun May 5 16:22:02 2013 +0900
|
| Add feature-A
|
* commit fd0cbf0d4a25f747230694d95cac1be72d33441d
| Author: hirocaster <hohtsuka@gmail.com>
| Date: Sun May 5 16:10:15 2013 +0900
|
| Add index
|
* commit 9f129bae19b2c82fb4e98cde5890e52a6c546922
Author: hirocaster <hohtsuka@gmail.com>
Date: Sun May 5 16:06:49 2013 +0900
First commit
```

git log --graph命令可以用图表形式输出提交日志，非常直观，请大家务必记住。 



# 更改提交的操作

## git reset——回溯历史版本 

Git 的另一特征便是可以灵活操作历史版本。借助分散仓库的优势，可以在不影响其他仓库的前提下对历史版本进行操作。 

要让仓库的 HEAD、暂存区、当前工作树回溯到指定状态，需要用到 git rest --hard命令。只要提供目标时间点的哈希值 ，就可以 完全恢复至该时间点的状态。事不宜迟，让我们执行下面的命令。 

```shell
$ git reset --hard fd0cbf0d4a25f747230694d95cac1be72d33441d (使用时这里需要个人更改哈希值)
HEAD is now at fd0cbf0 Add index
```

**git log命令只能查看以当前状态为终点的历史日志。所以这里要使用 git reflog命令，查看当前仓库的操作日志。在日志中找出回溯历史之前的哈希值，通过 git reset --hard命令恢复到回溯历史前的状态 。**

## 消除冲突 

### 查看冲突部分并将其解决 

用编辑器打开 README.md （如果你发生了冲突，查看相应的冲突文件）文件，就会发现其内容变成了下面这个样子。 （这是书上的例子）

```shell
# Git教程
<<<<<<< HEAD
- feature-A
=======
- fix-B
>>>>>>> fix-B
```

`======= `以上的部分是当前 HEAD 的内容，以下的部分是要合并的 fix-B 分支中的内容。我们在编辑器中将其改成想要的样子。 

```shell
# Git教程
- feature-A
- fix-B
```

如上所示，本次修正让 feature-A 与 fix-B 的内容并存于文件之中。但是在实际的软件开发中，往往需要删除其中之一，所以各位在处理冲突时，务必要仔细分析冲突部分的内容后再行修改。 

### 提交解决后的结果 

冲突解决后，执行 git add命令与 git commit命令。 



## git commit --amend——修改提交信息 



## git rebase -i——压缩历史 

在合并特性分支之前，如果发现已提交的内容中有些许拼写错误等，不妨提交一个修改，然后将这个修改包含到前一个提交之中，压缩成一个历史记录。这是个会经常用到的技巧，让我们来实际操作体会一下。 

首先，新建一个 feature-C 特性分支。 

作为 feature-C 的功能实现，我们在 README.md 文件中添加一行文字，并且故意留下拼写错误，以便之后修正。 

```shell
$ git checkout -b feature-C
Switched to a new branch 'feature-C'
```

```shell
# Git教程
- feature-A
- fix-B
- faeture-C
```

提交这部分内容。这个小小的变更就没必要先执行 git add命令再执行 git commit命令了，我们**用 git commit -am命令来一次完成这两步操作**。 

```shell
$ git commit -am "Add feature-C"
[feature-C 7a34294] Add feature-C
1 file changed, 1 insertion(+)
```

现在来修正刚才预留的拼写错误。 然后进行提交。 

```shell
$ git commit -am "Fix typo"
[feature-C 6fba227] Fix typo
1 file changed, 1 insertion(+), 1 deletion(-)
```

错字漏字等失误称作 typo，所以我们将提交信息记为 "Fix typo"。 实际上，我们不希望在历史记录中看到这类提交，因为健全的历史记录并不需要它们。如果能在最初提交之前就发现并修正这些错误，也就不会出现这类提交了。 

我们来更改历史。将 " Fix typo"修正的内容与之前一次的提交合并，在历史记录中合并为一次完美的提交。为此，我们要用到git rebase命令。 

```shell
$ git rebase -i HEAD~2
```

用上述方式执行 git rebase命令，可以选定当前分支中包含HEAD（最新提交）在内的两个最新历史记录为对象，并在编辑器中打开。 

```shell
pick 7a34294 Add feature-C
pick 6fba227 Fix typo
# Rebase 2e7db6f..6fba227 onto 2e7db6f
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

我们将 6fba227 的 Fix typo 的历史记录压缩到 7a34294 的 Add feature-C里。按照下图所示，将 6fba227 左侧的 pick 部分删除，改写为 fixup。 

```shell
pick 7a34294 Add feature-C
fixup 6fba227 Fix typo
[detached HEAD 51440c5] Add feature-C
1 file changed, 1 insertion(+)
Successfully rebased and updated refs/heads/feature-C.
```

这样一来， Fix typo 就从历史中被抹去，也就相当于 Add feature-C中从来没有出现过拼写错误。这算是一种**良性的历史改写**。 



# 推送至远程仓库

## git remote add——添加远程仓库 

在 GitHub 上创建的仓库路径为“git@github.com:用户名 /git-tutorial.git”。现在我们用 git remote add命令将它设置成本地仓库的远程仓库。

 ```shell
$ git remote add origin git@github.com:github-book/git-tutorial.git
 ```

按照上述格式执行 git remote add命令之后， Git 会自动git@github.com:github-book/git-tutorial.git远程仓库的名称设置为 origin（标识符）。 



## git push——推送至远程仓库 

### 推送至 master 分支 

如果想将当前分支下本地仓库中的内容推送给远程仓库，需要用到git push命令。现在假定我们在 master 分支下进行操作。 

```shell
$ git push -u origin master
Counting objects: 20, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (10/10), done.
Writing objects: 100% (20/20), 1.60 KiB, done.
Total 20 (delta 3), reused 0 (delta 0)
To git@github.com:github-book/git-tutorial.git
* [new branch] master -> master
Branch master set up to track remote branch master from origin.
```

像这样执行 git push命令，当前分支的内容就会被推送给远程仓库origin 的 master 分支。 -u参数可以在推送的同时，将 origin 仓库的 master 分支设置为本地仓库当前分支的 upstream（上游）。添加了这个参数，将来运行 git pull命令从远程仓库获取内容时，本地仓库的这个分支就可以直接从 origin 的 master 分支获取内容，省去了另外添加参数的麻烦。执行该操作后，当前本地仓库 master 分支的内容将会被推送到GitHub 的远程仓库中。在 GitHub 上也可以确认远程 master 分支的内容 和本地 master 分支相同。 



### 推送至 master 以外的分支 

除了 master 分支之外，远程仓库也可以创建其他分支。举个例子，我们在本地仓库中创建 feature-D 分支，并将它以同名形式 push 至远程仓库。 

```shell
$ git checkout -b feature-D
Switched to a new branch 'feature-D'
```

我们在本地仓库中创建了 feature-D 分支，现在将它 push 给远程仓库并保持分支名称不变。 

```shell
$ git push -u origin feature-D
Total 0 (delta 0), reused 0 (delta 0)
To git@github.com:github-book/git-tutorial.git
* [new branch] feature-D -> feature-D
Branch feature-D set up to track remote branch feature-D from origin.
```



# 从远程仓库获取 

## git clone——获取远程仓库 



### 获取远程仓库 

首先我们换到其他目录下，将 GitHub 上的仓库 clone 到本地。注意 不要与之前操作的仓库在同一目录下。 

```shell
$ git clone git@github.com:github-book/git-tutorial.git
Cloning into 'git-tutorial'...
remote: Counting objects: 20, done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 20 (delta 3), reused 20 (delta 3)
Receiving objects: 100% (20/20), done.
Resolving deltas: 100% (3/3), done.
$ cd git-tutorial
```

执行 git clone命令后我们会默认处于 master 分支下，同时系统会自动将 origin 设置成该远程仓库的标识符。也就是说，当前本地仓库的 master 分支与 GitHub 端远程仓库（origin）的 master 分支在内容上是完全相同的。

 ```shell
$ git branch -a
* master
remotes/origin/HEAD -> origin/master
remotes/origin/feature-D
remotes/origin/master
 ```

我们用 git branch -a命令查看当前分支的相关信息。添加 -a参数可以同时显示本地仓库和远程仓库的分支信息。
结果中显示了 remotes/origin/feature-D，证明我们的远程仓库中已经有了 feature-D 分支 。

### 获取远程的 feature-D 分支 

我们试着将 feature-D 分支获取至本地仓库。 

```shell
$ git checkout -b feature-D origin/feature-D
Branch feature-D set up to track remote branch feature-D from origin.
Switched to a new branch 'feature-D'
```

-b 参数的后面是本地仓库中新建分支的名称。为了便于理解，我们仍将其命名为 feature-D，让它与远程仓库的对应分支保持同名。新建分支名称后面是获取来源的分支名称。例子中指定了 origin/feature-D，就是说以名为 origin 的仓库（这里指 GitHub 端的仓库）的 feature-D 分支为来源，在本地仓库中创建 feature-D 分支。 



## git pull——获取最新的远程仓库分支 

远程仓库的 feature-D 分支中已经有了我们刚刚推送的提交。这时我们就可以使用 git pull 命令，将本地的 feature-D 分支更新到最新状态。当前分支为 feature-D 分支。 

```shell
$ git pull origin feature-D
remote: Counting objects: 5, done.
remote: Compressing objects: 100% (1/1), done.
remote: Total 3 (delta 1), reused 3 (delta 1)
Unpacking objects: 100% (3/3), done.
From github.com:github-book/git-tutorial
* branch feature-D -> FETCH_HEAD
First, rewinding head to replay your work on top of it...
Fast-forwarded feature-D to ed9721e686f8c588e55ec6b8071b669f411486b8.
```



-----

# 如何用Github的gh-pages分支展示自己的项目

```
git subtree push --prefix=dist origin gh-pages
```

意思就是把指定的dist文件提交到gh-pages分支上