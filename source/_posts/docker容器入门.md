---
title: docker容器入门
date: 2017-04-08 10:52:59
categories: docker
tags: docker
---
# Docker容器入门
## docker简介
> Docker是一个开源的引擎,可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器

_ _ _

### centos7安装docker
~~~shell
# yum install docker
~~~
### 启动docker
~~~shell
# systemctl start docker
~~~
### 常用命令
> 1、查看镜像。
docker images 
2、查看本机运行的镜像
docker ps 
3、从dockerhub上pull 镜像
docker pull镜像名称
4、查看docker 版本
   docker version
5、登陆docker Hub的账号
   docker login
6、运行容器
docker run -it #启动docker容器在前端
docker run -d #启动docker容器在后台
7、保存镜像
docker save –o xxx.tar.gz xxx/xxx:tag
8、导入镜像
docker load < /path/xxx.tar.gz

_ _ _

### 自己创建一个镜像
先网上拉下一个做为基础镜像
```
# docker pull centos
```
下载jdk-8u111-linux-x64.tar.gz和apache-tomcat-7.0.29.tar.gz包到本地
创建Dockerfile文件
```
# vi Dockerfile
```
```
FROM centos:latest
MAINTAINER author
#设置工作目录
WORKDIR /usr/local

ADD jdk-8u111-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-7.0.29.tar.gz /usr/local/
# 配置环境变量 
ENV JAVA_HOME /usr/local/jdk1.8.0_111
ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-7.0.29

#暴露端口
EXPOSE 8080  
#设置tomcat 自启动  
CMD [ "/usr/local/apache-tomcat-7.0.29/bin/catalina.sh", "run" ]  
```
执行创建镜像命令
```
# docker build -t tomcat/centos:latest .
```
**注意：**最后面的`.`符号
运行容器
```
# docker run -d -p 8080:8080 tomcat/centos:latest
```


