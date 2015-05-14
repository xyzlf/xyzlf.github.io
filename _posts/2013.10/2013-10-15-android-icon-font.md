---
layout: post
title:  Android中使用IconFont代替icon，减小app体积
date: 2013-10-15 16:59:00
category: Android
tags: Android UI
---
&#160; &#160; &#160; &#160;现在网页中使用IconFont已经开始流行，比如新版的淘宝首页，以及36kr等。使用IconFont的好处是可以大量节省资源，同时可以无损缩放，web中减少图片请求。缺点是由于和文字一样，只支持纯色或渐变色，所以颜色上比较单一。
其实在App开发中同样可以使用IconFont，尤其Win8推出后开始刮起的“扁平风”，对于一些icon，完全可以使用纯色的IconFont代替。
<!--more-->
&#160; &#160; &#160; &#160;什么是IconFont?其实通俗点讲就是一个Icon就相当于一个文字，将其存放在字体文件中，然后指定其映射关系，就可以使用了。

&#160; &#160; &#160; &#160;网上制作IconFont的教程很多，下面简单介绍下怎么在Android中使用IconFont。

1.  将制作好的字体放在assets中，通过Typeface获得字体;
2.  然后只需要设置TextView的字体，并将内容设置为要显示的icon对应的值即可。  

##  示例：

{% highlight java %}
 Typeface tf = Typeface.createFromAsset(getAssets(), "fonts/icomoon.ttf”);
 mText = (TextView) findViewById(R.id.content);
 mText.setTypeface(tf);
 mText.setTextColor(Color.parseColor("#FFFF3333"));
 mText.setTextSize(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, 60.0f, getResources().getDisplayMetrics()));
 char[] charArry = { 0xe001, 0xe002, 0xe003, 0xe004 };
 mText.setText(String.valueOf(charArry));
{% endhighlight java %}

##  效果图：

![icon font 效果图](http://img.blog.csdn.net/20131015170030609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWluemhvbmczOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)