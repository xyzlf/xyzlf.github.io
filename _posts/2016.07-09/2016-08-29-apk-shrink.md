---
layout: post
title: 关于Apk瘦身的实践
date: 2016-08-29 11:40:00
categories: [Android]
tags: [Android]
---


Apk大小，是App性能优化中的一个比较重要的指标。App越小，性能越好，用户使用的就会越方便，用户也会更加喜欢。下面记录下自己对App瘦身的一些实战经验~
<!--more-->

1、关于APK瘦身值得分享的一些经验：<http://zmywly8866.github.io/2015/04/06/decrease-apk-size.html>

2、如何缩减APK包大小？：<https://github.com/android-cn/android-discuss/issues/51>

3、那些你不知道的Apk瘦身：<http://blog.csdn.net/vfush/article/details/52266843>

4、西北野狼 Apk瘦身：<http://www.cnblogs.com/androidsuperman/p/4201626.html>

5、瘦身终极指南：<http://jayfeng.com/categories/android/>

以上文章，都很详细的介绍了瘦身方法。我根据上面的提示，对App进行相应的实战操作。我就简单讲讲实际操作。

## 压缩图片

减小Apk大小，压缩资源文件能起到立杆见影的效果。上面的文章提到了不错的压缩图片的软件：<https://tinypng.com/>，提供批量在线压缩，能减小很多。

## 用Lint删除废弃资源

用Android Studio自带的工具，进行Lint检测，然后将废弃资源删除。

**1、Step1**

<img src="/assets/drawable/shrink_lint_step1.png"  alt="pic" />

**2、Step2**

<img src="/assets/drawable/shrink_lint_step2.png"  alt="pic" />

**3、Step3**

<img src="/assets/drawable/shrink_lint_step3.png"  alt="pic" />


当然，如果有兴趣，可以研究下自定义lint，然后检测到废弃资源后，通过自定义脚本一一删除，这也算是自动化build的范畴了。

## 添加shrinkResources字段

1、在build.gradle下面，增加shrinkResources字段，配合混淆，能够减小不少大小，去试试吧。

	buildTypes {
		release {
			// ***
			shrinkResources true
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			// ***
		}
	}
	
2、增加配置 resConfigs 'en','zh-rCN'，只保留可用的string资源。

	defaultConfig {
        //...
        resConfigs 'en', 'zh-rCN'
        //...
    }