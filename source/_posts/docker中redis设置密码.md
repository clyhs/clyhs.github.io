---
title: docker中redis设置密码
date: 2019-01-19 14:48:39
category: docker
tags: [docker,redis]
---

### redis启动
```
docker run -d --name redis_test -p 6378:6379 --restart=always redis:latest redis-server --appendonly yes --requirepass "123456"
```
*redis-server --appendonly yes在容器执行redis-server启动命令，并打开redis持久化*
1、访问
```
docker exec -it c0de7caa5d1c redis-cli -a '123456'
```
或者
```
docker exec -it c0de7caa5d1c redis-cli -h 127.0.0.1 -p 6378 -a '123456'
```
```
127.0.0.1:6379> keys *
(empty list or set)
```
*c0de7caa5d1c为容器ID*

### redis挂载目录
* /home/domains/pascloud/redis/redis.conf:/etc/redis/redis.conf
* /home/domains/pascloud/redis/data:/data
```
docker run -d --privileged=true -p 6379:6379 -v /home/domains/pascloud/redis/redis.conf:/etc/redis/redis.conf -v /home/domains/pascloud/redis/data:/data --name pascloud_redis redis:latest redis-server /etc/redis/redis.conf --appendonly yes
```

/etc/redis/redis.conf 关键配置，让redis以指定的配置文件启动，而不是默认无配置启动。

--appendonly yes redis启动后开启数据持久化