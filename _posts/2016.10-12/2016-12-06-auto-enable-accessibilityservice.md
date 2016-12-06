---
layout: post
title: 自动开启辅助的黑科技之实践操作
date: 2016-12-06 12:40:00
categories: [Android]
tags: [Android]
---

看完这篇文章[揭秘360手机助手未经用户同意，自动开启辅助的“黑科技”](http://mp.weixin.qq.com/s/QW6oNOFI0HlJmMCuFSFYKg) ，觉得挺有意思的，动手实践操作了一遍，还是遇到蛮多问题，学习到了不少东西！做技术，动手更重要~ 再此，灰常感谢大神的分享。
<!--more-->

##  Android独立运行Java程序

在看到这篇文章前，没想过纯java程序能够在Android上运行。这篇文章有详细教程，[Android上app_process启动java进程](http://blog.csdn.net/u010651541/article/details/53163542) 。我参照着这个文章进行了实践操作，记录下遇到的问题。

**一、实例讲解**

最简单不过的实例了HelloWorld.java

{% highlight java %}

public class HelloWorld {

    public static void main(String[] args) {
        System.out.println("Hello, I am started by app_process!");
        while (true) {
            try {
                Thread.sleep(1500);
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("I am running HelloWorld.^_^");
        }
    }
}

{% endhighlight java %}

**二、生成Dex命令**

{% highlight java %}

dx --dex --output=D:\Hellworld.dex Helloworld.class

{% endhighlight java %}

**错误1：dx不是内部命令**

出现这个错误的原因，是没有配置环境变量，将dx的环境配置的path中，就可以使用正常了，在我机器上，如下路径：

{% highlight java %}

//设置dx的环境变量路径
D:\Android\sdk\build-tools\24.0.2

{% endhighlight java %}

**错误2：unsupported class file version 52.0**

<img src="/assets/drawable/error_version_52.png"  alt="pic" />

在我的机器上，jdk是1.8的版本（你可以用如下命令，查看下你的jdk版本：java -version），应该是jdk版本过高，出现了不匹配。按照这个文章 [Android上app_process启动java进程](http://blog.csdn.net/u010651541/article/details/53163542) 中的教程，可以如下编译：

{% highlight java %}

//编译，这里主要是Platform tool上用的是Java 7，所以显式指定1.7
javac -source 1.7 -target 1.7 C:\Users\Venscor\Desktop\app_process\dump.java

{% endhighlight java %}

在我机器上，没有1.7的jdk。我就用Android Studio生成一个项目，然后通过Android Studio的rebuild project自动编译生成class。

<img src="/assets/drawable/as_rebuild_project.png"  alt="pic" />  <img src="/assets/drawable/as_build.png"  alt="pic" />

如果还会出现jdk版本不匹配问题，可以在build.gradle里面指定编译环境。

{% highlight java %}

android {
    //....
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}

{% endhighlight java %}

**错误3：class name (...) does not match path(...class)**

<img src="/assets/drawable/error_has_pkgname.png"  alt="pic" />

这个问题是因为写的HelloWorld.java带了包名，然后编译路径不对应，如下解决：

以D盘为例，在D盘根目录新建目录 com\xyzlf\myapp （该路径跟类包名路径保持一致），然后将HelloWorld.class放在该目录下面，接着执行如下命令就OK了。

{% highlight java %}

D:\>dx --dex --output=D:\HelloWorld.dex com\xyzlf\myapp\HelloWorld.class

{% endhighlight java %}

这样就由class生成了dex了。 HelloWorld.java-->HelloWorld.class-->HelloWorld.dex。

**三、运行HelloWorld.dex**

运行HelloWorld.dex需要一部root手机，我身边没有root的手机，所以我用Genymotion的模拟器，这个模拟器特别好用，推荐~

（1）将HelloWorld.dex push到/data/local/tmp目录下面。

<img src="/assets/drawable/adm.png"  alt="pic" />

（2）执行HelloWorld.dex。

{% highlight java %}

adb shell

cd data/local/tmp

app_process -Djava.class.path=HelloWorld.dex  /data/local/tmp HelloWorld

{% endhighlight java %}

运行成功效果：

<img src="/assets/drawable/adb_run_java.png"  alt="pic" />

<img src="/assets/drawable/ps.png"  alt="pic" />

**四、被启动的Java的Pid,Uid与权限**

<img src="/assets/drawable/ps_view.png"  alt="pic" />

**五、启动的Java程序拥有的权限及Uid**

adb shell中，dumpsys activity命令是查看Activity栈信息的，只有shell权限(可能有shell权限也不行，必须Uid为shell，没有去看)才能调用。所以可以通过这个来验证启动的Java程序是否真的运行于shell的Uid下，测试源代码如下：

{% highlight java %}

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class dump {
    public static void main(String[]args){
        String cmd="dumpsys activity";
        System.out.println("Hello, I am started by app_process!");
        try {
            Process p=Runtime.getRuntime().exec(cmd);
            BufferedReader br=new BufferedReader(new InputStreamReader(p.getInputStream()));
            String readLine=br.readLine();
            while(readLine!=null){
                System.out.println(readLine);
                readLine=br.readLine();
            }
            if(br!=null){
                br.close();
            }
            p.destroy();
            p=null;
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

{% endhighlight java %}

和前面demo一样编译到Android下运行，查看结果，发现可以正常输出，说明app_process启动的Java程序的确具有shell级别的权限(其实其Uid就是shell)。

##  360手助黑科技实践操作

首先找一台Android手机（无需root）,然后需要安装360手机助手。然后通过如下命令行，然后你就会发现手助的“智能安装”开关打开了，在设置的辅助页面，也发现辅助功能开关打开了。

**一、命令行**

{% highlight java %}

adb shell

settings put secure enabled_accessibility_services com.qihoo.appstore/com.qihoo.appstore.accessibility.AppstoreAccessibility

settings put secure accessibility_enabled 1

{% endhighlight java %}

**二、代码实现**

按照上面的教程，执行如下java代码：

{% highlight java %}

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Helloworld {
    public static void main(String[] args) {
        System.out.println("Hello, I am started by app_process!");
        String cmd1 = "settings put secure enabled_accessibility_services com.qihoo.appstore/com.qihoo.appstore.accessibility.AppstoreAccessibility";
        String cmd2 = "settings put secure accessibility_enabled 1";
        execShell(cmd1);
        execShell(cmd2);
    }

    private static void execShell(String cmd) {
        try {
            Process p = Runtime.getRuntime().exec(cmd);
            BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
            String readLine = br.readLine();
            while (readLine != null) {
                System.out.println(readLine);
                readLine = br.readLine();
            }
            if (br != null) {
                br.close();
            }
            p.destroy();
            p = null;
        } catch (IOException e) {            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}

{% endhighlight java %}

**三、效果图**

<img src="/assets/drawable/360_zhushou1.png"  alt="pic" /><img src="/assets/drawable/360_zhushou2.png"  alt="pic" /><img src="/assets/drawable/setting.png"  alt="pic" />


##  参考文献

揭秘360手机助手未经用户同意，自动开启辅助的“黑科技”：<http://mp.weixin.qq.com/s/QW6oNOFI0HlJmMCuFSFYKg>

Android Accessibility安全性研究报告：<http://blogs.360.cn/360mobile/2016/09/07/research_of_accessibility/>

Android上app_process启动java进程：<http://blog.csdn.net/u010651541/article/details/53163542>

Android静默安装实现方案，仿360手机助手秒装和智能安装功能：<http://blog.csdn.net/guolin_blog/article/details/47803149#reply>

使用dx命令在cmd环境下执行的正确方法：<http://www.tuicool.com/articles/7ZrMjm>
 
 