---
layout: post
title: Gradle自定义插件开发
date: 2017-04-05 20:39:00
categories: [Android]
tags: [Android]
---

Gradle是一个强大的构建工具，能够给我们的项目带来极大的方便。Gradle插件是用groovy语言编写，而groovy完美兼容java代码，因此对于写java的童鞋来说，只需要很低的学习成本。接下来描述下，如何构建一个自定义的Gradle插件~
<!--more-->

## 自定义Gradle插件及使用

**1、基本准备**

```

1、首先，新建一个Android项目。

2、之后，新建一个Android Module项目，类型选择Android Library。

3、在新建的module中新建文件夹src，接着在src文件目录下新建main文件夹，在main目录下新建groovy目录，resources目录。在groovy目录下面新建包名，增加代码。resources目录下新建文件夹META-INF，META-INF文件夹下新建gradle-plugins文件夹。
这样，就完成了gradle 插件的项目的整体搭建，之后就是小细节了。目前，项目的结构是这样的。

```

结构如图：

<img src="/assets/drawable/gradle_plugin_struct.png"  alt="pic" />

**2、修改Module下面的build.gradle（修改上图pluginlib下面的build.gradle）**

{% highlight java %}

apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    compile gradleApi()
    compile localGroovy()
}

repositories {
    mavenCentral()
}
group='com.xyzlf.plugin'
version='1.0.0'

uploadArchives {
    //使用当前目录下面的maven仓库，仓库在当前目录的repo文件目录中
    repositories {
        mavenDeployer {
            repository(url: uri('../repo'))
        }
    }
}

{% endhighlight java %}

**3、增加自定义插件groovy代码**

（1）在groovy目录下，新建包名，如：com.xyzlf.pluginlib。

（2）增加插件代码PluginImpl.groovy及TimeListener.groovy

PluginImpl.groovy

{% highlight java %}

package com.xyzlf.pluginlib

import org.gradle.api.Plugin
import org.gradle.api.Project

public class PluginImpl implements Plugin<Project> {
    void apply(Project project) {
//        project.task('testTask') << {
//            println "Hello gradle plugin"
//        }
        project.gradle.addListener(new TimeListener())
    }
}

{% endhighlight java %}

TimeListener.groovy

{% highlight java %}

package com.xyzlf.pluginlib

import org.gradle.BuildListener
import org.gradle.BuildResult
import org.gradle.api.Task
import org.gradle.api.execution.TaskExecutionListener
import org.gradle.api.initialization.Settings
import org.gradle.api.invocation.Gradle
import org.gradle.api.tasks.TaskState
import org.gradle.util.Clock

class TimeListener implements TaskExecutionListener, BuildListener {
    private Clock clock
    private times = []

    @Override
    void beforeExecute(Task task) {
        clock = new org.gradle.util.Clock()
    }

    @Override
    void afterExecute(Task task, TaskState taskState) {
        def ms = clock.timeInMs
        times.add([ms, task.path])
        task.project.logger.warn "${task.path} spend ${ms}ms"
    }

    @Override
    void buildFinished(BuildResult result) {
        println "Task spend time:"
        for (time in times) {
            if (time[0] >= 50) {
                printf "%7sms  %s\n", time
            }
        }
    }

    @Override
    void buildStarted(Gradle gradle) {}

    @Override
    void projectsEvaluated(Gradle gradle) {}

    @Override
    void projectsLoaded(Gradle gradle) {}

    @Override
    void settingsEvaluated(Settings settings) {}
}


{% endhighlight java %}

（3）在resources/META-INF/gradle-plugins/目录下新建一个文件：xyzlf.plugin.time.properties.

{% highlight java %}

注意： xyzlf.plugin.time就是应用的插件名，  如使用的地方引用：apply plugin: 'xyzlf.plugin.time'

{% endhighlight java %}


## 将Gradle插件发布到本地仓库

1、填写完上面的插件代码，接下来就是发不到本地仓库了。

2、在项目的根目录下面执行：Windows下执行： gradlew uploadArchives（）, Mac下执行： ./gradlew uploadArchives

3、这样，在项目根目录会生成repo文件夹，里面即有我们的插件jar包。

4、使用本地的自定义插件，在项目的 app/build.gradle文件中，引用自定义插件。

{% highlight java %}

buildscript {
    repositories {
        //使用当前目录下面的maven仓库，仓库在当前目录的repo文件目录中
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath 'com.xyzlf.plugin:time:0.0.1'
    }
}
apply plugin: 'xyzlf.plugin.time'

{% endhighlight java %}

5、在项目根目录执行：gradlew clean build 就会出现task的耗时统计。

<img src="/assets/drawable/gradle_plugin_time.png"  alt="pic" />

## 将Gradle插件上传到jcenter

1、具体步骤其实也简单。

（1）在项目的根目录下面加入上传到jcenter的插件依赖库： CustomPlugin/build.gradle。

{% highlight java %}

classpath 'com.novoda:bintray-release:0.3.4' // 上传自定义插件到jcenter仓库的依赖库

{% endhighlight java %}

（2）在module目录下增加如下内容：

{% highlight java %}

apply plugin: 'com.novoda.bintray-release'//添加上传到jcenter仓库用到的插件

//------------------------------ 以下是上传到jcenter仓库的代码 ------------------------------//
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
    groupId = 'com.'+ localBintrayUser +'.plugin'//jcenter上的路径
    artifactId = 'time'//项目名称
    publishVersion = '0.0.1'//版本号
    desc = '统计gradle task耗时的插件'                                   // 增加你的插件描述
    website = 'https://github.com/xyzlf/CustomPlugin'                  // 修改你的项目地址
}


{% endhighlight java %}

（3）在根目录的local.properties配置你的bintray.user名，及bintray.apikey值。

{% highlight java %}

bintray.apikey=你的bintray.apikey值
bintray.user=你的bintray.user名称

{% endhighlight java %}

（4）配置完，在项目根目录下面，执行命令行：

{% highlight java %}

gradlew clean build bintrayUpload

{% endhighlight java %}

不出意外，应该就上传到jcenter中心了，然后你去 http://bintray.com/ 中增加到jcenter，审核通过后，就能使用了。

（5）使用jcenter依赖：

{% highlight java %}

buildscript {
    repositories {
       jcenter()
    }
    dependencies {
        classpath 'com.xyzlf.plugin:time:0.0.1'
    }
}
apply plugin: 'xyzlf.plugin.time'

{% endhighlight java %}

2、如果对于上传到jcenter有不懂的，也可以参照此文：[Android Studio aar上传到jcenter](http://xyzlf.github.io/2016/07/10/android-aar-jcenter.html)

## Gradle Flavor构建渠道包


## 参考资料

1、[如何使用Android Studio开发Gradle插件](http://blog.csdn.net/sbsujjbcy/article/details/50782830)

2、[构建神器Gradle](http://jiajixin.cn/2015/08/07/gradle-android/)

3、(1)[ 自定义Gradle插件（一）](http://blog.csdn.net/liuhongwei123888/article/details/50541759)   (2)[ 自定义Gradle插件（二）](http://blog.csdn.net/liuhongwei123888/article/details/50542104)

4、(1)[美团Android自动化之旅—生成渠道包](http://tech.meituan.com/mt-apk-packaging.html)   (2)[美团Android自动化之旅—适配渠道包](http://tech.meituan.com/mt-apk-adaptation.html)
