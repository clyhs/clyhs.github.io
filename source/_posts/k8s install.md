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
kube-system            calico-node-4xnrr                            1/1     Running   0          7m59s
kube-system            calico-node-tj7w9                            1/1     Running   0          5m23s
kube-system            calico-node-tp55n                            1/1     Running   0          7m59s
```

