---
layout: post
title: Android签名相关
date: 2016-05-23 16:00:00
categories: [Android]
tags: [Android]
---

记录一些Android签名相关介绍。
<!--more-->

##  Android 签名相关操作

### 1、创建Key

创建key，需要用到keytool.exe (位于jdk1.6.0_24\jre\bin目录下)，使用产生的key对apk签名用到的是jarsigner.exe (位于jdk1.6.0_24\bin目录下)，把上两个软件所在的目录添加到环境变量path后，打开cmd输入如下命令：

{% highlight java %}

keytool -genkey -alias demo.keystore -keyalg RSA -validity 40000 -keystore demo.keystore

/**

说明

-genkey 产生密钥

-alias demo.keystore 别名 demo.keystore

-keyalg RSA 使用RSA算法对签名加密

-validity 40000 有效期限4000天

-keystore demo.keystore

*/

{% endhighlight java %}

### 2、签名

{% highlight java %}

jarsigner -verbose -keystore {your.keystore} -signedjar {new-singed.apk}  {app-unaligned.apk} {your keyAlias}

/*

说明：

-verbose 输出签名的详细信息

-keystore  demo.keystore 密钥库位置

-signedjar demo_signed.apk demo.apk keyAlias 正式签名，三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和 keyAlias.

*/

{% endhighlight java %}

### 3、查看App签名信息 

{% highlight java %}

jarsigner -verify -verbose -certs {apkpath} 


jarsigner -verify -verbose:summary -certs {apkpath}

{% endhighlight java %}

### 4、验证签名
{% highlight java %}

jarsigner -verify {apkpath}

keytool -printcert -jarfile {apkpath}

apksigner verify -v --print-certs

{% endhighlight java %}
