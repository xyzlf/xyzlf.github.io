---
layout: post
title: Android Studio Jni实例
date: 2016-07-17 17:25:00
categories: [Android]
tags: [Android]
---

最近看了下Jni的配置与使用，因此写个Demo练习一下。在Android Studio里面，编写Jni，使用so还是非常方便的。参考文章：<http://www.jianshu.com/p/d8cde65cb4f7>，感谢作者的分享~
<!--more-->

## NDK下载配置

1、在Android Studio里面先下载NDK。可能需要翻墙，如果下载失败，可以尝试去网络上找找。

<img src="/assets/drawable/ndk_download.jpg"  alt="pic" />

2、配置NDK环境

<img src="/assets/drawable/ndk_config_1.png"  alt="pic" />

<img src="/assets/drawable/ndk_config_2.png"  alt="pic" />

好啦，这样，NDK环境就算配置完成了。

## 建立NdkJniDemo项目

1、新建立工程NdkJniDemo。

2、建立一个JniUtil.java类：
{% highlight java %}
package com.xyzlf.jni.demo;

/**
 * Created by zhanglifeng on 2016/7/17
 */

public class JniUtil {

    public static native String getStringFromJni();
 
}
{% endhighlight java %}

然后Clean Project，Rebuild Project，之后，你就能在项目的路径：app/build/intermediates/classes/debug 看到 com.xyzlf.jni.demo.JniUtil.class
{% highlight java %}
cd app/build/intermediates/classes/debug

javah -jni com.xyzlf.jni.demo.JniUtil
{% endhighlight java %}
<img src="/assets/drawable/ndk_jni_h.png"  alt="pic" />

以上操作如果没有问题的话，就会生成com_xyzlf_jni_demo_JniUtil.h文件。在项目的src/main下面建立一个jni文件夹，将生成的com_xyzlf_jni_demo_JniUtil.h文件拷贝进去。

在jni文件夹里面，新建一个与.h对应的.c文件，名字可以随便，我取得名字为：JniUtil.c，加入以下代码：
{% highlight java %}
//
// Created by zhanglifeng on 2016/7/17.
//

#include "com_xyzlf_jni_demo_JniUtil.h"
/*
* Class:     com_xyzlf_jni_demo_JniUtil
* Method:    getStringFromJni
* Signature: ()Ljava/lang/String;
*/
JNIEXPORT jstring JNICALL Java_com_xyzlf_jni_demo_JniUtil_getStringFromJni
        (JNIEnv *env, jobject obj) {
   return (*env)->NewStringUTF(env, "这里是来自c的string");
}
{% endhighlight java %}

3、Jni类的配置及引用

在项目app下的build.gradle下面配置ndk:
{% highlight java %}
ndk {
	moduleName "NdkJniDemo"          //生成的so名字
	abiFilters "armeabi", "armeabi-v7a", "x86" //输出指定三种abi体系结构下的so库，目前可有可无。
}
{% endhighlight java %}
<img src="/assets/drawable/ndk_config_3.png"  alt="pic" />

在JniUtil里面引用"NdkJniDemo"，记住需要跟上面gradle里面配置的名字一致，否则会找不到，完整代码如下:
{% highlight java %}
package com.xyzlf.jni.demo;

/**
 * Created by zhanglifeng on 2016/7/17
 */

public class JniUtil {

    static {
        System.loadLibrary("NdkJniDemo");//之前在build.gradle里面设置的so名字，必须一致
    }

    public static native String getStringFromJni();

}
{% endhighlight java %}

在MainActivity中调用：
{% highlight java %}
package com.xyzlf.jni.demo;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        TextView textView = (TextView) findViewById(R.id.ndk_text);
        textView.setText(JniUtil.getStringFromJni());
    }
}
{% endhighlight java %}

如果build过程中失败的话，在gradle.properties加入配置
{% highlight java %}
android.useDeprecatedNdk=true
{% endhighlight java %}

以下是运行效果图：

<img src="/assets/drawable/ndk_view.png"  alt="pic" />

工程目录结构：

<img src="/assets/drawable/ndk_jni_struct.png"  alt="pic" />

至此就完成了。

## 项目中引用so
以上项目编译完后，可以在app/build/intermediates/ndk/debug/lib下面找到生成的so:

<img src="/assets/drawable/ndk_so.png"  alt="pic" />

将这几个so，拷贝至app/src/main/jniLibs目录下：（如果没有jniLibs目录，新建一个），这样就直接引用so，可以将，jni文件下面的.h，.c文件删除了。

<img src="/assets/drawable/ndk_so2.png"  alt="pic" />

## ERROR
{% highlight java %}
Error:(36) undefined reference to '__android_log_print'
{% endhighlight java %}

解决方案，在buil.gradle里面添加：ldLibs "log"//实现__android_log_print

完整的build.gradle如下：
{% highlight java %}
android {
    compileSdkVersion 24
    buildToolsVersion "24.0.0"
    defaultConfig {
        applicationId "com.xyzlf.jni.demo"
        minSdkVersion 8
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"

        ndk {
            moduleName "NdkJniDemo"          //生成的so名字
            ldLibs "log"//实现__android_log_print
            abiFilters "armeabi", "armeabi-v7a", "x86" //输出指定三种abi体系结构下的so库，目前可有可无。
        }

    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
{% endhighlight java %}


## 完整项目地址

完整的项目地址：<https://github.com/xyzlf/JniDemo>
