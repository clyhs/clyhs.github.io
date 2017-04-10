---
title: shipyard集群部署
date: 2017-04-10 21:05:14
category: shipyard
tags: shipyard
---
### 准备服务器centos7
| 服务器 | IP |
|--------|--------|
|    master   |     192.168.0.100   |
|    slave01   |     192.168.0.101   |
|    slave02   |     192.168.0.102  |

### 修改HOST
192.168.0.100  master
192.168.0.101  slave01
192.168.0.102  slave02

### 关掉防火墙
```

$ service iptables stop
$ service network restart 
$ systemctl stop firewalld.service
$ systemctl disable firewalld.service
$ sysctl net.ipv6.conf.all.disable_ipv6=1

$ iptables -P INPUT ACCEPT
$ iptables -P OUTPUT ACCEPT
$ iptables -P FORWARD ACCEPT

```

### master
```
$ docker pull swarm 
$ docker pull shipyard/shipyard
$ docker pull rethinkdb
$ docker pull microbox/etcd
$ docker pull ehazlett/curl 
$ docker pull shipyard/docker-proxy

$ docker run -ti -d --restart=always --name shipyard-rethinkdb rethinkdb

$ docker run -ti -d -p 4001:4001 -p 7001:7001 --restart=always --name shipyard-discovery microbox/etcd -name discovery

$ docker run -ti -d -p 2375:2375 --hostname=$HOSTNAME --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest


$ docker run -ti -d --restart=always --name shipyard-swarm-manager swarm:latest manage --host tcp://0.0.0.0:3375 etcd://192.168.0.16:4001

$ docker run -ti -d --restart=always --name shipyard-swarm-agent swarm:latest join --addr 192.168.0.16:2375 etcd://192.168.0.16:4001

$ docker run -ti -d --restart=always \
    --name shipyard-controller \
    --link shipyard-rethinkdb:rethinkdb \
    --link shipyard-swarm-manager:swarm \
    -p 6969:8080 \
    shipyard/shipyard:latest \
    server \
    -d tcp://swarm:3375

```
### slave01/02
```
$ docker  pull microbox/etcd
$ docker  pull shipyard/docker-proxy
$ docker  pull swarm 
$ docker  pull alpine

$ curl -sSL https://shipyard-project.com/deploy > docker.sh
$ export ACTION=node DISCOVERY=etcd://192.168.0.16:4001 && bash docker.sh
如果是用ZK作为注册中心
$ export ACTION=node DISCOVERY=zk://192.168.0.16:2181 && bash docker.sh 


```
