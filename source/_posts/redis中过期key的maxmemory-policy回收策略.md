---
title: redis中过期key的maxmemory-policy回收策略
date: 2019-06-11 16:19:58
category: redis
tags: redis
---

### redis内存问题

Redis内存太大，会造成redis挂掉

#### 最大内存maxmemory

maxmemory设置最大内存，达到最大内存设置后，redis根据maxmemory-policy配置的策略来清理数据 释放空间

#### 六种maxmemory-policy

* noeviction:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）
* allkeys-lru: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。
* volatile-lru: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放
* allkeys-random: 回收随机的键使得新添加的数据有空间存放。
* volatile-random: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。
* volatile-ttl: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。



#### 设置方式

```shell
config set maxmemory-policy volatile-lru 
```
设置最大的内存
```shell
config set maxmemory 12884901888  # 12*1024*1024*1024 12G
CONFIG SET maxmemory 12288MB      # 12* 1024
```



