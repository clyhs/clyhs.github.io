---
title: redis的持久化策略
date: 2019-06-11 15:54:37
category: redis
tags: redis
---

### redis持久化

Redis 有两种持久化方案，RDB （Redis DataBase）和 AOF （Append Only File）

#### rdb方案

RDB 是 Redis 默认的持久化方案。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个dump.rdb文件。Redis 重启会通过加载dump.rdb文件恢复数据。

*在redis.conf中*

```shell
#   save ""
save 900 1
save 300 10
save 60 10000
```

上面的意思是：

*900秒内有1个更改，300秒内有10个更改以及60秒内有10000个更改，则将内存中的数据快照写入磁盘*。

*若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释。*

RDB文件一般采用默认的 dump.rdb

*在redis.conf中*

```shell
dbfilename dump.rdb   #
rdbcompression yes    #配置存储至本地数据库时是否压缩数据，默认为yes
```

redis的恢复

将dump.rdb 文件拷贝到redis的安装目录的bin目录下，重启redis服务即可。

**优缺点：**

优点：
1 适合大规模的数据恢复。
2 如果业务对数据完整性和一致性要求不高，RDB是很好的选择。

缺点：
1 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
2 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍），最后再将临时文件替换之前的备份文件。
所以Redis 的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。

#### aof方案

AOF ：Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个**写操作**，并**追加**到文件中

```shell
appendonly yes                             #开启需要手动把no改为yes
appendfilename "appendonly.aof"            #指定本地数据库文件名，默认值为 appendonly.aof
# appendfsync always
appendfsync everysec                       #出厂默认推荐，每秒异步记录一次（默认值）     
# appendfsync no
#配置重写触发机制
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

*根据配置文件触发，可以是每次执行触发，可以是每秒触发，可以不同步。*

**数据恢复**

将appendonly.aof 文件拷贝到redis的安装目录的bin目录下，重启redis服务即可。但在实际开发中，可能因为某些原因导致appendonly.aof 文件格式异常，从而导致数据还原失败，可以通过命令redis-check-aof --fix appendonly.aof 进行修复

如：

```shell
redis-check-aof --fix appendonly.aof 
```

**优缺点：**

优点：数据的完整性和一致性更高
缺点：因为AOF记录的内容多，文件会越来越大，数据恢复也会越来越慢。



*参考：<https://www.cnblogs.com/itdragon/p/7906481.html>*





