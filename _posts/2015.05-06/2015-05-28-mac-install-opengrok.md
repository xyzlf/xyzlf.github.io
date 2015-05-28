---
layout: post
title: Mac下安装OpenGrok
date: 2015-05-28 14:49:18
categories: [Mac, Android]
tags: [OpenGrok]
---  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OpenGrok一个快速、便于使用的源代码搜索与对照引擎。它帮助你搜索，对照，定位你的源代码树。它能够明白各种程序文件格式和版本控制历史记录如SCCS,RCS,CVS与Subversion。OpenGrok是OpenSolaris操作系统源文件浏览和搜索的工具。  
<!--more-->  
下面介绍下Mac的安装方式：  
1. 下载Tomcat: http://tomcat.apache.org/download-80.cgi  
2. 下载OpenGrok：https://github.com/OpenGrok/OpenGrok  
3. 下载ctags：http://ctags.sourceforge.net/，安装过程很简单，解压后执行：./configure && make && make install  
4. 配置一些必要的环境，可以配置在.bash_profile中  
export JAVA_HOME=/opt/java 
export OPENGROK_TOMCAT_BASE=/opt/tomcat/ 
export OPENGROK_APP_SERVER=Tomcat 
export OPENGROK_INSTANCE_BASE=/opt/opengrok  
5. 在OpenGrok根目录下执行mkdir data;mkdir etc;  
6. 在bin目录下执行./OpenGrok deploy，注意这一步后记得把Tomcat/webapps路径下的source.war删除掉，否则可能会在下次启动时覆盖上次的web.xml中的配置，如果web.xml中的配置被覆盖，也可以手动修改下，如:  
&lt;param-name>CONFIGURATION&lt;/param-name>  
&lt;param-value>/var/opengrok/etc/configuration.xml&lt;/param-value>  
7. 在需要建立索引的源码路径执行 /opt/opengrok/OpenGrok index . (.表示当前路径)  
8. done，打开http://localhost:8080/source/ 就可以看源码咯~    

#### 另外推荐一个使用OpenGrok在线查看Android源码的网站：
[http://androidxref.com/](http://androidxref.com/)