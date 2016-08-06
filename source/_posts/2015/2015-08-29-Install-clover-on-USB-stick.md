title: Install clover on USB stick
date: 2015-08-29 17:46:56
tags:
category:
---
clover支持UEFI和legacy两种BIOS启动，并且可以随意搭配GPT分区和MBR分区安装的系统，但是需要调教参数。
首先有一些概念要区别。
We can boot from a UEFI system and install Windows in UEFI\GPT mode, or boot in BIOS\CSM mode and install Windows in MBR mode.