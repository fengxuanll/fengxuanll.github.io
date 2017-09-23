---
title: Java从入门到放弃 - Maven篇
date: 2017-09-24 10:52:33
tags: 
	- Java
	- Maven
categories: Java从入门到放弃
---


### Maven介绍
Maven是Java的一个项目管理工具，类似于.NET中nuget，不过做.NET的一般是在Visul Studio里使用nuget，因为有可视化界面，而Maven一般是在xml文件里配置，当然nuget也可以手动配置，只是没有可视化界面操作那么的方便。这也是.NET开发和其他语言的开发比较大的区别，毕竟.NET有宇宙最强大的IDE。


<!-- more -->


### 准备工作

#### 第一步：当然是安装Maven

##### Mac
1. Mac下执行命令：brew install maven
2. 配置环境变量：vi ~/.bash_profile
3. 添加两行配置：
```
	export M2_HOME=/Users/用户名/apache-maven-3
	export PATH=$PATH:$M2_HOME/bin
```


##### Windows
1. Windows下需要在[官网](http://maven.apache.org/download.cgi)下载Maven的包并解压。
2. 配置M2_HOME和MAVEN_HOME环境变量，路径都配置Maven解压后的目录。
3. 在PATH环境变量后，添加Maven的bin目录，使用 %M2_HOME%/bin 即可。


安装完成后，执行 mvn –version 看看输出是否正常。


#### 第二步：使用阿里云的共有仓库
由于Maven的服务器是在国外，国内访问比较慢，阿里云提供了一个同步的Maven仓库，基本就可以秒下jar包了。
修改.m2目录下的setting.xml文件，在mirrors节点下增加以下内容：
```xml
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>
```


### 如何使用



### 最后
从.NET转Java估计最大的不习惯就是各种配置都需要手动去改，不像.NET都是各种图形化界面，慢慢习惯吧，😓
