---
title: 不重装迁移系统到新磁盘
date: 2016-12-24 15:03:36
tags:
category:
layout: post
---
几年前700多大洋入的浦科特128G SSD 近来愈发觉得不够用，趁着今年双十一终于决定再入一块三星256G SSD，仍然是700多大洋。虽然并不喜欢三星MLC工艺，无奈这么多年过去接口升级换代，当年的mSATA也只有三星还在稳定出货价钱合适。这应该是小黑T420最后一次磁盘升级了。

用了这么多年的系统，装了无数大小软件，实在不想再重装系统，之前从HDD到SSD，使用分区助手的迁移系统功能，轻松完成转换。这次因为系统盘上有双系统，所以并不能直接用"系统迁移到SSD/HDD"，而是选了"扇区到扇区复制"，希望保留双系统配置。当然小黑只有一个mSATA接口，为了完成这次任务，又入了佳翼的mSATA移动硬盘盒。然后选择正确的磁盘，根据提示重启后开始复制，这个过程大概进行了1小时40分钟，因为小黑那个时代还只有USB2.0，接口速度比较慢只有150Mbps，所以可以算出120G*8/150Mbps = 1.8小时。
![JEYI](/img/2016-12-24_161700.jpg)
![分区助手复制磁盘](/img/2016-12-24_162431.png)

复制完成后，满怀希望地换上新的SSD，以为开机重启就焕发新生了，前面Clover引导都很正常，然后就碰到下面这个鬼画面。
![0xc000000e](/img/2016-12-24_161500.jpg)

网上搜索都指向修复`\boot\BCD`文件，但是首先需要一张系统恢复DVD，而且更关键的是貌似需要重置MBR，而我的MBR已经配置了Clover，当初好不容易搞定三系统，轻易不想再动它。于是继续搜索，意识到系统认为硬件变动了，那体现在BCD里面，就只有分区GUID不同，为新分区重建一个entry应该就可以了。换回旧SSD启动电脑，这里用了EasyBCD来操作，其背后会调用bcdedit.exe来操作BCD文件。首先配置下EasyBCD编辑的BCD文件，然后重启程序让设置生效。这里直接指向了新SSD系统使用的BCD，但是安全起见，最好复制一份出来然后在上面编辑，最后再复制回去。
![设定BCD文件位置](/img/2016-12-24_165701.png)
然后新建启动入口。
![新建启动项](/img/2016-12-24_155520.png)
这里选择了新分区所在的盘符K。

然后用bcdedit.exe(可以在EasyBCD安装目录下找到)找到新建项目的GUID
```
bcdedit /store <path_to_bcd> /enum all
bcdedit /store <path_to_bcd> /enum all /v
```

将盘符改回c盘，设置新建条目为默认启动项。
```
bcdedit /store <path_to_bcd> /set {bootmgr} default <new guid>
bcdedit /store <path_to_bcd> /set {default} osdevice c:
bcdedit /store <path_to_bcd> /set {default} device c:
```

因为换到新磁盘启动时，系统分区一般就是c盘。可能有人会疑问为什么新建条目时，不直接选c盘，我的理解是GUID是从分区算出来的，选c盘可能会算出旧分区的GUID，这与我们的要求不符。改完之后重启，还是失败，但是错误原因已经变了，是找不到启动程序winload，而不再是压根都找不到设备，看来猜测靠谱。
![0xc000000e again](/img/2016-12-24_161600.jpg)

但是明明winload是不会丢的，系统找不到，可能跟多系统有关，引导程序错误地在一个分区搜索winload。继续搜索找到了解决之道，将启动分区从c:改成boot，意思是让引导程序在当前分区寻找winload。重启搞定。
```
bcdedit /store <path_to_bcd> /set {default} osdevice boot
bcdedit /store <path_to_bcd> /set {default} device boot
```
