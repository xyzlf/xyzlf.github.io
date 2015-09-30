---
layout: post
title: 1px图片的Crash
date: 2015-09-30 12:29:19
categories: [Android]
tags: [Crash]
---
###问题描述
最近在项目中遇到一个奇葩的Crash，在创建bitmap时Crash，log如下：
{% highlight java %}
java.lang.IllegalArgumentException: width and height must be > 0
  at android.graphics.Bitmap.nativeCreate(Native Method)
  at android.graphics.Bitmap.createBitmap(Bitmap.java:477)
  at android.graphics.Bitmap.createBitmap(Bitmap.java:444)
  at android.graphics.Bitmap.createScaledBitmap(Bitmap.java:349)
  at android.graphics.BitmapFactory.finishDecode(BitmapFactory.java:614)
  at android.graphics.BitmapFactory.decodeStream(BitmapFactory.java:589)
  at android.graphics.BitmapFactory.decodeResourceStream(BitmapFactory.java:439)
  at android.graphics.drawable.Drawable.createFromResourceStream(Drawable.java:697)
  at android.content.res.Resources.loadDrawable(Resources.java:1709)
  at android.content.res.TypedArray.getDrawable(TypedArray.java:601)
  at android.view.View.<init>(View.java:1952)
  at android.widget.ImageView.<init>(ImageView.java:112)
  at android.widget.ImageView.<init>(ImageView.java:108)

{% endhighlight java %}
<!--more-->
一开始以为是OOM导致的Bitmap创建失败，后来查看了下当前的内存状态，并非OOM导致的，同时发现Crash全部分布在API <= 19的设备中。于是看了下源码，发现是因为资源图片导致的。

###问题原因  

##### 首先看Crash代码  
[http://androidxref.com/4.0.4/xref/frameworks/base/graphics/java/android/graphics/Bitmap.java#601](http://androidxref.com/4.0.4/xref/frameworks/base/graphics/java/android/graphics/Bitmap.java#601)  
Crash是因为bitmap的宽或者高为0了。  

##### 再看出问题的代码  
[http://androidxref.com/4.0.4/xref/frameworks/base/graphics/java/android/graphics/BitmapFactory.java#502](http://androidxref.com/4.0.4/xref/frameworks/base/graphics/java/android/graphics/BitmapFactory.java#502)
{% highlight java %}
float scale = targetDensity / (float)density;
// TODO: This is very inefficient and should be done in native by Skia
final Bitmap oldBitmap = bm;
bm = Bitmap.createScaledBitmap(oldBitmap, (int) (bm.getWidth() * scale + 0.5f), (int) (bm.getHeight() * scale + 0.5f), true);
{% endhighlight java %}
以上是在创建bitmap后根据当前设备的density对bitmap进行缩放的代码，同时只会当inTargetDensity和inDensity不一致时才执行，其中inTargetDensity也就是当前手机的density，而inDensity就是Bitmap的density。  
举例说：
假如图片放在xhdpi下，那么bitmap的density就是320，xxhdpi，bitmap的density就是480。而如果当程序运行在density为120的手机上时，如果没有ldpi的图片，那么就会去取最接近的，假如这时取xhdpi的图片，那么inTargetDensity=120, inDensity=320, 这时
{% highlight java %}
float scale = 120 / 320.0f;
int width = (int) (1 * 120 / 320.0f + 0.5f) = 0;
{% endhighlight java %}
可以查看width计算结果为0，从而抛出异常。 

以上只是api<=19的情况，在api19及以上，createBitmap以及缩放的代码全部放在了native中进行，但是当遇到上面的case时，虽然不会crash，但是得到的bitmap size为0，在绘制的时候显示为一片黑色。

###解决方式
很简单，不使用1px的图片就行，同理，不仅在xhdpi中会出问题，在xxhdpi中也会出问题，只要上面的计算得到0都会crash.
因此有条简单的规则就是xhpdi和xxhdpi下都不能使用<=1px的图片，xxxhdpi下不能使用<=2px的图片。