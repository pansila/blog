title: Resize MAC OS X (Yosemite) Disk
date: 2015-09-05 02:58:20
tags:
category:
---
### 调整MAC OS X系统所在分区大小

开始安装Yosemite时，划分的分区过小，安装Xcode和IOS SDK时磁盘已经不够。作为爱偷懒的程序员，自然是能不重新安装最好不重装。

第一个想到的办法是Windows时代常用的Ghost。首先新划分一个合适大小的分区，格式不格式化无所谓，设置分区为激活状态。这里推荐分区助手，无损调节分区大小。
然后USB启动到PE，Ghost搬移partition，全程无压力。完成后启动新的Yosemite分区，发现分区大小也复制了过来。此法不通。

很快发现，Yosemite自带有Disk Utility，里面有Restore功能，专业系统搬移！Apple简直是太有预见性！
所以只要把原来的分区内容restore到一个更大的分区，就等于调节了分区大小。
不过要搬移的disk在Yosemite启动后就不能操作了，所以要从Yosemite的安装盘启动，里面也有这个工具。而且速度比Ghost还要快的多，3分钟搞定。