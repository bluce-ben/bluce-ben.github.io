---
title: PhpStorm中添加自定义函数注释
date: 2019-01-08 16:29:19
categories:
- 编辑器
tags:
- 编辑器
---
点击查看更多
<!--more-->
#### 一、创建模板
1、首先我们要添加自己的一套模板，点击 file->settings->editor->live Templates
![](/uploads/2019/01/phpstorm_editor_01.png)

2、点击右侧加号，选择第二个选项“template group”，创建一个分组。（注：不需要分组，可跳过此步）
![](/uploads/2019/01/phpstorm_editor_02.png)

3、选中刚才添加的分组，点击右侧加号，选择第一个选项“live template”添加一个具体模板。
![](/uploads/2019/01/phpstorm_editor_03.png)

4、按照图中步骤：其中第四步不一定要改，可以根据个人习惯，我习惯用enter键。
![](/uploads/2019/01/phpstorm_editor_04.png)

5、这是第五步点击内容，分别对DATE, TIME标签做设置。
![](/uploads/2019/01/phpstorm_editor_05.png)

设置完后点击”Apply”。


#### 二、使用模板
上面是设置要用的标签模板，下面是如何来使用它： 
点击“file and code templates->includes->PHP Function Doc Comment”，在里面输入自己想要的注释内容，注意Times对应添加的模板名称。
![](/uploads/2019/01/phpstorm_editor_06.png)

之后点击”apply”。


#### 三、测试使用
在项目中使用 
输入”/**”后点击enter键，会出现刚才添加的注释，再次点击enter键，会自动出现系统时间。 
注意：如果刚才第四步没有设置”enter”，要点击tab键，默认是tab键。 
![](/uploads/2019/01/phpstorm_editor_07.png)
