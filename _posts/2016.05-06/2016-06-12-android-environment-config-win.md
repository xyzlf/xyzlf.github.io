---
layout: post
title: Windows开发环境配置
date: 2016-06-12 17:00:00
categories: [Android]
tags: [Android]
---

Windows下面的开发环境配置，jdk安装，Android Studio的安装，以及git的安装及配置~
<!--more-->

##  JDK的下载及安装

从官网下载最新jdk：<http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html> 。
然后一步步安装，配置环境变量，这个没啥说的，一路点击下一步就OK。

##  Android Studio的下载及安装 

1、国内网络目前还是需要翻墙才能访问，所以呐先安装一个lantern吧，里面有各个平台的版本，挺好用的：<https://github.com/getlantern/lantern>。

2、从官网下载Android Studio：<https://developer.android.com/studio/index.html> 。

3、也是一步步点击下一步就OK了。

##  Git的下载及安装

1、从官网下载Git，<https://git-scm.com/download/win>，然后按照默认步骤一步步安装即可。

2、配置SSH key文件，生成SSH key文件：<https://help.github.com/articles/generating-an-ssh-key/>。

<b>Generating a new SSH key</b>

(1)Open Git Bash.

(2)Paste the text below, substituting in your GitHub email address.
{% highlight java %}
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
Generating public/private rsa key pair.
{% endhighlight java %}
(3）When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.
{% highlight java %}
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
{% endhighlight java %}
(4)At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases".
{% highlight java %}
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
{% endhighlight java %}

<b>Adding your SSH key to the ssh-agent</b>

Before adding a new SSH key to the ssh-agent, you should have checked for existing SSH keys and generated a new SSH key.

If you have GitHub for Windows installed, you can use it to clone repositories and not deal with SSH keys. It also comes with the Git Bash tool, which is the preferred way of running git commands on Windows.

(1)Ensure ssh-agent is enabled:

If you are using Git Bash, turn on ssh-agent:
{% highlight java %}
# start the ssh-agent in the background
eval "$(ssh-agent -s)"
Agent pid 59566
{% endhighlight java %}
If you are using another terminal prompt, such as Git for Windows, turn on ssh-agent:
{% highlight java %}
# start the ssh-agent in the background
eval $(ssh-agent -s)
Agent pid 59566
{% endhighlight java %}
(2)Add your SSH key to the ssh-agent. If you used an existing SSH key rather than generating a new SSH key, you'll need to replace id_rsa in the command with the name of your existing private key file.
{% highlight java %}
$ ssh-add ~/.ssh/id_rsa
{% endhighlight java %}
(3)Add the SSH key to your GitHub account.<https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/>

<b>ssh-keygen不是内部或外部命令</b>

解决方案：

（1）找到Git/usr/bin目录下的ssh-keygen.exe(如果找不到，可以在计算机全局搜索)


（2）属性-->高级系统设置-->环境变量-->系统变量,找到Path变量，进行编辑，End到最后，输入分号，粘贴复制的ssh-keygen所在的路径，保存；

3、记得配置用户名和邮箱：
设置本地机器默认commit的昵称与Email，请使用有意义的名字与Email。

{% highlight java %}
git config --global user.name "zhanglifeng"  
git config --global user.email "zhanglifeng@nicaifu.com"  
{% endhighlight java %}

这个博客写得也挺好的，用中文翻译了一遍，英文不好的同学参照这里吧：
<http://blog.csdn.net/renfufei/article/details/41647875>

##  Notepad配置Markdown以及预览
这篇文章写得不错：<http://www.jianshu.com/p/cdb42773fffe>
总结起来总共两步：

1、下载配置文件，format文件，导入。

2、导入预览dll。