---
layout: post
title: Gradle常用命令行
date: 2016-05-22 16:00:00
categories: [Android]
tags: [Android]
---

记录一些常用的Gradle命令行。
<!--more-->

##  相关操作

1、查看依赖关系 

{% highlight java %}

gradle :app:dependencies

{% endhighlight java %}

{% highlight java %}

gradle tasks --all
eg: gradle -q app:depIn --configuration compile --dependency appcompat-v7

{% endhighlight java %}


2、打包 

{% highlight java %}

gradle assembleRelease

{% endhighlight java %}

## Windows中gradle本地仓库地址

{% highlight java %}
../android-studio/gradle/gradle-2.10
{% endhighlight java %}

## Error

**1、Failed to open zip file**

<img src="/assets/drawable/gradle_error.png"  alt="pic" />

（1）官网下载对应的gradle版本，这里是下载2.14.1版本：<http://downloads.gradle.org/distributions/gradle-2.14.1-all.zip >

（2）将下载的gradle放置在对应的目录下面。

{% highlight java %}
/Applications/Android Studio.app/Contents/gradle/gradle-2.14.1
{% endhighlight java %}

（3）setting - gradle - Use local gradle distribution : 然后填上如上路径。

参照该篇博文修改：[Android Studio 出现Failed to open zip file的问题](http://blog.csdn.net/captain_magicer/article/details/52076338)

##  相关资料

Gradle官方文档：<http://tools.android.com/tech-docs/new-build-system/user-guide>

Gradle文档：<https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html>

Gradle的中文教程：<http://www.androidchina.net/2155.html>

Gradle依赖项学习总结：<http://www.paincker.com/gradle-dependencies>