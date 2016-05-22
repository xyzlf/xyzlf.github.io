---
layout: post
title: 常用命令行
date: 2016-05-22 16:00:00
categories: [Android]
tags: [Android]
---

Android adb相关

1、查看栈顶Activity 

adb shell dumpsys activity adb shell dumpsys activity | grep mResumed

2、通过命令行打开Activity界面 

adb shell am start -n pkgname/pkgname.activity 

例如：adb shell am start -n com.my.demo/.activity.MainActivity 

     adb shell am start -n com.my.demo/com.my.demo.error.ErrorActivity

3、模拟器真机同时存在的情况下，安装Apk 

adb -d install {apkpath}

Android gradle相关

1、查看依赖关系 

gradle :app:dependencies

2、打包 

gradle assembleRelease

Android 签名相关

1、查看App签名信息 

jarsigner -verify -verbose -certs {apkpath} jarsigner -verify -verbose:summary -certs {apkpath}