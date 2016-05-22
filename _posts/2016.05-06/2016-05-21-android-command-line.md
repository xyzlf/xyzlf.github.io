---
layout: post
title: 常用命令行
date: 2016-05-21 16:00:00
categories: [Android]
tags: [Android]
---

记录一些常用的一些命令行，比如：Android中的adb命令行，gradle命令行，签名相关的命令行。
<!--more-->

##  Android adb相关

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

##  Android gradle相关

1、查看依赖关系 
{% highlight java %}
gradle :app:dependencies
{% endhighlight java %}

2、打包 
{% highlight java %}
gradle assembleRelease
{% endhighlight java %}

##  Android 签名相关

查看App签名信息 
{% highlight java %}
jarsigner -verify -verbose -certs {apkpath} jarsigner -verify -verbose:summary -certs {apkpath}
{% endhighlight java %}