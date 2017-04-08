---
title: mesos+marathon搭建集群
date: 2017-04-08 13:11:08
category: mesos
tags: [mesos,marathon]
---
# mesos+marathon搭建集群
## mesos简介
> Mesos是Apache下的开源分布式资源管理框架，它被称为是分布式系统的内核。Mesos最初是由加州大学伯克利分校的AMPLab开发的，后在Twitter得到广泛使用。

## marathon简介
> 它是一个mesos框架，能够支持运行长服务，比如web应用等。

_ _ _

### 准备三台虚拟机
| 主机名 | IP |
|--------|--------|
|     mesos_master   |     192.168.137.121   |
|mesos_slave01|192.168.137.122|
|mesos_slave02|192.168.137.123|

### 开发搭建
1. 三台机都安装docker
 > yum install docker*
2. 三台机都安装mesos
 > rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
   rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-mesosphere
   yum install mesos -y
3. 修改hosts
> 192.168.137.121 mesos_master
   192.168.137.122 mesos_slave01
   192.168.137.123 mesos_slave02
4. mesos_master安装
> 1、关闭selinux
>     systemctl stop firewalld && systemctl disable firewalld
> 2、安装 yum -y install mesosphere-zookeeper
> 3、给每一个master的zookeeper设置一个唯一id
>     echo 1 > /var/lib/zookeeper/myid
>     vi /etc/zookeeper/conf/zoo.cfg
>     server.1=192.168.137.121:2888:3888
>     vi /etc/mesos/zk
>     zk://192.168.137.121:2181/mesos
>     vi /etc/mesos-master/quorum
>     1（zookeeper的数量）
> 4、停掉mesos-slave
>     systemctl stop mesos-slave.service
>     systemctl disable mesos-slave.service
> 5、安装marahton
>     yum install -y marathon
> 6、其他
>     vi /etc/mesos-master/ip
      (添加master的ip，默认是127.0.0.1，只做显示用)
      vi /etc/mesos-master/hostname
      (添加master的hostname，默认为localhost，主要在mesos集群间使用，不是机器的hostname，只做显示用)
> 7、marathon配置
> 这个设置和上面配置mesos的hostname效果一样，不配置会显示默认的localhost，不影响使用
>     mkdir -p /etc/marathon/conf/ && touch hostname
       echo 192.168.137.121 | sudo tee /etc/marathon/conf/hostname

5. mesos_slave01\slave02安装
> 1、设置zookeeper
       vi /etc/mesos/zk
       zk://192.168.1.24:2181/mesos
   2、关闭mesos-master服务
       systemctl stop mesos-master.service
       systemctl disable mesos-master.service
   3、指定使用docker容器化
        echo 'docker,mesos' > /etc/mesos-slave/containerizers
   4、考虑到拉取容器镜像等的操作，适当增加timeout的时间
        echo '5mins' > /etc/mesos-slave/executor_registration_timeout
6. 启动服务
>   1、MASTER
        service zookeeper start
        service mesos-master start
        service marathon start
   2、SLAVE01\02
       service mesos-slave start








