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

** 错误详情 **

```
Information:Gradle tasks [:app:generateWandoujiaReleaseSources, :app:prepareWandoujiaReleaseUnitTestDependencies, :app:mockableAndroidJar]
Error:Execution failed for task ':app:processWandoujiaReleaseManifest'.
Manifest merger failed with multiple errors, see logs
```

<img src="/assets/drawable/android_studio_error1.png"  alt="pic" />

** 解决方案 **

在AndroidManifest.xml的application节点下，添加这句：

```
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
```

