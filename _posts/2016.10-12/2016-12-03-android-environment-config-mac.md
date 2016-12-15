---
layout: post
title: Mac开发环境配置
date: 2016-12-03 13:16:00
categories: [Android]
tags: [Android]
---

Mac下面的开发环境配置，jdk的下载安装，Android Studio的下载安装，Android SDK的安装配置，git的配置。具体步骤跟Windows环境下差不太多，Mac是自带git，配置下就行了，详细内容可以参照这篇文章：[Windows开发环境配置](http://xyzlf.github.io/2016/06/12/android-environment-config-win.html)
<!--more-->

## Android SDK的下载与adb命令的配置

1、这个比较费劲，需要有翻墙代理，好在我公司网络自带VPN。如果没有的童鞋，可以去网上购买一个VPN，设置好代理，就能比较轻松的下载完成了。

2、下载完成Android SDK后，配置下adb命令的路径。具体步骤可参照：<http://jingyan.baidu.com/article/59703552c0f8818fc1074041.html>


	(1)在用户的HOME目录下，找到.bash_profile文件，如果没有该文件就创建一个 touch .bash_profile。

	(2)打开.bash_profile文件：open -e .bash_profile，进行路径配置。

	(3)在打开的编辑器里面输入如下路径：
	export PATH=${PATH}:/Users/apple/Library/Android/sdk/platform-tools   （apple就是你电脑用户的名字了）
	export PATH=${PATH}:/Users/apple/Library/Android/sdk/tools
	我的Android SDK放在了主目录下面，所以路径如下：
	export PATH=${PATH}:/Users/zhanglifeng/Android/platform-tools
	export PATH=${PATH}:/Users/zhanglifeng/Android/tools

	(4)保存配置，更新配置环境变量：source .bash_profile。

	(5)打开终端，试试adb命令是否生效。


## Apktool工具的配置

Apktool的github地址：<https://github.com/iBotPeaches/Apktool>

官网教程地址：<https://ibotpeaches.github.io/Apktool/install/>

apktool版本列表官网地址：<https://bitbucket.org/iBotPeaches/apktool/downloads>


## Homebrew的安装配置

官方github地址：<https://github.com/Homebrew/brew>

中文教程地址：<https://aaaaaashu.gitbooks.io/mac-dev-setup/content/Homebrew/index.html>