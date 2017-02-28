---
layout: post
title: Gradle常用命令行
date: 2016-05-22 16:00:00
categories: [Android]
tags: [Android]
---

记录一些常用的Gradle命令行。
<!--more-->

##  gradle相关操作

1、查看依赖关系 

{% highlight java %}

gradle :app:dependencies

{% endhighlight java %}

{% highlight java %}

gradle tasks --all
eg: gradle -q app:depIn --configuration compile --dependency appcompat-v7

{% endhighlight java %}

<https://docs.gradle.org/current/userguide/tutorial_gradle_command_line.html>

2、打包 

{% highlight java %}

gradle assembleRelease

{% endhighlight java %}

##  gradle相关资料

Gradle官方文档：<http://tools.android.com/tech-docs/new-build-system/user-guide>

Gradle的中文教程：<http://www.androidchina.net/2155.html>

Gradle依赖项学习总结：<http://www.paincker.com/gradle-dependencies>