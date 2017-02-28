---
layout: post
title: Android反编译及Java相关操作
date: 2016-05-24 16:00:00
categories: [Android]
tags: [Android]
---

记录一些关于Android反编译相关的操作。
<!--more-->

##  反编译相关

1、反编译得到布局文件，资源文件

下载一个apktool工具，然后 apktool d xxx.apk。 apktool官方教程网站：<https://ibotpeaches.github.io/Apktool/install/>


2、反编译Java文件

（1）Apk反编译：将.apk改成.zip后缀，解压拿到dex。 

（2）Mac下：brew install dex2jar， 然后用dex2jar将dex生成jar。

（3）用gui工具查看，推荐这个：<https://github.com/skylot/jadx>

##	Java

1、查看字节码详细信息
{% highlight java %}
javap -v FileUtils.class
{% endhighlight java %}	

2、解压jar包：
{% highlight java %}
jar xf weibosdkcore.jar
{% endhighlight java %}

3、重新打包：
{% highlight java %}
jar cvf weibosdkcore.jar *
{% endhighlight java %}

