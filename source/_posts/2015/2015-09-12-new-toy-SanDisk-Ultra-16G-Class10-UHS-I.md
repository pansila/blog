title: New Toy - SanDisk Ultra 16G Class10 UHS-I
date: 2015-09-12 04:10:32
tags:
category:
---
自从iPhone开启自带Flash Rom风潮以来，外置TF卡在手机上越来越不受待见，这次为了把吃灰多年的Raspberry Pi拿出来晒晒太阳，决心新入手一块TF卡。
吐槽下京东可能除了物流价格上真心没啥优势。。。
先来一张大头照
![美照](/img/20150912041903.jpg)
然后照例验证正品，没问题。
测试读写速度
![ATTO Disk BenchMark](/img/2015-09-12_035642.png)
最大速度，读45MB/s，写17MB/s左右。比官网号称48MB/s稍显乏力。。。
写入Raspberry Pi image
![Win32 Disk Imager](/img/2015-09-12_040514.png)
写入速度差不多在15~16MB/s。

附上对比测试，金士顿未知Class卡
![Kinston unknown TF](/img/20150912062844.jpg)
速度测试
![ATTO Disk BenchMark](/img/2015-09-12_050307.png)
看起来应该也是Class 10级别的卡。
~~Raspberry Pi板载速度测试敬请期待。。。~~
接上文接着测试Raspberry Pi上，方法简单粗暴，就是dd拷贝测试：
写
```
dd if=/dev/zero of=~/test.tmp bs=500K count=1024
524288000 bytes (524 MB) copied, 36.7958 s, 14.2 MB/s
```
读
```
dd if=~/test.tmp of=/dev/null bs=500K count=1024
524288000 bytes (524 MB) copied, 28.128 s, 18.6 MB/s
```
这个数据与官方同款测试基本一致（详见下方链接），猜测受限于raspberry pi的SDIO总线速度，读写速度均有一定程度下降。

[树莓派SD卡参考速度](http://elinux.org/RPi_SD_cards#SD_card_performance)