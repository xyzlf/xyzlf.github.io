---
layout: post
title: Android Studio依赖本地aar
date: 2016-07-15 15:51:00
categories: [Android]
tags: [Android]
---

写代码都希望高内聚，低耦合。Android Studio中aar的开发模式，可以让大项目，业务独立，互不影响。子业务开发完成后，将Android Library生成的aar，提交到公共平台，当项目足够大的时候，个人觉得这是一种比较不错的开发模式。
<!--more-->

## 本地Library生成的aar路径
在当前的Library下面会有个build目录，在改目录下会生成对应的aar，如我的项目：

    D:\workspace\ShareSDK\shareLibrary\build\outputs\aar

## 依赖本地aar

1、在工程目录app下面建立"libs"目录(如果没有的话)。

2、把aar文件拷入libs目录(例如：sharesdk_release_1.0.aar)。

3、在工程目录app的build.gradle中加入：
{% highlight java %}
android {
	...
	repositories {
		flatDir {
			dirs 'libs'
		}
	}
	...
	compile(name: 'sharesdk_release_1.0', ext: 'aar') {
		exclude group: 'com.android.support', module: 'appcompat-v7'
	}
}
{% endhighlight java %}

## Module依赖aar

当工程大了的时候，会有多个Module。比如一个Demo工程，下面有module: app，userlib，downloadlib，frameworklib。如果userlib，downloadlib都需要依赖一个aar，为了不每个module里面放置一份，可以将aar给frameworklib依赖，然后其他module依赖frameworklib即可。

配置在Demo目录下的build.gradle里面加入下配置：
{% highlight java %}
buildscript {
   ... 
}
allprojects {

    repositories {

        flatDir {
            dirs '../frameworklib/libs'
        }
		....
    }
}
{% endhighlight java %}