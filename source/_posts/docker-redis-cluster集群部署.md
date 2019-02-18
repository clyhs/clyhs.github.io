---
title: docker redis cluster集群部署
date: 2019-02-18 11:49:13
category: redis
tags: [redis,docker]
---

### docker部署redis集群

用两台虚拟机模拟6个节点，一台机器3个节点，创建出3 master、3 salve 环境。

| name   | ip           | port | remark |
| ------ | ------------ | ---- | ------ |
| redis1 | 192.168.0.16 | 8001 | master |
| redis2 | 192.168.0.16 | 8002 | master |
| redis3 | 192.168.0.16 | 8003 | master |
| redis4 | 192.168.0.7  | 8004 | slave  |
| redis5 | 192.168.0.7  | 8005 | slave  |
| redis6 | 192.168.0.7  | 8006 | slave  |

#### 准备镜像

publicisworldwide/redis-cluster:latest

inem0o/redis-trib:latest

```
docker pull publicisworldwide/redis-cluster
docker pull inem0o/redis-trib
```

#### 新建容器

在192.168.0.16新建/app/app/redis/docker-compose.yml

```
vi /app/app/redis/docker-compose.yml

version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001

 redis2:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002

 redis3:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
```

在192.168.0.7新建/app/app/redis/docker-compose.yml

```
vi /app/app/redis/docker-compose.yml

version: '3'
 redis4:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004

 redis5:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005

 redis6:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
```

*若同一台宿主机，不想使用host模式同一台，也可以把network_mode去掉，但就要加ports映射。
redis-cluster的节点端口共分为2种，一种是节点提供服务的端口，如6379；一种是节点间通信的端口，固定格式为：10000+6379。*

*如:*

```
docker-compose.yml

version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001
  ports:
    - '8001:8001'
    - '18001:18001'
    
 redis2:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002
  ports:
    - '8002:8002'
    - '18002:18002'
    
 redis3:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
  ports:
    - '8003:8003'
    - '18003:18003'
    
 redis4:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004
  ports:
    - '8004:8004'
    - '18004:18004'
    
 redis5:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005
  ports:
    - '8005:8005'
    - '18005:18005'
    
 redis6:
  image: publicisworldwide/redis-cluster
  restart: always
  volumes:
   - /app/app/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006
  ports:
    - '8006:8006'
    - '18006:18006'
```



#### 直接启动服务

```
窗口模式
docker-compose up
后台进程
docker-compose up -d

```

*以上镜像不能设置永久密码，其实redis一般是内网访问，可以不需密码。*

#### redis容器集群配置

##### 方法一、直接启动

不指定master

```
docker run --rm -it inem0o/redis-trib create --replicas 1 192.168.0.16:8001 192.168.0.16:8002 192.168.0.16:8003 192.168.0.7:8004 192.168.0.7:8005 192.168.0.7:8006

>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.0.16:8001
192.168.0.16:8002
192.168.0.16:8003
Adding replica 192.168.0.7:8004 to 192.168.0.16:8001
Adding replica 192.168.0.7:8005 to 192.168.0.16:8002
Adding replica 192.168.0.7:8006 to 192.168.0.16:8003
M: 0e594d4e46fef74366c0ce9f43f5d805af3bf639 192.168.0.16:8001
   slots:0-5460 (5461 slots) master
M: 0f86275fa7a965295153206c872d901d28461e6b 192.168.0.16:8002
   slots:5461-10922 (5462 slots) master
M: 25b5951be791956cc8aa7b93b8cecda25927d0f4 192.168.0.16:8003
   slots:10923-16383 (5461 slots) master
S: 9bd2ee334963a472a7ae183d69133c0cc0ede8d8 192.168.0.7:8004
   replicates 0e594d4e46fef74366c0ce9f43f5d805af3bf639
S: 2bd8d9a3e24b142d19bb4922f5aace9e60b9a16f 192.168.0.7:8005
   replicates 0f86275fa7a965295153206c872d901d28461e6b
S: c844c9277b34f685fc979dcdb47a1e71011a185a 192.168.0.7:8006
   replicates 25b5951be791956cc8aa7b93b8cecda25927d0f4
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 192.168.0.16:8001)
M: 0e594d4e46fef74366c0ce9f43f5d805af3bf639 192.168.0.16:8001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 25b5951be791956cc8aa7b93b8cecda25927d0f4 192.168.0.16:8003@18003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 9bd2ee334963a472a7ae183d69133c0cc0ede8d8 192.168.0.7:8004@18004
   slots: (0 slots) slave
   replicates 0e594d4e46fef74366c0ce9f43f5d805af3bf639
S: 2bd8d9a3e24b142d19bb4922f5aace9e60b9a16f 192.168.0.7:8005@18005
   slots: (0 slots) slave
   replicates 0f86275fa7a965295153206c872d901d28461e6b
M: 0f86275fa7a965295153206c872d901d28461e6b 192.168.0.16:8002@18002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: c844c9277b34f685fc979dcdb47a1e71011a185a 192.168.0.7:8006@18006
   slots: (0 slots) slave
   replicates 25b5951be791956cc8aa7b93b8cecda25927d0f4
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

##### 方法二、指定master

```
docker run --rm -it inem0o/redis-trib create  192.168.0.16:8001 192.168.0.16:8002 192.168.0.16:8003

