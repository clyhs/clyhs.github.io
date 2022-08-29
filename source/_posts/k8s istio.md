---
title: k8s istio
date: 2022-08-19 13:38:37
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

## 安装kiali

```
 kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.14/samples/addons/kiali.yaml
```

设置可视化界面kiali为外部访问模式并查询nodeport号

```
kubectl patch svc -n istio-system kiali -p '{"spec": {"type": "NodePort"}}'

kubectl describe svc -n istio-system kiali
Name:                     kiali
Namespace:                istio-system
Labels:                   app=kiali
                          app.kubernetes.io/instance=kiali
                          app.kubernetes.io/managed-by=Helm
                          app.kubernetes.io/name=kiali
                          app.kubernetes.io/part-of=kiali
                          app.kubernetes.io/version=v1.50.0
                          helm.sh/chart=kiali-server-1.50.0
                          version=v1.50.0
Annotations:              <none>
Selector:                 app.kubernetes.io/instance=kiali,app.kubernetes.io/name=kiali
Type:                     NodePort
IP Families:              <none>
IP:                       10.96.108.219
IPs:                      10.96.108.219
Port:                     http  20001/TCP
TargetPort:               20001/TCP
NodePort:                 http  31255/TCP
Endpoints:                172.31.131.167:20001
Port:                     http-metrics  9090/TCP
TargetPort:               9090/TCP
NodePort:                 http-metrics  32081/TCP
Endpoints:                172.31.131.167:9090
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

（1）**在istio上部署测试应用bookinfo**

```
1、开启sidecar自动注入
kubectl label namespace default istio-injection=enabled
2、使用kubectl部署bookinfo并创建网关
kubectl apply -f /usr/local/istio-1.14.3/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f /usr/local/istio-1.14.3/samples/bookinfo/networking/bookinfo-gateway.yaml 
3、设置访问网关的 INGRESS_HOST 和 INGRESS_PORT 变量，查询是否有外部负载均衡器
kubectl get svc istio-ingressgateway -n istio-system

EXTERNAL-IP是none或是pending说明没有问题
修改istio的网关为nodeport模式　

kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'
采用nodeport方式暴露istio-ingressgateway

export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')

export SECURE_INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="https")].nodePort}')

#获取 ingress IP 地址：
export INGRESS_HOST=$(kubectl get po -l istio=ingressgateway -n istio-system -o 'jsonpath={.items[0].status.hostIP}')

查询URL

echo $INGRESS_HOST:$INGRESS_PORT

http://192.168.2.220:30290/productpage
```

（2）**[Prometheus](https://prometheus.io/)**

```
kubectl apply -f /usr/local/istio-1.14.3/samples/addons/prometheus.yaml 
```

