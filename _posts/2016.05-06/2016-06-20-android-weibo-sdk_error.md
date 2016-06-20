---
layout: post
title: 微博客户端导入Android Studio编译错误
date: 2016-06-20 02:00:00
categories: [Android]
tags: [Android]
---

从官网下拉微博SDK代码：<https://github.com/sinaweibosdk/weibo_android_sdk>。导入工程后，总是有各种编译错误。记录下遇到的问题，及解决方法~
<!--more-->

##1、libpng error

{% highlight java %}
:weiboSDKDemo:mergeDebugAssets
:weiboSDKDemo:generateDebugResValues UP-TO-DATE
:weiboSDKDemo:generateDebugResources
:weiboSDKDemo:mergeDebugResources
AAPT err(Facade for 1333507005): libpng error: Not a PNG file
Error:Execution failed for task ':weiboSDKDemo:mergeDebugResources'.
> Some file crunching failed, see logs for details
{% endhighlight java %}

解决方法：
在weiboSDKDemo的build.gradle中加入：

{% highlight java %}
apply plugin: 'com.android.application'

android {
 **
	aaptOptions{
        cruncherEnabled = false
    }
}
{% endhighlight java %}

##2、9-patch error

{% highlight java %}
:weiboSDKDemo:mergeDebugResources
AAPT err(Facade for 1324592560): libpng error: Not a PNG file
AAPT err(Facade for 1324592560): ERROR: 9-patch image D:\workspace\WeiboSDKDemo_2.5.1\weiboSDKDemo\src\main\res\drawable\ic_login_button_blue_normal.9.png malformed.
AAPT err(Facade for 1324592560):        No marked region found along edge.
AAPT err(Facade for 1324592560):        Found along top edge.
Error:Execution failed for task ':weiboSDKDemo:mergeDebugResources'.
> Some file crunching failed, see logs for details
{% endhighlight java %}

解决方法：修复图片 ic_login_button_blue_normal.9.png 重新设置下图片伸缩域

##3、multiple dex 微博 3.0.1 bug

{% highlight java %}
Error:Error converting bytecode to dex:
Cause: com.android.dex.DexException: Multiple dex files define Lcom/sina/weibo/sdk/BuildConfig;
:weiboSDKDemo:transformClassesWithDexForDebug FAILED
Error:Execution failed for task ':weiboSDKDemo:transformClassesWithDexForDebug'.
> com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process 'command 'C:\Program Files\Java\jdk1.8.0_91\bin\java.exe'' finished with non-zero exit value 2
{% endhighlight java %}

解决方法：解包 weibosdkcore.jar，删除里面的 BuildConfig.class，重新打包 jar。
本来这个文件不应该存在于jar里，不知微博的开发怎么搞进去的。
<https://github.com/sinaweibosdk/weibo_android_sdk/issues/32>

3、解压包步骤

(1)解压jar包：
{% highlight java %}
	jar xf weibosdkcore.jar
{% endhighlight java %}
(2)删除解压后目录里面的com/sina/weibo/sdk/BuildConfig.class
(3)重新打包：
{% highlight java %}
	jar cvf weibosdkcore.jar *
{% endhighlight java %}