---
title: k8s安装kubesphere
date: 2022-05-10 14:48:39
category: k8s
tags: [k8s]
---
# kubesphere on k8s

> helm install on master
>
> tiller install on master
>
> openEBS install on master

## helm

* 包含两个组件，分别是 helm 客户端 和 Tiller 服务器
* **Tiller** 是 Helm 的服务端。Tiller 负责接收 Helm 的请求，与 k8s 的 apiserver 交互，根据chart
  来生成一个 release 并管理 release
* **chart** Helm的打包格式叫做chart，所谓chart就是一系列文件, 它描述了一组相关的 k8s 集群资源
* **release** 使用 helm install 命令在 Kubernetes 集群中部署的 Chart 称为 Release
* **Repoistory** Helm chart 的仓库，Helm 客户端通过 HTTP 协议来访问存储库中 chart 的索引文件和压缩包

* download https://get.helm.sh/helm-v3.6.1-linux-amd64.tar.gz

```
tar -zvxf helm-v2.16.3-linux-amd64.tar.gz
cd cd linux-amd64/
cp helm /usr/local/bin/
cp tiller /usr/local/bin/

[root@master ~]# helm init
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com 
Error: error initializing: Looks like "https://kubernetes-charts.storage.googleapis.com" is not a valid chart repository or cannot be reached: Failed to fetch https://kubernetes-charts.storage.googleapis.com/index.yaml : 403 Forbidden

[root@master ~]# helm version
Client: &version.Version{SemVer:"v2.16.3", GitCommit:"1ee0254c86d4ed6887327dabed7aa7da29d7eb0d", GitTreeState:"clean"}
Error: could not find tiller
```

## tiller

tiller.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
 - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

初始化helm服务端

```
helm init --service-account tiller --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
如下：
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
#查询
kubectl get pod -n kube-system
tiller-deploy-7bf45f97c7-c2978             0/1     ContainerCreating   0          63s
#过一会儿再查
tiller-deploy-7bf45f97c7-c2978             1/1     Running   0          3m37s
```

修改镜像地址：

```
[root@master ~]# helm repo add stable http://mirror.azure.cn/kubernetes/charts
"stable" has been added to your repositories
```



## openEBS

确认 master 节点是否有 Taint，如下看到 master 节点有 Taint

```
[root@master ~]# kubectl describe node master | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

先去掉master的Taint:

```
[root@master ~]# kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
node/centosdb untainted
```

安装openEBS

```
kubectl create ns openebs
helm install --namespace openebs --name openebs stable/openebs --version 1.5.0

NOTES:
The OpenEBS has been installed. Check its status by running:
$ kubectl get pods -n openebs

For dynamically creating OpenEBS Volumes, you can either create a new StorageClass or
use one of the default storage classes provided by OpenEBS.

Use `kubectl get sc` to see the list of installed OpenEBS StorageClasses. A sample
PVC spec using `openebs-jiva-default` StorageClass is given below:
```

等几份钟，安装 OpenEBS 后将自动创建 4 个 StorageClass，查看创建的 StorageClass：

```
[root@master ~]# kubectl get pods -n openebs
NAME                                           READY   STATUS    RESTARTS   AGE
openebs-admission-server-5dbc9f4456-jwv5f      1/1     Running   0          13m
openebs-apiserver-659d656db5-xjkb5             1/1     Running   3          13m
openebs-localpv-provisioner-6cb9d78965-fhmgz   1/1     Running   0          13m
openebs-ndm-4zbrt                              1/1     Running   0          13m
openebs-ndm-operator-5ff78c45f6-25hjf          1/1     Running   1          13m
openebs-ndm-q58pr                              1/1     Running   0          13m
openebs-ndm-td58x                              1/1     Running   0          13m
openebs-provisioner-77b84d8cc-tnmn9            1/1     Running   0          13m
openebs-snapshot-operator-6dcc7b6fbd-ssqn4     2/2     Running   0          13m

[root@master ~]# kubectl get sc
NAME                        PROVISIONER                                                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device              openebs.io/local                                           Delete          WaitForFirstConsumer   false                  2m4s
openebs-hostpath            openebs.io/local                                           Delete          WaitForFirstConsumer   false                  2m4s
openebs-jiva-default        openebs.io/provisioner-iscsi                               Delete          Immediate              false                  2m5s
openebs-snapshot-promoter   volumesnapshot.external-storage.k8s.io/snapshot-promoter   Delete          Immediate              false                  2m5s

