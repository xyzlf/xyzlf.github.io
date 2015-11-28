---
layout: post
title: 从一次实例的注入看ClassLoader的一个坑
date: 2015-11-28 20:25:53
categories: [Android]
tags: [inject]  
---  
#### 背景
在Android上使用MediaPlayer时发现某些情况下的会导致莫名的NPE：

{% highlight java %}
java.lang.NullPointerException
at android.media.MediaPlayer$EventHandler.handleMessage(MediaPlayer.java:2398)
at android.os.Handler.dispatchMessage(Handler.java:110)
at android.os.Looper.loop(Looper.java:193)
at android.app.ActivityThread.main(ActivityThread.java:5331)
at java.lang.reflect.Method.invokeNative(Native Method)
at java.lang.reflect.Method.invoke(Method.java:515)
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:832)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:648)
at dalvik.system.NativeStart.main(Native Method) 
{% endhighlight java %}
<!--more-->
但是这些机型都是非原生ROM，因此不好从源码判断是哪些出了问题，想到一个比较极端的方式，就是替换MediaPlayer内部的EventHandler的实例，然后try/catch住handleMessage方法。

#### 一、首先看MediaPlayer的源码：
[http://androidxref.com/4.4.2_r2/xref/frameworks/base/media/java/android/media/MediaPlayer.java#2181](http://androidxref.com/4.4.2_r2/xref/frameworks/base/media/java/android/media/MediaPlayer.java#2181)  
发现MediaPlayer.EventHandler类是一个private的成员类  
对于private的成员访问方式一种是反射，一种是通过API欺骗。但是这里因为要继承MediaPlayer.EventHandler，所以只能通过API欺骗。   

#### 二、构造SafeEventHandler

{% highlight java %}
package android.media;

import android.os.Looper;
import android.os.Message;

public class SafeEventHandler extends android.media.MediaPlayer.EventHandler {
	public SafeEventHandler(MediaPlayer mp, Looper looper) {
		mp.super(mp, looper);
	}

	public void handleMessage(Message msg) {
		try {
			super.handleMessage(msg);
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}

{% endhighlight java %}

注意：mp.super(mp, looper); 是用宿主类的实例去调用super来构造实例，写法比较特殊

#### 三、替换MediaPlayer的mEventHandler
这里替换逻辑很简单，反射赋值即可。  
一切看似很合理，但是一运行，在load class SafeEventHandler时直接抛出异常：superclass not accessible，字面意思就是父类无法访问。  
第一反应是：API欺骗有问题了？于是写了简单的demo，在电脑上直接运行，一切正常。因此可以判断是DVM对Class Loader过程做了手脚。
于是全局grep了这个异常，发现异常抛出代码是：
[http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/Class.cpp#2677](http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/Class.cpp#2677)  

{% highlight java %}
		} else if (!dvmCheckClassAccess(clazz, clazz->super)) {
            ALOGW("Superclass of '%s' (%s) is not accessible",
                clazz->descriptor, clazz->super->descriptor);
            dvmThrowIllegalAccessError("superclass not accessible");
            goto bail;
        }
{% endhighlight java %}
意思就是dvmCheckClassAccess验证失败

[http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/AccessCheck.cpp#125](http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/AccessCheck.cpp#125)    

{% highlight java %}
bool dvmCheckClassAccess(const ClassObject* accessFrom,
    const ClassObject* clazz)
{
    if (dvmIsPublicClass(clazz))
        return true;
    return dvmInSamePackage(accessFrom, clazz);
}
{% endhighlight java %}
dvmIsPublicClass函数在[http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/Object.h#734](http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/Object.h#734)中定义。  
首先会判断父类是否是public的，显然这里不是，因此进入下一个判断条件。
[http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/AccessCheck.cpp#39](http://androidxref.com/4.4.2_r2/xref/dalvik/vm/oo/AccessCheck.cpp#39)

{% highlight java %}
/* quick test for intra-class access */
if (class1 == class2)
    return true;

/* class loaders must match */
if (class1->classLoader != class2->classLoader)
    return false;
{% endhighlight java %}
首先判断是否同一个类，显然这里不是，再判断是否是相同的ClassLoader，这里有待确认。
因此直接打印出MediaPlayer.EventHandler的ClassLoader和SafeEventHandler的ClassLoader比较一下，发现，果然是因为ClassLoader不同导致的。
进一步分析发现，在Java中默认的缺省ClassLoader是java.lang.BootClassLoader，系统的库，比如Android的Framework的class都是缺省的ClassLoader，但是Android工程中自己写的Class，也就是最终放入Dex中的Class都是通过dalvik.system.PathClassLoader来loader的。因此当我们尝试去继承系统的private类的时候，在class load检查中会失败。

#### 解决方案
既然已经找到失败原因了，解决起来就有了方向。
第一次尝试使用同一个ClassLoader去load两个类，但是失败了。因为两个MediaPlayer并不是在dex中，因此无法通过PathClassLoader去load。  
那么是否可以修改MediaPlayer.EventHandler的ClassLoader和SafeEventHandler保持一致呢。
于是通过源码发现Dalvik的Class实现和ART的Class实现有所不同：  
**ART**  
[http://androidxref.com/4.4.2_r2/xref/libcore/libart/src/main/java/java/lang/Class.java](http://androidxref.com/4.4.2_r2/xref/libcore/libart/src/main/java/java/lang/Class.java)

**Dalvik**   
[http://androidxref.com/4.4.2_r2/xref/libcore/libdvm/src/main/java/java/lang/Class.java](http://androidxref.com/4.4.2_r2/xref/libcore/libdvm/src/main/java/java/lang/Class.java)

ART中classloader就是一个成员变量，因此可以通过反射去修改，但是Dalvik中的classLoader是一个native方式，因此无法修改。

至于一次不完美的系统类class注入就完成了。
