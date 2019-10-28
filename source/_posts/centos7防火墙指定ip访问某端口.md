---
title: centos7防火墙指定ip访问某端口
date: 2019-06-25 09:29:44
category: iptables
tags: [docker,centos7,iptables]
---

docker 启动的服务想用防火墙来限制ip访问，怎么做？

 ### 防火墙命令

systemctl enable firewalld 允许防火墙

systemctl disable firewalld 禁止防火墙

systemctl start firewalld 开启防火墙

systemctl stop firewalld 停止防火墙



#### 先开启防火墙

```shell
systemctl enable firewalld
systemctl start firewalld
```

查看防火墙规则

```
iptables -L
```

```shell
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            
INPUT_direct  all  --  anywhere             anywhere            
INPUT_ZONES_SOURCE  all  --  anywhere             anywhere            
INPUT_ZONES  all  --  anywhere             anywhere            
DROP       all  --  anywhere             anywhere             ctstate INVALID
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            
FORWARD_direct  all  --  anywhere             anywhere            
FORWARD_IN_ZONES_SOURCE  all  --  anywhere             anywhere            
FORWARD_IN_ZONES  all  --  anywhere             anywhere            
FORWARD_OUT_ZONES_SOURCE  all  --  anywhere             anywhere            
FORWARD_OUT_ZONES  all  --  anywhere             anywhere            
DROP       all  --  anywhere             anywhere             ctstate INVALID
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
OUTPUT_direct  all  --  anywhere             anywhere            

Chain FORWARD_IN_ZONES (1 references)
target     prot opt source               destination         
FWDI_public  all  --  anywhere             anywhere            [goto] 
FWDI_public  all  --  anywhere             anywhere            [goto] 

Chain FORWARD_IN_ZONES_SOURCE (1 references)
target     prot opt source               destination         

Chain FORWARD_OUT_ZONES (1 references)
target     prot opt source               destination         
FWDO_public  all  --  anywhere             anywhere            [goto] 
FWDO_public  all  --  anywhere             anywhere            [goto] 

Chain FORWARD_OUT_ZONES_SOURCE (1 references)
target     prot opt source               destination         

Chain FORWARD_direct (1 references)
target     prot opt source               destination         

Chain FWDI_public (2 references)
target     prot opt source               destination         
FWDI_public_log  all  --  anywhere             anywhere            
FWDI_public_deny  all  --  anywhere             anywhere            
FWDI_public_allow  all  --  anywhere             anywhere            
ACCEPT     icmp --  anywhere             anywhere            

Chain FWDI_public_allow (1 references)
target     prot opt source               destination         

Chain FWDI_public_deny (1 references)
target     prot opt source               destination         

Chain FWDI_public_log (1 references)
target     prot opt source               destination         

Chain FWDO_public (2 references)
target     prot opt source               destination         
FWDO_public_log  all  --  anywhere             anywhere            
FWDO_public_deny  all  --  anywhere             anywhere            
FWDO_public_allow  all  --  anywhere             anywhere            

Chain FWDO_public_allow (1 references)
target     prot opt source               destination         

Chain FWDO_public_deny (1 references)
target     prot opt source               destination         

Chain FWDO_public_log (1 references)
target     prot opt source               destination         

Chain INPUT_ZONES (1 references)
target     prot opt source               destination         
IN_public  all  --  anywhere             anywhere            [goto] 
IN_public  all  --  anywhere             anywhere            [goto] 

Chain INPUT_ZONES_SOURCE (1 references)
target     prot opt source               destination         

Chain INPUT_direct (1 references)
target     prot opt source               destination         

Chain IN_public (2 references)
target     prot opt source               destination         
IN_public_log  all  --  anywhere             anywhere            
IN_public_deny  all  --  anywhere             anywhere            
IN_public_allow  all  --  anywhere             anywhere            
ACCEPT     icmp --  anywhere             anywhere            

Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh ctstate NEW

Chain IN_public_deny (1 references)
target     prot opt source               destination         

Chain IN_public_log (1 references)
target     prot opt source               destination         

Chain OUTPUT_direct (1 references)
target     prot opt source               destination 
```

