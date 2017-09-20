---
layout: post
title: 初识AspectJ
date: 2017-09-20 16:00:00
categories: [AOP]
tags: [AOP]
---

AOP(Aspect Oriented Programming),中文通常翻译成面向切面编程。在Java当中我们常常提及到的是OOP(Object Oriented Programming)面向对象编程。其实这些都只是编程中从不同的思考方向得出的一种编程思想、编程方法。
<!--more-->
在开发过程中，对于很多场景可以借助AOP来完成，比如日志监控，异常处理，无侵入的在宿主中插入一些代码等。这里有一篇博文介绍的非常详细[《深入理解Android之AOP》][1]，感兴趣的可以深入阅读一下~

本文主要从实践层面来学习下AspectJ的使用，以及实际项目中中，可以给程序开发带来的方便。

## AspectJ的配置

1、工程根目录build.gradle配置

{% highlight java %}

buildscript {
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'org.aspectj:aspectjtools:1.8.6'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

...

{% endhighlight java %}

2、app目录下build.gradle配置

{% highlight java %}

import org.aspectj.bridge.IMessage  // 添加aspectj的相关类
import org.aspectj.bridge.MessageHandler // 添加aspectj的相关类
import org.aspectj.tools.ajc.Main // 添加aspectj的相关类

apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.1"
    defaultConfig {
        applicationId "com.xyzlf.aspectjdemo"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:26.+'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'

    compile 'org.aspectj:aspectjrt:1.8.6' // 添加aspectj的依赖
}

//-------------------------------------------
//-----------------添加如下代码----------------
//-------------------------------------------
final def log = project.logger
final def variants = project.android.applicationVariants
//在构建工程时，执行编织
variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.8",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}

{% endhighlight java %}

## 代码编写

1、自定义切片，在People类的static静态代码块执行完成后，打印一句话。

2、自定义Pointcut && 组合Pointcut， 对MainActivity中的crash进行处理。

{% highlight java %}

package com.xyzlf.aspectjdemo;

import android.util.Log;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

/**
 * Created by zhanglifeng on 2017/9/20.
 */

@Aspect
public class TestAspect {

    public static final String TAG = "TestAspect";

    //@After，表示使用After类型的advice，里面的value其实就是一个poincut
    @After(value = "staticinitialization(*..People)")
    public void afterStaticInitial(){
        Log.d(TAG,"the static block is initial");
    }

    @Pointcut(value = "handler(Exception)")
    public void handleException(){

    }

    @Pointcut(value = "within(*..MainActivity)")
    public void codeInMain(){

    }

    // 这里通过&&操作符，将两个Pointcut进行了组合
    // 表达的意思其实就是：在MainActivity当中的catch代码块

    @Before(value = "codeInMain() && handleException()")
    public void catchException(JoinPoint joinPoint){
        Log.d(TAG,"this is a try catch block. " + joinPoint.toString());
    }

}

{% endhighlight java %}

## 运行程序后日志

{% highlight java %}

09-20 03:56:56.672 10847-10847/? D/TestAspect: the static block is initial
09-20 03:56:56.672 10847-10847/? D/TestAspect: this is a try catch block.)
                                               
                                               [ 09-20 03:56:56.694 10847:10847 D/         ]
                                               HostConnection::get() New Host Connection established 0xe0cb1280, tid 10847


{% endhighlight java %}

## Demo地址

地址：<https://github.com/xyzlf/AspectjDemo>

## 参考资料 

1、Android AOP学习之：AspectJ实践：<https://juejin.im/post/58ad3944b123db00672cdeeb>

2、看 AspectJ 在 Android 中的强势插入：<https://juejin.im/post/587989f48d6d810058bbae01>

3、深入理解Android之AOP：<http://blog.csdn.net/innost/article/details/49387395>


[1]: http://blog.csdn.net/innost/article/details/49387395