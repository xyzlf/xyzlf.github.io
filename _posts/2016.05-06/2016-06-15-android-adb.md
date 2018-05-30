---
layout: post
title: Adb相关问题
date: 2016-06-15 14:20:00
categories: [Android]
tags: [Android]
---

记录一些常用的Adb命令，以及一些常见的错误。
<!--more-->

##  常用命令

1、查看栈顶Activity ：
{% highlight java %}
adb shell dumpsys activity 

adb shell dumpsys activity | grep mResumed
{% endhighlight java %}

2、通过命令行打开Activity界面 ：
{% highlight java %}
adb shell am start -n pkgname/pkgname.activity 
{% endhighlight java %}

例如：
{% highlight java %}
adb shell am start -n com.my.demo/.activity.MainActivity 
adb shell am start -n com.my.demo/com.my.demo.error.ErrorActivity
{% endhighlight java %}

3、adb启动service：
{% highlight java %}
adb shell
am startservice -n ｛包(package)名｝/｛包名｝.{服务(service)名称}
如：启动自己应用中一个service
am startservice -n com.android.traffic/com.android.traffic.maniservice
{% endhighlight java %}

4、adb发送broadcast：
{% highlight java %}
adb shell
am broadcast -a <广播动作>
如：发送一个网络变化的广播
am broadcast -a android.net.conn.CONNECTIVITY_CHANGE
{% endhighlight java %}

5、模拟器真机同时存在的情况下，安装Apk：
{% highlight java %}
adb -d install {apkpath}
{% endhighlight java %}

6、查看App启动消耗时间

You can also measure the time to initial display by running your app with the ADB [Shell Activity Manager](https://developer.android.com/studio/command-line/adb.html#shellcommands) command. Here's an example:

{% highlight java %}

adb [-d|-e|-s <serialNumber>] shell am start -S -W

com.example.app/.MainActivity

-c android.intent.category.LAUNCHER

-a android.intent.action.MAIN

{% endhighlight java %}

The Displayed metric appears in the logcat output as before. Your terminal window should also display the following:

{% highlight java %}

Starting: Intent

Activity: com.example.app/.MainActivity

ThisTime: 2044

TotalTime: 2044

WaitTime: 2054

Complete

{% endhighlight java %}

The -c and -a arguments are optional and let you specify [category](https://developer.android.com/guide/topics/manifest/category-element.html) and [action](https://developer.android.com/guide/topics/manifest/action-element.html) for the intent.

7、查看进程启动

过滤关键字: start proc

##  Adb Error相关

1、使用Genymotion遇到错误：
{% highlight java %}
adb server version (32) doesn't match this client (36); killing...
{% endhighlight java %}

反反复复的 adb kill-server --> start-server，还是同样的错误。

原因：genymotion的adb设置。

解决方案如下：

<img src="/assets/drawable/adb_genymotion_error.jpg"  alt="pic" />


## Adb相关资料

Adb官网教程：<https://developer.android.com/studio/command-line/adb.html#shellcommands>

官网性能优化教程：<https://developer.android.com/topic/performance/launch-time.html>

《Android ADB命令?这一次我再也不死记了！》：<http://mp.weixin.qq.com/s/fWaa1rutwfoIIrje8RfWBw>

adb logcat 命令行用法:<http://blog.csdn.net/tumuzhuanjia/article/details/39555445>