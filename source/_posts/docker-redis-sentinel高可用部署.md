---
title: docker redis sentinel高可用部署
date: 2019-02-16 10:55:44
category: redis
tags: [redis,docker]
---
### docker redis sentinel高可用部署

目前来说，高可用(主从复制、主从切换)redis集群有两种方案。

* 一种是redis-sentinel，只有一个master，各实例数据保持一致；
* 一种是redis-cluster，也叫分布式redis集群，可以有多个master，数据分片分布在这些master上。一种是redis-sentinel，只有一个master，各实例数据保持一致；

本文基于docker和redis-sentinel的高可用redis集群搭建。

![img](https://clyhs.github.io/images/redis/redis01.png)

*单个redis-sentinel进程来监控redis集群是不可靠的，由于redis-sentinel本身也有single-point-of-failure-problem(单点问题)，当出现问题时整个redis集群系统将无法按照预期的方式切换主从。官方推荐：一个健康的集群部署，至少需要3个Sentinel实例。另外，redis-sentinel只需要配置监控redis master，而集群之间可以通过master相互通信。*

分别有3个Sentinel节点，1个主节点，2个从节点组成一个Redis Sentinel集群。

| name         | ip           | port  |
| ------------ | ------------ | ----- |
| redis-master | 192.168.0.7  | 6300  |
| redis-slave1 | 192.168.0.16 | 6301  |
| redis-slave2 | 192.168.0.16 | 6302  |
| sentinel1    | 192.168.0.7  | 26000 |
| sentinel2    | 192.168.0.16 | 26001 |
| sentinel3    | 192.168.0.16 | 26002 |

### docker 部署redis

在192.168.0.7上运行

```
docker run -it --name redis-master --network host -d redis --appendonly yes --port 6300
```

在192.168.0.16上运行

```
docker run -it --name redis-slave1 --network host -d redis --appendonly yes --port 6301 --slaveof 192.168.0.7 6300

docker run -it --name redis-slave2 --network host -d redis --appendonly yes --port 6302 --slaveof 192.168.0.7 6300
```

### 搭建sentinel环境

#### 配置文件

通过网络下载sentinel.conf文件

```
wget http://download.redis.io/redis-stable/sentinel.conf
```

下载到**/app/app/**目录下

并复制sentinel1.conf，sentinel2.conf，sentinel3.conf

```
# mymaster:自定义集群名，如果需要监控多个redis集群，只需要配置多次并定义不同的<master-name> <master-redis-ip>:主库ip <master-redis-port>:主库port <quorum>:最小投票数，由于有三台redis-sentinel实例，所以可以设置成2
sentinel monitor mymaster <master-redis-ip> <master-redis-port> <quorum>
	
# 注：redis主从库搭建的时候，要么都不配置密码(这样下面这句也不需要配置)，不然都需要设置成一样的密码
sentinel auth-pass mymaster redispassword
	
# 添加后台运行
daemonize yes
```

内容如下：

**sentinel1.conf**

```
port 26000
daemonize yes
dir "/tmp"
sentinel monitor mymaster 192.168.0.7 6300 2

sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

**sentinel2.conf**

```
port 26001
daemonize yes
dir "/tmp"
sentinel monitor mymaster 192.168.0.7 6300 2

sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

**sentinel3.conf**

```
port 26002
daemonize yes
dir "/tmp"
sentinel monitor mymaster 192.168.0.7 6300 2

sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

#### 方法一

##### 搭建sentinel主节点

在192.168.0.7上运行

```
docker run -it --name redis-sentinel1 --network host -v /app/app/sentinel1.conf:/usr/local/etc/redis/sentinel.conf -d redis /bin/bash

```

##### 搭建sentinel从节点

在192.168.0.16上运行

```
docker run -it --name redis-sentinel2 --network host -v /app/app/sentinel2.conf:/usr/local/etc/redis/sentinel.conf -d redis /bin/bash

docker run -it --name redis-sentinel3 --network host -v /app/app/sentinel3.conf:/usr/local/etc/redis/sentinel.conf -d redis /bin/bash

```

*分别进入以上三个容器启动redis-sentinel：*

```
docker exec -it redis-sentinel(x) bash
redis-sentinel /usr/local/etc/redis/sentinel.conf
或
redis-server /usr/local/etc/redis/sentinel.conf --sentinel
```

#### 方法二

也可以一步完成，在/app/app/建目录sentinel01,sentinel02,sentinel03,目录结构如*

```
sentinel01/data
          /conf/sentinel1.conf
sentinel02/data
          /conf/sentinel2.conf
sentinel03/data
          /conf/sentinel3.conf
```

*然后依次执行*

```
docker run -d --network host --name sentinel1 \
-v /app/app/sentinel01/data:/var/redis/data \
-v /app/app/sentinel01/conf:/usr/local/etc/redis/sentinel.conf \
redis \ 
/usr/local/etc/redis/sentinel.conf --sentinel

```



#### 查询是否成功

在192.168.0.7或192.168.0.16上执行

```
redis-cli -p 26000或26001、26002
sentinel master mymaster或sentinel slaves mymaster

如下：
[root@centoss2 app]# redis-cli -p 26000
127.0.0.1:26000> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "192.168.0.7"
 5) "port"
 6) "6300"
 7) "runid"
 8) "f2eee0697508217b9dbd151feca340587aee87d7"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "395"
19) "last-ping-reply"
20) "395"
21) "down-after-milliseconds"
22) "30000"
23) "info-refresh"
24) "2598"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "62813"
29) "config-epoch"
30) "0"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "1"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"

```



*参考：https://blog.csdn.net/OneZhous/article/details/80679352*



