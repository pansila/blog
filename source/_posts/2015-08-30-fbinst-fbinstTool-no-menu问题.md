title: fbinst/fbinstTool no menu问题
date: 2015-08-30 11:30:40
tags:
category:
---
安利下，fbinst是个万能启动盘制作工具，能引导多种类型系统，网管必备。。。
使用fbinstTool制作启动盘时，碰到无法启动引导程序问题，提示"no menu"信息。

![错误提示](/img/2015-08-30_113215.png)

google搜索没有结果，只好尝试自己排查问题。首先fbinst是个[开源程序](http://download.gna.org/grubutil/)，因此最简单办法就是看代码为何出现如此错误。
搜索no menu，可以看到相关代码在fbmbr.S
```
	movb	$FBM_TYPE_MENU, %cl
	call	find_first_item

1:
	jc	1f
	subb	$1, %bl
	jc	boot_item
	call	find_next_item
	jmp	1b

1:
	movw	$ABS(err_no_menu), %si
	jmp	fail
	
	...

err_no_menu:
	.ascii	"no menu\0"
```
可以看到是在查找FBM_TYPE_MENU这个类型的数据时失败所致，继续回溯调用路径，可以看到这部分代码在parse_menu label下，而调用parse_menu的地方只有一个
```
	movw	$ABS(menu_file), %si
	pushw	%si
	call	check_file
	popw	%si
	jc	1f
	call	load_file
	jmp	parse_menu
	
	...
	
	menu_file:
	.ascii	FB_MENU_FILE "\0"
```
FB_MENU_FILE就是要查找的目标，看看它是什么，搜索可以发现其定义在fbinst.h，`#define FB_MENU_FILE		"fb.cfg"`
所以问题是这个文件找不到，这个文件其实是有的，它就是fbinst菜单

![fbinst菜单](/img/2015-08-30_122425.png)
原来默认启动项指向了一个不存在的条目，这才想起来因为之前配置过其它引导条目，后又删除忘记恢复default所致。。。
修改后，正常启动。
![正常启动](/img/2015-08-30_113423.png)