#### 重启docker

如果应用部署在docker中，那么启动防火墙，必须重启docker，**docker会自动**把端口写入防火墙

再次查看防火墙，*iptables -L*，可以看到多了一条记录

```shell
......
Chain DOCKER (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:2375
......
```

因为docker 启动后，会自己启动时面已经加载的shipyard-proxy的容器，端口为2375，而anywhere表示任何机器都可以访问。

#### 对外开放端口

##### 对所有IP都开放端口,不限制ip

如tomcat:8870

```shell
firewall-cmd --zone=public --add-port=8870/tcp --permanent   //添加端口
#如果想加入其它端口8080可以为：
#firewall-cmd --zone=public --add-port=8080/tcp --permanent  

firewall-cmd --reload                                        //必须重加载，才生效

```

再次查看*iptables -L*，多了一条记录

```shell
......
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh ctstate NEW
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:8870 ctstate NEW
......
```

这样你就可以在浏览器访问8870

##### 指定ip访问redis的6300端口

一般启动docker的redis服务，docker写入防火墙是不限制ip，比如通过docker启动mysql，zookeeper等

通过*iptables -L*自动添加了如下：

```shell
......
Chain DOCKER (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             172.17.0.2           tcp dpt:2375
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:9066
ACCEPT     tcp  --  anywhere             172.17.0.3           tcp dpt:8066
ACCEPT     tcp  --  anywhere             172.17.0.4           tcp dpt:mysql
ACCEPT     tcp  --  anywhere             172.17.0.5           tcp dpt:sun-as-jmxrmi
ACCEPT     tcp  --  anywhere             172.17.0.5           tcp dpt:eforward
ACCEPT     tcp  --  anywhere             172.17.0.6           tcp dpt:61616
......
```

但是你发现，**哨兵的端口**没有在里面，因为哨兵是在DOCKER容器里面启动的，不在这里，需要手动添加

```shell
firewall-cmd --zone=public --add-port=26000/tcp --permanent
firewall-cmd --reload

# iptables -L查看
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh ctstate NEW
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:26000 ctstate NEW
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:8870 ctstate NEW

#通过telnet可以访问26000
```

再查看一下，里面没有redis的端口规则，因为redis只能指定Ip访问，所以添加如下：

**指定192.168.0.60能访问6300**

```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.60" port protocol="tcp" port="6300" accept"

firewall-cmd --reload

#iptables -L
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.0.60         anywhere             tcp dpt:bmc-grx ctstate NEW
```



通过其它机器的telnet验证

在192.168.0.60

```
[root@localhost ~]# telnet 192.168.0.54 6300
Trying 192.168.0.54...
Connected to 192.168.0.54.
Escape character is '^]'.
```

表示成功

在192.168.0.16

```
[root@appserver1 ~]#  telnet 192.168.0.54 6300
Trying 192.168.0.54...
telnet: connect to address 192.168.0.54: No route to host
```

表示失败.

这样就成功了。

**如果我也想把192.168.0.16加入访问192.168.0.54的6300**

同样如下：

```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.0.16" port protocol="tcp" port="6300" accept"

firewall-cmd --reload

#iptables -L 查看
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.0.16         anywhere             tcp dpt:bmc-grx ctstate NEW
ACCEPT     tcp  --  192.168.0.60         anywhere             tcp dpt:bmc-grx ctstate NEW
```

在192.168.0.16

```shell
[root@appserver1 ~]# telnet 192.168.0.54 6300
Trying 192.168.0.54...
Connected to 192.168.0.54.
Escape character is '^]'.
```

这样就可以成功了。

**如果上面无效、需要增加下面操作**

```
`docker 会在iptables上加上自己的转发规则，如果直接在input链上限制端口是没有效果的。这就需要限制docker的转发链上的DOCKER表。``# 查询DOCKER表并显示规则编号``iptables -L DOCKER -n --line-number``# 修改对应编号的iptables 规则，这里添加了允许访问ip的限制``iptables -R DOCKER 5 -p tcp -m tcp -s 192.168.1.0``/24` `--dport 3000 -j ACCEPT`
```















