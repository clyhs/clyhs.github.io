---
title: lvs+keepalived+nginx服务搭建过程
date: 2018-03-13 09:47:35
category: lvs
tags: [nginx,lvs,keepalived]
---

### 准备环境
| 服务 | 地址 |
|--------|--------|
|   nginx1     |    192.168.0.7:80    |
|nginx2|192.168.0.16:80|
|VIP|192.168.0.80|

两台服务器都是centos7

### 分别安装环境,ipvsadm,keepalived 
1、安装环境依赖包,两台服务器都安装
```
# yum install -y gcc gcc-c++ makepcre pcre-devel kernel-devel openssl-devel libnl-devel popt-devel popt-static
```
2、安装 ipvsadm 
```
# wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz
# tar zxf ipvsadm-1.26.tar.gz
# cd ipvsadm-1.26
# make ; make install
```
有系统有带的可以不安装
3、安装 keepalived
```
# tar xf keepalived-1.2.12.tar.gz
# cd keepalived-1.2.12
# ./configure --sysconf=/etc --sbindir=/usr/sbin/
# make && make install
# ln -s /usr/local/sbin/keepalived /sbin/keepalived
# chkconfig --add keepalived
# chkconfig --level 2345 keepalived on
```
* 若/usr/src/kernel目录下没有内核目录，则需要安装内核开发包
yum install -y kernel-devel安装
yum install –y libnfnetlink-devel   (libnfnetlink-devel-1.0.1-4.el7.x86_64.rpm)
* CentOS7跟CentOS6的头文件路径有差别，把最后一个参数去掉
* CentOS6下面编译
./configure --sysconf=/etc --with-kernel-dir=/usr/src/kernels/2.6.32-504.23.4.el6.x86_64

### 配置keepalived的主从文件
* 192.168.0.16(MASTER)

```
! Configuration File for keepalived

global_defs {
   router_id MASTER
}

vrrp_script chk_nginx {  #检测nginx服务是否在运行有很多方式，比如进程，用脚本检测等等
   script "killall -0 nginx"  #用shell命令检查nginx服务是否存在
   interval 1  #时间间隔为1秒检测一次
   weight -2   #当nginx的服务不存在了，就把当前的权重-2
   fall 2      #测试失败的次数
   rise 1      #测试成功的次数
}

vrrp_instance VI_1 {
    state MASTER
    interface eno16777984
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.80
    }
    track_script {
        chk_nginx   #引用上面的vrrp_script定义的脚本名称
    }
}

virtual_server 192.168.0.80 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0 
    persistence_timeout 5
    protocol TCP

    real_server 192.168.0.16 80 {
        weight 1
        #HTTP_GET {
        #    url {
        #      path /
        #  status_code 200
        #    }
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.0.7 80 {
        weight 1
        #HTTP_GET {
        #    url {
        #      path /
        #  status_code 200
        #    }
        TCP_CHECK {    
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
```
* 192.168.0.7(BACKUP)

```
! Configuration File for keepalived

global_defs {
   router_id BACKUP
}
vrrp_script chk_nginx {  #检测nginx服务是否在运行有很多方式，比如进程，用脚本检测等等
   script "killall -0 nginx"  #用shell命令检查nginx服务是否存在
   interval 1  #时间间隔为1秒检测一次
   weight -2   #当nginx的服务不存在了，就把当前的权重-2
   fall 2      #测试失败的次数
   rise 1      #测试成功的次数
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens160
    virtual_router_id 51
    priority 80
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.80
    }
    track_script {
        chk_nginx   #引用上面的vrrp_script定义的脚本名称
    }
}

virtual_server 192.168.0.80 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0 
    persistence_timeout 5
    protocol TCP

    real_server 192.168.0.16 80 {
        weight 1
        #HTTP_GET {
        #    url {
        #      path /
        #  status_code 200
        #    }
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }

    real_server 192.168.0.7 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}


```

### 为16与07两台服务器添加VIP
脚本(realserver.sh)如下：
```
#!/bin/bash

VIP='192.168.0.80'

. /etc/init.d/functions

case "$1" in
  start)
    /sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
    echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
    echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
    echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
    echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
    echo " LVS Real-Server Start Success"
      ;;
    stop)
     /sbin/ifconfig lo:0 down
     echo "0" > /proc/sys/net/ipv4/conf/lo/arp_ignore
     echo "0" > /proc/sys/net/ipv4/conf/lo/arp_announce
     echo "0" > /proc/sys/net/ipv4/conf/all/arp_ignore
     echo "0" > /proc/sys/net/ipv4/conf/all/arp_announce
     echo " LVS Real-Server Stop Success"
      ;;
       *)
     echo "Usage: $0 ( start | stop )"
     exit 1
esac

```
### 启动服务 
1、启动16与07两台nginx
```
# service nginx start
```
2、分别在16与07执行脚本realserver.sh
```
# ./realserver.sh
```
启动后可以输入ifconfig看到
```
lo:0: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 192.168.0.80  netmask 255.255.255.255
        loop  txqueuelen 1  (Local Loopback)
```

