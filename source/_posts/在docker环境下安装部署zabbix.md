---
title: 在docker环境下安装部署zabbix
date: 2017-10-27 16:26:47
category: zabbix
tags: [docker,zabbix]
---

### 安装MYSQL
此步略
### 安装zabbix/zabbix-server-mysql
```shell
# docker pull zabbix/zabbix-server-mysql
# docker run --name some-zabbix-server-mysql  -p 10051:10051 --net=host -e DB_SERVER_HOST="192.168.0.16" -e DB_SERVER_PORT=3306 -e MYSQL_USER="root" -e MYSQL_PASSWORD="123456" -d zabbix/zabbix-server-mysql

```

### 安装zabbix/zabbix-web-apache-mysql
```shell
# docker pull zabbix/zabbix-web-apache-mysql
# docker run --name some-zabbix-web-apache-mysql -p 8188:80  -e DB_SERVER_HOST="192.168.0.16" -e DB_SERVER_PORT=3306 -e MYSQL_USER="root" -e MYSQL_PASSWORD="123456" -e ZBX_SERVER_HOST="192.168.0.16" -e TZ="Asia/Shanghai" -d zabbix/zabbix-web-apache-mysql

```

### 安装zabbix/zabbix-agent
```shell
# docker pull zabbix/zabbix-agent
# docker run --name some-zabbix-agent -p 10050:10050 -e ZBX_HOSTNAME="192.168.0.16" -e ZBX_SERVER_HOST="192.168.0.16" -e ZBX_SERVER_PORT=10051 -d zabbix/zabbix-agent

```
### 访问WEB
帐号密码：Admin/zabbix
地址：http://192.168.0.16:8188
![img](https://clyhs.github.io/images/zabbix/zabbix01.png)

### 配置
![img](https://clyhs.github.io/images/zabbix/zabbix02.png)
![img](https://clyhs.github.io/images/zabbix/zabbix03.png)
![img](https://clyhs.github.io/images/zabbix/zabbix04.png)

