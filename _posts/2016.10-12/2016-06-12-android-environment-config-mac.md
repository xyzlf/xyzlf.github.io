---
layout: post
title: Mac开发环境配置
date: 2016-12-03 13:16:00
categories: [Android]
tags: [Android]
---

Mac下面的开发环境配置，jdk的下载安装，Android Studio的下载安装，Android SDK的安装配置，git的配置。
<!--more-->

##  JDK的下载及安装

Mac下一般带有JDK环境的，我买的16款新Mac发现没有JDK，因此我从官网下载了JDK7，下载了JDK8安装。

JDK7官网地址：<http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html>

JDK8官网地址：<http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html> 


##  Android Studio的下载及安装 

1、国内网络目前还是需要翻墙才能访问，所以呐先安装一个lantern吧，里面有各个平台的版本，挺好用的：<https://github.com/getlantern/lantern>。

2、从官网下载Android Studio：<https://developer.android.com/studio/index.html> 。

3、下载完成后，拖进Application里面即可。

## Android SDK的下载

1、这个比较费劲，需要有翻墙代理，好在我公司网络自带VPN。如果没有的童鞋，可以去网上购买一个VPN，设置好代理，就能比较轻松的下载完成了。

2、下载完成Android SDK后，配置下adb命令的路径。具体步骤可参照：<http://jingyan.baidu.com/article/59703552c0f8818fc1074041.html>

{% highlight java %}
	(1)在用户的HOME目录下，找到.bash_profile文件，如果没有该文件就创建一个 touch .bash_profile。

	(2)打开.bash_profile文件：open -e .bash_profile，进行路径配置。

	(3)在打开的编辑器里面输入如下路径：
	export PATH=${PATH}:/Users/apple/Library/Android/sdk/platform-tools   （apple就是你电脑用户的名字了）
	export PATH=${PATH}:/Users/apple/Library/Android/sdk/tools
	我的Android SDK放在了主目录下面，所以路径如下：
	export PATH=${PATH}:/Users/zhanglifeng/Android/platform-tools
	export PATH=${PATH}:/Users/zhanglifeng/Android/tools

	(4)保存配置，更新配置环境变量：source .bash_profile。

	(5)打开终端，试试adb命令是否生效。
{% endhighlight java %}


##  Git的配置

Mac自带git，所以没有下载安装这一步骤，我们只需要生成一个SSH key，然后配置就行了。


生成SSH key文件，配置SSH key，具体内容可参照官网：<https://help.github.com/articles/generating-an-ssh-key/>，下面是我从官网拷贝过来的，当做笔记。

**<font color="#FF0000">Generating a new SSH key</font>**

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

**<font color="#FF0000">Adding your SSH key to the ssh-agent</font>**

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

3、记得配置用户名和邮箱： 设置本地机器默认commit的昵称与Email，请使用有意义的名字与Email。

{% highlight java %}
git config --global user.name "zhanglifeng"  
git config --global user.email "zhanglifeng@nicaifu.com"  
{% endhighlight java %}

这个博客写得也挺好的，用中文翻译了一遍，英文不好的同学参照这里吧： <http://blog.csdn.net/renfufei/article/details/41647875>


