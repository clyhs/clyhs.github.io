---
title: rancher2安装
date: 2019-06-18 14:47:28
category: rancher
tags: [rancher,docker]
---

### rancher安装

#### 服务器

* 主：192.168.137.101
* agent：192.168.137.100

#### 准备

关掉swap

```shell
swapoff -a
```

设置selinux

```shell
setenforce 0
```

或者/etc/selinux/config中设置为disabled

关掉防火墙

```shell
systemctl stop firewalld
systemctl disable direwalld
```

软件情况

docker:18+

rancher:2.2.4

#### 安装 

**1、在192.168.137.101执行**：

```shell
docker run -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher
```

成功后，在浏览器访问https://192.168.0.16，设置初始密码

![img](https://clyhs.github.io/images/rancher/01.png)

下一步

![img](https://clyhs.github.io/images/rancher/02.png)

下一步

![img](https://clyhs.github.io/images/rancher/03.png)

选择右下角中文

![img](https://clyhs.github.io/images/rancher/04.png)

添加集群，选择右下角custom，并滚动页面，在下面填定集群名称

![img](https://clyhs.github.io/images/rancher/05.png)

![img](https://clyhs.github.io/images/rancher/06.png)

下一步，勾选所有选项

![img](https://clyhs.github.io/images/rancher/07.png)

并且可以看到以下：

复制以下命令在主机的SSH终端运行。这个是在agent执行的命令

**2、在192.168.137.100执行**：

```shell
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run rancher/rancher-agent:v2.2.4 --server https://192.168.137.101 --token jpbmr8k27l7r7hxps5q9p6lxhzjpjk8sspqvx9flbb2rxlpslgcsg9 --ca-checksum ccba9db8ad0f6e204968ec2dc5f87de5fea16c05032f6f1c19e141b03f8edcf5 --etcd --controlplane --worker
```

![img](https://clyhs.github.io/images/rancher/07.png)

红色字体提示没有rancher/hyperkube:v1.13.5-rancher1镜像，连网不成功。

最好192.168.0.7是连网状态，因为加上agent节点，会从网上拉下镜像。

添加成功后如下：

![img](https://clyhs.github.io/images/rancher/10.png)

![img](https://clyhs.github.io/images/rancher/11.png)

![img](https://clyhs.github.io/images/rancher/12.png)

![img](https://clyhs.github.io/images/rancher/13.png)





