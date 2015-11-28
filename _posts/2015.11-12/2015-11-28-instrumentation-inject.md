---
layout: post
title: 从淘宝广告SDK看Instrumentation注入
date: 2015-11-28 17:28:12
categories: [Android]
tags: [Instrumentation, inject]  
---  
#### 用途
监听Activity的创建，生命周期等  
<!--more-->
淘宝广告sdk：[http://mu.tanx.com/m/sdk/doc.htm?type=mu&platform=android](http://mu.tanx.com/m/sdk/doc.htm?type=mu&platform=android)

#### 初始化代码
下面是淘宝广告的初始化代码

    private void setupAlimama(ViewGroup nat, String slotId) {
        MmuSDK mmuSDK = MmuSDKFactory.getMmuSDK();
        mmuSDK.init(getApplication());//初始化SDK,该方法必须保证在集成代码前调用，可移到程序入口处调用
        properties = new BannerProperties(slotId, nat);
        mController = (BannerController) properties.getMmuController();
        mmuSDK.attach(properties);
    }

MmuSDKFactory.getMmuSDK();// 获得单例的实例
mmuSDK.init(getApplication());// 同步初始化（initAsync 异步初始化）
GodModeHacks.inject(application.getBaseContext());

// 设置Instrumentation
Instrumentation instrumentation = (Instrumentation)ActivityThread_mInstrumentation.on(activityThread).get();
InstrumentationHook instrumentationHook = new InstrumentationHook(instrumentation, context);
ActivityThread_mInstrumentation.on(activityThread).set(instrumentationHook);

#### 原理
原理很简单，反射设置ActivityThread的mInstrumentation即可。

#### 示例

    public static void inject() {
        try {
            Class<?> ActivityThread = Class.forName("android.app.ActivityThread");
            Object sCurrentActivityThreadObj;
            if (Build.VERSION.SDK_INT < 11) {
                Method currentActivityThread = ActivityThread.getDeclaredMethod("currentActivityThread", null);
                currentActivityThread.setAccessible(true);
                sCurrentActivityThreadObj = currentActivityThread.invoke(null);
            } else {
                Field sCurrentActivityThread = ActivityThread.getDeclaredField("sCurrentActivityThread");
                sCurrentActivityThread.setAccessible(true);
                sCurrentActivityThreadObj = sCurrentActivityThread.get(null);
            }
            Field mInstrumentation = ActivityThread.getDeclaredField("mInstrumentation");
            mInstrumentation.setAccessible(true);
            mInstrumentation.set(sCurrentActivityThreadObj, new InstrumentationHook());
            Log.d("SS", "BaseApplication.injectInstrumentation: ok");
        } catch (Throwable e) {
            e.printStackTrace();
            Log.d("SS", "BaseApplication.injectInstrumentation: exception = " + Log.getStackTraceString(e));
        }
    }