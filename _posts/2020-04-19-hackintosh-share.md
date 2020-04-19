---
layout:     post
title:      非常啰嗦的黑果之路分享
subtitle:   技嘉Z390M GAMING / 9600K / RX580-O8G / 94331
date:       2020-04-19
author:     Bokey Lee
header-img: img/post-bg-article.jpg
catalog: true
tags:
    - 黑苹果
---
### 1.前言
本人从Leopard时代开始接触黑苹果，直到上大学换笔记本后作罢，如今已毕业了，日常拍拍片子剪剪视频，恰逢笔记本配置有点跟不上，需要配新机，同时又想要回到苹果环境中，于是几番爬贴，从clover折腾到OC引导，再折腾出通过BOOTCAMP的系统切换。非常感谢论坛里的诸位大佬对黑苹果做出的技术贡献。
在这段时间我也感受到，各类教程都分布得挺分散，所以这篇分享也是希望尽可能记录下黑果过程参考的一些资料与解决办法，帮到更多的人。

### 2.配置
主板	技嘉 Z390M GAMING（BIOS:F9G）
CPU	i5-9600k
内存	镁光英睿达 DDR4 8GX2
显卡	华硕 dual-rx580-o8g（hdmi hdmi dp dp dvi）
SSD	东芝RC500
网卡蓝牙	BCM94331CD

键盘	罗技 K580
显示器	自己组装的23.8寸4K屏幕（MV238QUM-N20）

### 3.目前体验
睡眠	OK 正常
电源	OK CPU变频，开启节能五项
HIDPI	OK 自动开启
WIFI	OK 正常 (10.15.4需加入AirportBrcmFixup.kext方可正常)
蓝牙	OK 隔空投送、接力正常（随航没IPAD无法测试）
显卡	OK 注入FBNAME与缓冲帧双硬解，FCPX工作正常
声卡	OK 注入layout-id=1 正常（调节音量时会卡顿）
NVRAM	OK 正常 可使用“启动磁盘”功能
SMBIOS	OK iCloud imessage facetime 正常

### 4.安装过程
最初安装时使用黑果小兵的10.15.3镜像安装，每次都卡最后2分钟，后来改用10.14.6镜像安装成功
最初选购配置时有参考过如下两贴子，也有尝试使用其EFI
http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1836332&highlight=Z390M
http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1808442
不过初试OC引导时完全搞不懂，甚至连引导界面都看不到，后来改用clover安装
随后成功升级10.15.3.

#### 4.1.BIOS设置
SETTINGS
-INTERNAL GRAPHICS==ENABLED
-ABOVE 4G DECODING==ENABLED
-USB CONFIGURATION:
--LEGACY USB SUPPORT==ENABLED
--XHCI HAND-OFF==ENABLED
BOOT
       -FAST BOOT==ULTRA FAST
       -WINDOWS 8/10 FEATURES==WINDOWS 8/10 WHQL
       -CSM SUPPORT==DISABLED
       -SECURE BOOT==DISABLED
VT-D==DISABLED

#### 4.2.折腾OC
最初接触OC简直一头雾水，这里要隆重感谢XJN大佬与黑果小兵大佬的两篇关于OC的说明文章
https://blog.daliansky.net/OpenCore-BootLoader.html#group-9
https://blog.xjn819.com/?p=543
结合他们的教程，我才一步一步完成了今天这个EFI文件夹。

##### 4.2.1.解锁CFG LOCK
刚刚前面提及的帖子中，有提及OC需要解锁CFG LOCK，于是爬贴摸索，通过setup_var的方法进行解锁
最早参考的帖子现在找不到了，又重新搜了一篇类似的：
https://blog.csdn.net/shuiyunxc/article/details/104295837
如果有跟我使用同款主板的，倒是可以省点力气
4.2.1.1.准备U盘，FAT32格式，建立EFI/BOOT 文件夹
4.2.1.2.下载modeGRUBShell.efi并放入上述文件夹
[attach]4197855[/attach]
4.2.1.3.将上一步的文件改名为bootx64.efi
4.2.1.4.重启通过U盘启动
4.2.1.5.输入setup_var_3 0x5C1 0x00 回车
4.2.1.6.重启，完成。
PS:上述命令仅适合同一块主板的朋友，经查询这块主板的CFG LOCK位于0x5C1，其他主板自己跟随教程操作。

