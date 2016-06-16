---
layout: post
title: 常用命令行
date: 2016-05-21 16:00:00
categories: [Android]
tags: [Android]
---

记录一些常用的一些命令行，比如：Android中的adb命令行，gradle命令行，签名相关的命令行。
<!--more-->

##  Android gradle相关

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

##  Android 签名相关

创建key，需要用到keytool.exe (位于jdk1.6.0_24\jre\bin目录下)，使用产生的key对apk签名用到的是jarsigner.exe (位于jdk1.6.0_24\bin目录下)，把上两个软件所在的目录添加到环境变量path后，打开cmd输入

{% highlight java %}
keytool -genkey -alias demo.keystore -keyalg RSA -validity 40000 -keystore demo.keystore
/**说明
	-genkey 产生密钥
    -alias demo.keystore 别名 demo.keystore
    -keyalg RSA 使用RSA算法对签名加密
    -validity 40000 有效期限4000天
    -keystore demo.keystore
*/
{% endhighlight java %}

签名

{% highlight java %}
jarsigner -verbose -keystore {your.keystore} -signedjar {new-singed.apk}  {app-unaligned.apk} {your keyAlias}
/*说明：
	-verbose 输出签名的详细信息
    -keystore  demo.keystore 密钥库位置
    -signedjar demor_signed.apk demo.apk keyAlias 正式签名，三个参数中依次为签名后产生的文件demo_signed，要签名的文件demo.apk和 keyAlias.
*/
{% endhighlight java %}

查看App签名信息 
{% highlight java %}
jarsigner -verify -verbose -certs {apkpath} 
jarsigner -verify -verbose:summary -certs {apkpath}
{% endhighlight java %}

验证签名
{% highlight java %}
jarsigner -verify {apkpath}
{% endhighlight java %}


##  反编译相关

1、反编译得到布局文件，资源文件

下载一个apktool工具，然后 apktool d xxx.apk

2、反编译Java文件

（1）Apk反编译：将.apk改成.zip后缀，解压拿到dex。 

（2）Mac下：brew install dex2jar， 然后用dex2jar将dex生成jar。

（3）用gui工具查看：推荐<https://github.com/skylot/jadx>
