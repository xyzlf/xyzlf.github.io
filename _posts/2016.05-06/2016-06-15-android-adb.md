---
layout: post
title: Adb相关问题
date: 2016-06-15 14:20:00
categories: [Android]
tags: [Android]
---

Adb命令相关命令记录，以及一些常见的错误。
<!--more-->

##  常用命令

1、查看栈顶Activity 
{% highlight java %}
adb shell dumpsys activity adb shell dumpsys activity | grep mResumed
{% endhighlight java %}

2、通过命令行打开Activity界面 
{% highlight java %}
adb shell am start -n pkgname/pkgname.activity 
{% endhighlight java %}

例如：adb shell am start -n com.my.demo/.activity.MainActivity 
     adb shell am start -n com.my.demo/com.my.demo.error.ErrorActivity

3、模拟器真机同时存在的情况下，安装Apk 
{% highlight java %}
adb -d install {apkpath}
{% endhighlight java %}

##  Adb Error相关

1、使用Genymotion遇到错误：adb server version (32) doesn't match this client (36); killing...

反反复复的 adb kill-server --> start-server，还是同样的错误。

原因：genymotion的adb设置。
解决方案如下：
<img src="/assets/ico/adb_genymotion_error.jpg"  alt="pic" />
