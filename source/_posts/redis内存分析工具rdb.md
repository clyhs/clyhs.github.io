---
title: redis内存分析工具rdb
date: 2019-06-11 18:22:40
category: redis
tags: redis
---

### rdb

redis-rdb-tools是由Python写的用来分析Redis的rdb快照文件用的工具，它可以把rdb快照文件生成json文件或者生成报表用来分析Redis的使用详情、使用标准的diff工具比较两个dump文件，总之是比较实用的工具，至于安装可以通过Python的pip来安装

#### 安装

```shell
yum -y install python-pip python-redis

正在解决依赖关系
--> 正在检查事务
---> 软件包 python2-pip.noarch.0.8.1.2-8.el7 将被 安装
---> 软件包 python2-redis.noarch.0.2.10.6-1.el7 将被 安装

pip install rdbtools
```

rdb常用的几个参数：

```
  -h，--help                         显示此帮助信息并退出
  -c FILE, --command=FILE            要执行的命令，转出的类型。有效的命令是json(转成json)，diff（差异比对），justkeys（仅有key），justkeyvals（仅有value），memory（内存报告）， protocol（导成添加指令）

  -f FILE, --file=FILE               文件输出文件
  -n DBS, --db=DBS                   DBS数据库号码。可以提供多个数据库。如果未指定，则将包括所有数据库。
  -k KEYS, --key=KEYS                显示出的Redis的key。这可以是一个正则表达式
  -o NOT_KEYS, --not-key=NOT_KEYS    显示忽略的key。这可以是一个正则表达式
  -t TYPES, --type=TYPES             显示出数据类型，key的数据类型string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型），如果没有指定，全部数据类型将被返回

  -b BYTES, --bytes=BYTES            将内存输出限制为大于或等于的key，单位字节
  -l LARGEST, --largest=LARGEST      将内存输出限制为只有前N个key（按大小）
  -e ESCAPE, --escape=ESCAPE         将字符串转义为编码：raw（默认），print，utf8或base64。
```

#### 运用

把匹配到的key的key和value用json的格式打印

```
Example : rdb --command json -k "user.*" /var/redis/6379/dump.rdb
```



把Redis的rdb内存分析报告生成csv文件，可以使用awk等相关工具分析，也可以导入数据库用以分析

rdb -c memory dump.rdb > memory.csv

**创建SQL**

```
CREATE TABLE `memory` (
  `database` int(128) DEFAULT NULL,
  `type` varchar(128) DEFAULT NULL,
  `KEY` varchar(128) DEFAULT NULL,
  `size_in_bytes` bigint(20) DEFAULT NULL,
  `encoding` varchar(128) DEFAULT NULL,
  `num_elements` bigint(20) DEFAULT NULL,
  `len_largest_element` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`KEY`)
);
```

将memory.csv导入数据库中，数据如下：

| database | type   | key                   | size_in_bytes | encoding | num_elements | len_largest_element | expiry |
| -------- | ------ | --------------------- | ------------- | -------- | ------------ | ------------------- | ------ |
| 0        | string | dn1.jgyjhgl           | 41016         | string   | 38035        | 38035               |        |
| 0        | string | dn0.hydkjxgzhzcx      | 14400         | string   | 12712        | 12712               |        |
| 0        | string | dn0.zxllgl.para       | 16448         | string   | 15143        | 15143               |        |
| 0        | string | dn0.zhjstzdr2         | 16440         | string   | 16081        | 16081               |        |
| 0        | string | dn1.zhsthyrjrlye.para | 10304         | string   | 8365         | 8365                |        |
| 0        | string | dn0.cdkqxlrfx         | 20536         | string   | 19333        | 19333               |        |
| 0        | string | dn1.eckhjxfpdr        | 14392         | string   | 13945        | 13945               |        |
| 0        | string | dn0.bldkqsjxcx.para   | 16448         | string   | 14736        | 14736               |        |



