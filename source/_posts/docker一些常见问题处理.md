---
title: docker一些常见问题处理
date: 2019-07-12 13:47:03
category: docker
tags: docker
---

### docker常见问题

1、docker启动后发现端口是*tcp6*的，无法访问，如：

```
tcp6       0      0 :::3306                 :::*                    LISTEN      9863/docker-proxy
```

解决方法：

配置/etc/sysctl.conf，添加以下：

```
#开户ipv4路由转发
net.ipv4.ip_forward=1
#关闭ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

让参数生效

```
sysctl -p
```

重启docker

```
systemctl restart docker
```

重启docker服务就可以解决。