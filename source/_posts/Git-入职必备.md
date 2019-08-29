---
title: Git 入职必备
date: 2019-08-29 14:13:38
categories:
- 版本控制
- Git
tags:
- Git
---
#### 一、配置操作
##### 1、配置文件
Git自带一个 git config 的工具来帮助设置控制 Git外观和行为的配置变量。这些变量存储在三个不同的位置：
1. **/etc/gitconfig 文件**：包含系统上每一个用户及他们仓库的通用配置。如果使用带有 \-\-system 选项的 git config时，它会从此文件读写配置变量。
2. **~/.gitconfig 或 ~/.config/git/config 文件**：只针对当前用户。可以传递 \-\-global 选项让 Git读写此文件。
3. 当前使用仓库的Git目录中的config文件（就是 **.git/config**)：针对该仓库。

每一个级别覆盖上一个级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。
<!--more-->

##### 2、用户信息
当安装完 Git应该做的第一件事就是设置用户名称与邮件地址。这样做很重要，因为每一个Git 的提交都会使用这些信息，并且它会写入到每一次提交中，不可更改。
```
git config --global user.name "zhengbenwu"
git config --global user.email "zhengbenwu@soyoung.com"
```
再次强调，如果使用了 \-\-global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情，Git都会使用这些信息。当你想针对特定项目使用不同的用户名称与邮件地址时，可以在哪个项目目录下运行不使用 \-\-global选项的命令来设置。
Git提交示例：
```
commit 2a82ac5a1bfcd8e052e9053e512e14a8880c02ce (HEAD -> dev-sms)
Merge: 8e4c451 3b03b5e
Author: zhengbenwu <zhengbenwu@soyoung.com>
Date:   Thu Aug 29 10:17:11 2019 +0800
```

##### 3、常用配置命令
1. git config <key> 查看某个配置项
2. git config --list 列出所有Git当时能找到的配置
3. git help config  查看config命令的手册
>扩展：可使用 `git help <verb>` 命令来查看对应某个命令的手册信息。


#### 二、Git基本介绍
##### 1、概述
Git是一个版本控制工具，**版本控制是一种记录一个或若干个文件内容变化，以便将来查阅特定版本修订情况的系统。**版本控制可以有如下分类：
* **集中化的版本控制系统**，例如SVN，更久远的CVS。这类系统，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或提交更新。优点是集中管理，可以轻松掌控每个开发者的权限等等。缺点就是中央服务器的单点故障，如果宕机，那么谁都无法提交更新，也无法协同工作。如果数据没有备份，出现磁盘损坏，数据也会丢失等等。
* **分布式版本控制系统**，例如Git。 客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。那么，如果出现服务器故障，事后都可以用任何一个镜像出来的本地仓库恢复。

##### 2、Git开发流程
1. 开通或注册账号，获取项目仓库权限
2. 克隆仓库到本地开发环境
3. 新建分支并进行开发，此处不建议在master上开发
4. 开发完成后，需要更新分支，同步远程分支协同工作的开发修改
5. 将更新提交到分支上，合并到master，推到远程仓库中
（注：此处可直接将分支推到远程仓库中，有Git的开源项目管理工具，通过该工具进行合并master）

##### 3、本地开发需要了解
首先，需要了解什么是工作区、暂存区和版本库的概念。
* 工作区：就是你在电脑里能看到的目录
* 暂存区：英文叫stage，或index。一般存放在".git"目录下的index文件中，所以我们也把暂存区叫做索引(index)。
* 版本库：工作区有一个隐藏目录 .git，这个不算工作区，而是Git的版本库。

