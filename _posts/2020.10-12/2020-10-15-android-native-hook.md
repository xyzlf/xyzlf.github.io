---
layout: post
title: Android Native Hook
date: 2020-10-15 19:30:00
categories: [Android]
tags: [Android]
---

Android开发中，为了提高安全性，很多重要的、敏感的信息都会用C/C++开发。C/C++编写的代码，编译成so引入apk文件中，so文件相比于dex文件，有更高的安全性、更优越的性能。

在App使用过程中，也会发现很多由JNI带来的崩溃，这些崩溃无明显的堆栈信息，导致很难定位问题。如果我们可以Hook一些Native方法，然后通过代理原Native方法，截取一些重要信息上报，能帮助我们快速定位一些JNI问题。最近在工作中用到了Native Hook的方案，整理记录一下...
<!--more-->


## Native Hook方式
Android Native Hook的方式，目前有两种
	
	- PLT Hook
	- Inline Hook

因为笔者主要使用了PLT Hook方式，接下来就主要介绍一下PLT Hook的使用。

## PLT Hook
C/C++编译后生成的so文件，是一种ELF(Executable and Linkable Format)格式的文件，需要Hook Native方法，我们需要了解ELF的格式。

<img src="/assets/drawable/android-native-hook-elf.png"  alt="pic" />

主要了解一下EFL的节头表SHT(section_header_table)，ELF详细介绍参考文章：[《深度解密Android中基于plt/got的hook实现原理》][1]。

<img src="/assets/drawable/android-native-hook-sht.png"  alt="pic" />

	.dynsym：为了完成动态链接，最关键的还是所依赖的符号和相关文件的信息。为了表示动态链接这些模块之间的符号导入导出关系，ELF有一个叫做动态符号表(Dynamic Symbol Table)的段用来保存这些信息。
	.rel.dyn：实际上是对数据引用的修正，它所修正的位置位于.got以及数据段。
	.rel.plt：是对函数引用的修正，它所修正的位置位于.got。
	.plt：程序链接表(Procedure Link Table)，外部调用的跳板。
	.text：为代码段，也是反汇编处理的部分，以机器码的形式存储。
	.dynamic：描述了模块动态链接相关的信息。
	.got：全局偏移表(Global Offset Table)，用于记录外部调用的入口地址。
	.data：数据段，保存的那些已经初始化了的全局静态变量和局部静态变量。

需要重点了解的就是PLT，程序链接表(Procedure Link Table)，它是一个外部调用的跳板。 所以PLT Hook的方式，只有其他so，调用被Hook的so的函数时，才会触发hook动作，被Hook的函数在本身so内部调用时不会触发hook操作。

	eg： 
	（1）libtest.so是需要被hook的so，假设hook  void *test(const char *name)函数。
	（2）hook.so 将test函数进行hook，当test函数调用时，触发 test_proxy 函数。
	（3）当libother.so调用test函数时，能触发执行 test_proxy函数；但是当libtest.so内部调用 test函数时，不会触发 test_proxy函数。

PLT Hook的优秀开源工具是爱奇艺开源的 [xhook][2]，该开源项目下面有详细的介绍。

## 示例

cpp文件：http://androidxref.com/9.0.0_r3/xref/art/runtime/thread_list.cc

hook的函数

	void ThreadList::SuspendAll(const char* cause, bool long_suspend)

**步骤一 找到thread_list所在的so**

（1）在网址中<https://cs.android.com/>，直接搜索ThreadList::SuspendAll这个native方法。

（2）然后如下图，找到Android.bp，找到对应的so名称:

<img src="/assets/drawable/android-native-hook-findso.png"  alt="pic" />

（3）在Android手机的目录下，将so pull出来

	adb pull /apex/com.android.runtime/lib/libart.so   
	
**步骤二 查询hook函数的签名信息**
	
通过 readelf -s libart.so，查找 ThreadList::SuspendAll 函数的签名信息。通过这个命令查询，会发现函数签名不全，被截取了。可以下载一个软件辅助查看。cutter下载地址：<https://cutter.re/>，cutter github地址：<https://github.com/radareorg/cutter>

**步骤三 通过爱奇艺xhook工具进行hook**

	static void (*suspend_all_origin)(char const *, bool);
	static void suspend_all_proxy(char const *cause, bool long_suspend = false) {
   	 	LOG("suspend_all _ZN3art16ScopedSuspendAllC1EPKcb cause. %s %d\n", cause, long_suspend);
   	 	suspend_all_origin(cause, long_suspend);
	}
		
		
	if (0 != xhook_register(".*\\.so$", "_ZN3art10ThreadList10SuspendAllEPKcb",
	                        (void *) suspend_all_proxy, (void **) suspend_all_origin)) {
	    LOG("hooooook SuspendAll errror\n");
	} else {
	    LOG("hooooook SuspendAll success\n");
	}

	    
 以上三步，就完成了native hook的操作。
 
 
**完整Demo地址**
NativeHookDemo： <https://github.com/xyzlf/NativeHookDemo>

## 参考资料

深度解密Android中基于plt/got的hook实现原理:<https://cloud.tencent.com/developer/article/1601496>

Android Native Hook: <https://juejin.im/post/6844903698435407879>

Android Native Hook技术路线概述: <https://gtoad.github.io/2018/07/05/Android-Native-Hook/>

爱奇艺xhook开源地址: <https://github.com/iqiyi/xHook>

readelf elf文件格式分析: <https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/readelf.html>


[1]: https://cloud.tencent.com/developer/article/1601496
[2]: https://github.com/iqiyi/xHook