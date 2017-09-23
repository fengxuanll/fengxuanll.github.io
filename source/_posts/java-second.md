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


#### 第一步：从IDEA生成Web项目


#### 第二步：部署到tomcat
1. 在tomcat的webapps目录下，新建个app的文件夹
2. 将Java项目target目录下的文件复制到app文件夹中
3. 启动tomcat
4. 打开浏览器访问：http://127.0.0.1:8080/app
5. OK


#### 第三步：部署到tomcat2
1. 将tomcat的目录复制一个新的文件夹：tomcat2
2. 修改下tomcat2/webapps/app里的某个静态文件内容，让它在被访问的时候能看到和上一步部署的是不一样的内容
3. 修改tomcat2的server.xml配置文件，将每个端口都加100
4. 启动tomcat2
5. 打开浏览器访问：http://127.0.0.1:8180/app
6. OK


#### 第四步：部署nginx反向代理
1. 修改nginx/conf/nginx.conf文件，主要配置如下：
```
#设定负载均衡的服务器列表
upstream app {
    #weigth参数表示权值，权值越高被分配到的几率越大
    server 127.0.0.1:8080 weight=2;
    server 127.0.0.1:8180 weight=1;
}

server {
    listen       81;
    server_name  app;

    location / {
        root   html;
        index  index.html index.htm;
        proxy_pass http://app;
    }

    // 省略内容......
}```


2. 启动nginx
3. 打开浏览器访问：http://127.0.0.1:81/app/
4. nginx部署就结束了，多刷新几次，可以看到访问的是不同的tomcat
5. 这就是传说中的反向代理



### 最后
tomcat和nginx还有很多的功能和配置，还得好好学习下。