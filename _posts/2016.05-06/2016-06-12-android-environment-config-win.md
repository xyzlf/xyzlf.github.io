---
layout: post
title: Windows开发环境配置
date: 2016-06-12 17:00:00
categories: [Android]
tags: [Android]
---

Windows下面的开发环境配置，jdk安装，Android Studio的安装，以及git的安装及配置~
<!--more-->

##  JDK的下载及安装

从官网下载最新jdk：<http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html> 。
然后一步步安装，配置环境变量，这个没啥说的，一路点击下一步就OK。

##  Android Studio的下载及安装 

国内网络目前还是需要翻墙才能访问，所以呐先安装一个lantern吧，里面有各个平台的版本，挺好用的：<https://github.com/getlantern/lantern>
从官网下载Android Studio：<https://developer.android.com/studio/index.html> 也是一步步点击下一步就OK了。

##  Git的下载及安装

从官网下载Git，<https://git-scm.com/download/win>，然后按照默认步骤一步步安装即可。
配置SSH key文件，生成SSH key文件：<https://help.github.com/articles/generating-an-ssh-key/>

记得配置用户名和邮箱：
设置本地机器默认commit的昵称与Email. 请使用有意义的名字与email.

{% highlight java %}
git config --global user.name "tiemaocsdn"  
git config --global user.email "tiemaocsdn@qq.com"  
{% endhighlight java %}

这个博客写得也挺好的，用中文翻译了一遍，英文不好的同学参照这里吧：<http://blog.csdn.net/renfufei/article/details/41647875>

##  Notepad配置Markdown以及预览
这篇文章写得不错：<http://www.jianshu.com/p/cdb42773fffe>
总结起来总共两步：
1、下载配置文件，format文件，导入。
2、导入预览dll。