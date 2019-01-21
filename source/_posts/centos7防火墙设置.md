---
title: centos7防火墙设置
date: 2019-01-21 21:12:02
category: firewall
tags: [centos7,firewall]
---
### centos7防火墙设置
1、firewalld的基本使用
启动： systemctl start firewalld
关闭： systemctl stop firewalld
查看状态： systemctl status firewalld 
开机禁用  ： systemctl disable firewalld
开机启用  ： systemctl enable firewalld

2、配置firewalld-cmd

查看版本： firewall-cmd --version
查看帮助： firewall-cmd --help
显示状态： firewall-cmd --state
查看所有打开的端口： firewall-cmd --zone=public --list-ports
更新防火墙规则： firewall-cmd --reload
查看区域信息:  firewall-cmd --get-active-zones
查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0
拒绝所有包：firewall-cmd --panic-on
取消拒绝状态： firewall-cmd --panic-off
查看是否拒绝： firewall-cmd --query-panic

3、开启一个端口
添加
```
firewall-cmd --zone=public --add-port=80/tcp --permanent    （--permanent永久生效，没有此参数重启后失效）
```
重新载入
```
firewall-cmd --reload
```
查看
```
firewall-cmd --zone= public --query-port=80/tcp
```
删除
```
firewall-cmd --zone= public --remove-port=80/tcp --permanent
```

4、指定IP与端口
```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.40.107.11" port protocol="tcp" port="2375" accept"
```
5、删除规则
```
firewall-cmd --permanent --remove-rich-rule="rule family="ipv4" source address="10.40.107.11" port protocol="tcp" port="2375" accept"
```
6、然后对指定的IP开放指定的端口段
```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="10.40.107.11" port protocol="tcp" port="30000-31000" accept"
```