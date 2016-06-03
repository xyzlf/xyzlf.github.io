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

gradle tasks --all
eg: gradle -q app:depIn --configuration compile --dependency appcompat-v7
https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html

2、打包 
{% highlight java %}
gradle assembleRelease
{% endhighlight java %}

##  Android 签名相关

查看App签名信息 
{% highlight java %}
jarsigner -verify -verbose -certs {apkpath} jarsigner -verify -verbose:summary -certs {apkpath}
{% endhighlight java %}

##  反编译相关

1、反编译得到布局文件，资源文件

下载一个apktool工具，然后 apktool d xxx.apk

2、反编译Java文件

（1）Apk反编译：将.apk改成.zip后缀，解压拿到dex。 

（2）Mac下：brew install dex2jar， 然后用dex2jar将dex生成jar。

（3）用gui工具查看：推荐https://github.com/skylot/jadx
