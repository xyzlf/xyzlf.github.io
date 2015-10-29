---
layout: post
title: Android中Touch事件分发过程全解析
date: 2015-10-29 11:33:13
categories: [Android]
tags: [UI]
---
## 分发过程
     首先事件从native分发，会传递给 ViewRootImpl.WindowInputEventReceiver.onInputEvent ，然后会经过ViewRootImpl的分发到DecorView，DecorView会根据之前给Window设置的CallBack（具体可以参考Activity的启动过程系列文章），也就是Activity，会调用Activity的dispatchTouchEvent，然后Activity会调用DecorView的superDispatchTouchEvent，从而开始到ViewHierarchy的事件分发。
     ViewHierarchy中事件分发，简单来说就是ViewGroup.dispatchTouchEvent->ViewGroup.onInterceptTouchEvent->View.dispatchTouchEvent->View.OnTouchListener.onTouch->View.onTouchEvent，具体过程如下：
  
<!--more-->    

## Down事件
     在 ViewGroup.dispatchTouchEvent 方法中。
     首先会判断是否拦截该事件，如果有调用过ViewGroup.requestDisallowInterceptTouchEvent，且参数为true，那么就不会去拦截，也就是不会调用ViewGroup.onInterceptTouchEvent，如果参数为false或者没有调用(默认是允许拦截)，那么会调用ViewGroup.onInterceptTouchEvent，同时根据这个方法的返回值来判断是否拦截该down事件。
     如果ViewGroup拦截了该事件，那么就会把该ViewGroup当做是个普通的view，然后调用并返回父类 View.dispatchTouchEvent（后续就是 onTouchEvent ），如果返回值为true，那么该ViewGroup成为事件消费者。
     如果ViewGroup不拦截该事件，那么就遍历子view，根据drawingOrder从最上层的View开始，判断事件坐标点是否在view的显示区域内，如果在，那么会调用View.dispatchTouchEvent方法（这里View.dispatchTouchEvent方法简单来说，首先会调用OnTouchListener的onTouch方法，如果消费了，则直接放回，如果不消费，则调用并返回View.onTouchEvent方法），根据View.dispatchTouchEvent的返回值来判断该View是否消费该事件，如果消费，那么将该View保存在ViewGroup的TouchTarget链表中，以后所有事件都会由该View来处理。
     如果没有子View来处理该事件，同样会把ViewGroup当做一个普通的view，调用并返回它的父类 View.dispatchTouchEvent（后续就是 onTouchEvent ），如果返回true，那么该ViewGroup就是事件消费者，后续的事件都直接传递给ViewGroup。
     如果所有View或ViewGroup都不消费down事件，那么因为Activity的最上层的ViewGroup是DecorView(ViewRootImpl不是ViewGroup)，所以后续事件都只会传递给DecorView，而DecorView本身不处理事件，因此最终都会分发给Activity，Activity会成为最终的事件消费者，也就是会调用Activity的onTouchEvent。

## Down以后的其他事件
     在 ViewGroup.dispatchTouchEvent 方法中。
     如果有TouchTarget，首先会判断是否拦截该事件，如果有调用过ViewGroup.requestDisallowInterceptTouchEvent，且参数为true，那么就不会去拦截，也就是不会调用ViewGroup.onInterceptTouchEvent。如果参数为false或者没有调用(默认是允许拦截)，那么会调用ViewGroup.onInterceptTouchEvent，如果ViewGroup.onInterceptTouchEvent返回false，则正常的执行事件分发，如果 ViewGroup.onInterceptTouchEvent 返回true拦截了该事件，那么之前的TouchTarget会收到一个ACTION_CANCEL事件，并从TouchTarget链表中移除掉，同时 ViewGroup.mFirstTouchTarget 置为 null，该ViewGroup成为了事件消费者，后续事件都传给该ViewGroup。
     如果没有TouchTarget，直接调用并返回父类View.dispatchTouchEvent（后续就是 onTouchEvent ）。

## 总结

- 谁消费 ACTION_DOWN 事件（无论是View还是ViewGroup），后续所有事件都传给它;
- 都不消费 ACTION_DOWN ，所有事件传给Activity（准确说是DecorView，DecorView默认分发给Activity）。
- 子View或ViewGroup消费事件，父ViewGroup可以在onInterceptTouchEvent中拦截该事件，如果事件消费者请求不允许拦截事件（ViewGroup.requestDisallowInterceptTouchEvent），那么将不会被拦截，也既不会调用onInterceptTouchEvent。如果onInterceptTouchEvent拦截了该事件，那么ViewGroup成为事件消费者，后续事件都分发给ViewGroup。