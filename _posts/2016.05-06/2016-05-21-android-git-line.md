---
layout: post
title: Git常用命令行
date: 2016-05-21 16:00:00
categories: [Android]
tags: [Android]
---

记录一些常用的Git命令行~
<!--more-->

## Git 操作

### (1)查看本地标签。
{% highlight java %}
git tag
{% endhighlight java %}


### (2)打标签。
{% highlight java %}	
git tag xxx(标签名)

eg:
	git tag V1.0.0

{% endhighlight java %}


### (3)推送本地标签。
{% highlight java %}
git push origin tag名

eg:
	git push origin V1.0.0

{% endhighlight java %}


### (4)删除标签。
{% highlight java %}

//删除本地标签
git tag -d tag名

eg:
	git tag -d V1.0.0

//删除远程标签
git push origin :refs/tags/标签名 

eg:
	git push origin :refs/tags/V1.0.0

{% endhighlight java %}


## Git 相关资料

Pro Git（中文版）： <http://git.oschina.net/progit/>

Git from the inside out：<https://maryrosecook.com/blog/post/git-from-the-inside-out>