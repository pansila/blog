---
title: ETL利器Kettle数据库同步实战
layout: post
abbrlink: ba926855
date: 2016-12-25 18:19:36
tags:
category:
---
最近有需求在SQL server和MySQL之间数据库同步，在比较多种方案后选择了Kettle，因其开源免费，用户多文档完善，图形化编辑上手容易。这里使用了其最新版本，Pentaho Data Integration 7.0。

具体需求不复杂，要从MySQL数据库一张表定期更新到SQL server数据库。这里面数据库之间的差异，Kettle已经通过SQL语言抽象，所以只要处理表结构的差异和转换，主要是数据类型的转换和值映射。这些转换都使用了`Script Values / Mod`模块处理。而类型和值都匹配仅字段名不同的转换，可以直接在`Insert / Update`模块里面映射。

为了便于后续调试，为远程MySQL数据库做了一个本地映射，这里用到了合并模块`Merge Rows (diff)`和同步模块`Synchronize after merge`。

首先为这三个数据分别建立连接。从左边设计面板拖出一个表输入模块`Table input`。
![Table input](/img/2016-12-25_203233.png)

点击New按钮，为数据库连接配置参数。配置好后可以点下面的Test按钮，测试配置是否正确。为本地备份数据库也如此建立连接。
![Edit connection for MySQL](/img/2016-12-25_203404.png)
这是SQL Server数据库的连接配置。
![Edit connection for SQL server](/img/2016-12-25_203821.png)
注意从官网下载的Kettle缺少这两个数据库的驱动文件。分别是`jtds-1.3.1.jar`和`mysql-connector-java-5.1.40-bin.jar`。放到kettle的lib目录下即可。

然后建立远端数据库到本地数据库的映射，同样从左边面板拖出来`Merge Rows (diff)`，这个模块根据若干Key来检索两边数据，然后根据选定的若干Value来检查数据是否一致，哪边的数据较新，哪边的过时。
![Merge Rows (diff)](/img/2016-12-25_202709.png)
注意这里的`Flag fieldname`字段，它会作为输出之一送到后续模块，后面会看到它的作用。

接着放上同步模块`Synchronize after merge`，这里有个坑，可以看到General页旁边还有个Advanced页面，一般这种并排折叠的tab页都是些高级配置，不配置默认就不起作用。然而这里不是，它不配置就无法正常工作。
![Synchronize after merge](/img/2016-12-25_204752.png)
![Synchronize after merge Advanced](/img/2016-12-25_205129.png)
这是配置后的样子，可以看到前面的flagfield字段的作用，它会保存前面Merge模块的比较结果，有三个值`new`，`changed`和`deleted`，意思是对照远程参考数据库，正在检索的数据是否是新添加的，是否是已经有的数据但是需要更新的，是否是过时的数据需要删除的。

接着来到`Script Values / Mod`模块，这是整个任务的重点，
``` javascript
//类型映射，datetime to string可以方便地用内置的JS函数库date2str来完成
i_FStartDate = date2str(WRITE_DATE, "yyyy-mm-dd hh:mm:ss");
i_FCreateDate = i_FStartDate;
var d1 = str2date(i_FStartDate, "yyyy-mm-dd hh:mm:ss");

// 值映射
switch (true) {
case (PASS_TYPE.equals("员工车辆")):
  // 还可以对数据做一些计算后转换
	i_FEndDate = dateAdd(d1, "y", year(VALID_DATE));
	i_FEndDate = date2str(i_FEndDate, "yyyy-mm-dd hh:mm:ss");
	i_FCardTypeID = 1010;
	break;
case (PASS_TYPE.equals("客户车辆")):
	i_FEndDate = dateAdd(d1, "y", year(VALID_DATE));
	i_FEndDate = date2str(i_FEndDate, "yyyy-mm-dd hh:mm:ss");
	i_FCardTypeID = 1020;
	break;
case (PASS_TYPE.equals("作业车辆")):
	i_FEndDate = dateAdd(d1, "y", year(VALID_DATE));
	i_FEndDate = date2str(i_FEndDate, "yyyy-mm-dd hh:mm:ss");
	i_FCardTypeID = 1030;
	break;
default:
	i_FEndDate = dateAdd(d1, "y", year(VALID_DATE));
	i_FEndDate = date2str(i_FEndDate, "yyyy-mm-dd hh:mm:ss");
	i_FCardTypeID = 1020;
	break;
}
```
datetime to string可以方便地用内置的JS函数库date2str来完成，甚至进而可以对数据做一些计算，这里体现了用javascript转换的灵活性，这在`Insert / Update`是不可能的。 注意转换后的字段，在javascript可以当成普通变量赋值，只要在下面的Fields里面声明过。
![Script Values / Mod](/img/2016-12-25_210515.png)

最后将处理好的字段和可以直接映射的字段，在`Insert /Update`里面做好一一映射。这里也有个新手坑，根据look up keys检索不到的数据会被insert到目标数据库，检索到的数据会接着去比较下面的Update fields映射表，值不同的字段会接着去Update，除非后面Update字段指定了N。然而对于没有在Update fields表里出现的字段，该模块即不会去insert也不会去update。这点区别很重要，因为一般数据表都会有一个主键，这个键是不能重复的，如果insert或update时包含了这个字段，会报错该字段无法更新。所以不要把主键或者Identity属性字段放到Update fields里面，让数据库内部自动分配主键，这也适用于其它自动过程控制的字段。
![Insert / Update](/img/2016-12-25_210645.png)

这样就得到了最终的转换方案。
![Final solution](/img/2016-12-25_200159.png)

为了定期运行这个转换任务(Transformation)，还要建立Kettle里面的Job任务，也很简单，新建一个job，拖出start模块。设置运行间隔。
![Job Scheduling](/img/2016-12-25_213058.png)
然后拖出Transformation模块，指定之前配置的转换任务文件路径。
![Transformation](/img/2016-12-25_213327.png)
连接两个模块，搞定。
![Job](/img/2016-12-25_213426.png)
