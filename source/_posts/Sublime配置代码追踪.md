---
title: Sublime配置代码追踪
date: 2018-07-18 19:44:28
categories: 编辑器
tags: 编辑器
---
在阅读别人代码的时候，非常喜欢IDE中的追踪代码功能。也想在sublime text中也使用。从网上查找发现可以使用ctags插件。

下面介绍下如何在sublime text中安装使用ctags：
#### 1、安装package control ####
点击`Preferences > Browse Packages`菜单，进入打开的目录的上层目录，然后再进入Installed Packages/目录，下载Package Control.sublime-package并复制到Installed Packages/目录
pControl.sublime-package下载地址：[点击下载](/uploads/2018/07/Package Control.sublime-package)
<!--more-->

输入Ctrl+Shift+P（菜单 - Tools - Command Paletter），输入Install Package并回车，输入或选择你需要的插件回车就安装了。
如果快捷键不好使，重启Sublime Text。

#### 2、安装ctags ####
##### （1）安装sublime text插件 #####
输入ctags安装，会发现编辑器左下角正在下载安装。

##### （2）安装可执行程序 #####
下载ctags可执行程序，地址为`http://prdownloads.sourceforge.net/ctags/ctags58.zip`，解压到一个目录，注意要是纯ASCII字符的目录不要带空格或中文命名的目录。

##### （3）配置 #####
配置可执行文件路径：打开菜单在Preferences菜单中打开`Package settings->ctags->settings-user和settings-default `把default中的配置全部复制到user中，然后改一下command配置项，为ctags的可执行文件路径,即ctags.exe路径
```
// Path to ctags executable.
//
// Alter this value if your ctags command is not in the PATH, or if using
// a different version of ctags to that in the path (i.e. for OSX).
//
// NOTE: You *should not* place entire commands here. These commands are
// built automatically using the values below. For example, this is OK:
//
//     "command": "/usr/bin/ctags"
//
// This, on the other hand, won't work!
//
//     "command": "ctags -R -f .tags --exclude=some/path"
//
"command": "C:/Windows/System32/ctags58/ctags.exe",
```
配置快捷键：配置在sublime中使用Ctrl+左键单击函数跳转、Ctrl+右键单击跳回函数调用位置
复制以下代码到 `Preferences->Package Settings->Ctags->Mouse Bindings-User `
```
[
    {
        "button": "button1",
        "count": 1,
        "press_command": "drag_select",
        "modifiers": ["ctrl"],
        "command": "navigate_to_definition"
    },
    {
        "button": "button2",
        "count": 1,
        "modifiers": ["ctrl"],
        "command": "jump_prev"
    }
]
```

##### （4）使用 #####
在使用函数调转功能前，需要先生成.tags文件，只需在<font color="red">项目文件管理器的项目文件上右键点击Ctags:Rebuild Tags即可</font>**（注意，在改动文件之后也许重新生成.tags）**

##### （5）所有工作都准备充分之后，就可以在函数名上 Ctrl+左键 点击指定函数跳转了，Ctrl+右键返回上一个跳转函数； #####
