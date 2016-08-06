title: 被GRUB2破坏MBR后Yosemite的恢复
date: 2015-09-05 01:45:14
tags:
category:
---
之前已经成功安装了Yosemite到Thinkpad T420上，然而不幸的是，楼主手贱在玩Ubuntu 14.04时把grub2安装到了所有硬盘上。
不得不吐槽Linux系统桌面软件不如Windows成熟，这么重要的操作，Ubuntu自带的这个grub2配置GUI程序，在应用修改时居然没有一声提示，所有tab下的设置都统统设了下去。欲哭无泪。

由于之前已经成功安装Yosemite，而Grub2理论上只会破坏MBR以及MBR到VBR之间的数据，所以只要恢复这部分数据就可以了。
那么原来的这部分数据是什么呢，是Yosemite自己的bootloader，由于楼主之前用Multibeast安装了Chimera bootloader，所以首先要找到这个bootloader的boot0数据，即MBR数据。

有两种办法可以拿到这个数据：
1. 找到相应版本的Chimera pkg，这个文件可以用解压缩软件打开，在路径`Chimera 4.1.0.pkg\chimeraV41.pkg\Payload\Payload~\.\usr\standalone\i386\`
下可以看到有四个文件：`boot`，`boot0`，`boot0md`，`boot1h`，boot是存放于HFS文件系统下的Chimera主体程序，boot0存放于MBR，boot1h存放于VBR，这几个文件的关系有兴趣的可以看[GRUB的维基介绍](https://en.wikipedia.org/wiki/GNU_GRUB)，原理一样。

2. 另一个地方就在Yosemite系统里面，`/usr/standalone/i386`，里面也有这几个文件。

找到这几个文件后，就可以动手实施恢复。**以下高能，模仿慎重**
首先要独立于要操作的硬盘所在的系统进行操作，比如可以用USB盘启动PE，或者Yosemite的安装盘，或者其它硬盘上的系统。否则会提示无法操作成功，目标硬盘被占用。
楼主使用其他硬盘上的Ubuntu操作，使用最简单的dd指令即可恢复。
`sudo dd if=/media/username/bakup/boot0 of=/dev/sdc bs=440 count=1`
里面的路径注意替换成自己的，440字节是为了避免把分区表覆盖，同时还要保留分区表前面6个字节的磁盘签名。关于MBR可以猛戳[MBR维基](https://en.wikipedia.org/wiki/Master_boot_record)。
最后，注意要把活动分区设为Yosemite所在的分区，这个boot0，似乎只认活动分区，不认HFS分区。
