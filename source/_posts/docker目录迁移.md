---
title: docker目录迁移
date: 2019-01-10 09:22:24
category: docker
tags: docker
---
###  docker目录迁移
Docker的默认目录是在/var/lib/docker目录下
目录包括：
![img](https://clyhs.github.io/images/docker/201901101.png)
将/var/lib/docker的目录迁移到/home/docker/lib/docker目录下

1、	用于查看Docker的磁盘使用情况
```shell
docker system df
```
![img](https://clyhs.github.io/images/docker/201901102.png)

2、	关闭docker
```shell
systemctl stop docker
```

3、	命令查看磁盘使用情况
```shell
du -hs /var/lib/docker/
```
![img](https://clyhs.github.io/images/docker/201901103.png)

4、	创建新的docker目录，执行命令df -h,找一个大的磁盘。 我在 /home目录下面建了
```shell
mkdir -p /home/docker/lib/docker
```

5、	迁移/var/lib/docker目录下面的文件到 /home/docker/lib/docker
```shell
cp -R /var/lib/docker/* /home/docker/lib/docker/
```

6、	配置 /etc/systemd/system/docker.service.d/devicemapper.conf。查看 devicemapper.conf 是否存在。如果不存在，就新建
```shell
mkdir -p /etc/systemd/system/docker.service.d/
vi /etc/systemd/system/docker.service.d/devicemapper.conf
```
7、	然后在 devicemapper.conf 写入
内容如下：
```shell
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd  --graph=/home/docker/lib/docker
```
8、	重新加载 docker
```shell
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```
9、	启动成功后，再确认之前的镜像还在：
```shell
docker images
```
![img](https://clyhs.github.io/images/docker/201901104.png)

10、确定容器没问题后删除/var/lib/docker/目录中的文件。
