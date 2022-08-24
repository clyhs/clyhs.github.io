---
title: k8s安装部署
date: 2022-05-10 10:48:39
category: k8s
tags: [k8s]
---
# k8s install

> Kubernetes Cluster = N Master Node + N Worker Node：N主节点+N工作节点； N>=1

## ready server

centos7 64

* master 192.168.2.221
* node1  192.168.2.219
* node2  192.168.2.220

## kubeadm install

*all server install such as:*

### base setting

> 1、setting hostname

```
hostnamectl set-hostname master
hostnamectl set-hostname node1
hostnamectl set-hostname node2
```

> 2、setting selinux

```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

> 3、turn off swap

```
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab 
```

> 4、iptables could bridge

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

or do

```
vi /etc/sysctl.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
# 
sysctl -p
```

> 5、install kubelet、kubeadm、kubectl
>
> kubectl 是供程序员使用的命令行，一般只安装在mater(总部)。
>
> kubeadm 帮助程序员管理集群 (如设置主节点并创建主节点中的控制平面的组件)。

```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

```

```
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes
```

> 6、start kubelet

```
sudo systemctl enable --now kubelet
# look at kubelet status
systemctl status kubelet
```

> 7、use kubeadm to Cluster 

```
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
```

```
chmod +x ./images.sh && ./images.sh

docker images
```

*finish install*

**next copy vm to node1 node2**

> 9、setting domain mapping to all vm(master,node1,node2)

```
echo "192.168.2.221  cluster-endpoint" >> /etc/hosts

```

> 10、init master

```
kubeadm init \
--apiserver-advertise-address=192.168.2.221 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=172.31.0.0/16

