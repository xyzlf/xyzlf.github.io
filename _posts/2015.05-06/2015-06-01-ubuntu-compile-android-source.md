---
layout: post
title: Ubuntu15.04下编译Android5.1源码
date: 2015-06-01 20:16:41
categories: [Android, Ubuntu]
tags: [Android, 源码]
---
#### 编译环境  
- ubuntu：ubuntu-15.04-desktop-amd64  
- android：android-5.1.0_r3    
- 硬盘空间100G，最好预留比较多的硬盘空间，以后可以扩展用。如果不设置ccache，编译完后差不多占用47G+，设置ccache，会多占用8G+。  
<!--more-->

#### 编译步骤 ####  
&emsp;
1. 下载安装JDK   
{% highlight java %}
sudo apt-get update
sudo apt-get install openjdk-7-jdk
{% endhighlight java %}
在Ubuntu下编译最新的Android源码需要OpenJDK环境  
&emsp;
2. 更新下默认的java版本（可选）  
{% highlight java %}
sudo update-alternatives --config java
sudo update-alternatives --config javac
{% endhighlight java %}  
&emsp;
3. 安装依赖包  
{% highlight java %}
sudo apt-get install bison g++-multilib git gperf libxml2-utils make zlib1g-dev:i386 zip
{% endhighlight java %}
&emsp;
4. 设置ccache（可选） 
在.bashrc中添加  
{% highlight java %}
export USE_CCACHE=1
export CCACHE_DIR=<path-to-your-cache-directory># 默认路径为~/.ccache
{% endhighlight java %}
在Android5.1源码根目录下执行
{% highlight java %}
prebuilts/misc/linux-x86/ccache/ccache -M 50G
{% endhighlight java %}
ccache会在重新编译时加快编译速度。  
&emsp;
5. 配置环境
{% highlight java %}
source build/envsetup.sh
{% endhighlight java %}
&emsp;
6. 设置编译target
{% highlight java %}
lunch aosp_arm-eng
{% endhighlight java %}
默认就是aosp_arm-eng，如果想换成其他的话，可以先输入lunch，然后会显示可设置的target列表  
&emsp;
7.  更新API
{% highlight java %}
make update-api
{% endhighlight java %}
这一步如果不执行的话，在后面编译过程中可能会报错  
&emsp;
8. 编译
{% highlight java %}
make -j16
{% endhighlight java %}
j后面的数字表示最大任务数，视机器的配置自行设定，配置高的尽量设置大一点，可以节省编译时间，笔者编译过程大概花费了两个半小时。如果编译中编译失败，可以使用make -k继续编译  
&emsp;
9. 编译完成
编译成功后，可以看到下面的输出：
{% highlight java %}
#### make completed successfully (02:24:19 (hh:mm:ss)) ####
{% endhighlight java %}
然后可以使用模拟器来运行试试：
{% highlight java %}
emulator
{% endhighlight java %}
在第5步设置环境的时候已经把emulator加入到PATH中，所以可以直接执行。如果不能执行，可以直接运行prebuilts/android-emulator/linux-x86_64/emulator，或者重新执行下第5步和第6步。

#### 参考：
[http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)  
[http://source.android.com/source/building-running.html](http://source.android.com/source/building-running.html)  
[http://blog.csdn.net/luoshengyang/article/details/6559955/](http://blog.csdn.net/luoshengyang/article/details/6559955/)  
