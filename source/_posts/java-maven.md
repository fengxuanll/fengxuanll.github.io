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

在没有Maven之前，如果有依赖其它的包，需要下载包放到项目类路径，手动将其包含到需要的项目，这一些都需要手动去做，相当麻烦。
而有了Maven之后，只需要在pom.xml里配置依赖包的Maven的坐标即可，Maven就会自动将对应的依赖包下载下来，并保存到Maven的本地仓库。


<!-- more -->


### 准备工作

#### 第一步：当然是安装Maven

##### Mac
1. Mac下执行命令：brew install maven
2. 配置环境变量：vi ~/.bash_profile
3. 添加两行配置：
```
	export M2_HOME=Maven目录
	export PATH=$PATH:$M2_HOME/bin
```


##### Windows
1. Windows下需要在[官网](http://maven.apache.org/download.cgi)下载Maven的包并解压。
2. 配置M2_HOME和MAVEN_HOME环境变量，路径都配置Maven解压后的目录。
3. 在PATH环境变量后，添加Maven的bin目录，使用 %M2_HOME%/bin 即可。


安装完成后，执行 mvn –version 看看输出是否正常。


#### 第二步：使用阿里云的共有仓库
由于Maven的服务器是在国外，国内访问比较慢，阿里云提供了一个同步的Maven仓库镜像，基本就可以秒下jar包了。
修改.m2目录下的setting.xml文件，在mirrors节点下增加以下内容：
```xml
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>
```

### Maven仓库
Maven仓库分为本地仓库、远程仓库，Maven查找包时，会先搜索本地仓库，如果本地仓库没有，就会去搜索远程仓库。
远程仓库又分为：中央仓库、私服、其它公共库


#### 本地仓库
顾名思义，本地仓库就是在自己电脑上的，当我们需要什么包时，会从远程仓库缓存到本地，然后第二次使用这个包时，就不用去下载新的了。
本地仓库存储在用户目录下的.m2/repository/文件夹下，这是Maven的默认路径，当然，也是可以修改的，修改.m2/repository、settings.xml文件里的localRepository节点，例如：
```xml
<settings>  
    <localRepository>D:\maven_new_repository</localRepository>  
</settings>
```


#### 远程仓库

##### 中央仓库
中央仓库是官方默认的远程仓库，Maven在安装时，自带的就是中央仓库的配置。
中央仓库中包含了绝大多数流行的开源Java构件，基本上你需要的都能在这里找得到，这也是Java生态繁荣的一个表现。
Maven中央仓库地址：[http://mvnrepository.com/](http://mvnrepository.com/)，如果需要什么包，可以直接去这上面找。

##### 私服
如果是团队开发项目，最好是能够自己搭建一个Maven私服仓库，我们一些自己内部的包就可以放到私服仓库里供团队使用，并且也可以从外部的远程仓库下载其它的包，缓存在私服中。
关于怎么搭建Nexus，可以参考这篇文章：[Maven仓库—Nexus环境搭建及简单介绍](http://blog.csdn.net/wang379275614/article/details/43940259)

##### 其它共同库
前面配置的阿里云的Maven镜像，就属于这一类，它基本上是同步了中央仓库的内容。




### 如何使用


#### POM文件
POM是Maven项目的配置文件，包含了各种配置信息，每个项目只能有一个POM文件，是一个xml文件，文件名为：POM.xml。
一个基本的POM文件内容为：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.zsean.project-group</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
<project>
```
因为每个Maven项目都会生成一个jar包，这些配置就是对当前项目的一些Maven定义


##### Super POM
所有的POM都继承自一个父POM，无论是否有显示定义，父POM也叫做Super POM，他包含了一些可以被继承的默认设置。



##### POM外部依赖
```xml
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>3.8.1</version>
	</dependency>
</dependencies>
```
这里是引用了junit的依赖，当保存后，Maven就会自动从中央仓库下载3.8.1版本的junit包到本地仓库，并引用到当前项目，然后就可以使用junit了。




##### POM插件
```xml
<plugins>
	<plugin>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-antrun-plugin</artifactId>
		<version>1.1</version>
	</plugin>
</plugins>
```


### 最后
写着写着就写不下去了，越写越觉得自己没弄清楚的地方太多了，这篇是还没写完的，等自己再多了解下再回来写吧......

关于Maven要学习的地方还很多，比如命令行工具...，现在还只是学习到了一点点皮毛的东西。

从.NET转Java估计最大的不习惯就是各种配置都需要手动去改，不像.NET都是各种图形化界面，慢慢习惯、学习吧，😓
