---
layout: post
title: Android Studio中忽略目录不显示在Project文件列表中
date: 2015-05-19 12:01:39
category: Android
tags: AndroidStudio
---
&nbsp;&nbsp;&nbsp;&nbsp;在Android Studio中，有时一个Projcet下Module比较多，可能有些不需要显示，这时需要隐藏掉一些Module不显示。<!--more-->但是Studio默认没有提供可视化的操作，但是可以通过修改iml文件来做到：
方法：编辑Porject根目录下的xxx.iml文件，加入以下排除列表：  

{% highlight java %}
<content url="file://$MODULE_DIR$">
    <excludeFolder url="file://$MODULE_DIR$/.gradle" />
    <excludeFolder url="file://$MODULE_DIR$/.idea" />
    <excludeFolder url="file://$MODULE_DIR$/tools" />
</content>  
{% endhighlight java %}