```

将 openebs-hostpath设置为 默认的 StorageClass：

```
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

[root@master ~]# kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-hostpath patched

kubectl create ns storgeclass


```

* 创建deployment

```
#nfs-client-provisioner.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: openebs
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: mynfs
            - name: NFS_SERVER
              value: 192.168.2.221
            - name: NFS_PATH
              value: /nfs/data
      volumes:
        - name: nfs-client-root
          nfs:
            server: 192.168.2.221
            path: /nfs/data




```

* rbac.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: openebs
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: openebs
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: openebs
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: openebs
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: openebs
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```



* 

```
#storgeclass.yaml

apiVersion: storage.k8s.io/v1
kind: storageClass
metadata:
  name: managed-nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: mynfs # or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"
```

*https://openebs.io/docs/concepts/localpv*

## kubesphere

```
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/kubesphere-installer.yaml
   
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/cluster-configuration.yaml

```

查看日志

```
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f

Error from server (BadRequest): container "installer" in pod "ks-installer-85dcfff87d-wgcn2" is waiting to start: trying and failing to pull image
#在拉镜像，等会

Waiting for all tasks to be completed ...
task network status is successful  (1/4)
task openpitrix status is successful  (2/4)
task multicluster status is successful  (3/4)
task monitoring status is successful  (4/4)
**************************************************
Collecting installation results ...
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.2.219:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components 
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2022-05-10 10:30:52
#####################################################
```





**注**：

（1）prometheus-k8s-0容器一直处于SchedulerError状态，无法使用监控功能

running PreBind plugin "VolumeBinding": binding volumes: timed out waiting for the condition

```
kubectl logs -n openebs -l openebs.io/component-name=openebs-localpv-provisioner
```

"openebs-hostpath": unexpected error getting claim reference: selfLink was empty, can't make referen

elfLink was empty 在k8s集群 v1.20之前都存在，在v1.20之后被删除，需要在/etc/kubernetes/manifests/kube-apiserver.yaml 添加参数
增加 - --feature-gates=RemoveSelfLink=false

```
spec:
containers:
- command:
    - kube-apiserver
    - --feature-gates=RemoveSelfLink=false

```

```
kubectl apply -f /etc/kubernetes/manifests/kube-apiserver.yaml
```

（2）有一台安装了gitlab，那台的node-expoter一直说端口9100被占用

```
gitlab status
[root@node2 ~]# gitlab-ctl status
/opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:684: warning: Insecure world writable dir /home/domains/maven/bin in PATH, mode 040777
run: gitaly: (pid 705) 33304154s; run: log: (pid 698) 33304154s
run: gitlab-monitor: (pid 684) 33304154s; run: log: (pid 683) 33304154s
run: gitlab-workhorse: (pid 704) 33304154s; run: log: (pid 695) 33304154s
run: logrotate: (pid 15501) 566s; run: log: (pid 696) 33304154s
run: nginx: (pid 699) 33304154s; run: log: (pid 688) 33304154s
run: node-exporter: (pid 692) 33304154s; run: log: (pid 686) 33304154s
run: postgres-exporter: (pid 702) 33304154s; run: log: (pid 690) 33304154s
run: postgresql: (pid 693) 33304154s; run: log: (pid 691) 33304154s
run: prometheus: (pid 706) 33304154s; run: log: (pid 703) 33304154s
run: redis: (pid 682) 33304154s; run: log: (pid 681) 33304154s
run: redis-exporter: (pid 687) 33304154s; run: log: (pid 685) 33304154s
run: sidekiq: (pid 701) 33304154s; run: log: (pid 694) 33304154s
run: unicorn: (pid 700) 33304154s; run: log: (pid 689) 33304154s
```

关掉

```
gitlab-ctl stop node_exporter

[root@node2 ~]# gitlab-ctl stop node_exporter
/opt/gitlab/embedded/lib/ruby/gems/2.3.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:684: warning: Insecure world writable dir /home/domains/maven/bin in PATH, mode 040777【没有权限】

chmod go-w /home/domains/manve/bin
再执行
gitlab-ctl stop node_exporter
```