3、分别启动16与07的keepalived
```
# service keepalived start
```
```
[root@centoss1 home]# ipvsadm -ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.80:80 rr persistent 5
  -> 192.168.0.7:80               Route   1      0          0         
  -> 192.168.0.16:80              Route   1      0          0  
```

4、访问服务
http://192.168.0.80

5、关掉两台服务器中的一个nginx
比如关掉16
日志如下
```
Mar 13 08:42:14 centoss1 Keepalived_vrrp[16789]: VRRP_Instance(VI_1) Effective priority = 98
Mar 13 08:42:14 centoss1 Keepalived_vrrp[16789]: pid 17221 exited with status 1
Mar 13 08:42:15 centoss1 Keepalived_vrrp[16789]: pid 17223 exited with status 1
Mar 13 08:42:16 centoss1 Keepalived_vrrp[16789]: pid 17225 exited with status 1
Mar 13 08:42:17 centoss1 Keepalived_vrrp[16789]: pid 17227 exited with status 1
Mar 13 08:42:17 centoss1 Keepalived_healthcheckers[16788]: Error connecting server [192.168.0.16]:80.
Mar 13 08:42:18 centoss1 Keepalived_vrrp[16789]: pid 17229 exited with status 1
Mar 13 08:42:19 centoss1 Keepalived_vrrp[16789]: pid 17231 exited with status 1
Mar 13 08:42:20 centoss1 Keepalived_vrrp[16789]: pid 17233 exited with status 1
Mar 13 08:42:20 centoss1 Keepalived_healthcheckers[16788]: Error connecting server [192.168.0.16]:80.
Mar 13 08:42:21 centoss1 Keepalived_vrrp[16789]: pid 17235 exited with status 1
Mar 13 08:42:22 centoss1 Keepalived_vrrp[16789]: pid 17237 exited with status 1
Mar 13 08:42:23 centoss1 Keepalived_vrrp[16789]: pid 17239 exited with status 1
Mar 13 08:42:23 centoss1 Keepalived_healthcheckers[16788]: Error connecting server [192.168.0.16]:80.
Mar 13 08:42:24 centoss1 Keepalived_vrrp[16789]: pid 17241 exited with status 1
Mar 13 08:42:25 centoss1 Keepalived_vrrp[16789]: pid 17243 exited with status 1
Mar 13 08:42:26 centoss1 Keepalived_vrrp[16789]: pid 17245 exited with status 1
Mar 13 08:42:26 centoss1 Keepalived_healthcheckers[16788]: Error connecting server [192.168.0.16]:80.
Mar 13 08:42:26 centoss1 Keepalived_healthcheckers[16788]: Check on service [192.168.0.16]:80 failed after 3 retry.
Mar 13 08:42:26 centoss1 Keepalived_healthcheckers[16788]: Removing service [192.168.0.16]:80 from VS [192.168.0.80]:80
Mar 13 08:42:27 centoss1 Keepalived_vrrp[16789]: pid 17247 exited with status 1
Mar 13 08:42:28 centoss1 Keepalived_vrrp[16789]: pid 17249 exited with status 1
Mar 13 08:42:29 centoss1 Keepalived_vrrp[16789]: pid 17251 exited with status 1
Mar 13 08:42:30 centoss1 Keepalived_vrrp[16789]: pid 17253 exited with status 1
Mar 13 08:42:31 centoss1 systemd: Starting nginx - high performance web server...
Mar 13 08:42:31 centoss1 nginx: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 13 08:42:31 centoss1 nginx: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 13 08:42:31 centoss1 systemd: Started nginx - high performance web server.
Mar 13 08:42:31 centoss1 Keepalived_vrrp[16789]: VRRP_Script(chk_nginx) succeeded
Mar 13 08:42:32 centoss1 Keepalived_vrrp[16789]: VRRP_Instance(VI_1) Effective priority = 100
Mar 13 08:42:32 centoss1 Keepalived_healthcheckers[16788]: HTTP status code success to [192.168.0.16]:80 url(1).
Mar 13 08:42:32 centoss1 Keepalived_healthcheckers[16788]: Remote Web server [192.168.0.16]:80 succeed on service.
Mar 13 08:42:32 centoss1 Keepalived_healthcheckers[16788]: Adding service [192.168.0.16]:80 to VS [192.168.0.80]:80

```
