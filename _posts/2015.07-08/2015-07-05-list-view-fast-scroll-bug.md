---
layout: post
title: 解决Android5.0 ListView快速滚动后，接近顶部时滚动方向反向的bug
date: 2015-07-06 20:14:15
categories: android
tags: listview
---
issue描述：[https://code.google.com/p/android/issues/detail?id=159739](https://code.google.com/p/android/issues/detail?id=159739)  
原因：ListView内部的一个bug，在接近顶部时，mFirstPosition提前变为0，导致其他根据mFirstPosition来判断的逻辑错误，从而引发滚动事件异常。  
<!--more-->
目前该issue的状态是assigned，并没有fixed。  
曲线救国的方案是：  
```
ViewCompat.setOverScrollMode(lv, ViewCompat.OVER_SCROLL_NEVER); 
```