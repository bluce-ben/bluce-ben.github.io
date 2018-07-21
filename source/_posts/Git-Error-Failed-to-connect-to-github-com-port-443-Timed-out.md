---
title: 'Git Error: Failed to connect to github.com port 443: Timed out'
date: 2017-12-08 14:39:58
categories:
- [版本控制, Git, Git问题]
- [Hexo, Hexo问题]
tags:
- Git
- Hexo
- Git问题
- Hexo问题
---
### 错误信息：
> fatal: unable to access 'https://github.com/bluce-ben/bluce-ben.github.io.git/': Failed to connect to github.com port 443: Timed out
> FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
> Error: fatal: unable to access 'https://github.com/bluce-ben/bluce-ben.github.io.git/': Failed to connect to github.com port 443: Timed out
<!--more-->
　依据错误可清晰知道是因为无法连接Git导致的超时错误。而我配置的Hexo+GitHub Pages中使用的Git仓库就是 HTTPS 协议的，所以是使用443端口，也由此我 hexo deploy 部署Git失败。

### 解决方案：
*在百度搜索到一个解决方案，现整理如下。*
**1、测试git是否成功连接GitHub，使用：**
　`ssh -T git@github.com`
> 连接失败：ssh: connect to host github.com port 22: Connection timed out
> 连接成功：Hi bluce-ben! You've successfully authenticated, but GitHub does not provide shell access.

**2、修改Git配置文件，首先找到git的安装目录，找到/etc/ssh/ssh_config文件**
　打开该文件并将下列配置添加末尾：
```
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443
```
　保存修改。
*（注：如果保存失败提示没有权限，可用 Notepad++打开会提示使用管理员权限，点击确定即可。）*

**3、再次测试是否成功连接GitHub。成功后再次 hexo deploy 部署GitHub即可。**