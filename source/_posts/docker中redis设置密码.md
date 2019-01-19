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


