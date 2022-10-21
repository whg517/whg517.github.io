---
title: 使用 k3s 部署 k8s 环境
author: wanghuagang
date: 2022-10-22 00:37:28
updated: 2022-10-22 00:37:28
tags:
- k3s
- k8s
categories:
- 运维
---

在中小集群环境中，可以使用 k3s 快速简便安装 k8s 节点和初始化集群。

参考文档：[Quick-Start Guide](https://docs.k3s.io/quick-start)
或者 [中文快速入门指南](https://docs.k3s.io/quick-start)

**注意：** 中文教程中的使用均为国内阿里云加速。

<!--more-->

## 1. 集群配置

| hostname        | ip            | desc         |
|-----------------|---------------|--------------|
| km1.iloveyou.cn | 192.168.31.21 | HA 时为 master |
| km2.iloveyou.cn | 192.168.31.22 | HA 时为 master |
| km3.iloveyou.cn | 192.168.31.23 | HA 时为 master |
| kn1.iloveyou.cn | 192.168.31.24 | HA 时为 node   |
| kn2.iloveyou.cn | 192.168.31.25 | HA 时为 node   |
| kn3.iloveyou.cn | 192.168.31.26 | HA 时为 node   |

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
- 配置主机名：`hostnamectl set-hostname km1.iloveyou.cn` 。主机名根据具体节点调整。
- 配置域名解析或者 hosts 文件。
- 安装基本软件：`dnf install wget vim tar net-tools telnet`

### 2.2 软件环境

**注意：**这一步不是必须的，本步骤的操作只是为了想使用官方文档中的步骤安装，同时
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
下载 k3s 二进制文件并使用 k3s 安装脚本安装。

#### 3.1.1 使用加速镜像安装

```shell
curl -sfL \
    https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    INSTALL_K3S_MIRROR=cn \
    sh -
```

执行结果如下：

```textmate
[root@km1 ~]# curl -sfL \
    https://rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/k3s-install.sh | \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    INSTALL_K3S_MIRROR=cn \
    sh -
[INFO]  Finding release for channel stable
[INFO]  Using v1.24.6+k3s1 as release
[INFO]  Downloading hash rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/v1.24.6-k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary rancher-mirror.oss-cn-beijing.aliyuncs.com/k3s/v1.24.6-k3s1/k3s
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
    INSTALL_K3S_SKIP_DOWNLOAD=true \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    sh -
 
```

这里解释一下上述几个执行步骤：

- 首先从本地服务器下载前面准备的 `k3s` 二进制文件到 `/usr/local/bin/k3s` ，并赋予可执行权限
- 从本地服务器下载前面准备的 `install-k3s.sh` 安装脚本
- `INSTALL_K3S_SKIP_DOWNLOAD=true`：跳过二进制文件下载，使用本地二进制文件，也就是上面已经下载到 `/usr/local/bin/k3s` 的文件了
- `INSTALL_K3S_SKIP_SELINUX_RPM=true`：跳过 SELINUX RPM 检查
- `INSTALL_K3S_SELINUX_WARN=false`：跳过 SELINUX 警告

运行结果如下：

```textmate
[root@km1 ~]# curl -sfl http://192.168.31.10:8080/k3s -o /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s

curl -sfL \
    http://192.168.31.10:8080/install-k3s.sh | \
    INSTALL_K3S_SKIP_DOWNLOAD=true \
    INSTALL_K3S_SKIP_SELINUX_RPM=true \
    INSTALL_K3S_SELINUX_WARN=false \
    sh -

[INFO]  Skipping k3s download and verify
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

### 3.2 检查 k8s

命令执行完成，并没有任何错误后，使用操作检查 k8s 是否正常：

- 检查 k3s 服务：`systemctl status k3s`
- 检查 k8s 的 Pods ：`kubectl get pods -A`

```
[root@km1 ~]# kubectl get pods -A
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-7b7dc8d6f5-tnsnd   1/1     Running     0          70s
kube-system   coredns-b96499967-mrnj4                   1/1     Running     0          70s
kube-system   helm-install-traefik-crd-wzb8j            0/1     Completed   0          70s
kube-system   metrics-server-668d979685-dc8fd           1/1     Running     0          70s
kube-system   helm-install-traefik-7gpq9                0/1     Completed   2          70s
kube-system   svclb-traefik-c7d0cbe9-n8bxr              2/2     Running     0          19s
kube-system   traefik-7cd4fcff68-z5l9n                  0/1     Running     0          19s
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
  replicas: 1
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

该 Web 应用暴露 80 端口。

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
[root@km1 ~]# kubectl get deployments
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
whoami   1/1     1            1           12s
```

使用如下命令查看 deployment 详细内容：

```shell
kubectl describe deployment whoami
```

输出如下：

```textmate
[root@km1 ~]# kubectl describe deployment whoami
Name:                   whoami
Namespace:              default
CreationTimestamp:      Sat, 22 Oct 2022 00:22:36 +0800
Labels:                 app=whoami
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=whoami
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
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
NewReplicaSet:   whoami-7f55677887 (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  72s   deployment-controller  Scaled up replica set whoami-7f55677887 to 1
 
```

可以看到在最后 Events 的 Message 中的信息提示，启动了一个 replica set `whoami-7f55677887` 。

使用如下命令获取 replicaset ：

```shell
kubectl get replicaset
# 等价与 kubectl get rs
```

输出如下：

```textmate
[root@km1 ~]# kubectl get replicaset
NAME                DESIRED   CURRENT   READY   AGE
whoami-7f55677887   1         1         1       2m3s
```

使用如下命令获取 replicaset 详细信息：

```shell
kubectl describe rs whoami
# 等价与 kubectl describe rplicaset whoami
```

输出如下：

```textmate
[root@km1 ~]# kubectl describe rs whoami
Name:           whoami-7f55677887
Namespace:      default
Selector:       app=whoami,pod-template-hash=7f55677887
Labels:         app=whoami
                pod-template-hash=7f55677887
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/whoami
Replicas:       1 current / 1 desired
Pods Status:    1 Running / 0 Waiting / 0 Succeeded / 0 Failed
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
  Normal  SuccessfulCreate  2m30s  replicaset-controller  Created pod: whoami-7f55677887-twvct

```

可以看到在最后 Events 的 Message 中的信息提示中，创建了一个 pod `whoami-7f55677887-twvct` 。

使用如下命令查看 pod：

```shell
kubectl get pods
```

输出如下：

```textmate
[root@km1 ~]# kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
whoami-7f55677887-twvct   1/1     Running   0          3m12s
```

使用如下命令查看 pod 详细信息：

```shell
kubectl describe pod whoami
```

输出如下：

```textmate
[root@km1 ~]# kubectl describe pod whoami
Name:         whoami-7f55677887-twvct
Namespace:    default
Priority:     0
Node:         km1.iloveyou.cn/192.168.31.21
Start Time:   Sat, 22 Oct 2022 00:22:36 +0800
Labels:       app=whoami
              pod-template-hash=7f55677887
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:           10.42.0.9
Controlled By:  ReplicaSet/whoami-7f55677887
Containers:
  whoami:
    Container ID:   containerd://7f5d95d6959b8b7d99c11e199e153d55b5b9c0a4e0175b8b19e320c6e3b11a5a
    Image:          traefik/whoami
    Image ID:       docker.io/traefik/whoami@sha256:24829edb0dbaea072dabd7d902769168542403a8c78a6f743676af431166d7f0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sat, 22 Oct 2022 00:22:46 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-f6c77 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-f6c77:
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
  Normal  Scheduled  3m40s  default-scheduler  Successfully assigned default/whoami-7f55677887-twvct to km1.iloveyou.cn
  Normal  Pulling    3m40s  kubelet            Pulling image "traefik/whoami"
  Normal  Pulled     3m30s  kubelet            Successfully pulled image "traefik/whoami" in 9.861788031s
  Normal  Created    3m30s  kubelet            Created container whoami
  Normal  Started    3m30s  kubelet            Started container whoami
```

到此 deployment 所有信息查看完成。所有内容都是根据 `whoami-deployment.yml`
生产的，而最终的 pod `whoami-7f55677887-twvct` 也就是一个容器，具有创建该容器的镜像，
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
```

输出如下：

```textmate
[root@km1 ~]# kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
kubernetes   ClusterIP      10.43.0.1      <none>          443/TCP          10m
whoami       LoadBalancer   10.43.98.133   192.168.31.21   8080:32745/TCP   5s
```

可以看到已经有了一个 `whoami` 的 service ，并且使用 `EXTERNAL-IP` 和对应的端口 `32745` 。

我们可以访问 `http://192.168.31.21:32745` 查看 whoami 应用的内容。

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
