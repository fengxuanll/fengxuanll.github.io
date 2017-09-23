---
title: Java从入门到放弃 - 部署篇
date: 2017-09-23 12:16:18
tags: Java
comments: true
categories: Java从入门到放弃
---


### 如何部署Java项目
今天尝试了下Java项目的部署，还是挺方便的，以前也没用过nginx来做过反向代理，现在觉得这种配置下就能用的东西还是挺好玩的。


#### 准备工作
1. 下载tomcat
	* tomcat就下载绿色版的，等下做反向代理时，直接复制个目录就可以玩了
2. 下载nginx


<!-- more -->


#### 第一步：从IDEA发布Web项目
1. 复制target目录下的文件


#### 第二步：部署到tomcat
1. 在tomcat的webapps目录下，新建个app的文件夹
2. 将Java项目target目录下的文件复制到app文件夹中
3. 修改tomcat配置文件


#### 第三步：部署nginx



#### 测试



### 最后
tomcat和nginx还有很多的功能和配置，还得好好学习下。