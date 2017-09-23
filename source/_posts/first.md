---
title: 开篇
date: 2017-09-21 22:15:42
tags: 
	- hexo
	- next
comments: true
categories: Others
---


### 开篇
折腾了两个晚上，终于把hexo搭好了
当然，这也是网络发达的好处，网上一搜索大把的教程就出来了，也说明了hexo的受欢迎程度
最近又要学习Java了，在这个博客上记录点东西

<!-- more -->

### hexo折腾记
1. 安装hexo
	1. 先安装nodeJs、git
	2. 再安装hexo：npm install hexo-cli -g
	3. 创建博客目录：hexo init hexoblog
2. 在github上增加仓库，名称为：yourname.github.io
3. 解析域名
	1. 阿里云增加CNAME记录
	2. source目录下增加CNAME文件，内容为域名
4. 修改配置文件
	1. 在deploy节点下配置type为git
	2. repository为前面在github上新建的github.io仓库地址
5. 安装next主题(也可以用其它的，hexo官网上有很多的主题)
	1. cd到blog目录
	2. git clone https://github.com/iissnan/hexo-theme-next.git themes/next
	3. 修改配置文件的theme节点为next
6. 本地验证：hexo server
7. 推送到仓库
	1. hexo clean
	2. hexo d -g


### 待完善的地方
1. 评论系统
2. 分类
3. 谷歌浏览器缓存很严重