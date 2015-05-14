---
layout: post
title: Android动画执行过程源码分析
date: 2015-05-06 10:30:00
category: Android
tags: Android UI
---
### 以下是Android Animation的动画的完整流程。源码参考的是Android5.0。  
1. View.startAnimation
{% highlight java %}
public void startAnimation(Animation animation) {
    animation.setStartTime(Animation.START_ON_FIRST_FRAME);
    setAnimation(animation);
    invalidateParentCaches();
    invalidate(true);
}
{% endhighlight java %}
setAnimation 将mCurrentAnimation设置为当前要执行的animation
invalidateParentCaches将当前view的mParent的flag设为 PFLAG_INVALIDATED
然后调用invalidate，同时参数为true，使当前view的缓存失效。  
<!--more-->
2. View.invalidate 
{% highlight java %}
// Propagate the damage rectangle to the parent view.
final AttachInfo ai = mAttachInfo;
final ViewParent p = mParent;
if (p != null && ai != null && l < r && t < b) {
    final Rect damage = ai.mTmpInvalRect;
    damage.set(l, t, r, b);
    p.invalidateChild(this, damage);
}
{% endhighlight java %}
这里会将失效的矩阵传递给ViewParent，ViewParent的直接子类有ViewGroup和ViewRootImpl，通常情况下，view的直接parent是ViewGroup，所以接下来会进入ViewGroup.invalidateChild。  
3. ViewGroup.invalidateChild
在这个方法中
{% highlight java %}
final boolean drawAnimation = (child.mPrivateFlags & PFLAG_DRAW_ANIMATION) == PFLAG_DRAW_ANIMATION;
do {
    View view = null;
    if (parent instanceof View) {
        view = (View) parent;
    }
    ...
    if (drawAnimation) {
        if (view != null) {
            view.mPrivateFlags |= PFLAG_DRAW_ANIMATION;
        } else if (parent instanceof ViewRootImpl) {
            ((ViewRootImpl) parent).mIsAnimating = true;
        }
    }
    // ...
    parent = parent.invalidateChildInParent(location, dirty);
} while (parent != null);
{% endhighlight java %}
这里会循环调用parent的invalidateChildInParent方法，在ViewGroup.invalidateChildInParent中，会返回它的mParent，但是最下层的ViewRootImpl.invalidateChildInParent方法返回null，因为它没有父view，所以循环终止。drawAnimation在第一次判断时，并不为true，因为这个PFLAG_DRAW_ANIMATION flag只有当执行了一次绘制后才会被设置，参见后面View.drawAnimation。当动画执行过程中的invalidateChildParent时，drawAnimation会为true，这时下面的while循环就会给ViewGroup设置PFLAG_DRAW_ANIMATION flag，给ViewRootImpl.mIsAnimating设置为true。  
4. ViewRootImpl.invalidateChildInParent
{% highlight java %}
final boolean intersected = localDirty.intersect(0, 0,
        (int) (mWidth * appScale + 0.5f), (int) (mHeight * appScale + 0.5f));
