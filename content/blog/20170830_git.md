+++
title= "git版本控制"
date= 2017-08-30T16:19:13+08:00
categories = ["Development", "git"]
trags = ["Development", "git"]
type = "post"
+++

# Git简介

Git是一款免费、开源的分布式版本控制系统。

Linus在1991年创建了Linux内核开源项目，有着来自全世界的贡献者参与该项目，使其不断壮大发展。在早期(1991-2002)，绝大多数的Linux内核维护工作是由Linus本人通过手工方式进行合并与归档。直到2002年，随着代码库的逐渐壮大，Linus最终选择了启用BitMover的分布式版本控制系统BitKeeper，BitMover授权Linux社区免费使用。

到了2005年，由于社区内存在试图破解BitKeeper的协议被发现，所以BitMover收回了免费授权。于是，Linus Torvalds花了两周时间用C写了一个分布式版本控制系统--Git，一个月内Linux社区的源码就由Git管理了，Git也迅速成为最流行的分布式版本控制系统！

## Git的提交流程

Git和其他的版本控制系统如SVN的一个不同之处就是除了工作区、版本库之外还有暂存区的概念。

**工作区(Working Directory)**

工作区即本地项目目录。

**版本库(Repository)**

工作区有一个隐藏目录``.git``，这个不算工作区，而是Git的版本库。