##### 4.2.2.SMBIOS与三码
关于机型的选择是参考了这个帖子：
http://bbs.pcbeta.com/viewthread-1835322-1-1.html
关于三码的生成参考了这个帖子
https://blog.csdn.net/weixin_40684028/article/details/85270633
建议各位在测试时不要急着登录自己的常用id，可以先申请多一个小号，以免主号被拉黑

##### 4.2.3.核显缓冲帧与USB定制
这一部分主要参考了黑果小兵的hackintool教程
https://blog.daliansky.net/Intel-FB-Patcher-tutorial-and-insertion-pose.html
核显选择注入3E980003以配合FCPX，所以下图的DVI和HDMI都是无输出
USB定制的端口如下图，红字是无效的端口（HS为2.0，SS为3.0）

##### 4.2.4.独显FBNAME与端口修订
独显一开始我是使用whatevergreen驱动，后来爬了些贴，被“抛弃WEG”风潮吸引，于是通过爬贴注入了Orinoco这个FBname并抛弃weg，但重启后发现黑屏无输出。
后来才发现，Orinoco的接口是DP/DP/HDMI/HDMI/DVI/DP
而这张华硕RX580的接口是HDMI/HDMI/DP/DP/DVI，刚好完美错开
于是乎继续爬以下贴，了解了修正端口的方法：
http://bbs.pcbeta.com/viewthread-1839231-1-1.html
http://bbs.pcbeta.com/viewthread-1635607-1-1.html
后来又得到指引发现如下贴：
http://zuoyoulz.top/2020/4-9/
http://bbs.pcbeta.com/viewthread-1853631-1-1.html
于是将Orinoco的FB参数一起也注入进去。
 
目前显卡工作正常，不过折腾下来发现其实580两种驱动方法性能表现差异不大。
而且，去掉WEG后，HDMI显示器如果开机前没有通电，进入系统后会没信号，需要重新插拔信号线，dp接口正常。
所以如果想要用我的EFI，省事点建议直接去掉ACPI-PATCH中的GFX0 to IGPU, PEGP to GFX0，HECI to IMEI，删掉DEVICE PROPERTIES中带Orinoco字样的条目，关闭Kernel-Patch中FBNAME开头的几项端口补丁，然后打开Kernel-Add中的WhateverGreen。

##### 4.2.5.NVRAM
刚开始折腾OC时，Z390还“无法支持原生NVRAM”，后来很快又支持了，这里参照了如下帖子：
http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1827430
https://blog.zuiyu1818.cn/posts/z390_NVRAM.html
为后续折腾BOOTCAMP的启动磁盘功能做出了重要铺垫（有空补充）

##### 4.2.6.wifi与蓝牙
94360现在越来越贵，想了一下电脑旁有网线口，但又想配合iphone使用隔空投送，于是选购了94331cd
但是升级到10.15后，94331的wifi驱动会出现问题，这里加入AirportBrcmFixup.kext即可解决。

#### 4.3.添加OC引导项
这个主板最新bios没办法直接添加自定义引导项，bootx64.efi又经常会被windows覆盖，我的解决方案是把OC的bootx64.efi文件放到OC文件夹中，在windows里使用UEFI工具把OC引导项写进bios里。

### 5.其他疑难杂症
#### 5.1.【解决】睡眠后USB非正常推出：
这个据说是白果也会有的问题，解决办法：
http://bbs.pcbeta.com/viewthread-1680369-1-1.html
#### 5.2.【或可解决】调整音量时出现卡顿：
http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=1843112&highlight=%D2%F4%C1%BF%2B%BF%A8
据说BIOS中关闭serial port接口解决，但技嘉把这选项隐藏了，需要刷bios，暂未实践。
#### 5.3.【解决】允许任意来源app：
终端执行sudo spctl --master-disable后输入密码即可
#### 5.4.【未解决】蓝牙只能接收数据无法发送：
不知道是我买的这块94331硬件有问题还是我驱动有问题，状况表现为：可以搜索到附近的蓝牙设备，一旦点击连接，就会疯狂地在已连接与未连接间闪动，无法连接；windows下会一直显示正在连接，随后失败；iphone可以隔空投送发送文件到电脑，电脑隔空投送看不到任何设备。
