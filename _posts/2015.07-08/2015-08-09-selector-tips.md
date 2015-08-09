---
layout: post
title: Selector的一个坑
date: 2015-08-09 14:00:47
categories: [Android]
tags: [UI]
---
创建selector时，有个小坑，就是默认没有指明任何状态的item必须写在末尾。因为系统会拿第一个item去匹配当前状态，如果没有指明state的item放在第一个的话，会匹配任何state，会造成与实际期望的效果不符合的情况。  
参考：[http://developer.android.com/intl/zh-cn/guide/topics/resources/drawable-resource.html#StateList](http://developer.android.com/intl/zh-cn/guide/topics/resources/drawable-resource.html#StateList)   
 
> Note: Remember that Android applies the first item in the state list that matches the current state of the object. So, if the first item in the list contains none of the state attributes above, then it is applied every time, which is why your default value should always be last (as demonstrated in the following example).
