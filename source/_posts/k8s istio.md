---
title: k8s istio
date: 2017-04-10 15:38:37
category: k8s
tags: istio
---

## istio 安装

istio-1.14.3.tar.gz

```
mv istio-1.14.3.tar.gz /usr/local/
tar -zvxf istio-1.14.3
cd ~
vi .bash_profile
# 开始
export ISTIO_HOME=/usr/local/istio-1.14.3
export PATH=$PATH:$ISTIO_HOME/bin
# 结束
source .bash_profile
#安装
istioctl install --set profile=demo -y
✔ Istio core installed                         
✔ Istiod installed                          
✔ Ingress gateways installed                              
✔ Egress gateways installed            
✔ Installation complete                                                                                                         Making this installation the default for injection and validation.

#给命名空间添加标签，指示 Istio 在部署应用的时候，自动注入 Envoy 边车代理：
kubectl label namespace default istio-injection=enabled

kubectl get svc -n istio-system
NAME                                    READY   STATUS    RESTARTS   AGE
istio-egressgateway-68cbb757cc-jdc8l    1/1     Running   0          8m8s
istio-ingressgateway-6fbd8dddbf-2b25p   1/1     Running   0          8m8s
istiod-6fcfb4fbd9-6jklv                 1/1     Running   0          10m
```

