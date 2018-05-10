---
layout: post
title: Android动态更新so
date: 2018-05-09 17:05:00
categories: [Android]
tags: [Android]
---

说明：此文章完全参照《Android热修复升级探索——SO库修复方案》<https://yq.aliyun.com/articles/217377>，整理做个笔记。

Android开发过程中，经常会引入so文件，因为so具有安全，高效，方便移植的特点。有些时候，我们需要在不发版的情况下，更新so文件，因此就有了这篇文章啦。下面主要讲讲动态更新so的实践...
<!--more-->


## SO的注册方式

动态更新SO，首先需要了解JNI的两个注册方式

- 静态注册

{% highlight java %}

#include <jni.h>
#include <string>
	
//静态注册
extern "C"
JNIEXPORT jstring JNICALL
Java_com_sankuai_ndk_demo_NdkManager_stringFromJNI(JNIEnv *env, jclass type) 	{
    std::string hello = "我是补丁so，动态加载即时生效";
    return env->NewStringUTF(hello.c_str());
}
	
{% endhighlight java %}

- 动态注册

{% highlight java %}

#include <jni.h>
#include <string>
	
//动态注册
jint test(JNIEnv *env, jclass clazz) {
    return 10086;
}
	
JNINativeMethod nativeMethods[] = {"test", "()I", (void *) test};
	
#define JNIREG_CLASS "com/sankuai/ndk/demo/NdkManager"
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    printf("old JNI_OnLoad");
	
    JNIEnv* env = NULL;
    //判断虚拟机状态是否有问题
    if(vm->GetEnv((void**)&env,JNI_VERSION_1_6)!= JNI_OK){
        return -1;
    }
	
    jclass clz = env->FindClass(JNIREG_CLASS);
    if (env->RegisterNatives(clz, nativeMethods, sizeof(nativeMethods)/ sizeof(nativeMethods[0])) != JNI_OK) {
        return JNI_ERR;
    }
	
    return JNI_VERSION_1_6;
}

{% endhighlight java %}

- Android代码

{% highlight java %}

package com.sankuai.ndk.demo;

public class NdkManager {
    static {
        System.loadLibrary("native-lib");
    }
    /**
     * A native method that is implemented by the 'native-lib' native 
     * library, which is packaged with this application.
     */
    public static native String stringFromJNI();
	
    public static native int test();
	
}

{% endhighlight java %}


## Android加载SO的两种方式

Java Api提供以下两个接口加载一个so库。这两种方式最后都调用了nativeLoad(name, loader, librarySearchPath)这个方法，参数name：so库在磁盘中的完整路径名。

System.loadLibrary(String libName)：传进去的参数：so库名称， 表示的so库文件，位于apk压缩文件中的libs目录，最后复制到apk安装目录下。

System.load(String pathName)：传进去的参数：so库在磁盘中的完整路径， 加载一个自定义外部so库文件 。

java.lang.Runtime类里面：

{% highlight java %}

private String doLoad(String name, ClassLoader loader) {
    // ...
    String librarySearchPath = null;
    if (loader != null && loader instanceof BaseDexClassLoader) {
        BaseDexClassLoader dexClassLoader = (BaseDexClassLoader) loader;
        librarySearchPath = dexClassLoader.getLdLibraryPath();
    }
    // nativeLoad should be synchronized so there's only one LD_LIBRARY_PATH in use regardless
    // of how many ClassLoaders are in the system, but dalvik doesn't support synchronized
    // internal natives.
    synchronized (this) {
        return nativeLoad(name, loader, librarySearchPath);
    }
}

{% endhighlight java %}

## so库热部署实时生效可行性分析

- **jni动态注册**

分析SO的加载原理，动态注册的native方法，每次JNI_OnLoad方法都会重新完成一次映射。所以是否只要先加载原来的so库,，然后再加载补丁so库，就能完成Java层native方法到native层patch后的新方法映射，这样就完成动态注册native方法的patch实时修复。实际操作发现：

**ART**

能正常工作。

**Dalvik**

Dalvik下存在bug，Dalvik下根据so的文件名称去查找句柄，发现加载过，直接返回so的句柄。因此热修复so的时候，只需要在so文件后面加个随机数就行..

- **jni静态注册**

静态注册有两个难点：

(1)首先需要解注册静态注册的native方法， 这个也是难点， 因为我们很难知道so库中哪几个静态注册的native方法发生了变更。

(2)假设就算我们知道如果静态注册的native方法需要解注册， 重新load补丁so库也有可能被修复也有可能不被修复。

上面对补丁so进行了第二次加载， 那么肯定是多消耗了一次本地内存， 如果补丁so库够大， 补丁so够多，那么JNI层的OOM也不是没可能。

## so库冷部署重启生效实现方案

- 接口调用替换

SOPatchManager.loadLibrary接口加载so库的时候优先尝试去加载sdk指定目录下的补丁so， 加载策略如下：

如果存在则加载补丁so库而不会去加载安装apk安装目录下的so库。

如果不存在补丁so， 那么调用System.loadLibrary去加载安装apk目录下的so库。

优点：不需要对不同sdk版本进行兼容， 因为所有的sdk版本都有System.loadLibrary这个接口。

缺点： 调用方需要替换掉System默认加载so库接口为sdk提供的接口， 如果是已经编译混淆好的三方库的so库需要patch， 那么是很难做到接口的替换。

虽然这种方案实现简单， 同时不需要对不同sdk版本区分处理，但是有一定的局限性没法修复三方包的so库同时需要强制侵入接入方接口调用。

-  反射注入


## 如果正确复制补丁so库

查找出对应的CpuABI，然后复制对应的so。

{% highlight java %}

private String[] getPrimaryCpuAbi() {
    String[] cpuAbi = null;
    try {
	
        PackageManager pm = getPackageManager();
        if (null != pm) {
            ApplicationInfo applicationInfo = pm.getApplicationInfo(getPackageName(), 0);
	
            if (null != applicationInfo) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                    Field field = ApplicationInfo.class.getField("primaryCpuAbi");
                    field.setAccessible(true);
                    String abi = (String) field.get(applicationInfo);
	
                    cpuAbi = new String[]{abi};
                } else {
                    cpuAbi = new String[]{Build.CPU_ABI, Build.CPU_ABI2};
                }
            }
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
    return cpuAbi;
}
	
{% endhighlight java %}

## 参考资料

Android热修复升级探索——SO库修复方案:<https://yq.aliyun.com/articles/217377>