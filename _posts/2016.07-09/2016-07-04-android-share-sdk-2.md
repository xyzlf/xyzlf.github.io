---
layout: post
title: 分享SDK系列二：SDK封装思路整理
date: 2016-07-04 16:58:00
categories: [Android]
tags: [Android]
---

这段时间以来，因为公司组件化的推动，因此决定将App中的分享模块，抽离出来，封装成SDK。一来，做到代码解耦；二来，公司内部App都可复用。提升开发效率，节省开发成本！
<!--more-->

## 1、SDK封装原则

1、能够快速接入。
2、使用简单。
3、渠道可配。
4、分享数据可扩展。


## 2、分享SDK简介

分享SDK的基本思路是，用Activity实现一个基本的分享界面。分享渠道通过传递的int值，动态展示。分享中接受分享的基本数据结构，或者分享基本数据结构的SparseArray类型。

## 3、分享流程

分享是一个比较简单的过程，一般就是，传递分享需要的数据结构，分享SDK分享，然后回调结果，整个过程就完成了。如下图：

<img src="/assets/drawable/share_flow.png"  alt="pic" />


## 4、分享渠道可配置

根据简单的与或关系，提供一个整形值来控制分享渠道的可配。
{% highlight java %}
	/// -------------------------- 分享渠道 --------------------------
    /**
     * 微信好友分享渠道
     */
    public static final int SHARE_CHANNEL_WEIXIN_FRIEND = 1; //微信好友
    /**
     * 微信朋友圈分享渠道
     */
    public static final int SHARE_CHANNEL_WEIXIN_CIRCLE = 1 << 1; //微信朋友圈
    /**
     * 新浪微博渠道
     */
    public static final int SHARE_CHANNEL_SINA_WEIBO = 1 << 2; //新浪微博
    /**
     * QQ分享渠道
     */
    public static final int SHARE_CHANNEL_QQ = 1 << 3; //QQ
    /**
     * QQ空间渠道
     */
    public static final int SHARE_CHANNEL_QZONE = 1 << 4;//qzone分享
    /**
     * 短信分享渠道
     */
    public static final int SHARE_CHANNEL_SMS = 1 << 5; //短信
    /**
     * 邮箱分享渠道
     */
    public static final int SHARE_CHANNEL_EMAIL = 1 << 6; //邮箱
    /**
     * 更多渠道
     */
    public static final int SHARE_CHANNEL_SYSTEM = 1 << 10; //更多
    /**
     * 所有渠道
     */
    public static final int SHARE_CHANNEL_ALL = ~0;

    /**
     * 默认渠道 微信，朋友圈，微博，QQ，QZONE，More
     */
    public static int SHARE_CHANNEL_DEFAULT = SHARE_CHANNEL_WEIXIN_FRIEND | SHARE_CHANNEL_WEIXIN_CIRCLE | SHARE_CHANNEL_SINA_WEIBO | SHARE_CHANNEL_QQ | SHARE_CHANNEL_QZONE | SHARE_CHANNEL_SYSTEM;

    /// -------------------------- 分享渠道 --------------------------
{% endhighlight java %}

比如传递分享渠道为如下，那么分享界面，里面只会展现微信，朋友圈两个分享渠道。
{% highlight java %}
	int channel = SHARE_CHANNEL_WEIXIN_FRIEND | SHARE_CHANNEL_WEIXIN_CIRCLE;
{% endhighlight java %}

渠道是否展示的使用方式如下：
{% highlight java %}
	/**
     * 初始化分享渠道
     */
    protected void initShareApps() {
        shareAppList = new ArrayList<AppBean>();

        if (AppUtil.isWeixinInstall(this)) {
            if ((channel & ShareChannel.SHARE_CHANNEL_WEIXIN_FRIEND) > 0) {
                shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_WEIXIN_FRIEND, R.drawable.share_weixin, getString(R.string.share_channel_weixin_friend)));
            }

            if ((channel & ShareChannel.SHARE_CHANNEL_WEIXIN_CIRCLE) > 0) {
                shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_WEIXIN_CIRCLE, R.drawable.share_weixin_friends, getString(R.string.share_channel_weixin_circle)));
            }
        }

        if ((channel & ShareChannel.SHARE_CHANNEL_SINA_WEIBO) > 0) {
            shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_SINA_WEIBO, R.drawable.share_sina_weibo, getString(R.string.share_channel_sina_weibo)));
        }

        if (AppUtil.isQQInstall(this)) {
            if ((channel & ShareChannel.SHARE_CHANNEL_QQ) > 0) {
                shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_QQ, R.drawable.share_qq, getString(R.string.share_channel_qq)));
            }

            if ((channel & ShareChannel.SHARE_CHANNEL_QZONE) > 0) {
                shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_QZONE, R.drawable.share_qzone, getString(R.string.share_channel_qzone)));
            }
        }

        if ((channel & ShareChannel.SHARE_CHANNEL_MORE) > 0) {
            shareAppList.add(new AppBean(ShareChannel.SHARE_CHANNEL_MORE, R.drawable.share_more, getString(R.string.share_channel_more)));
        }
    }
{% endhighlight java %}


## 5、分享数据结构

基础数据结构如下描述：

{% highlight java %}
public class ShareEntity implements Parcelable {

    private String title;
    private String content;
    private String url;
    private String imgUrl;

    public ShareEntity(String title, String content) {
        this(title, content, null);
    }

    public ShareEntity(String title, String content, String url) {
        this(title, content, url, null);
    }

    public ShareEntity(String title, String content, String url, String imgUrl) {
        this.title = title;
        this.content = content;
        this.url = url;
        this.imgUrl = imgUrl;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getImgUrl() {
        return imgUrl;
    }

    public void setImgUrl(String imgUrl) {
        this.imgUrl = imgUrl;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    protected ShareEntity(Parcel in) {
        title = in.readString();
        content = in.readString();
        url = in.readString();
        imgUrl = in.readString();
    }

    public static final Creator<ShareEntity> CREATOR = new Creator<ShareEntity>() {
        @Override
        public ShareEntity createFromParcel(Parcel in) {
            return new ShareEntity(in);
        }

        @Override
        public ShareEntity[] newArray(int size) {
            return new ShareEntity[size];
        }
    };

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(title);
        dest.writeString(content);
        dest.writeString(url);
        dest.writeString(imgUrl);
    }
}
{% endhighlight java %}

