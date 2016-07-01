---
layout: post
title: 分享SDK系列一：遇到的坑
date: 2016-07-01 13:58:00
categories: [Android]
tags: [Android]
---

这段时间以来，开发了两个分享SDK，遇到了许多小问题。很开心的是，都一一解决了。在这里总结一下这些坑。
<!--more-->


在这里说明一下，我封装的分享SDK，包含微信、微博、QQ、QZone、短信、邮箱、以及系统更多，几个分享渠道。在我开发的App中，有这几个主流的分享渠道就够了。

### AndroidManifest中的配置相关的坑

问题：关于配置分享渠道，将微博的key直接配置在value中，如：
{% highlight java %}
<meta-data
	android:value="42648357859"
	android:name="sina_weibo_key" />
{% endhighlight java %}
取出来的值会自动转化为 sina_weibo_key=4.264835E9 形式，试了几种方式，依然失败，系统会自动转化为长整型。

解决方案：将纯数字的key值保存在string.xml中，然后配置在manifest文件中，如下：
{% highlight java %}
<meta-data
	android:value="@string/weixin_key"
	android:name="weixin_key" />
<meta-data
	android:value="http://www.xxx.com"
	android:name="weixin_redirecturi" />
<meta-data
	android:value="@string/sina_weibo_key"
	android:name="sina_weibo_key" />
<meta-data
	android:value="http://www.xxx.com"
	android:name="sina_weibo_redirecturi" />
{% endhighlight java %}

### 微博分享遇到的坑

(1)问题一：微博分享时，分享的文字不能带有 %s 类似字符，会分享失败。

(2)问题二：分享文案中如果包含URL，并且是如下格式：
{% highlight java %}
http://www.xxx.com?a=xxx&b=xxx&c=xxx
{% endhighlight java %}
该url会被截取成
{% highlight java %}
http://www.xxx.com?a=xxx
{% endhighlight java %}
如果所传的参数是必须的，则会出现跳转异常。

解决方案：将包含的URL进行编码。

如下使用场景遇到的坑：
{% highlight java %}
String[] params = new String[]{
		"status", "分享内容",
		"url", Uri.encode(imgUrl.replace(".webp", "")),
		"access_token", "获取的token"
};

new HttpsRequest(, params) {
	@Override
	protected void onSuccess(String t) throws Exception {
		super.onSuccess(t);
	}

	@Override
	protected void onException(Exception e) {
		super.onException(e);
	 }
}.excute();
{% endhighlight java %}

(3)问题三：微博分享必须是正式签名包，不然会认证失败，打包的时候记得签名包就可以了。

### 微信分享遇到的坑

微信分享API，实在是太简洁，也许微信都是大神级别，不太重视文档相关的工作。相比于QQ的文档，简直差出了几个数量级。网上有人评价说：“实在太懒，能省一行代码，绝对不多写”，甚为贴切。

(1)微信分享必须是签名正式包，并且在微信开放平台已经注册成功，不然认证时会闪退。
(2)如果分享过程用过debug的包，进行过认证分享，安装正式签名包后，需要清除微信的数据，重新启动，否则会出现微信闪退现象。
(3)分享微信过程中，图片不能超过，有些说32kb，有些说64kb，总之在分享过程中需要对图片进行处理，不然分享失败，微信闪退。
我的图片处理方式：
{% highlight java %}
protected Bitmap getWxShareBitmap(Bitmap targetBitmap) {
	float scale = Math.min((float) 150 / targetBitmap.getWidth(), (float) 150 / targetBitmap.getHeight());
	Bitmap fixedBmp = Bitmap.createScaledBitmap(targetBitmap, (int) (scale * targetBitmap.getWidth()), (int) (scale * targetBitmap.getHeight()), false);
	return fixedBmp;
}
{% endhighlight java %}

(4)微信分享，必须注册一个Activity：WXEntryActivity，该WXEntryActivity必须实现该接口：com.tencent.mm.sdk.openapi.IWXAPIEventHandler;
并且WXEntryActivity的包名必须是：appPkgName.wxapi; 如：com.demo.wxapi.WXEntryActivity。
{% highlight java %}
<activity
	android:name=".wxapi.WXEntryActivity"
	android:configChanges="keyboardHidden|orientation|screenSize"
	android:exported="true"
	android:screenOrientation="portrait"
	android:theme="@android:style/Theme.Translucent.NoTitleBar" />
{% endhighlight java %}

微信分享的回调结果，就是通过WXEntryActivity这个Activity回调回来。
{% highlight java %}
@Override
public void onResp(BaseResp resp) {
	//处理回调结果
	finish();
}
{% endhighlight java %}

### QQ分享遇到的坑

QQ分享对key值好像不是特别重要，在之前的公司做分享SDK的时候，给兄弟App用的过程中，因为非该App注册的key也能用，就没太多注意。

加入QZone分享后该问题就暴露了，使用同一个key值，分享回调后会出现选择框。主要原因如下：
{% highlight java %}
<!-- QQ SDK 需要註冊Activity -->
<activity
	android:name="com.tencent.connect.common.AssistActivity"
	android:configChanges="orientation|keyboardHidden"
	android:screenOrientation="behind"
	android:theme="@android:style/Theme.Translucent.NoTitleBar" />

<activity
	android:name="com.tencent.tauth.AuthActivity"
	android:launchMode="singleTask"
	android:noHistory="true" >
	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />
		<data android:scheme="tencent222222" />                           -----------------主要是因为这个scheme，回调时候因为找这个Activity才出现选择框
		<!-- 100380359 100381104 222222 -->
	</intent-filter>
</activity>
<!-- QQ SDK 需要註冊 Activity -->
{% endhighlight java %}

谨记：每个App分享，都独自去开发平台注册相应的账号，这种问题，大部分人应该不会遇到，很不幸，我遇到了。