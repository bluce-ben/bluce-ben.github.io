---
title: Git 详解之常用命令汇总（四）
date: 2018-03-14 17:48:01
categories:
- 版本控制
- Git
tags:
- Git
---
#### Git提交三步走：
（1）git add xx
git add xx命令可以将xx文件添加到暂存区，如果有很多改动可以通过 git add -A .来一次添加所有改变的文件。
注意 -A 选项后面还有一个句点。 git add -A表示添加所有内容， git add . 表示添加新文件和编辑过的文件不包括删除的文件; git add -u 表示添加编辑或者删除的文件，不包括新添加的文件
（2）git commit -m "注释"
（3）git push origin 分支名称，一般使用：git push origin master

正常来说，这三步基本满足需求了。 
<!--more-->

#### 下面进行命令的详细介绍
`git checkout`  查看当前分支

`git branch name`  新建分支name
`git checkout name`  切换当前分支为name
<font color="red">合并语句：</font>`git checkout -b name`  新建分支并切换到分支name

`git add filename`  增加文件到分支
`git commit -m "remark"`  提交修改内容到分支并添加注释
<font color="red">合并语句：</font>`git commit -m "remark" filename`

`git diff filename`  查看文件修改了什么内容

`git status`  查看当前工作区状态

`git checkout master`
`git merge name`  合并分支到master分支

`git branch -d name`  删除name分支

`git log --graph --pretty=oneline --abbrev-commit`  用git log查看分支历史


#### 特殊需求详解：
##### 1、合并文件：
`git merge name` 这种合并 Git会优先选择 Fast forward模式，这种模式下，删除分支后，会丢掉分支信息
如果要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样从分支历史上就可以看出来分支信息。
`git merge --no-ff -m "merge with no-ff" dev`  合并分支dev并禁用Fast forward模式 


##### 2、冲突解决：
① 手动修改冲突文件并再次提交 
打开产生冲突的文件后，Git用<<<<<<<，=======，\>\>\>\>\>\>\>标记出不同分支的内容，其中`<<<HEAD`是指主分支修改的内容，`>>>>>fenzhi1` 是指fenzhi1上修改的内容，我们可以修改后保存即解决冲突。


##### 3、版本回退：
（1）使用`HEAD^`回退版本，`HEAD^`回退上个版本，`HEAD^^`回退上上个版本，以此类推。如果回退前100个版本的话，使用`HEAD~100`即可。
`git reset --hard HEAD^`
`git reset --hard HEAD~100`

（2）使用commit_id回退版本
`git log` 查看需要回退的 commit_id
`git reflog` 查看未来版本的 commit_id
`git reset --hard commit_id`
（注：这个回退可以回到以前版本，也可回到未来版本，只要知道commit_id即可）


##### 4、撤销修改：
在工作区：（就是开发环境本地）
`git checkout -- file`  可以丢弃工作区的修改。**（注：如果没有 \-\- 的话，那么命令就变成创建分支了。）**

在暂存区：（已经 git add了）
`git reset HEAD file`  可以把暂存区的修改撤销掉，重新放回到工作区

在本地版本中：（已经提交到分支）
只能版本回退了


##### 5、删除文件：
工作区删除：
`rm file`
工作区删除后文件恢复：
`git checkout -- file`
版本库删除：
`git rm file`
`git commit -m 'remark'`
（注：删除后必须得提交更新版本库）


##### 6、BUG分支
`git stash`  把当前工作现场“储藏”起来，等以后恢复现场后继续工作
`git stash list`  查看工作现场存储到什么地方
`git stash apply`  恢复，但stash内容并不删除
`git stash pop`  恢复，但同时把stash内容也删除


##### 7、多人协作
`git remote`  查看远程库的信息 -v 更详细信息
推送分支： 就是把该分支上的所有本地提交推送到远程库。
`git push origin master`
抓取分支：把线上最新的提交抓下来
`git pull`


#### 工作模式通常是这样：
1、首先，可以试图用git push origin branch-name推送自己的修改；
2、如果推送失败，则因为远程分支比你的本地更新，需要先用git pull试图合并；
3、如果合并有冲突，则解决冲突，并在本地提交；
4、没有冲突或者解决掉冲突后，再用git push origin branch-name推送就能成功！
如果`git pull`提示“no tracking information”，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream branch-name origin/branch-name`。
这就是多人协作的工作模式，一旦熟悉了，就非常简单。



##### Git工作命令汇总（外派百度）：
1) iCode 上新建分支，并拉取iCode分支到本地
   `git checkout -b branch_name origin/branch_name`
（注：iCode新建分支后，一定要git pull 更新本地分支，才能把新建分支更新到本地）

2）本地开发完成，合并到分支
```
   git add filename
   git commit -m "remark"
   git push origin HEAD:refs/for/branch_name
```

3）特殊操作：
① 撤销修改：
在工作区：
`git checkout -- file  可以丢弃工作区的修改`
在暂存区：（已经 git add了）
`git reset HEAD file  可以把暂存区的修改撤销掉，重新放回到工作区`
在本地版本中：
只能版本回退了
② 版本回退：
`git log 查看需要回退的 commit_id`
`git reflog 查看未来版本的 commit_id`
`git reset --hard commit_id`
*（注：这个回退可以回到以前版本，也可回到未来版本，只要知道commit_id即可）*
③ 冲突解决：
① 手动修改冲突文件并再次提交 