Git的版本库中有很多东西，包括称为stage(或者index)的暂存区，还有Git为我们自动创建的第一个分支``master``，以及指向``master``的一个指针``HEAD``。
<p align="center">
![git_stage_01](http://r.photo.store.qq.com/psb?/V13rKAGe2Rexzc/ThmErLemr0OXgQatXKJutobsoEbMF1OfBTg*JpA*xTY!/r/dD4BAAAAAAAA)
</p>

在我们把文件往Git仓库里添加的时候，是分两步执行的：

* 用``git add``把文件添加进去，实际上就是把文件修改添加到暂存区。
* 用``git commit``提交更改，实际上就是把暂存区的所有内容提交到当前分支。

值得一提的是，Git追踪并管理的是**修改**而非文件，所以提交到仓库的是**暂存区内的修改**，而不是文件本身，而每个版本都是一份**文件快照**。

当我们执行如下过程：

* 第一次修改
* git add
* 第二次修改
* git commit
* git status

会发现，``git status``的消息显示第二次的修改并没有被提交到仓库中。是由于``git add``只将第一次修改存入了暂存区，``git commit``只负责把暂存区的修改提交到仓库，所以第二次的修改没有在暂存区内也没有被提交。

再执行一次``git add``与``git commit``或在第一次``git commit``之前执行``git add``将第二次修改加入到暂存区，都可以达到将第二次修改提交的效果。

## Git的分支管理

每次的版本提交，Git都会把它们串成一条条时间线，这些时间线就是一个个分支。其中有一条主分支``master``，在我们创建仓库时即存在。``HEAD``指针指向当前版本,在下图中``HEAD``指向``master``，即当前版本为``master``所在的节点。
<p align="center">![git_branch_01](http://a3.qpic.cn/psb?/V13rKAGe2Rexzc/PwqUDv1ajji.u*FqOnqlZJeFFwzxxiDCRJTe7*fEBQ4!/b/dPIAAAAAAAAA&bo=LQGXAAAAAAADAJ4!&rf=viewer_4)
</p>

Git中对分支的操作可以看作是对链表的操作。当我们创建新分支``dev``时，Git只需新建一个指针叫``dev``，指向上次提交，再把``HEAD``指向``dev``，就表示当前分支在``dev``上。
<p align="center">![git_branch_02](http://r.photo.store.qq.com/psb?/V13rKAGe2Rexzc/kQBCK4qIyNJ7diphEIcttOE5QTz7aTYEZ7BbicvLnfU!/r/dPMAAAAAAAAA)
</p>

分支的合并可以看作一次提交。假如我们在``dev``上的工作完成了，就可以把``dev``合并到``master``上，相对于``master``来说合并的过程类似一次提交，同样也需要解决冲突，再将``master``指向``dev``当前提交就完成了合并。
<p align="center">![git_branch_03](http://r.photo.store.qq.com/psb?/V13rKAGe2Rexzc/4L7m3QD0utXMT0XkPzDsko3v.MLDuv21HnH*sXZfaK8!/r/dPMAAAAAAAAA)
</p>

合并完之后可以选择将``dev``分支删除，即将``dev``指针删掉。Git鼓励大量使用分支进行开发，在我们修正bug时可以新建bug分支，写实验性特性时可以新建feature分支，从而实现在不影响稳定版本的基础上进行开发。

## 远端仓库

Git作为分布式版本控制系统，同一个Git仓库可以通过克隆分布到不同的机器上。在实际的生产中，通常会找一台电脑或者像github(在线Git仓库托管服务平台)充当Git仓库服务器。

本地仓库与github仓库的通信通常通过SSH或http传输。配置SSH时需在**shell**或**git bash**中使用``$ ssh-keygen -t rsa -C "email@example.com"``，将在用户主目录的``.ssh``中生成密钥对(``id_rsa``和``id_rsa.pub``)，然后github中打开**SSH Keys**页面将``id_rsa.pub``的内容加入到其中。

# Git操作

### 配置Git

    # 配置用户名与邮箱
    $ git config --global user.name "My Name"
    $ git config --global user.email email@example.com

### 创建仓库

仓库即为版本库(Resposity)，可以理解为一个目录，该目录下的所有文件都可以被Git管理，每个文件的修改、删除都可以被追踪与记录。

创建一个Git仓库只需以下几步：

    # 新建或移动到需要管理的项目根目录
    $ mkdir folder
    $ cd folder
    
    # 初始化仓库
    $ git init

如此便将仓库初始化完成，并在folder下生产一个.git的隐藏文件夹，这个文件夹是Git用来存储仓库信息的。

### 检查状态

    # 该命令可以返回仓库当前状态并提示可用操作命令
    $ git status

### 提交修改

提交修改需要将修改先添加到暂存区:

    # 添加单个文件
    $ git add readme.txt
    # 添加文件夹
    $ git add foldername
    # 添加当前目录下所有内容
    $ git add .

再从暂存区提交到仓库：

    # 将暂存区的修改提交到仓库
    $ git commit -m "commit message"
    
    # 此外还有
    # 先将所有已track的修改add到暂存区，然后提交(对于没有track的仍需要git add)
    $ git commit -a
    # 增补提交，会使用与当前提交节点相同的父节点进行一次新的提交(旧提交会被覆盖)
    $ git commit --amend

### 关联远端仓库

为了能将修改上传到远端仓库，我们需要先进行关联。如：

    $ git remote add origin https://github.com/YaoShuHan/YaoShuHan.github.io.git

其中``origin``可以看作是指向的远端仓库的别名，代表的就是该远端库，是Git默认的叫法，当然也可以改成其它的，选用不同的名字代表多个远端仓库。

    # 列出远端主机别名(如origin)
    $ git remote
    # 列出每一个别名对应的实际url
    $ git remote -v
    # 删除一个存在的别名
    $ git remote rm <alias>
    # 重命名别名
    $ git remote rename <old_alias> <new_alias>
    # 为别名设置拉取(fetch)地址
    # 可以通过加上--push参数，为同一别名设置不同的存取地址
    $ git remote set-url [--push] <alias> <url>

### 克隆仓库

    #在当前文件夹下创建本地仓库，并自动设立远端分支
    $ git clone https://github.com/YaoShuHan/YaoShuHan.github.io.git

### 取回更新并合并

    # 基本格式
    $ git pull <远程主机名> <远程分支名>:<本地分支名>

    # 相当于以下两步
    $ git fetch origin master
    $ git merge origin/master

    # 取回到本地但不合并
    $ git fetch <alias> [branch]

### 推送到远端

    # git push 命令用于将本地分支的更新推送到远端主机，其格式为
    $ git push <远程主机名> <本地分支名>:<远程分支名>

    # 将当前分支推送到origin主机的对应分支，如果当前分支只有一个追踪分支，那么主机名可以省略
    $ git push origin
    $ git push
    # 如果当前分支与多个主机存在追踪关系，则可以使用 -u 选项制定一个默认主机，这样后面可以不加任何参数使用 git push
    # 为master配置默认主机
    $ git push -u origin master
    # 如果省略本地分支名，则表示删除指定的远程分支
    $ git push origin :master
    # 推送所有分支到origin主机
    $ git push --all origin
    # 强制推送至远端，将产生“非直进式”合并(non-fast-forward merge)
    $ git push --force origin
    # git push 默认不推送标签(tag)，除非使用 --tags 选项
    $ git push origin --tags

### 分支管理

    # 创建新分支
    $ git branch <new branch_name>

    # 切换分支
    $ git checkout <branch_name>

    # 创建并切换分支
    $ git checkout -b <branch_name>

    # 删除分支
    # 删除当前分支需要先切换到其它分支才能删除此分支
    $ git branch -D <branch_name>

    # 查看分支列表
    $ git branch
    # 查看远端分支列表
    $ git branch -r

    # 重命名分支
    $ git branch -m <old_name> <new_name>

    # 合并分支
    # 将dev分支合并到当前分支
    $ git merge dev

### 查看提交

    # 每次提交都会有一个由SHA1计算出的十六进制大数作为commit id
    # 查看由近至远的提交日志
    $ git log
    # 简化日志输出信息
    $ git log --pretty=oneline
    # 显示整个提交历史记录，但跳过合并
    $ git log --no-merges
    # 显示最近两周的更改文件
    # --since --after --until --before
    $ gitk --since="2 weeks ago"

    # commit id很长，但我们查看某次提交的信息只需使用其中的前一小段就足够了，Git会自动进行检索匹配。
    # 查看某一次提交更新内容
    $ git show <commit id>

    # 比较当前文件和暂存区文件差异
    $ git diff <file>
    # 比较两次提交之间的差异
    $ git diff <commit_id> <commit_id>
    # 比较两个分支之间的差异
    $ git diff <branch> <branch>
    
    # 查看尚未暂存的文件修改
    $ git diff
    # 查看已经git add，但没有git commit的修改
    $ git diff --cached
    # 查看工作目录与上次提交之间的所有差异，这条命令显示的内容都会在执行"git commit -a"命令时被提交
    $ git diff HEAD

### 回滚提交

    # 撤销文件修改
    # 从本地仓库恢复已删除的文件
    $ git checkout <file>
    # 撤销添加到暂存区的文件，并还原文件状态
    $ git checkout head -- <file>

    # 仅重置HEAD指针到指定提交，而不会破坏任何东西
    $ git reset --soft
    # 将所有文件(包括本地文件与HEAD)重置到指定提交，将丢弃本地文件修改
    $ git reset --hard
    # 重置HEAD与暂存区，但不破坏本地文件。为reset默认参数
    $ git reset --mixed

### 撤销提交

    # 撤销提交
    $ git revert <commit_id>

### 清理工作区

    # 从工作目录中移除没有track的文件
    # -d:同时移除目录，-f:强制执行
    $ git clean

### 配置.gitignore

``.gitignore``顾名思义是Git用来配置忽略文件的。在项目中有些文件或文件夹是我们不想提交的，为了以防万一不小心提交，我们可以将他们添加到``.gitignore``文件中。过程如下：

    1. 在项目根目录创".gitignore"文件
    2. 在文件中列出不需要提交的文件及文件夹名，每行一个
    3. ".gitignore"文件需要被提交，就像普通文件一样

    # 因为.gitignore文件是点开头，没有文件名所以没法直接在windows目录下创建，必须通过Git Bash用命令行方式创建
    $ vim .gitignore
