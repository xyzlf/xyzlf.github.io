---
layout: post
title: ApplicationContext、ActivityContext、ViewContext和ContextImpl(BaseContext)的区别
date: 2015-11-21 12:14:32
categories: [Android]
tags: [Context]
---
### ContextWrapper
- ContextWrapper 是 ContextImpl(也就是BaseContext) 的包装，所有对 ContextWrapper 的操作都会用代理到 BaseContext 上。
- Application和Activity和Service都继承自ContextWrapper
- android.content.ContextWrapper#getApplicationContext 获得Application实例，和Application.this 是同一个对象。因此Activity.getApplicationContext == Application.getApplicationContext == Service.getApplicationContext == Application.this == Activity.getApplication
- android.content.ContextWrapper#getBaseContext  都获得的是ContextImpl对象。
<!--more-->

### Application中BaseContext的创建过程：
- android.app.LoadedApk#makeApplication，首先判断Application实例是否存在，如果存在则直接返回，不存在继续下面步骤。
- 首先调用 android.app.ContextImpl#createAppContext 创建一个ContextImpl实例；
- 调用 android.app.Instrumentation#newApplication 创建一个Application实例；
- 调用 android.app.Application#attach->android.content.ContextWrapper#attachBaseContext 为Application的BaseContext赋值；
- 最后调用 android.app.ContextImpl#setOuterContext 将Application实例赋值给ContextImpl的mOuterContext，这个mOuterContext的作用：
    - startActivity时，如果是ActivityContext的话，就不需要加 FLAG_ACTIVITY_NEW_TASK 标记之类的
    - 创建部分SystemService（如：android.content.Context#NOTIFICATION_SERVICE）时，也会用到mOuterContext。具体参考ContextImpl的static块。
- 因为一个进程中只会有 Application的一个实例，因此Application的BaseContext也只会存在一个。

### Activity的BaseContext的创建过程：
- android.app.ActivityThread#performLaunchActivity
- android.app.Instrumentation#newActivity 创建Activity实例，直接调用Activity的无参构造
- android.app.ActivityThread#createBaseContextForActivity 为Activity创建BaseContext。
- android.app.ContextImpl#createActivityContext 创建ContextImpl实例
- 设置ContextImpl的OuterContext为Activity
- android.app.Activity#attach->android.content.ContextWrapper#attachBaseContext 为Activity的BaseContext赋值；
- 因此Activity的BaseContext每次都是新的。

### Service的BaseContext的创建过程：
参考：android.app.ActivityThread#handleCreateService 和Activity类似，同样每次BaseContext都是新的。

### ViewContext
参考：android.view.LayoutInflater#createViewFromTag
如果ViewParent使用了不同的Theme之类的，就会和ViewParent使用同一个Context，否则使用 LayoutInflater 实例化时的Context，不过一般ViewContext == Activity.this。ViewContext不可能是ContextImpl，因为在ContextImpl对LAYOUT_INFLATER_SERVICE的获得是通过getOuterContext，也就是获得的是Activity，Service或Application。
### 总结：
- android.content.ContextWrapper#getBaseContext  都获得的是ContextImpl对象。
- Activity.getApplicationContext == Application.getApplicationContext == Service.getApplicationContext == Application.this == Activity.getApplication
- Application的BaseContext只会存在一个，而Activity和Service的BaseContext每次都是新的.
- 通常情况下 ViewContext == Activity.this，特殊情况比如：使用LayoutInflater.from(ApplicationContext) inflate的View的ViewContext == Application.this。ViewContext不可能是ContextImpl。