---
layout: post
title: Android多语言及字体大小的设置
date: 2017-03-08 14:00:00
categories: [Android]
tags: [Android]
---

之前做国际化App的时候，App内部一般会要求有个多语言设置功能，做国内App，很少遇到添加这个功能。对于多语言这个功能，个人觉得价值不大，但对于测试来说，可以方便测试多语言适配。之前做过这个功能，整理整理。

在做App开发过程中，有一次测试过来说，系统字体的大小更换，会对App造成很大的适配问题，为了一劳永逸，采用了一个比较trick的方法，也分享下，希望能给有需要的人一些帮助。
<!--more-->


##  App多语言更换

- 效果图

<img src="/assets/drawable/language_1.png"  alt="pic" />
<img src="/assets/drawable/language_2.png"  alt="pic" />

- 项目地址

Github地址：<https://github.com/xyzlf/MultiLanguage>

## App字体不受系统字体大小影响

主要是在App的Application中，对字体大小进行限制，这样App内部的字体大小就不会受系统大小的更换而影响了。虽然一劳永逸，但对于需要随着系统字体大小而变化的应用，就不能这样做了，做页面时就得一点点适配了。直接贴代码吧：

{% highlight java %}

	/**
	 * Created by zhanglifeng on 2017/3/8.
	 * Language Application
	 */
	public class LanguageApplication extends Application {

	    @Override
	    public void onCreate() {
	        super.onCreate();
	    }
	
	    @Override
	    protected void attachBaseContext(Context base) {
	        super.attachBaseContext(base);
	        keepFontSize(base);
	
	    }
	
	    private void keepFontSize(Context base) {
	        // 取消系统设置文字大小时对App的影响
	        Resources resources = base.getResources();
	        if (resources != null) {
	            Configuration configuration = resources.getConfiguration();
	            configuration.fontScale = 1f;
				//configuration.setToDefaults();
	            resources.updateConfiguration(configuration, resources.getDisplayMetrics());
	        }
	    }
	
	    @Override
	    public void onConfigurationChanged(Configuration newConfig) {
	        super.onConfigurationChanged(newConfig);
	        keepFontSize(this);
	    }

	}
{% endhighlight java %}