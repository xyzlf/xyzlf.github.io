---
layout: post
title: Android Studio常见错误集
date: 2017-04-10 13:36:00
categories: [Tools]
tags: [Tools]
---

记录Android Studio中的一些错误，以及解决方案~
<!--more-->

## Manifest merger failed with multiple errors  

**错误详情**

{% highlight java %}
Information:Gradle tasks [:app:generateWandoujiaReleaseSources, :app:prepareWandoujiaReleaseUnitTestDependencies, :app:mockableAndroidJar]
Error:Execution failed for task ':app:processWandoujiaReleaseManifest'.
Manifest merger failed with multiple errors, see logs
{% endhighlight java %}

<img src="/assets/drawable/android_studio_error1.png"  alt="pic" />

**解决方案**

在AndroidManifest.xml的application节点下，添加这句tools:replace="..."， 参考[Android Studio使用心得 - 常见问题集锦][1]

{% highlight java %}
tools:replace="android:allowBackup, android:theme, android:icon, android:label, android:supportsRtl"

//完整的
<application
	android:allowBackup="false"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="false"
    android:theme="@style/AppTheme"
    tools:replace="android:allowBackup, android:theme, android:icon, android:label, android:supportsRtl">

	// ...

</application>
{% endhighlight java %}


## 使用Rxjava 跟CompileSdkVersion冲突， 导致4.0手机崩溃的问题。

参考文章：<https://stackoverflow.com/questions/37619596/rxjava-noclassdeffounderror-rx-plugins-rxjavaplugins-on-api-16>

接入一个SDK后，方法数超标了。 所以Rxjava中的各种类找不着...继承MultiDexApplication解决问题。


{% highlight java %}

compile 'com.android.support:multidex:1.0.1'

{% endhighlight java %}


{% highlight java %}

In thread: Thread[main,5,main]                                                             
UncaughtException detected: java.lang.NoClassDefFoundError: rx.subscriptions.CompositeSubscription

{% endhighlight java %}

## Gralde Failed to open zip file

**错误详情**

<img src="/assets/drawable/gradle_error2.png"  alt="pic" />

**解决方案**

指定正确的gradle路径。

<img src="/assets/drawable/gradle_error2_solution.png"  alt="pic" />

## Error:Execution failed for task ':app:transformClassesWithExtractJarsForDebug'.

**错误详情**

<img src="/assets/drawable/error_transformClassesWithExtractJars.png"  alt="pic" />

**解决方案**

关掉instant run之后解决


[1]: http://blog.csdn.net/codezjx/article/details/38669939