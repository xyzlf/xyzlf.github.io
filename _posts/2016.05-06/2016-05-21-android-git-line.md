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

##  Git的下载及安装

1、从官网下载Git，<https://git-scm.com/download/win>，然后按照默认步骤一步步安装即可。

2、配置SSH key文件，生成SSH key文件：<https://help.github.com/articles/generating-an-ssh-key/>。

**<font color="#FF0000">Generating a new SSH key</font>**

(1)Open Git Bash.

(2)Paste the text below, substituting in your GitHub email address.
{% highlight java %}
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
Creates a new ssh key, using the provided email as a label
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
start the ssh-agent in the background
eval "$(ssh-agent -s)"
Agent pid 59566
{% endhighlight java %}
If you are using another terminal prompt, such as Git for Windows, turn on ssh-agent:
{% highlight java %}
start the ssh-agent in the background
eval $(ssh-agent -s)
Agent pid 59566
{% endhighlight java %}
(2)Add your SSH key to the ssh-agent. If you used an existing SSH key rather than generating a new SSH key, you'll need to replace id_rsa in the command with the name of your existing private key file.
{% highlight java %}
$ ssh-add ~/.ssh/id_rsa
{% endhighlight java %}
(3)Add the SSH key to your GitHub account.<https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/>

**<font color="#FF0000">ssh-keygen不是内部或外部命令</font>**

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


## Git 相关资料

Pro Git（中文版）： <http://git.oschina.net/progit/>

Git from the inside out：<https://maryrosecook.com/blog/post/git-from-the-inside-out>