if (!intersected) {
    localDirty.setEmpty();
}
if (!mWillDrawSoon && (intersected || mIsAnimating)) {
    scheduleTraversals();
}
{% endhighlight java %}
intersected 这个变量，顾名思义是指矩阵是否相交，用来判断需要失效的矩阵是否和当前ViewRootImpl的矩阵相交，如果相交的话就执行下面的scheduleTraversals.同时如果是在动画执行过程中，mIsAnimating变量也为true，也能走到scheduleTraversals这一步。  
5. ViewRootImpl.scheduleTraversals
{% highlight java %}
mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
{% endhighlight java %}
这里主要就是调用了Choreographer的postCallBack，mTraversalRunnable 里面就是调用了View measure，layout，draw的非常经典的performTraversals方法。然后我们再看看Choreographer这个类.  
6. Choreographer.postCallback
{% highlight java %}
mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);
if (dueTime <= now) {
    scheduleFrameLocked(now);
} else {
    Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
    msg.arg1 = callbackType;
    msg.setAsynchronous(true);
    mHandler.sendMessageAtTime(msg, dueTime);
}
{% endhighlight java %}
addCallbackLocked这个方法会将当前Runnable按执行时间先后进行排队（所以如果在UI线程中最很多事情的话，肯定会导致动画的掉帧）.  
7. Choreographer.scheduleFrameLocked  
{% highlight java %}
if (USE_VSYNC) {
    if (DEBUG) {
        Log.d(TAG, "Scheduling next frame on vsync.");
    }
    // If running on the Looper thread, then schedule the vsync immediately,
    // otherwise post a message to schedule the vsync from the UI thread
    // as soon as possible.
    if (isRunningOnLooperThreadLocked()) {
        scheduleVsyncLocked();
    } else {
        Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_VSYNC);
        msg.setAsynchronous(true);
        mHandler.sendMessageAtFrontOfQueue(msg);
    }
} else {
    final long nextFrameTime = Math.max(
            mLastFrameTimeNanos / TimeUtils.NANOS_PER_MS + sFrameDelay, now);
    if (DEBUG) {
        Log.d(TAG, "Scheduling next frame in " + (nextFrameTime - now) + " ms.");
    }
    Message msg = mHandler.obtainMessage(MSG_DO_FRAME);
    msg.setAsynchronous(true);
    mHandler.sendMessageAtTime(msg, nextFrameTime);
}
{% endhighlight java %}
USE_VSYNC，判断这个VSync（VSync是Android 4.1以后引入的新的view绘制技术，可以参见http://blog.chinaunix.net/uid-26669815-id-3272173.html）是否打开，如果打开，走VSync的定时刷新逻辑，VSync刷新时间由VSync控制，不受Choreographer.sFrameDelay（默认为10ms，也就是100FPS）延迟控制，如果VSync没有打开走下面的延迟刷新逻辑，会判断上次刷新绘制帧+sFrameDelay的时间和当前请求绘制的时间哪个大，选择大的最后下一帧的刷新时间，但是由于使用Handler处理的，所以不一定会在规定时间点开始执行。不管哪种方式最终都会走到Choreographer.doFrame方法。  
8. Choreographer.doFrame  
{% highlight java %}
mFrameScheduled = false;
mLastFrameTimeNanos = frameTimeNanos;
doCallbacks(Choreographer.CALLBACK_INPUT, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_ANIMATION, frameTimeNanos);
doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
{% endhighlight java %}
将最后一次绘制帧的时间设置为当前的时间，精确到纳秒，同时调用callback.run方法。而之前CALLBACK_TRAVERSAL对应的Runnable是TraversalRunnable，其run方法调用的是ViewRootImpl.doTraversal.  
9. ViewRootImpl.doTraversal  
这个方法很简单，之前有介绍，就是调用了ViewRootImpl.performTraversals  
10. ViewRootImpl.performTraversals  
这个方法就不细说了，重点不在这，这里会进行绘制的分发，ViewRootImpl.performDraw->ViewRootImpl.draw->ViewGroup.dispatchDraw->ViewGroup.drawChild->View.draw（注意是这个：draw(Canvas canvas, ViewGroup parent, long drawingTime)）。  
11. ViewGroup.dispatchDraw  
值得注意的是ViewGroup.drawChild和View.draw这两个方法都有boolean返回值，这个返回值是用来标志是否需要继续执行invalidate方法，当动画没有结束，那么肯定返回true。而在方法最后，会判断一个标志位是否被置，如果置了将继续调用invalidate方法。
{% highlight java %}
if ((flags & FLAG_INVALIDATE_REQUIRED) == FLAG_INVALIDATE_REQUIRED) {
    invalidate(true);
}
{% endhighlight java %}
接下来再看的View的draw方法。  
12. View.draw(Canvas canvas, ViewGroup parent, long drawingTime)  
这里没有太多，主要看View.drawAnimation方法。  
13. View.drawAnimation  
这里会调用Animation.getTransformation方法，这个方法处理Animation的一些生命周期，比如Animation的start,end,repeat和applyTransformation，同时有个返回值，如果动画执行结束，返回false，没有结束就返回true。如果是true的话，就将修改parent的一些flag，比如PFLAG_DRAW_ANIMATION，参见第3步。
{% highlight java %}
if (!a.willChangeBounds()) {
    if ((flags & (ViewGroup.FLAG_OPTIMIZE_INVALIDATE | ViewGroup.FLAG_ANIMATION_DONE)) == ViewGroup.FLAG_OPTIMIZE_INVALIDATE) {
        parent.mGroupFlags |= ViewGroup.FLAG_INVALIDATE_REQUIRED;
    } else if ((flags & ViewGroup.FLAG_INVALIDATE_REQUIRED) == 0) {
        // The child need to draw an animation, potentially offscreen, so
        // make sure we do not cancel invalidate requests
        parent.mPrivateFlags |= PFLAG_DRAW_ANIMATION;
        parent.invalidate(mLeft, mTop, mRight, mBottom);
    }
} else {
    if (parent.mInvalidateRegion == null) {
        parent.mInvalidateRegion = new RectF();
    }
    final RectF region = parent.mInvalidateRegion;
    a.getInvalidateRegion(0, 0, mRight - mLeft, mBottom - mTop, region,
            invalidationTransform);
    // The child need to draw an animation, potentially offscreen, so
    // make sure we do not cancel invalidate requests
    parent.mPrivateFlags |= PFLAG_DRAW_ANIMATION;
    final int left = mLeft + (int) region.left;
    final int top = mTop + (int) region.top;
    parent.invalidate(left, top, left + (int) (region.width() + .5f),
            top + (int) (region.height() + .5f));
}
{% endhighlight java %}  
这里有几个分支，如果进入第一个分支，执行parent.mGroupFlags &#124;= ViewGroup.FLAG_INVALIDATE_REQUIRED，这个flag，会在ViewGroup.dispatchDraw方法最后有判断，如果有这个标志，继续执行View.invalidate，参见第11步。而如果进了其他几个分支的话，显而易见，直接调用了parent.invalidate方法，也就继续进入了第2步。所以只要动画没结束，都会继续调用invalidate，直到动画结束。  

#####  附1：dispatchDraw流程  

![dispatchDraw流程](http://img.blog.csdn.net/20150510155714385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWluemhvbmczOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)  

#####  附2：如果调用startAnimation，那么只有当该View和屏幕区域有交集，且可见时才会执行动画。否则不会执行，除非定时调用invalidate的方式。  
&nbsp;&nbsp;&nbsp;&nbsp;同时根据以上的源码分析，可以解释一些Animation上的一些诡异情况：  

*   通过WindowManager.addView方式添加的View是否可以直接执行（是指直接parent是ViewRootImpl的情况）基本动画（外层没有ViewGroup包裹）？  
答案是无法执行，因为外层没有ViewGroup，无法走正常的Animation流程。在ViewRootImpl.draw过程中无法通过ViewGroup.dispatchDraw走View.drawAnimation过程。  
*   为什么有的View调用了startAnimation动画不能执行？  
因为有可能这个view的矩阵和ViewRootImpl的矩阵没有交集，比如View不在屏幕内，在进行第3步时，得到的矩阵交集为0。而第4步则会失败。  