检查集群

docker run --rm -it inem0o/redis-trib check 192.168.0.16:8001
>>> Performing Cluster Check (using node 192.168.0.16:8001)
M: dbb6b214215f70b6a9e7ce150d1898ebadbf90fc 192.168.0.16:8001
   slots:0-5460 (5461 slots) master
   0 additional replica(s)
M: 6915f5f4bb5df77c72a943a5afb1b619bdc0d937 192.168.0.16:8002@18002
   slots:5461-10922 (5462 slots) master
   0 additional replica(s)
M: d80ee99cea1a6050a16fee952b012df455b2fb07 192.168.0.16:8003@18003
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

```

添加从节点

```
docker run --rm -it inem0o/redis-trib add-node --slave --master-id dbb6b214215f70b6a9e7ce150d1898ebadbf90fc 192.168.0.7:8004 192.168.0.16:8001

docker run --rm -it inem0o/redis-trib add-node --slave --master-id 6915f5f4bb5df77c72a943a5afb1b619bdc0d937 192.168.0.7:8005 192.168.0.16:8002

docker run --rm -it inem0o/redis-trib add-node --slave --master-id d80ee99cea1a6050a16fee952b012df455b2fb07 192.168.0.7:8006 192.168.0.16:8003

查询三个主节点，都有一个从节点

docker run --rm -it inem0o/redis-trib info 192.168.0.16:8001
192.168.0.16:8001 (dbb6b214...) -> 0 keys | 5461 slots | 1 slaves.
192.168.0.16:8002@18002 (6915f5f4...) -> 0 keys | 5462 slots | 1 slaves.
192.168.0.16:8003@18003 (d80ee99c...) -> 0 keys | 5461 slots | 1 slaves.
[OK] 0 keys in 3 masters.
0.00 keys per slot on average.
```

#### 查看redis-trib使用

```
docker run --rm -it inem0o/redis-trib help
```

*1、create：创建集群
2、check：检查集群
3、info：查看集群信息
4、fix：修复集群
5、reshard：在线迁移slot
6、rebalance：平衡集群节点slot数量
7、add-node：将新节点加入集群
8、del-node：从集群中删除节点
9、set-timeout：设置集群节点间心跳连接的超时时间
10、call：在集群全部节点上执行命令
11、import：将外部redis数据导入集群*

### 自己创建镜像

#### 准备脚本

/app/app/redis/entrypoint.sh

vi entrypoint.sh

```
#!/bin/sh
#只作用于当前进程,不作用于其创建的子进程
set -e
#$0--Shell本身的文件名 $1--第一个参数 $@--所有参数列表
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
    sed -i 's/REDIS_PORT/'$REDIS_PORT'/g' /usr/local/etc/redis.conf
    chown -R redis .  #改变当前文件所有者
    exec gosu redis "$0" "$@"  #gosu是sudo轻量级”替代品”
fi

exec "$@"
```

/app/app/redis/redis.conf

vi redis.conf

```
#端口
port REDIS_PORT
#开启集群
cluster-enabled yes
#配置文件
cluster-config-file nodes.conf
cluster-node-timeout 5000
#更新操作后进行日志记录
appendonly yes
#设置主服务的连接密码
# masterauth
#设置从服务的连接密码
# requirepass
```

- requirepass和masterauth不能启用，否则redis-trib创建集群失败。
- protected-mode 保护模式是禁止公网访问，但是不能设置密码和bind ip。

编写Dockerfile文件

/app/app/redis/Dockerfile

vi Dockerfile

```
#基础镜像
FROM redis
#修复时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone
#环境变量
ENV REDIS_PORT 8000
#ENV REDIS_PORT_NODE 18000
#暴露变量
EXPOSE $REDIS_PORT
#EXPOSE $REDIS_PORT_NODE
#复制
COPY entrypoint.sh /usr/local/bin/
COPY redis.conf /usr/local/etc/
#for config rewrite
RUN chmod 777 /usr/local/etc/redis.conf
RUN chmod +x /usr/local/bin/entrypoint.sh
#入口
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
#命令
CMD ["redis-server", "/usr/local/etc/redis.conf"]
```

#### 创建镜像

```
cd /app/app/redis
docker build -t pascloud/redis-cluster:1.0 .