```

**`-pod-network-cidr=172.31.0.0/16` 非常重要，用于为pod分配ip地址**

- 初始化主节点成功,同时给出一系列**提示**（接下来按提示操作）

```
# ...
# 【翻译】你的Kubernetes控制平面已经成功初始化
[init] Using Kubernetes version: v1.20.9
[preflight] Running pre-flight checks
        [WARNING Service-Docker]: docker service is not enabled, please run 'systemctl enable docker.service'
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [centosdb cluster-endpoint kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.2.221]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [centosdb localhost] and IPs [192.168.2.221 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [centosdb localhost] and IPs [192.168.2.221 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 16.003830 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node centosdb as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node centosdb as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: xxxid4.qfgy66yoykp07cjw
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join cluster-endpoint:6443 --token xxxid4.qfgy66yoykp07cjw \
    --discovery-token-ca-cert-hash sha256:c00aaa55766bdfef9f92c04a1c7e8f871e9f270dc1619f47519ab6d3223e3450 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token xxxid4.qfgy66yoykp07cjw \
    --discovery-token-ca-cert-hash sha256:c00aaa55766bdfef9f92c04a1c7e8f871e9f270dc1619f47519ab6d3223e3450 

```

> 10、按照英文提示执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

> 11、 look  at all nodes

```
kubectl get nodes
NAME        STATUS     ROLES                  AGE   VERSION
master     Ready    control-plane,master   17h   v1.20.9
```

### install network components calico

```
curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml

```

*修改 calico.yaml 配置，使网段同 `--pod-network-cidr=172.31.0.0/16` 一致，除非是192.168**就不用改*

```
vim calico.yaml 
?192.168 #vim工具搜索

# no effect. This should fall within `--cluster-cidr`.
- name: CALICO_IPV4POOL_CIDR
    value: "172.31.0.0/16"
# Disable file logging so `kubectl logs` works.
```

*常用命令*

```
#查看集群所有节点
kubectl get nodes

#根据配置文件，给集群创建资源
kubectl apply -f xxxx.yaml

#查看集群部署了哪些应用？
docker ps   
# 效果等于   
kubectl get pods -A

```

*look at master status*

```
[root@centosdb ~]# kubectl get pods -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            calico-kube-controllers-6fcb5c5bcf-q5mkg     1/1     Running   0          17h
kube-system            calico-node-8fnpb                            0/1     Running   0          17h
kube-system            calico-node-l49k6                            0/1     Running   0          17h
kube-system            calico-node-q5pd6                            0/1     Running   0          17h
kube-system            coredns-5897cd56c4-kqvxd                     1/1     Running   0          17h
kube-system            coredns-5897cd56c4-zkxmz                     1/1     Running   0          17h
kube-system            etcd-centosdb                                1/1     Running   0          17h
kube-system            kube-apiserver-centosdb                      1/1     Running   0          17h
kube-system            kube-controller-manager-centosdb             1/1     Running   0          17h
kube-system            kube-proxy-87h9h                             1/1     Running   0          17h
kube-system            kube-proxy-b8zxc                             1/1     Running   0          17h
kube-system            kube-proxy-vlhcs                             1/1     Running   0          17h
kube-system            kube-scheduler-centosdb                      1/1     Running   0          17h


```

> 将node1、node2 进入工作节点

```
kubeadm join cluster-endpoint:6443 --token xxxid4.qfgy66yoykp07cjw \
    --discovery-token-ca-cert-hash sha256:c00aaa55766bdfef9f92c04a1c7e8f871e9f270dc1619f47519ab6d3223e3450
```

```
[root@centosdb ~]# kubectl get nodes
NAME         STATUS   ROLES                  AGE   VERSION
node1   Ready    <none>                 17h   v1.20.9
node2   Ready    <none>                 17h   v1.20.9
master     Ready    control-plane,master   17h   v1.20.9
```

### install dashboard

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
# or do
kubectl apply -f recommended.yaml 
```

```
[root@centosdb ~]# kubectl get pods -A
kubernetes-dashboard   dashboard-metrics-scraper-79c5968bdc-qrbzf   1/1     Running   0          16h
kubernetes-dashboard   kubernetes-dashboard-7448ffc97b-64tf2        1/1     Running   0          16h
```

> 设置访问端口

```
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

提示: 进入文件将`type: ClusterIP`改为`type: NodePort`

```
kubectl get svc -A | grep kubernetes-dashboard

[root@centosdb ~]# kubectl get svc -A | grep kubernetes-dashboard
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.96.117.164   <none>        8000/TCP                 16h
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.96.181.30    <none>        443:30104/TCP            16h
```

*https://192.168.2.219:30104*

> setting dashboard accout 

```
#vim dash.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```

```
kubectl apply -f dash.yaml

# generate dashboard token

[root@centosdb ~]# kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
eyJhbGciOiJSUzI1NiIsImtpZCI6IndNSllNU3p0MzRJVDhoV0NWMEJqZkd5WmdveEpkcXExbEZmOHY1d21PUWcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXI4MnJiIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4ODQ3OWY5MC1jOTljLTQzNWYtYjkzMy04MWUwMDk5N2E0OWIiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.YA23lwjuF3CW4pRYLOSFAPWeFtOQXNa7RPsz12v_twxOacvoZoZsxEpBJj0WO_TNSLk4luvHY4fhG3udfZxI_-IvJaPY667-C4AYENOIAQ1QCIU_Qo_NoyJCQ5WQfM_RpIsCng7X7dJy1sSs5UiceXFQfMDEfbUoSAZ4GViU_bXuloEaWkIfGl4c5RT_tuaDoDvHNjLRGUv_tQgpzAt6IZoZooLgsON1p0F9AIX8wJ-1c7BdvMa4quAvJ_eFgR65flLrfeWJH9lkz_gnz671jPqkVV_fRpL-H_767jHD1sKlS9wHqYVm-eu1p_gjISyPQUTdwrw72P7x_saMaG2XPA
```

next login *https://192.168.2.219:30104*

**注：calico node不运行**

```
kube-system            calico-node-8fnpb                            0/1     Running   0          17h
kube-system            calico-node-l49k6                            0/1     Running   0          17h
kube-system            calico-node-q5pd6                            0/1     Running   0          17h

kubectl describe pod calico-node-8fnpb -n kube-system

IRD: unable to connect to BIRDv4 socket: dial unix /var/run/calico/bird.ctl: connect: connection refused
  Warning  Unhealthy  14m   kubelet            Readiness probe failed: 2022-05-07 03:01:44.179 [INFO][204] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:01:54.107 [INFO][243] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:02:04.112 [INFO][276] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:02:14.133 [INFO][317] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:02:24.106 [INFO][351] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:02:34.133 [INFO][385] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  13m  kubelet  Readiness probe failed: 2022-05-07 03:02:44.119 [INFO][426] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  12m  kubelet  Readiness probe failed: 2022-05-07 03:02:54.108 [INFO][458] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220
  Warning  Unhealthy  4m12s (x52 over 12m)  kubelet  (combined from similar events): Readiness probe failed: 2022-05-07 03:11:34.318 [INFO][2258] confd/health.go 180: Number of node(s) with BGP peering established = 0
calico/node is not ready: BIRD is not ready: BGP not established with 192.168.2.219,192.168.2.220

```

```

kubectl edit daemonset calico-node -n kube-system
- name: CALICO_IPV4POOL_CIDR
  value: 172.31.0.0/16
- name: IP_AUTODETECTION_METHOD
  value: interface=eth0

```

```
kubectl get pods -A
NAMESPACE              NAME                                         READY   STATUS    RESTARTS   AGE
kube-system            calico-node-bw8b9                            1/1     Running   0          35s
kube-system            calico-node-g4dvb                            1/1     Running   0          50s
kube-system            calico-node-tftp8                            1/1     Running   0          63s
```

*注意三台机必须相互能Ping得通*

### namespace

```
kubectl create ns hello
kubectl delete ns hello

# vi creates.yaml
apiVersion: v1 # 版本
kind: Namespace # 类型
metadata:
  name: hello

kubectl apply -f creates.yaml
```

> 运行Pod

```
kubectl run 【Pod名称】 --image=【镜像名称】

[root@master ~]# kubectl run mynginx --image=nginx
pod/mynginx created

[root@centosdb ~]# kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
mynginx   1/1     Running   0          81s

```

*常用命令*

```
#查看default名称空间的Pod
kubectl get pod 
#描述
kubectl describe pod 你自己的Pod名字
#删除
kubectl delete pod Pod名字
#查看Pod的运行日志
kubectl logs Pod名字

#每个Pod-k8s都会分配一个ip
kubectl get pod -owide

[root@master ~]# kubectl get pod -owide
NAME      READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
mynginx   1/1     Running   0          27m   172.31.145.131   centosapp1   <none>           <none>

#使用Pod的ip+pod里面运行容器的端口
curl 172.31.145.131

进入pod
kubectl exec -ti [pod-name] -n <your-namespace> -- /bin/sh
kubectl exec -ti mynginx -- /bin/sh
```

### Deployment

> 多副本

```
kubectl create deployment my-dep --image=nginx --replicas=3

```

deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-dep-02
  name: my-dep-02
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-dep-02
  template:
    metadata:
      labels:
        app: my-dep-02
    spec:
      containers:
      - image: nginx
        name: nginx

```

> 扩缩容

```
kubectl scale --replicas=5 deployment/my-dep
kubectl edit deployment my-dep

#修改replicas

```

> 滚动更新

将一个pod集群在正常提供服务时从V1版本升级成 V2版本

```
kubectl set image deployment/my-dep-02 nginx=nginx:1.16.1 --record
kubectl rollout status deployment/my-dep-02
```

通过修改deployment配置文件实现更新

```
kubectl edit deployment/my-dep-02
```

> 版本回退

```
[root@master ~]# kubectl rollout history deployment/my-dep-02
deployment.apps/my-dep-02 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         kubectl set image deployment/my-dep-02 nginx=nginx:1.16.1 --record=true
```

查看某个历史详情

```
kubectl rollout history deployment/my-dep-02 --revision=3
```

回滚到上次的版本

```
kubectl rollout undo deployment/my-dep-02
```

回滚到指定版本

```
kubectl rollout undo deployment/my-dep-02 --to-revision=1

```

除了Deployment，k8s还有 `StatefulSet` 、`DaemonSet` 、`Job` 等 类型资源。我们都称为 `工作负载`。

有状态应用使用 `StatefulSet` 部署，无状态应用使用 `Deployment` 部署

### service

以上内容的pod中的容器我们在外网都无法访问，使用Service来解决（–type=NodePort）。

> Service是 将一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 公开为网络服务的抽象方法。

```
[root@master ~]# kubectl get service
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   21h
```

> 暴露deployment的服务和端口,进行端口映射，创建出具有ip地址的Service (pod的集群)

```
kubectl expose deployment my-dep-02 --port=8000 --target-port=80

[root@master ~]# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    22h
my-dep-02    ClusterIP   10.96.226.142   <none>        8000/TCP   4s
```

> 查看pod的标签

```
kubectl get pod --show-labels

[root@master ~]# kubectl get pod --show-labels
NAME                         READY   STATUS    RESTARTS   AGE     LABELS
my-dep-02-7b9d6bb69c-4fvl8   1/1     Running   0          5m31s   app=my-dep-02,pod-template-hash=7b9d6bb69c
my-dep-02-7b9d6bb69c-j4mjk   1/1     Running   0          5m31s   app=my-dep-02,pod-template-hash=7b9d6bb69c
my-dep-02-7b9d6bb69c-k7lhl   1/1     Running   0          5m31s   app=my-dep-02,pod-template-hash=7b9d6bb69c
mynginx                      1/1     Running   0          157m    run=mynginx
#使用标签检索Pod
kubectl get pod -l app=my-dep-02
```

> 查看service/my-dep-02 的yaml配置文件

```
kubectl get service/my-dep-02 -o yaml

```

> 重里面pod访问

```
kubectl exec -ti my-dep-02-7b9d6bb69c-4fvl8 -- /bin/sh
# curl my-dep-02.default.svc:8000
```

> 删除Service

```
 kubectl delete service/my-dep-02
```



>clusterIP

* 默认就是ClusterIP 等同于没有–type的

```
kubectl expose deployment my-dep-02 --port=8000 --target-port=80 --type=ClusterIP
```

> NodePort

* 集群外可以访问

```
kubectl expose deployment my-dep-02 --port=8000 --target-port=80 --type=NodePort
```

```
[root@master ~]# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          22h
my-dep-02    NodePort    10.96.200.109   <none>        8000:30037/TCP   10s
```

 **ip+port映射**: 集群外`（mater、node1、node2的ip):30037` 映射到 ` 10.96.200.109:8000`

外网访问 可以 http://192.168.2.219:30037

### Ingress(网关)

> Ingress：Service的统一网关入口（如百度的统一域名访问，统一Service层），Ingress是k8s机器集群的统一入口，请求流量先经过Ingress（入口）再进入集群内接受服务。
>
>  service是为一组pod服务提供一个统一集群内访问入口或外部访问的随机端口，而ingress做得是通过反射的形式对服务进行分发到对应的service上。
>
> service一般是针对内部的，集群内部调用，而ingress应该是针对外部调用的
>
> service只是开了端口，可以通过服务器IP:端口的方式去访问，但是服务器IP还是可变的，Ingress应该就是作为网关去转发
>
> 因为有很多服务,入口不统一,不方便管理

> 安装ingress

```
wget https://github.com/kubernetes/ingress-nginx/blob/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml

```

```
vi deploy.yaml
#将image的值改为如下值：
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0

kubectl apply -f deploy.yaml
```

> 查看安装结果

```
kubectl get pod,svc -n ingress-nginx

[root@master ~]# kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS              RESTARTS   AGE
pod/ingress-nginx-admission-create-j92kq        0/1     Completed           0          2m27s
pod/ingress-nginx-admission-patch-2pmwz         0/1     Completed           2          2m27s
pod/ingress-nginx-controller-65bf56f7fc-8b4nk   0/1     ContainerCreating   0          2m27s
```

* 新建了Service，以NodePort方式暴露了端口

```
[root@master ~]# kubectl get service -A | grep ingress
ingress-nginx          ingress-nginx-controller             NodePort    10.96.160.237   <none>        80:32335/TCP,443:30536/TCP   3m29s
ingress-nginx          ingress-nginx-controller-admission   ClusterIP   10.96.138.149   <none>        443/TCP                      3m29s
```

映射：32335,30536

http://192.168.2.221:30536/，bad request

http://192.168.2.221:32335 , notfound

* 安装测试环境

> 应用配置文件，部署了2个Deployment,2个Service

ingresstest.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
```

```
[root@master ~]# kubectl apply -f ingresstest.yaml 
deployment.apps/hello-server created
deployment.apps/nginx-demo created
service/nginx-demo created
service/hello-server created

[root@master ~]# kubectl get service
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-server   ClusterIP   10.96.121.161   <none>        8000/TCP         69s
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP          22h
my-dep-02      NodePort    10.96.200.109   <none>        8000:30037/TCP   30m
nginx-demo     ClusterIP   10.96.221.97    <none>        8000/TCP         69s

[root@master ~]# kubectl get deployment -o wide
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                                                          SELECTOR
hello-server   2/2     2            2           2m48s   hello-server   registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server   app=hello-server
my-dep-02      3/3     3            3           55m     nginx          nginx                                                           app=my-dep-02
nginx-demo     2/2     2            2           2m48s   nginx          nginx                                                           app=nginx-demo

[root@master ~]#  kubectl get pods -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
hello-server-6cbb679d85-5t2t4   1/1     Running   0          3m34s   172.31.145.140   centosapp1   <none>           <none>
hello-server-6cbb679d85-88ndh   1/1     Running   0          3m34s   172.31.145.139   centosapp1   <none>           <none>
my-dep-02-7b9d6bb69c-4fvl8      1/1     Running   0          56m     172.31.145.136   centosapp1   <none>           <none>
my-dep-02-7b9d6bb69c-j4mjk      1/1     Running   0          56m     172.31.19.198    centosapp2   <none>           <none>
my-dep-02-7b9d6bb69c-k7lhl      1/1     Running   0          56m     172.31.145.137   centosapp1   <none>           <none>
mynginx                         1/1     Running   0          3h28m   172.31.145.131   centosapp1   <none>           <none>
nginx-demo-7d56b74b84-rkjcd     1/1     Running   0          3m34s   172.31.19.202    centosapp2   <none>           <none>
nginx-demo-7d56b74b84-twfms     1/1     Running   0          3m34s   172.31.19.201    centosapp2   <none>           <none>

```

> 域名访问

* 访问 hello.test.com 的请求由 hello-server (Service)集群处理
* 访问 demo.test.com 的请求由 nginx-demo (Service)集群处理
* Ingress(网关)根据请求的域名分配对应的Service去处理

ingresscom.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress  # 类型
metadata:
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.test.com" #域名
    http:
      paths:
      - pathType: Prefix # 前缀
        path: "/"
        backend:
          service:
            name: hello-server # Service 名称
            port:
              number: 8000 # 端口
  - host: "demo.test.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的service，下面的service一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  # java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000

```

```
[root@master ~]# kubectl apply -f ingresscom.yaml 
ingress.networking.k8s.io/ingress-host-bar created

[root@master ~]# kubectl get ingress
NAME               CLASS   HOSTS                          ADDRESS         PORTS   AGE
ingress-host-bar   nginx   hello.test.com,demo.test.com   192.168.2.220   80      2m47s
```

* windows 配置域名映射（域名映射文件地址：`C:\Windows\System32\drivers\etc`）

```
192.168.2.220 hello.test.com 
192.168.2.220 demo.test.com 
```

浏览器：http://hello.test.com:32335/

hello world!

浏览器：http://demo.test.com:30536/  nginx是由Ingress层返回的

400 bad request

nginx

* 修改Ingress配置文件,将`path: "/nginx"`改成`path: "/nginx.html"`

浏览器：http://demo.test.com:32335/nginx.html

404 not found

nginx/1.21.5

问题： path: “/nginx.html” 与 path: “/” 为什么会有不同的效果？

```
kubectl edit ingress ingress-host-bar
```

> - 修改配置文件 ingresscom.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  annotations: # 路径重写配置功能开启
    nginx.ingress.kubernetes.io/rewrite-target: /$2 
  name: ingress-host-bar
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.test.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.test.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx(/|$)(.*)"  # 配置忽略/nginx
        backend:
          service:
            name: nginx-demo 
            port:
              number: 8000

```

```
[root@master ~]# kubectl apply -f ingresscom.yaml 
ingress.networking.k8s.io/ingress-host-bar configured
```

http://demo.test.com:32335/nginx.html

http://demo.test.com:32335

都返回

404 not found

nginx

### 存储抽象

> 所有节点都安装nfs

```
yum install -y nfs-utils
```

> nfs主节点

```
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports

mkdir -p /nfs/data
systemctl enable rpcbind --now
systemctl enable nfs-server --now
#配置生效
exportfs -r

[root@master ~]# exportfs
/nfs/data       <world>
```

> 从节点

```
systemctl enable rpcbind --now
showmount -e 192.168.2.221
[root@node1 ~]# showmount -e 192.168.2.221
Export list for 192.168.2.221:
/nfs/data *

#执行以下命令挂载nfs服务器上的共享目录到本机路径/root/nfsmount
mkdir -p /nfs/data

mount -t nfs 192.168.2.221:/nfs/data /nfs/data
#写入一个测试文件
echo "hello nfs server" > /nfs/data/test.txt

```

在其它服务器查看test.txt

#### nfs（原生）方式数据挂载

在所有服务器创建目录

```
mkdir /nfs/data/nginx-pv/
```

mountnfs.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-pv-demo
  name: nginx-pv-demo
spec:
  replicas: 2 
  selector:
    matchLabels:
      app: nginx-pv-demo
  template:
    metadata:
      labels:
        app: nginx-pv-demo
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
        - name: html
          nfs:
            server: 192.168.2.221
            path: /nfs/data/nginx-pv/
```

```
kubectl apply -f mountnfs.yaml
cd /nfs/data/nginx-pv/
echo "Hello Mount" > index.html
```

```
kubectl get pods -o wide

nginx-pv-demo-587489dfcf-ljwqr   1/1     Running   0          85s     172.31.145.141   centosapp1   <none>           <none>
nginx-pv-demo-587489dfcf-ssc2c   1/1     Running   0          85s     172.31.19.203    centosapp2   <none>           <none>
```

```
[root@master ~]# curl 172.31.19.203
Hello Mount
```

#### PV(持久卷)和PVC(持久卷申明)

NFS(原生)方式数据挂载存在一些问题：

- 目录要自己创建
- Deployment及其pod删除后,服务器目录数据依旧存在
- 挂载容量没有限制

PV：持久卷（Persistent Volume），将应用需要持久化的数据保存到指定位置（存放持久化数据的目录就是持久卷）

 PVC：持久卷申明（Persistent Volume Claim），申明需要使用的持久卷规格 （申请持久卷的申请书）

静态供应： 提取指定位置和空间大小

动态供应：位置和空间大小由pv自动创建

> 创建pv池

```
在master
mkdir -p /nfs/data/01
mkdir -p /nfs/data/02
mkdir -p /nfs/data/03
```

创建三个 PV（持久卷）**静态供应的方式**，配置文件createPV.yaml

```
apiVersion: v1
kind: PersistentVolume # 类型
metadata:
  name: pv01-10m # 名称
spec:
  capacity:
    storage: 10M # 持久卷空间大小
  accessModes:
    - ReadWriteMany # 多节点可读可写
  storageClassName: nfs # 存储类名
  nfs:
    path: /nfs/data/01 # pc目录位置
    server: 192.168.2.221
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv02-1gi
spec:
  capacity:
    storage: 1Gi # 持久卷空间大小
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/02 # pc目录位置
    server: 192.168.2.221
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv03-3gi
spec:
  capacity:
    storage: 3Gi # 持久卷空间大小
  accessModes:
    - ReadWriteMany
  storageClassName: nfs
  nfs:
    path: /nfs/data/03 # pc目录位置
    server: 192.168.2.221

```

```
[root@master ~]# kubectl apply -f createPV.yaml
persistentvolume/pv01-10m created
persistentvolume/pv02-1gi created
persistentvolume/pv03-3gi created

[root@master ~]# kubectl get persistentvolume
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv01-10m   10M        RWX            Retain           Available           nfs                     48s
pv02-1gi   1Gi        RWX            Retain           Available           nfs                     48s
pv03-3gi   3Gi        RWX            Retain           Available           nfs                     48s
```

> 创建PVC

createPVC.yaml

```
kind: PersistentVolumeClaim # 类型
apiVersion: v1
metadata:
  name: nginx-pvc # PVC名称
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 200Mi # 需要空间
  storageClassName: nfs # 要对应PV的storageClassName

```

```
[root@master ~]# kubectl apply -f createPVC.yaml 
persistentvolumeclaim/nginx-pvc created
[root@master ~]# kubectl get persistentvolumeclaim
NAME        STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc   Bound    pv02-1gi   1Gi        RWX            nfs            30s
```

```
#查看PC,状态Bound(绑定),说明已经被使用,绑定信息： default/nginx-pvc => 名称空间/PVC名称
[root@master ~]# kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM               STORAGECLASS   REASON   AGE
pv01-10m   10M        RWX            Retain           Available                       nfs                     4m47s
pv02-1gi   1Gi        RWX            Retain           Bound       default/nginx-pvc   nfs                     4m47s
pv03-3gi   3Gi        RWX            Retain           Available                       nfs                     4m47s
```

> 创建Deployment ，让Deployment中的Pod绑定PVC

boundPVC.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy-pvc # Deployment名称
  name: nginx-deploy-pvc
spec:
  replicas: 2 # pod数量
  selector:
    matchLabels:
      app: nginx-deploy-pvc
  template:
    metadata:
      labels:
        app: nginx-deploy-pvc
    spec:
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html # 挂载目录
      volumes:
        - name: html
          persistentVolumeClaim:
            claimName: nginx-pvc  # pvc 的名称

```

```
[root@master ~]#  kubectl apply -f boundPVC.yaml
deployment.apps/nginx-deploy-pvc created
```

> 向挂载目录 `/nfs/data/02`写入测试文件

```
[root@master ~]# cd /nfs/data/02
[root@master 02]# echo "boundPVC test nginx" > index.html

[root@master 02]# kubectl get pod -o wide
nginx-deploy-pvc-79fc8558c7-8m9nn   1/1     Running   0          97s     172.31.19.204    centosapp2   <none>           <none>
nginx-deploy-pvc-79fc8558c7-s97vh   1/1     Running   0          97s     172.31.145.142   centosapp1   <none>           <none>
```

#### configMap(配置集)

ConfigMap 缩写为cm

ConfigMap（配置集）：用于配置文件挂载，抽取应用配置，并且可以自动更新。

> redis.conf内容如下：

```
appendonly yes
```

````
[root@master ~]# kubectl create configmap redis-conf --from-file=redis.conf
configmap/redis-conf created

[root@master ~]# kubectl get configmap
NAME               DATA   AGE
kube-root-ca.crt   1      24h
redis-conf         1      76s
````

```
kubectl get configmap redis-conf -o yaml
```

创建redistest.yaml

```
apiVersion: v1
kind: Pod # 类型
metadata:
  name: redis # pod名称
spec:
  containers:
  - name: redis
    image: redis # 镜像
    command:
      - redis-server
      - "/redis-master/redis.conf"  #指的是redis容器内部的位置
    ports:
    - containerPort: 6379 
    volumeMounts: # 配置卷挂载
    - mountPath: /data 
      name: data 	# 卷挂载名称  对应 下面的 挂载卷 data
    - mountPath: /redis-master
      name: config	 # 卷挂载名称  对应 下面的 挂载卷 config
  volumes: # 挂载卷
    - name: data 
      emptyDir: {} 
    - name: config 
      configMap:   # 配置集
        name: redis-conf
        items:
        - key: redis.conf
          path: redis.conf

```

```
[root@master ~]# kubectl apply -f redistest.yaml 
pod/redis created

kubectl exec -ti redis -- /bin/sh
cd ../redis-manager
more redis.conf
appendonly yes
```

修改配置集的redis.conf

```
kubectl edit configmap redis-conf 
```

加入：

```
requirepass 123456
```

```
[root@centosdb ~]# kubectl exec -ti redis -- /bin/sh
# cd ../
# ls
bin   data  etc   lib    media  opt   redis-master  run   srv  tmp  var
boot  dev   home  lib64  mnt    proc  root          sbin  sys  usr
# cd redis-master
# more redis.conf
appendonly yes
requirepass 123456
```

#### secret

 Secret 对象类型**用来保存敏感信息**，例如密码、OAuth 令牌和 SSH 密钥。 将这些信息放在 secret 中比放在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的定义或者 [容器镜像](https://kubernetes.io/zh/docs/reference/glossary/?all=true#term-image) 中来说更加安全和灵活。原理同ConfigMap

```
kubectl create secret docker-registry 【Secret的名称】 \
  --docker-server=【你的镜像仓库服务器】 \
  --docker-username=【你的用户名】 \
  --docker-password=【你的密码】 \
  --docker-email=【你的邮箱地址】

```

secret01.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: private-nginx # pod 名称
spec:
  containers:
  - name: private-nginx
    image: dockerywl/mysql # 私有镜像名称

```

查看

```
kubectl get secret
```





*来源：https://blog.csdn.net/weixin_46703850/article/details/122922090*