我们开发的时候，是在工作区写代码。然后通过"git add"命令，将工作区的修改或新增文件添加到暂存区，此时，暂存区的目录树被更新，同时工作区修改或新增的文件被写入到对象库中的一个新的对象中，而该对象的ID被记录在暂存区的文件索引中。
当执行提交操作（git commit）时，暂存区的目录树写到版本库中，此时，相应的分支会做相应的更新。即对应分支指向的目录树就是提交时暂存区的目录树。
当执行"git reset HEAD"命令时，暂存区的目录树会被重写，被分支指向的目录树所替换，但是工作区不受影响。
当执行"git rm \-\-cached <file>"命令时，会直接从暂存区删除文件，工作区则不做出改变。
当执行"git checkout ." 或 "git checkout \-\- <file>"命令时，会用暂存区全部或指定的文件替换工作区的文件。这个操作很危险，会清除工作区中未添加到暂存区的改动。
当执行"git checkout HEAD ." 或 "git checkout HEAD <file>"命令时，会用HEAD指向的分支中的全部或者部分文件替换暂存区和以及工作区中的文件。这个命令也非常危险，因为不但会清楚工作区中未提交的改动，也会清楚暂存区中未提交的改动。
>暂存区详情：[Git暂存区原理](https://blog.csdn.net/s646575997/article/details/52143586)

##### 4、与远程仓库的操作
上面说的版本库，仅仅只是对本地的仓库来说的。并没有涉及到远程仓库，因此，还需要将本地的仓库修改推到远程仓库中。一般的操作流程如下：
1. `git pull` 更新远程仓库到本地
2. `git push origin dev-sms` 将本地修改推动到远程仓库中

注意，一定要在推送前，先更新下远程仓库，因为在协同开发中有可能会出现冲突，遇到冲突的话，需要先解决冲突之后，才能推送。 


#### 三、Git常用命令
##### 1、基本操作
 可参考我以前的文章：[常用命令汇总](https://blog.zhengbenwu.com/2018/03/14/Git%E8%AF%A6%E8%A7%A3%E4%B9%8B%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4%E6%B1%87%E6%80%BB%EF%BC%88%E5%9B%9B%EF%BC%89/)
##### 2、实际工作中记录
1. git stash用法
 保存工作区到暂存区： git stash save "save message"
 查看暂存区：git stash list
 查看暂存区中某一个做了哪些改动：git stash show [stash@{$num}]
 应用某个存储单不会删除：git stash apply [stash@{$num}]
 应用某个存储并删除：git stash pop [stash@{$num}]
 删除某个存储：git stash drop stash@{$num}
 删除所有存储：git stash clear
2. git commit用法
 1）修改commit注释，前提是commit后未push
 `git commit --amend`进入vi编辑界面，修改注释后保存退出即可。
 2）还有一种用法是，当需要将本地的修改推到未push的commit中时使用，先add添加进去，在commit。
 （注：还有一种是将两个commit糅合在一起，通过 rebase \-i）
3. git 强制提交
```
 git push -f origin master
 -f/--force
```
4. 查看日志
 1）git log 查看以前日志
 2）git reflog 查看未来日志
 配合 git reset HEAD@{index}来回滚代码 或 git reset \-\-hard commitId
5. 比较版本内容
 git diff <file> 比较当前文件和暂存区文件差异
 git diff <id1><id2> 比较两次提交之间的差异
 git diff <branch1> <branch2> 在两个分支之间比较
 git diff \-\-staged  比较暂存区和版本库差异
 git diff \-\-cached  比较暂存区和版本库差异
 git diff \-\-stat 仅仅比较统计信息
6. 合并线上分支报文件冲突，询问是否覆盖本地。
 合并分支：`git merge origin/dev-new-swoole`
 解决单个文件冲突:
 `git checkout --patch dev-new-swoole SDK/Services/ShopService/ShopService_Shop.php`


#### 四、Git常见问题记录
##### 1、由于本地版本与Git上版本相差太多导致的
```
To git.culiu.org:dwxk/dwxk-api.git
 ! [rejected]            dev-turnplate -> dev-turnplate (non-fast-forward)
error: failed to push some refs to 'git@git.culiu.org:dwxk/dwxk-api.git'
hint: Updates were rejected because a pushed branch tip is behind its remote
hint: counterpart. Check out this branch and integrate the remote changes
hint: (e.g. 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
>注意点：
不要把自己修改的代码给合并没了，当发现自己代码没了，只能再从日志中拉取了（或log,或relog）。

>解决方案：
把远程库同步到本地就可以了，然后解决冲突，提交即可。
`git pull --rebase origin dev-turnplate`

>依据提示操作即可。 注意点，有可能版本相差有点多，导致需要每一个版本都要合并，所以，要仔细合并。完事再确认代码是否正确。


**参考网站：**
1、[Git语言快速入门](http://www.hechaku.com/git/Gitkuaisurumen.html)
2、[Git教程](https://www.runoob.com/git/git-tutorial.html)
