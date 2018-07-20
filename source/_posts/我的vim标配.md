---
title: 我的vim标配
date: 2018-01-24 17:33:52
categories:
- Linux
tags:
- vim
---
在Linux下开发程序，一般都会离不开vi编辑器，而vim是vi的进阶版IDE。目前vim已经取代了vi编辑器，正常使用vi的时候就默认启用的是vim编辑器，不信你可自己查看，在bash命令行下输入"alias"即可看到是否vim已经取代了vi编辑器。但vim的用法与vi编辑器是一模一样，所以，不用担心使用过vi，而不会使用vim，只不过vim有更高级的用法罢了。而vim的配置就是其中一个非常好的用法。
一千个vimer就有一千个配置，vim的配置之多让人眩目，但常用的基本配置也就那些——我的vim标配。
首先，要知道在什么地方配置vim，一般只修改用户自己的vim配置文件即可(~/.vimrc)。下面给出我的配置清单：
<!--more-->
```
set tabstop=4	   "设定tab缩进空格数
set expandtab      "设定tab自动转为空格
set number         "set nu 显示行号
set hlsearch       "搜索时高亮显示被找到的文本
set syntax=on      "自动语法高亮
set smartindent    "开启新行时使用智能自动缩进
set showmatch	   "自动匹配括号
```
（注：vim配置注释，使用双引号 " 注释。）

##### 常用配置：
"显示行号
set nu/nonu

"语法高亮
syntax on/off

"tab缩进
set tabstop=4
set shiftwidth=4
set expandtab
set smarttab

"智能缩进
set smartindent
（表示在换行的时候光标进行智能缩进，触发智能缩进的场景有：以"{"结尾，以C语言关键字开头，可以通过:h cinwords进行查看，包括：if, else, while, do, for, switch，以"}"开头，此情况只有在执行"O"命令时候有效，但是这个配置有一点问题，就是当换行之后本来已经处于缩进状态了，如果输入"#"，那么缩进就会取消，"#"就会被插入到第一列，可是我们在写Python代码的时候，往往不希望这样做，所以我们可以增加配置":inoremap # X^H"，其中 ^H 是先按CTRL-V再按CTRL-H输入的。）

"高亮被搜索的字符
set hlsearch

"背景色
set background=dark

"显示匹配（可自动匹配括号）
set showmatch

<font color="red">待补录...</font>