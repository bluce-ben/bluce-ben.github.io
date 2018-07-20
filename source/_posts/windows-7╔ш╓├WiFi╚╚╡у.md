---
title: windows 7设置WiFi热点
date: 2018-03-27 16:04:35
categories:
- Windows
tags:
- Windows
---
点击可查看...
<!--more-->
#### 1、查看是否支持并开启
首先打开自己的网络连接，看一下有没有上述网卡图标，若有则进行下一步，若没有则打开设备管理器，找到网卡选项，右击选择更新网卡驱动，若此时出现说明你的电脑支持此功能，若没有出现，则说明电脑不支持。
![](/uploads/2018/03/network_wifi_01.jpg)

#### 2、设置WiFi热点
点击 开始， 选择 附件，找到命令提示符，右击选择以管理员方式运行
输入：`netsh wlan set hostednetwork mode=allow ssid="无线热点的名字" key=“w ifi的密码”`，然后回车，如果出现如下的信息，则表示设置成功：
![](/uploads/2018/03/network_wifi_02.jpg)
![](/uploads/2018/03/network_wifi_03.jpg)

#### 3、开启WiFi
至此wifi已经设置完成，现在就是开始运行问题，输入：netsh wlan start hostednetwork 然后回车，则启动wifi
以后，每次重启电脑后都需要运行netsh wlan start hostednetwork命令来启动
![](/uploads/2018/03/network_wifi_04.jpg)