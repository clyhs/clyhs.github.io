---
title: centos6.8下PHP5.3.3升级到5.6
date: 2017-09-30 09:57:00 
category: php
tags: php
---
centos6.8的php预设5.3.3这个版本，其实对centos来说就是替换掉yum的资料库
分以下步骤進行
```shell
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm
sudo vim /etc/yum.repos.d/remi.repo
```

進去把所有的enabled参数改成1

yum –enablerepo=remi update php* mysql*  

最后再进行一次升级动作
```shell
yum -y update php*

```
全部安装成功后确认一下：

yum list installed | grep php

最后重启服务

service httpd restart





