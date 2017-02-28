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

1、反编译apk，获取布局文件和资源文件。

下载一个apktool工具，输入如下命令：(apktool官方教程网站：<https://ibotpeaches.github.io/Apktool/install/>)

{% highlight java %}
apktool d xxx.apk
{% endhighlight java %}	

2、反编译class文件。

（1）Apk反编译：将.apk改成.zip后缀，解压拿到dex。 

（2）dex2jar将dex生成jar。在Mac下安装dex2jar:

{% highlight java %}
brew install dex2jar
{% endhighlight java %}

（3）用gui工具查看，推荐这个：<https://github.com/skylot/jadx>

##	Java

1、查看字节码详细信息
{% highlight java %}
javap -v FileUtils.class
{% endhighlight java %}	

2、解压jar包
{% highlight java %}
jar xf weibosdkcore.jar
{% endhighlight java %}

3、重新打包
{% highlight java %}
jar cvf weibosdkcore.jar *
{% endhighlight java %}

