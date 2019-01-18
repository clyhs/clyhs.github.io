---
title: docker中mysql的数据导出导入
date: 2019-01-18 20:18:34
category: docker
tags: docker
---
### docker启动mysql
```
docker run --name pascloud_mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7 --lower_case_table_names=1 --character-set-server=utf8 --collation-server=utf8
```
1、导出数据
```
docker exec -it pascloud_mysql mysqldump -uroot -proot pascloud > /opt/pascloud.sql
```
2、复制文件到容器里面
```
docker cp /opt/pascloud.sql pascloud_mysql:/opt/pascloud.sql
```
3、进入容器
```
docker exec -it pascloud_mysql bash
```

4、还原数据
```
mysql -uroot -proot
>create database pascloud;
>use pascloud;
>source /opt/pascloud.sql
```


