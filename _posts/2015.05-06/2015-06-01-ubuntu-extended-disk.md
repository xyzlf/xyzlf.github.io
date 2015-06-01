---
layout: post
title: Windows、Ubuntu双系统下，给Ubuntu增加磁盘空间
date: 2015-06-01 20:51:01
categorys: [Ubuntu]
tags: [Ubuntu]
---
&emsp;因为本人安装的是Windows，Ubuntu双系统，所以当时给Ubuntu分配的空间比较小，现在想在Ubuntu下编译Android Rom，但是AOSP代码就是几十G，空间严重不够，因此，准备给Ubuntu增加磁盘空间。
<!--more-->  

1. 首先进入Windows系统，在Windows下使用磁盘管理-压缩卷给压缩出一个未分配的分区出来，然后新建简单卷，按NFTS格式格式化。
2. 重启进入Ubuntu系统。这时可能会进入不了系统，因为刚才多分了一个区，grub引导所在的分区变了。解决办法参见：[http://www.linuxidc.com/Linux/2012-06/61983.htm](http://www.linuxidc.com/Linux/2012-06/61983.htm)
3. 在Ubuntu下使用磁盘工具，找到刚在在windows下的那个分区，记住它的设备名：比如/dev/sda8。
4. 使用终端，输入：mkfs -t ext4 /dev/sda8   
 将刚在的分区格式化为ext4格式。
5. 编辑/etc/fstab使新分进来的设备自动挂载，追加一行即可  
{% highlight java %}
 /dev/sda8                                /home/open      ext4    defaults        0      1 
 {% endhighlight java %} 
&emsp;
6. 重启，它会自动挂载在/home/open下。