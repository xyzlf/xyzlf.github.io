---
layout: post
title: Android library上传到jcenter
date: 2016-07-10 20:44:00
categories: [Android]
tags: [Android]
---

在github上发布项目后，可以整理发布到jcenter，这样可以方便开发者使用。今天实战练习了一遍，主要参考文章<http://blog.csdn.net/tu_bingbing/article/details/49208873>，非常感谢这个作者的分享。 在练习过程中，还是遇到了一些问题，在这里记录一下..
<!--more-->

## 项目地址
Github项目地址：<https://github.com/xyzlf/ShareSDK>

## 注册用户名
首先要注册一个账户：<https://bintray.com>，我直接用github账户登录的，然后在里面设置了一个用户名。

## 项目aar上传jcenter配置
build.gradle中的配置
    
    buildscript {
	    repositories {
	    	jcenter()
    	}
    	dependencies {
		    //....
		    classpath 'com.novoda:bintray-release:0.3.4'
	    }
    }
    
    apply plugin: 'com.novoda.bintray-release'//添加
    
	//推送指令
	//gradlew clean build bintrayUpload -PbintrayUser=bintrayUser -PbintrayKey=bintrayKey -PdryRun=false
    //添加
    publish {
	    userOrg = 'xyzlf'//bintray.com用户名
	    groupId = 'com.xyzlf.share'//jcenter上的路径
	    artifactId = 'sharesdk'//项目名称
	    publishVersion = '0.0.1'//版本号
	    desc = 'Oh hi, this is a nice description for a project, right?'
	    website = 'https://github.com/xyzlf/ShareSDK'
    }

由于publish需要用户名密码，为了方便上传到bintray.com，不需要每次写用户名，apikey，也可以将这些基本信息配置在本地local.properties。

	buildscript {
	    repositories {
	        jcenter()
	    }
	    dependencies {
	        //....
	        classpath 'com.novoda:bintray-release:0.3.4'
	    }
	}
	
	apply plugin: 'com.novoda.bintray-release'//添加

    Properties properties = new Properties()
	properties.load(project.rootProject.file('local.properties').newDataInputStream())

	String localBintrayUser = properties.getProperty("bintray.user")
	String localBintrayApikey = properties.getProperty("bintray.apikey")

	//推送指令
	//gradlew clean build bintrayUpload
	//添加
	publish {
	    bintrayUser = localBintrayUser   //bintray.com用户名
	    bintrayKey = localBintrayApikey  //bintray.com apikey
	    dryRun = false
	    userOrg = localBintrayUser
	    groupId = 'com.'+ localBintrayUser +'.share'//jcenter上的路径
	    artifactId = 'sharesdk'//项目名称
	    publishVersion = '0.0.1'//版本号
	    desc = 'Oh hi, this is a nice description for a project, right?'
	    website = 'https://github.com/xyzlf/ShareSDK'
	}

完整build.gradle参照：

1、<https://github.com/xyzlf/ShareSDK/blob/master/build.gradle>

2、<https://github.com/xyzlf/ShareSDK/blob/master/shareLibrary/build.gradle>

## 上传命令

其中 bintrayUser bintrayKey你在这里<https://bintray.com>注册完成后，在用户中心可以找到。

	//直接使用用户名  apikey
    gradlew clean build bintrayUpload -PbintrayUser=bintrayUser -PbintrayKey=bintrayKey -PdryRun=false

	//配置了用户名 apikey
	gradlew clean build bintrayUpload
    
上传完成后，你能看到你上传的项目，然后发布申请，审核过程很快。实战练习，5分钟之内审核完成。然后其他开发者就可以使用了。

## 遇到的问题

将aar上传到jcenter的过程中遇到几个问题：

1、"编码 GBK 的不可映射字符"，项目中的一部分中文注释，会出现这个问题，在网上找了下解决方案，没有作用。我最终是逐个修改，才OK的。（只有部分中文注释会这样，不知道为啥）

2、每个注释都需要有完整的说明，不然就会抛错，如下：
错误信息：

    @param 没有说明

    /**
     * 显示toast
     * @param context
     * @param resId
     * @param isShort
     */
    public static void showToast(Context context, @IdRes int resId, boolean isShort) {
    	...
    }
    
修改后完整，能通过：

    /**
     * 显示toast
     * @param context context
     * @param resId 资源id
     * @param isShort isShort
     */
    public static void showToast(Context context, @IdRes int resId, boolean isShort) {
    	...
    }