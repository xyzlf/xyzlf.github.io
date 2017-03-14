---
layout: post
title: Android性能优化之App进程启动
date: 2017-03-14 15:38:00
categories: [Android]
tags: [Android]
---

App启动速度是App性能的一个比较关键指标，App启动速度越快，给用户带来的体验更好。最近看到了蛮多优秀的关于优化App启动速度的文章，整理记录下这些优秀的思路。
<!--more-->

首先看下启动的状态，从官方文档介绍 [《Launch-Time Performance》][1] 主要分为冷启动，温启动，热启动。

- Cold start


		A cold start refers to an app’s starting from scratch: the system’s process has not, until this start, created the app’s process. Cold starts happen in cases such as your app’s being launched for the first time since the device booted, or since the system killed the app. This type of start presents the greatest challenge in terms of minimizing startup time, because the system and app have more work to do than in the other launch states.

		At the beginning of a cold start, the system has three tasks. These tasks are:
		
		Loading and launching the app.
		Displaying a blank starting window for the app immediately after launch.
		Creating the app process.
		
		As soon as the system creates the app process, the app process is responsible for the next stages. These stages are:
		
		Creating the app object.
		Launching the main thread.
		Creating the main activity.
		Inflating views.
		Laying out the screen.
		Performing the initial draw.
		Once the app process has completed the first draw, the system process swaps out the currently displayed background window, replacing it with the main activity. At this point, the user can start using the app.

- Warm start

		A warm start of your application is much simpler and lower-overhead than a cold start. In a warm start, all the system does is bring your activity to the foreground. If all of your application’s activities are still resident in memory, then the app can avoid having to repeat object initialization, layout inflation, and rendering.
	
		However, if some memory has been purged in response to memory trimming events, such as onTrimMemory(), then those objects will need to be recreated in response to the warm start event.
	
		A warm start displays the same on-screen behavior as a cold start scenario: The system process displays a blank screen until the app has finished rendering the activity.

- Lukewarm start

		A lukewarm start encompasses some subset of the operations that take place during a cold start; at the same time, it represents less overhead than a warm start. There are many potential states that could be considered lukewarm starts. For instance:
		（1）The user backs out of your app, but then re-launches it. The process may have continued to run, but the app must recreate the activity from scratch via a call to onCreate().
		（2）The system evicts your app from memory, and then the user re-launches it. The process and the Activity need to be restarted, but the task can benefit somewhat from the saved instance state bundle passed into onCreate().


## 参考资料

[1]: https://developer.android.com/topic/performance/launch-time.html

Android App启动流程：<http://cheelok.com/aosp/54/>

Android性能优化（一）之启动加速35%：<http://www.jianshu.com/p/f5514b1a826c>

Android 你应该知道的的应用冷启动过程分析和优化方案：<http://blog.csdn.net/growing_tree/article/details/53183511>

Activity到底是什么时候显示到屏幕上的呢？：<http://blog.desmondyao.com/android-show-time/?from=timeline&isappinstalled=0>

App冷启动你要我怎样：<http://www.jianshu.com/p/84983a3bdbff>