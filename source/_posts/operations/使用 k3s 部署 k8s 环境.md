---
title: 使用 k3s 部署 k8s 环境
author: wanghuagang
date: 2022-10-22 00:37:28
updated: 2022-10-29 17:48:22
tags:
  - k3s
  - k8s
categories:
  - 运维
---

在中小集群环境中，可以使用 k3s 快速简便安装 k8s 节点和初始化集群。本教程通过安装一个单节点 k3s 来启动一个单节点的
k8s 集群，并在 k8s 启动完成后，使用 helm 在 k8s 之上安装 rancher 管理 k8s 集群。

通过阅读本文，你可以了解如下内容：

- 如何使用 k3s 初始化一个 k8s 集群
- 如何在 k8s 上发布一个多副本 Web 应用，并通过负载均衡访问该应用
- 如何手动安装 helm
- 如何通过 helm 安装 rancher


参考文档：[Quick-Start Guide](https://docs.k3s.io/quick-start)
或者 [中文快速入门指南](https://docs.k3s.io/quick-start)

**注意：** 中文教程中的使用均为国内阿里云加速。

<!--more-->

## 1. 集群配置

| hostname        | ip            | desc            |
|-----------------|---------------|-----------------|
| rm1.iloveyou.cn | 192.168.31.20 | rancher manager |


**注意：** 主机名域名访问可以通过 DNS 也可以通过主机名映射。

## 2. 环境准备

安装操作系统为 [RockyLinux 9](https://docs.rockylinux.org/release_notes/9_0/)
，其他操作系统环境可能稍有不同，可以根据官方文档进行调整。

### 2.1 系统环境

在安装前，请对所有节点操作系统做如下操作：

- 配置网络： 使用 `nmtui` 命令配置，或手动修改网卡配置文件。请注意 RHEL 9.x 的网卡文件
  在 `/etc/NetworkManager/system-connections/` 。
- 关闭 selinux ：`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`
- 禁用 firewalld 服务：`systemctl disable firewalld`
- 并停止 firewalld 服务：`systemctl stop firewalld`
- 配置主机名：`hostnamectl set-hostname rm.iloveyou.cn` 。主机名根据具体节点调整。
- 配置域名解析或者 hosts 文件。
- 安装基本软件：`dnf install wget vim tar net-tools telnet`

### 2.2 软件环境

软件环境版本如下：

- k3s : [v1.24.7+k3s1`](https://github.com/k3s-io/k3s/releases/tag/v1.24.7%2Bk3s1)
- rancher : v2.6

#### 2.2.1 本地加速安装

**注意：** 这一步不是必须的，本步骤的操作只是为了想使用官方文档中的步骤安装，同时
避免为网络问题出现而准备使用内网环境安装前置步骤。 当参考 [k3s 中文文档](https://docs.rancher.cn/docs/k3s/quick-start/_index/)时，
可以跳过此步骤。

- k3s 安装脚本： `wget -q -O instal-k3s.sh https://get.k3s.io` 也可以手动保存成 `install-k3s.sh` 
- 下载 k3s 二进制文件： 从 `https://github.com/k3s-io/k3s/releases` 下载指定版本的 k3s 。
注意：较新当版本可能在后续无法安装 Rancher ，如遇到版本情况请按照提示回退并重新安装环境。
- 下载 helm 二进制包：从 `https://github.com/helm/helm/releases` 下载最新版本的 Helm
- 准备内网 Web 文件服务，具体搭建不再赘述，使用 Nginx 或者 Caddy 都可以。
  简单的可以在本地节点使用 node 的 [http-server](https://www.npmjs.com/package/http-server)
  快速启动一个 Web 服务。

将下载的三个文件放到 Web 文件服务器中，以便可以通过 Web 服务器地址访问这些文件。

## 3. 单节点环境安装

### 3.1 安装 k3s

#### 3.1.1 使用加速镜像安装

```shell
curl -sfL \
    https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | \
    INSTALL_K3S_VERSION=v1.24.7+k3s1 \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    INSTALL_K3S_MIRROR=cn \
    sh -s - \
      server \
      --cluster-init \
      --data-dir /data/k8s/storage
```

这里解释一下上述几个执行步骤：

- 首先从本地服务器下载前面准备的 `k3s` 二进制文件到 `/usr/local/bin/k3s` ，并赋予可执行权限
- 从本地服务器下载前面准备的 `install-k3s.sh` 安装脚本
- `INSTALL_K3S_VERSION=v1.24` ：使用环境变量指定 k3s 版本
- `INSTALL_K3S_SKIP_DOWNLOAD=true`：跳过二进制文件下载，使用本地二进制文件，也就是上面已经下载到 `/usr/local/bin/k3s` 的文件了
- `INSTALL_K3S_SKIP_SELINUX_RPM=true`：跳过 SELINUX RPM 检查
- `INSTALL_K3S_SELINUX_WARN=false`：跳过 SELINUX 警告
- `server` 使用命令初始化 server 节点
- `--cluster-init` 使用参数初始化 etcd 作为数据存储
- `--data-dir /data/k8s/storage` 使用参数初始化 local-storage 存储类使用其他数据目录。

执行结果如下：

```textmate
[root@rm1 ~]# curl -sfL \
    https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | \
    INSTALL_K3S_VERSION=v1.24.7+k3s1 \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    INSTALL_K3S_MIRROR=cn \
    sh -s - \
      server \
      --cluster-init \
      --data-dir /data/k8s/storage
[INFO]  Using v1.24.7+k3s1 as release
[INFO]  Downloading hash rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/v1.24.7-k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/v1.24.7-k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Skipping installation of SELinux RPM
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

#### 3.1.2 使用内网环境安装

```shell
curl -sfl http://192.168.31.10:8080/k3s -o /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s

curl -sfL \
    http://192.168.31.10:8080/install-k3s.sh | \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    INSTALL_K3S_MIRROR=cn \
    sh -s - \
      server \
      --cluster-init \
      --data-dir /data/k8s/storage
    sh -
 
```

这里解释一下上述几个执行步骤：

- 首先从本地服务器下载前面准备的 `k3s` 二进制文件到 `/usr/local/bin/k3s` ，并赋予可执行权限
- 从本地服务器下载前面准备的 `install-k3s.sh` 安装脚本
- `INSTALL_K3S_SKIP_DOWNLOAD=true`：跳过二进制文件下载，使用本地二进制文件，也就是上面已经下载到 `/usr/local/bin/k3s` 的文件了
- `INSTALL_K3S_SKIP_SELINUX_RPM=true`：跳过 SELINUX RPM 检查
- `INSTALL_K3S_SELINUX_WARN=false`：跳过 SELINUX 警告
- `server` 使用命令初始化 server 节点
- `--cluster-init` 使用参数初始化 etcd 作为数据存储
- `--data-dir /data/k8s/storage` 使用参数初始化 local-storage 存储类使用其他数据目录。

### 3.2 检查 k8s

命令执行完成，并没有任何错误后，使用操作检查 k8s 是否正常：

- 检查 k3s 服务：`systemctl status k3s`
- 检查 k8s 的 Pods ：`kubectl get pods -A`

```text
[root@rm1 ~]# kubectl get pods -A
INFO[0000] Acquiring lock file /var/lib/rancher/k3s/data/.lock 
INFO[0000] Preparing data dir /var/lib/rancher/k3s/data/e430ebdc9a66bbba77e21922f682788d68338a4eb3c035740aeeef38be42b6f9 
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   coredns-75fc8f8fff-9fzjq                  1/1     Running     0          11m
kube-system   helm-install-traefik-crd-52pj5            0/1     Completed   0          11m
kube-system   helm-install-traefik-wqvlx                0/1     Completed   2          11m
kube-system   local-path-provisioner-5b5579c644-vwm2d   1/1     Running     0          11m
kube-system   metrics-server-5c8978b444-stnd5           1/1     Running     0          11m
kube-system   svclb-traefik-f0e3885e-khrck              2/2     Running     0          3m35s
kube-system   traefik-56cfcbb99f-t7fvf                  1/1     Running     0          3m35s
```

如果检查 pods 状态如上所示，说明一切正常。
如果节点启动失败，请使用 `systemctl -eu k3s` 查看日志。

### 3.3 部署应用

以下为使用 [whoami](https://hub.docker.com/r/traefik/whoami) 镜像部署一个只有一个副本应用。

#### 3.3.1 部署deployment

部署一个无状态的 [deployments](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 。

创建 `whoami-deployment.yml` 文件，并增加如下内容：

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: whoami
  labels:
    app: whoami

spec:
  replicas: 3
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
        - name: whoami
          image: traefik/whoami
          ports:
            - name: web
              containerPort: 80
```

文件内容是创建一个 deployment 类型资源，资源使用 `traefik/whoami` 镜像创建暴露出 80 端口的容器，并且会有 3 个副本。

使用如下命令启动：

```shell
kubectl create -f whoami-deployment.yml
```

启动后使用如下命令可以查看 deployment 的状态：

```shell
kubectl get deployments
```

输出如下：

```textmate
[root@rm1 ~]# kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
whoami   3/3     3            3           6s
```

使用如下命令查看 deployment 详细内容：

```shell
kubectl describe deployment whoami
```

输出如下：

```textmate
[root@rm1 ~]# kubectl describe deployment whoami
Name:                   whoami
Namespace:              default
CreationTimestamp:      Sat, 29 Oct 2022 16:49:38 +0800
Labels:                 app=whoami
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=whoami
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=whoami
  Containers:
   whoami:
    Image:        traefik/whoami
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   whoami-7f55677887 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  26s   deployment-controller  Scaled up replica set whoami-7f55677887 to 3

```

可以看到在最后 Events 的 Message 中的信息提示，启动了三个副本的 `whoami-7f55677887` 。

使用如下命令获取 replicaset ：

```shell
kubectl get replicaset
# 等价与 kubectl get rs
```

输出如下：

```textmate
[root@rm1 ~]# kubectl get replicaset
NAME                DESIRED   CURRENT   READY   AGE
whoami-7f55677887   3         3         3       3m55s
```

使用如下命令获取 replicaset 详细信息：

```shell
kubectl describe rs whoami
# 等价与 kubectl describe rplicaset whoami
```

输出如下：

```textmate
[root@rm1 ~]# kubectl describe rs whoami
Name:           whoami-7f55677887
Namespace:      default
Selector:       app=whoami,pod-template-hash=7f55677887
Labels:         app=whoami
                pod-template-hash=7f55677887
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 4
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/whoami
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=whoami
           pod-template-hash=7f55677887
  Containers:
   whoami:
    Image:        traefik/whoami
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  4m10s  replicaset-controller  Created pod: whoami-7f55677887-9lcck
  Normal  SuccessfulCreate  4m10s  replicaset-controller  Created pod: whoami-7f55677887-m7tcp
  Normal  SuccessfulCreate  4m10s  replicaset-controller  Created pod: whoami-7f55677887-rgdng

```

可以看到在最后 Events 的 Message 中的信息提示中，创建了三个 pod 。

使用如下命令查看 pod：

```shell
kubectl get pods
```

输出如下：

```textmate
[root@rm1 ~]# kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
whoami-7f55677887-9lcck   1/1     Running   0          4m50s
whoami-7f55677887-m7tcp   1/1     Running   0          4m50s
whoami-7f55677887-rgdng   1/1     Running   0          4m50s
```

使用如下命令查看其中一个 pod 的信息：

```shell
kubectl describe pod whoami-7f55677887-rgdng
```

输出如下：

```textmate
[root@rm1 ~]# kubectl describe pod whoami-7f55677887-rgdng
Name:         whoami-7f55677887-rgdng
Namespace:    default
Priority:     0
Node:         rm1.iloveyou.cn/192.168.31.20
Start Time:   Sat, 29 Oct 2022 16:49:38 +0800
Labels:       app=whoami
              pod-template-hash=7f55677887
Annotations:  <none>
Status:       Running
IP:           10.42.0.11
IPs:
  IP:           10.42.0.11
Controlled By:  ReplicaSet/whoami-7f55677887
Containers:
  whoami:
    Container ID:   containerd://29b77aacbae468caec850b345c726a6733f597c0148e7f2cc5a8ee799275d898
    Image:          traefik/whoami
    Image ID:       docker.io/traefik/whoami@sha256:24829edb0dbaea072dabd7d902769168542403a8c78a6f743676af431166d7f0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 29 Oct 2022 16:49:41 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-s22gp (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-s22gp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  5m32s  default-scheduler  Successfully assigned default/whoami-7f55677887-rgdng to rm1.iloveyou.cn
  Normal  Pulling    5m32s  kubelet            Pulling image "traefik/whoami"
  Normal  Pulled     5m29s  kubelet            Successfully pulled image "traefik/whoami" in 2.699086068s
  Normal  Created    5m29s  kubelet            Created container whoami
  Normal  Started    5m29s  kubelet            Started container whoami
```

到此 deployment 所有信息查看完成。所有内容都是根据 `whoami-deployment.yml`
生产的，而最终的三个 POD ， 也就是一个容器，具有创建该容器的镜像，
和生产容器的 IP ，端口等信息。

#### 3.3.2 部署 service

部署一个 [service](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/) ，
让上述 deployments 能对外提供访问的方式。

创建文件 `whoami-svc.yml` ，并增加如下内容：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami

spec:
  ports:
    - name: web
      port: 8080
      targetPort: web
  type: LoadBalancer

  selector:
    app: whoami
 
```

使用如下命令创建 services：

```shell
kubectl create -f whoami-svc.yml
```

创建完成后，使用如下命令查看 services：

```shell
kubectl get svc
# 等价 kubectl get service
```

输出如下：

```textmate
[root@rm1 ~]# kubectl create -f whoami-svc.yml
service/whoami created
[root@rm1 ~]# kubectl get svc
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
kubernetes   ClusterIP      10.43.0.1       <none>          443/TCP          20m
whoami       LoadBalancer   10.43.170.168   192.168.31.20   8080:30804/TCP   6s
```

可以看到已经有了一个 `whoami` 的 service ，并且使用 `EXTERNAL-IP` 和对应的端口 `32745` 。

我们可以访问 `http://192.168.31.21:30804` 查看 whoami 应用的内容。

```textmate
[root@km1 ~]# curl http://192.168.31.21:32745
Hostname: whoami-7f55677887-twvct
IP: 127.0.0.1
IP: ::1
IP: 10.42.0.9
IP: fe80::e8be:e1ff:fec7:7cd8
RemoteAddr: 10.42.0.1:52379
GET / HTTP/1.1
Host: 192.168.31.21:32745
User-Agent: curl/7.76.1
Accept: */*
```

至此，我们的应用部署完成，并且通过 service 提供了负载均衡访问。

## 4. 部署 rancher 单节点

为了更方便的管理 k8s 集群，安装 rancher 可以使用更加强大的 UI 界面管理 k8s 。

详细安装步骤请参考 [Helm CLI Quick Start](https://docs.ranchermanager.rancher.io/getting-started/quick-start-guides/deploy-rancher-manager/helm-cli)

### 4.1 配置 k8s 集群访问

参照 [Cluster Access](https://docs.k3s.io/cluster-access) ，生成 k8s 访问配置文件。

```bash
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

### 4.2 安装 helm

根据 [Installing Helm](https://helm.sh/docs/intro/install/) 描述，我们选择使用二进制文件安装。

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
 ./get_helm.sh
```

如果遇到网络问题，可以手动下载 release 包，然后手动安装。

```bash
wget -O /tmp/heml-linux-amd64.tar.gz https://get.helm.sh/helm-v3.10.1-linux-amd64.tar.gz
tar -zxvf /tmp/heml-linux-amd64.tar.gz -C /tmp
cp /tmp/linux-amd64/helm /usr/local/bin
chmod +x /usr/local/bin/helm
```

安装完后，查看现有 helm 应用：

```textmate
[root@rm1 ~]# helm list
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION
[root@rm1 ~]# helm list -A
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
traefik         kube-system     1               2022-10-29 08:43:49.69907183 +0000 UTC  deployed        traefik-12.0.000        2.9.1      
traefik-crd     kube-system     1               2022-10-29 08:43:29.776972256 +0000 UTC deployed        traefik-crd-12.0.000  
```

### 安装证书管理工具

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest

kubectl create namespace cattle-system

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.crds.yaml

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.1
  
 ```

### 安装 rancher

使用如下命令安装：

**注意：**调整访问主机名和默认管理员密码！！！

```bash
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rm1.iloveyou.cn \
  --set replicas=1 \
  --set bootstrapPassword=00000000
```

等两分钟后，访问 [https://rm1.iloveyou.cn/](https://rm1.iloveyou.cn/) 即可打开 rancher 登陆页面。

当遇到证书问题 `NET::ERR_CERT_INVALID` 无法打开时，使用 F5 刷新页面，然后直接使用键盘输入 `thisisusafe` 页面会自动跳转到登陆页面。