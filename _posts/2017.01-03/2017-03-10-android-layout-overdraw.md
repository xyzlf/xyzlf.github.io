---
layout: post
title: Android性能优化之过度绘制
date: 2017-03-10 18:20:00
categories: [Android]
tags: [Android]
---

最近看到蛮多关于性能优化的文章，感觉特别受鼓舞。按照文章的教程，自己也查阅官方文档资料，一步步优化下公司的App。
<!--more-->


App的性能指标按照我的理解可以分为以下几个部分（不一定全面）：

- 启动速度：特别是冷启动速度。（冷启动，温启动，热启动）

- App包大小

- 页面的流畅度：使用流畅不掉帧。（涉及UI层级，Overdraw）

- 内存占用

- 耗电

本文主要从UI层级，查看界面是否过度绘制了方面优化下App。

##  官方资料

在手机设置 - 开发者选项 - 打开GPU（图形处理单元）调试开关，步骤如下：
	
	On your mobile device, go to Settings and tap Developer Options.
	In the Hardware accelerated rendering section, select Debug GPU Overdraw.
	In the Debug GPU overdraw popup, select Show overdraw areas.

打开GPU调试开关，进入App之后，界面布局会显示对应的颜色值，颜色值如图：

<img src="/assets/drawable/overdraw.png"  alt="pic" />

颜色值对应的解释如下：

	The colors are hinting at the amount of overdraw on your screen for each pixel, as follows:
	True color: No overdraw
	Blue: Overdrawn once
	Green: Overdrawn twice
	Pink: Overdrawn three times
	Red: Overdrawn four or more times

	//翻译
	原色 – 没有过度绘制 – 这部分的像素点只在屏幕上绘制了一次。
	蓝色 – 1次过度绘制– 这部分的像素点只在屏幕上绘制了两次。
	绿色 – 2次过度绘制 – 这部分的像素点只在屏幕上绘制了三次。
	粉色 – 3次过度绘制 – 这部分的像素点只在屏幕上绘制了四次。
	红色 – 4次过度绘制 – 这部分的像素点只在屏幕上绘制了五次。

## 优化App

对照着官方介绍的资料，打开GPU调试开关，然后打开App，去优化界面布局，尽量减少绘制次数。

- 优化之前

<img src="/assets/drawable/overdraw_setting.png"  alt="pic" />


- 优化之后

<img src="/assets/drawable/overdraw_setting_new.png"  alt="pic" />

- 所做的工作

主要就是根据Hierarchy Viewer工具查看层级，去除不必要的层级。 将App中不必要设置的背景层去除！ 布局文件中，重复的布局文件使用 @layout 标签。相同的父布局文件，尽量使用 <Merge>标签，可减少层级。


## 层级优化工具

Hierarchy Viewer： Android studio - Tools - Android - Android Device Monitor， 打开工具后就能使用了。

使用Lint ： Android studio - Analyze - Inspect code， 检测全项目代码，根据报告依次修改。


## 参考资料

官方文档 Optimizing Layout Hierarchies：<https://developer.android.com/training/improving-layouts/optimizing-layout.html>

官方文档 Debug GPU Overdraw Walkthrough：<https://developer.android.com/studio/profile/dev-options-overdraw.html>

Android性能优化（二）之布局优化面面观：<http://www.jianshu.com/p/4f44a178c547>