Sending build context to Docker daemon 4.608 kB
Step 1/11 : FROM redis
 ---> 0f55cf3661e9
Step 2/11 : RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 ---> Running in 5aa77cdb9d96
 ---> 67c21c2f65cc
Removing intermediate container 5aa77cdb9d96
Step 3/11 : RUN echo 'Asia/Shanghai' >/etc/timezone
 ---> Running in b08571fcaca3
 ---> 75c88a71c3e2
Removing intermediate container b08571fcaca3
Step 4/11 : ENV REDIS_PORT 8000
 ---> Running in 264dd35607f9
 ---> 237723740c53
Removing intermediate container 264dd35607f9
Step 5/11 : EXPOSE $REDIS_PORT
 ---> Running in 3ec733186488
 ---> d44c858fd07a
Removing intermediate container 3ec733186488
Step 6/11 : COPY entrypoint.sh /usr/local/bin/
 ---> 53cc6f8e9979
Removing intermediate container 253c851483a9
Step 7/11 : COPY redis.conf /usr/local/etc/
 ---> 849df9aace37
Removing intermediate container 1cb7033269a5
Step 8/11 : RUN chmod 777 /usr/local/etc/redis.conf
 ---> Running in c755b51d08ef
 ---> 232eff722727
Removing intermediate container c755b51d08ef
Step 9/11 : RUN chmod +x /usr/local/bin/entrypoint.sh
 ---> Running in ec35a6c8b212
 ---> 555f5497fb9c
Removing intermediate container ec35a6c8b212
Step 10/11 : ENTRYPOINT /usr/local/bin/entrypoint.sh
 ---> Running in 8b0dd921d40a
 ---> c377c55da021
Removing intermediate container 8b0dd921d40a
Step 11/11 : CMD redis-server /usr/local/etc/redis.conf
 ---> Running in 080d23a389c3
 ---> d281c0f4ded9
Removing intermediate container 080d23a389c3
Successfully built d281c0f4ded9
```

#### 创建网络

```
docker network create redis-cluster-net
```

#### 编写docker-compose.yml

在192.168.0.16新建/app/app/redis/docker-compose.yml

```
version: '3'

services:
 redis1:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8001/data:/data
  environment:
   - REDIS_PORT=8001
    
 redis2:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8002/data:/data
  environment:
   - REDIS_PORT=8002
    
 redis3:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8003/data:/data
  environment:
   - REDIS_PORT=8003
```

在192.168.0.7新建/app/app/redis/docker-compose.yml

```
version: '3'

services:
 redis4:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8004/data:/data
  environment:
   - REDIS_PORT=8004
    
 redis5:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8005/data:/data
  environment:
   - REDIS_PORT=8005
    
 redis6:
  image: pascloud/redis-cluster:1.0
  network_mode: host
  restart: always
  volumes:
   - /app/app/redis/8006/data:/data
  environment:
   - REDIS_PORT=8006

```

#### 启动

分别在16和07上执行

```
docker-compose up -d
```

在16上执行

```
docker run --rm -it inem0o/redis-trib create --replicas 1 192.168.0.16:8001 192.168.0.16:8002 192.168.0.16:8003 192.168.0.7:8004 192.168.0.7:8005 192.168.0.7:8006
```

**设置集群密码**

分别通过端口设置密码

```
[root@localhost src]# ./redis-cli -c -p 8001(8002...8006)
127.0.0.1:8001> config set masterauth 123456
OK
127.0.0.1:8001> config set requirepass 123456
OK
127.0.0.1:8001> config rewrite
(error) NOAUTH Authentication required.
127.0.0.1:8001> auth 123456
OK
127.0.0.1:8001> config rewrite
OK
```

**查看密码文件**

```
docker exec -it 容器ID /bin/bash
cat /usr/local/etc/redis.conf

root@centoss1:/data# cat /usr/local/etc/redis.conf
#端口
port 8002
#开启集群
cluster-enabled yes
#配置文件
cluster-config-file "nodes.conf"
cluster-node-timeout 5000
#更新操作后进行日志记录
appendonly yes
#设置主服务的连接密码
# masterauth
#设置从服务的连接密码
# requirepass
# Generated by CONFIG REWRITE
dir "/data"
masterauth "123456"
requirepass "123456"
```

