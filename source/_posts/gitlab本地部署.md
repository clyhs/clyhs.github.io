---
title: gitlab本地部署
date: 2019-09-24 15:51:05
category: git
tags: gitlab
---

### gitlab本地部署

#### 准备

下载地址：https://packages.gitlab.com/gitlab/gitlab-ce/

系统：centos7

软件包：gitlab-ce-10.5.1-ce.0.el7.x86_64.rpm

#### 先安装依赖

```
yum install curl openssh-server openssh-clients postfix cronie policycoreutils-python
```

#### 安装gitlab

```
rpm -ivh gitlab-ce-10.5.1-ce.0.el7.x86_64.rpm
#安装完成后，执行配置
gitlab-ctl reconfigure

```

修改/etc/gitblab/gitlab.rb

将external_url，改成具体ip，如：http://192.168.0.7

```
#重新配置
gitlab-ctl reconfigure
#重启
gitlab-ctl restart
```

 查看gitlab版本 

head -1 /opt/gitlab/version-manifest.txt

通过浏览器登录

http://192.168.0.7

配置用户密码，默认用户为root

#### 汉化

```
git clone https://github.com/marbleqi/gitlab-ce-zh.git
cd gitlab-ce-zh
git pull origin #从远端获取最新库
git branch -a
git checkout remotes/origin/v10.5.1-zh-patch

```

将汉化文件覆盖到/opt/gitlab/embedded/service/gitlab-rails/

```
\cp -r -f gitlab-ce-zh/* /opt/gitlab/embedded/service/gitlab-rails/
```

修改/etc/gitlab/gitlab.rb

```
vi /etc/gitlab/gitlab.rb
gitlab_rails['time_zone'] = 'PRC' #将标准时修改为中国时间

gitlab-ctl reconfigure #使修改的配置文件生效
#注：如果上述命令执行后，未汉化或有异常，则执行以下命令
gitlab-ctl stop #停止gitlab服务
gitlab-ctl start #启动gitlab服务
```

![img](https://clyhs.github.io/images/gitlab/gitlab-01.png)

