---
title: kubernetes 安装
author: huagang
date: 2021-11-26 17:14:30
updated: 2021-11-26 17:14:30
tags: 
- k8s
- kubernetes
categories: 运维
---

虽然官方文档中已经有了很详细的安装说明。但是为了快速安装，并强调一些安装过程中的注意事项，仍然需要一篇简单易操作的 kubernetes 安装文档。

而本文的目的就是通过一篇完整的流程来安装 kubernetes ，并在必要的地方做出提示和指引到相关引用链接。

## 环境准备

本次安装环境为 Deepin （基于 debian） 操作系统安装。操作步骤适用于 Debian 和 Ubuntu 系列。如果是在 RHEL 系统中操作，除了步骤和特定操作系统的文件目录不同，其他基本一致。

安装时用了最新版本的 kubernetes 环境。使用了 containerd 作为容器运行层。

在安装过程中有些内容是必须使用科学上网才能进行的，所以在需要地方会配置代理。

<!-- more -->

禁用 SWAP

```bash
swapoff  -a
sed -i '/swap/s/^/#/' /etc/fstab
```

设置主机名

```bash
# 使用自定义的主机名设置。
# hostnamectl set-hostname xxx
```

增加解析。如果有 dns 可以配置 DNS 解析，如果么有，需要配置 hosts 文件解析

增加如下内容到 `/etc/hosts` 文件中：

```text
192.168.2.178 xxx
```

创建一个 kubernetes 安装准备目录，用于存放安装过程中需要的内容。

```bash
mkdir ~/k8s/
```

## 安装

### 安装运行时

kubernetes 是运行在容器运行时(Container Runtime)环境之上的，所以在安装 kubernetes 时，需要先准备容器运行时。当前主流的容器运行时有：

- Docker：在 1.20 弃用，更多细节请参考: [《别慌: Kubernetes 和 Docker》](https://kubernetes.io/zh/blog/2020/12/02/dont-panic-kubernetes-and-docker/)
- containerd
- CRI-O

更详细内容请参考： [容器运行时](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#container-runtimes)

#### 安装 containerd

根据当前社区环境，综合考虑后选择 containerd 作为容器运行时。如果需要使用 Docker 作为底层运行时，请查看文档 [Docker runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker) 。

在安装 containerd 之前，首先根据 [Containerd runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)设置系统环境：

```bash
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置必需的 sysctl 参数，这些参数在重新启动后仍然存在。
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 应用 sysctl 参数而无需重新启动
sudo sysctl --system
```

具体安装步骤请参考： [Getting started with containerd](https://containerd.io/docs/getting-started/)

##### 安装步骤

在 [https://github.com/containerd/containerd/releases] 找到最新的稳定版，下载 `cri-containerd-cni` ，然后直接解压到根目录。

这个过程中会安装 containerd 、相关服务 、 runc 、 cni 。

```bash
export https_proxy=http:127.0.0.1:1081
wget https://github.com/containerd/containerd/releases/download/v1.5.8/cri-containerd-cni-1.5.8-linux-amd64.tar.gz /tmp/
unset https_proxy
tar -zxvf /tmp/cri-containerd-cni-1.5.8-linux-amd64.tar.gz -C /

```

解压输入如下：

```text
etc/
etc/systemd/
etc/systemd/system/
etc/systemd/system/containerd.service
etc/crictl.yaml
etc/cni/
etc/cni/net.d/
etc/cni/net.d/10-containerd-net.conflist
usr/
usr/local/
usr/local/sbin/
usr/local/sbin/runc
usr/local/bin/
usr/local/bin/containerd
usr/local/bin/crictl
usr/local/bin/ctd-decoder
usr/local/bin/critest
usr/local/bin/containerd-shim-runc-v1
usr/local/bin/containerd-shim-runc-v2
usr/local/bin/containerd-stress
usr/local/bin/ctr
usr/local/bin/containerd-shim
opt/
opt/containerd/
opt/containerd/cluster/
opt/containerd/cluster/version
opt/containerd/cluster/gce/
opt/containerd/cluster/gce/configure.sh
opt/containerd/cluster/gce/cni.template
opt/containerd/cluster/gce/cloud-init/
opt/containerd/cluster/gce/cloud-init/node.yaml
opt/containerd/cluster/gce/cloud-init/master.yaml
opt/containerd/cluster/gce/env
opt/cni/
opt/cni/bin/
opt/cni/bin/dhcp
opt/cni/bin/firewall
opt/cni/bin/host-local
opt/cni/bin/ipvlan
opt/cni/bin/sbr
opt/cni/bin/vlan
opt/cni/bin/vrf
opt/cni/bin/tuning
opt/cni/bin/bridge
opt/cni/bin/macvlan
opt/cni/bin/bandwidth
opt/cni/bin/portmap
opt/cni/bin/host-device
opt/cni/bin/ptp
opt/cni/bin/flannel
opt/cni/bin/static
opt/cni/bin/loopback
```

记录一下 containerd 自动生成的 cni 配置 `/etc/cni/net.d/10-containerd-net.conflist` 。以备不时之需。

```json
{
  "cniVersion": "0.4.0",
  "name": "containerd-net",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "promiscMode": true,
      "ipam": {
        "type": "host-local",
        "ranges": [
          [{
            "subnet": "10.88.0.0/16"
          }],
          [{
            "subnet": "2001:4860:4860::/64"
          }]
        ],
        "routes": [
          { "dst": "0.0.0.0/0" },
          { "dst": "::/0" }
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true}
    }
  ]
}
```

##### 配置 containerd 使用代理

安装完成后，为了让 kubernetes 能从 google 拉去 kubernetes 镜像，需要配置 containerd 服务使用代理。
如果直接使用 `export https_proxy` 的方式设置代理， containerd 并不会使用，具体原因未知，
但猜测可能是读取环境变量的 key 值不一样。所以在此针对该服务直接配置。

修改 `/etc/systemd/system/containerd.service` 文件如下：

```ini
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:1080"
Environment="HTTPS_PROXY=socks5://127.0.0.1:1080"
Environment="NO_PROXY=localhost,127.0.0.1,https://registry-1.docker.io"
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

重载服务配置后，重启 containerd ，并检查服务状态：

```bash
systemctl daemon-reload
systemctl restart containerd
systemctl status containerd
```

##### 配置 runc 使用 cgroup

在 kubernetes 文档中提到了配置内容 [使用 systemd cgroup 驱动程序](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)

结合 runc 使用 systemd cgroup 驱动，修改 `/etc/containerd/config.toml` 的 `SystemdCgroup` 为 `true` 。

> 注意：containerd 使用的是默认配置，而且 `/etc/containerd/config.toml` 是没有创建的。根据 [Customizing containerd](https://github.com/containerd/containerd/blob/main/docs/getting-started.md#customizing-containerd) 中的内容，可以使用 `containerd config default > /etc/containerd/config.toml` 生成默认 containerd 默认配置

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

然后重启 containerd ，并查看服务状态：

```bash
systemctl restart containerd
systemctl status containerd
```

##### 镜像加速

在国内使用 docker.io 仓库会时不时抽风，可以通过调整 containerd 配置文件使用国内公共容器加速镜像或者个人的阿里云容器加速镜像。

具体配置说明请参考 [Registry Configuration](https://github.com/containerd/containerd/blob/main/docs/cri/config.md#registry-configuration) 。

增加阿里云个人容器镜像加速：

修改 `/etc/containerd/config.toml` 的配置，调整 `config_path = "/etc/containerd/certs.d"`

```toml
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/containerd/certs.d"

```

然后创建目录 `/etc/containerd/certs.d` 。

创建一个镜像仓库域名的目录。现在需要添加阿里云的镜像仓库 `https://xxxxx.mirror.aliyuncs.com` ，则需要创建一个 `aliyuncs.com` 的目录，并在该目录中
创建一个 `hosts.toml` 的文件，将一下内容加入到该文件中

```toml
server = "https://xxxxx.mirror.aliyuncs.com"
```

所以具体操作如下：

```bash
mkdir /etc/containerd/certs.d/aliyuncs.com/
cat << EOF | sudo tee /etc/containerd/certs.d/aliyuncs.com/hosts.toml
server = "https://xxxxx.mirror.aliyuncs.com"
EOF
```

操作结果如下：

```text
root@kevin:~# tree /etc/containerd/certs.d
/etc/containerd/certs.d
└── aliyuncs.com
    └── hosts.toml

1 directory, 1 file

root@kevin:~# cat /etc/containerd/certs.d/aliyuncs.com/hosts.toml
server = "https://zsh9cr4y.mirror.aliyuncs.com"
```

> 注意：此处配置的加速镜像会通过上面配置的代理进行访问。两边都配置的目的有两个：
>
> 1. kubernetes 中使用的一些镜像并不在 docker.io 中，所以需要使用代理加速
> 2. 普通镜像在 docker.io 中可以使用国内加速，毕竟国内访问还是会比代理快
>
> 所以最佳的做法是：加速和代理都配置。然后在代理服务器上将加速地址配置到白名单中。
> 这样镜像加速时会在代理的白名单中直接访问，除此之外的访问通过代理出去。

#### 安装 cni 插件

为了确保能使用最新稳定版，避免不必要的 BUG ，手动下载安装 cni 插件。

在 [https://github.com/containernetworking/plugins/releases] 中下载 `cni-plugins-linux-amd64` 的插件，然后将其解压：

```bash
export https_proxy=http:127.0.0.1:1081
wget https://github.com/containernetworking/plugins/releases/download/v1.0.1/cni-plugins-linux-amd64-v1.0.1.tgz /tmp/
# 将 cni 插件解压，会自动覆盖的。
tar -zxvf /tmp/cni-plugins-linux-amd64-v1.0.1.tgz -C /opt/cni/bin/
unset https_proxy
```

解压操作后的结果：

```text
./
./macvlan
./static
./vlan
./portmap
./host-local
./vrf
./bridge
./tuning
./firewall
./host-device
./sbr
./loopback
./dhcp
./ptp
./ipvlan
./bandwidth
```

由于在 cni 1.0 时已经将 flannel 独立维护了。所以后面用到的网络工具 flannel 也要升级到最新稳定版。

在 [https://github.com/flannel-io/cni-plugin] 下载 `flannel-amd64` ，然后移动到 cni 中：

```bash
export https_proxy=http:127.0.0.1:1081
# 直接会下载一个二进制文件
wget https://github.com/flannel-io/cni-plugin/releases/download/v1.0.0/flannel-amd64 /tmp/
mv /tmp/flannel-amd64 /opt/cni/flannel
unset https_proxy
```

#### 安装 crictl

将 kubernetes 容器运行时工具 CLI 工具升级到最新版本。

在 [https://github.com/kubernetes-sigs/cri-tools/releases] 中下载最新稳定版，然后解压复制到 `usr/local/bin/crictl` ：

> 注意：使用 wget 下载需要代理

```bash
# 设置系统全局代理
export https_proxy=http:127.0.0.1:1081
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.22.0/crictl-v1.22.0-linux-amd64.tar.gz /tmp/
# 会解压出一个 crictl 的可执行文件
tar -zxf /tmp/crictl-v1.22.0-linux-amd64.tar.gz /tmp
mv /tmp/crictl /usr/local/bin/crictl
# 清楚系统代理，避免后续操作造成影响
unset https_proxy
```

#### 检查 containerd 是否正常

重启 containerd 然后检查 containerd 服务是否正常：

```bash
systemctl restart containerd
systemctl status containerd
```

输出如下：

```text
root@kevin:~# systemctl status containerd
● containerd.service - containerd container runtime
   Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2021-11-26 11:08:31 CST; 2h 28min ago
     Docs: https://containerd.io
 Main PID: 13883 (containerd)
    Tasks: 20
   Memory: 982.0M
   CGroup: /system.slice/containerd.service
           └─13883 /usr/local/bin/containerd

11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.264709688+08:00" level=info msg="cleaning up dead shim"
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.270751576+08:00" level=warning msg="cleanup warnings time=\"2021-11-26T11:42:02+08:00\" level=info msg=\"starting signal loop\" namespace=k8s.io pid=2875\n"
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.308379595+08:00" level=info msg="shim disconnected" id=ac2e6f10600b7791b63442f4a4612618134a1740de6dff4412f31b3b8db218de
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.308409050+08:00" level=warning msg="cleaning up after shim disconnected" id=ac2e6f10600b7791b63442f4a4612618134a1740de6dff4412f31b3b8db218de namespace=k8s.io
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.308417347+08:00" level=info msg="cleaning up dead shim"
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.314814241+08:00" level=warning msg="cleanup warnings time=\"2021-11-26T11:42:02+08:00\" level=info msg=\"starting signal loop\" namespace=k8s.io pid=2927\n"
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.314994781+08:00" level=info msg="TearDown network for sandbox \"ac2e6f10600b7791b63442f4a4612618134a1740de6dff4412f31b3b8db218de\" successfully"
11月 26 11:42:02 kevin containerd[13883]: time="2021-11-26T11:42:02.315008342+08:00" level=info msg="StopPodSandbox for \"ac2e6f10600b7791b63442f4a4612618134a1740de6dff4412f31b3b8db218de\" returns successfully"
```

使用 ctr 命令，检查 containerd 插件是否正常：

```bash
ctr plugin ls
```

输出如下：

```text
root@kevin:~# ctr plugin ls
TYPE                            ID                       PLATFORMS      STATUS    
io.containerd.content.v1        content                  -              ok        
io.containerd.snapshotter.v1    aufs                     linux/amd64    skip      
io.containerd.snapshotter.v1    btrfs                    linux/amd64    skip      
io.containerd.snapshotter.v1    devmapper                linux/amd64    error     
io.containerd.snapshotter.v1    native                   linux/amd64    ok        
io.containerd.snapshotter.v1    overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1    zfs                      linux/amd64    skip      
io.containerd.metadata.v1       bolt                     -              ok        
io.containerd.differ.v1         walking                  linux/amd64    ok        
io.containerd.gc.v1             scheduler                -              ok        
io.containerd.service.v1        introspection-service    -              ok        
io.containerd.service.v1        containers-service       -              ok        
io.containerd.service.v1        content-service          -              ok        
io.containerd.service.v1        diff-service             -              ok        
io.containerd.service.v1        images-service           -              ok        
io.containerd.service.v1        leases-service           -              ok        
io.containerd.service.v1        namespaces-service       -              ok        
io.containerd.service.v1        snapshots-service        -              ok        
io.containerd.runtime.v1        linux                    linux/amd64    ok        
io.containerd.runtime.v2        task                     linux/amd64    ok        
io.containerd.monitor.v1        cgroups                  linux/amd64    ok        
io.containerd.service.v1        tasks-service            -              ok        
io.containerd.internal.v1       restart                  -              ok        
io.containerd.grpc.v1           containers               -              ok        
io.containerd.grpc.v1           content                  -              ok        
io.containerd.grpc.v1           diff                     -              ok        
io.containerd.grpc.v1           events                   -              ok        
io.containerd.grpc.v1           healthcheck              -              ok        
io.containerd.grpc.v1           images                   -              ok        
io.containerd.grpc.v1           leases                   -              ok        
io.containerd.grpc.v1           namespaces               -              ok        
io.containerd.internal.v1       opt                      -              ok        
io.containerd.grpc.v1           snapshots                -              ok        
io.containerd.grpc.v1           tasks                    -              ok        
io.containerd.grpc.v1           version                  -              ok        
io.containerd.grpc.v1           cri                      linux/amd64    ok
```

使用 crictl 检查 cri 是否正常：

```bash
crictl --version
crictl info
```

输入如下：
<!-- markdownlint-disable MD013 MD033-->
```text
root@kevin:~# crictl --version
crictl version v1.22.0
root@kevin:~# crictl info
DEBU[0000] get runtime connection                       
DEBU[0000] connect using endpoint 'unix:///run/containerd/containerd.sock' with '2s' timeout 
DEBU[0000] connected successfully using endpoint: unix:///run/containerd/containerd.sock 
DEBU[0000] StatusRequest: &StatusRequest{Verbose:true,} 
DEBU[0000] StatusResponse: &StatusResponse{Status:&RuntimeStatus{Conditions:[]*RuntimeCondition{&RuntimeCondition{Type:RuntimeReady,Status:true,Reason:,Message:,},&RuntimeCondition{Type:NetworkReady,Status:true,Reason:,Message:,},},},Info:map[string]string{cniconfig: {"PluginDirs":["/opt/cni/bin"],"PluginConfDir":"/etc/cni/net.d","PluginMaxConfNum":1,"Prefix":"eth","Networks":[{"Config":{"Name":"cni-loopback","CNIVersion":"0.3.1","Plugins":[{"Network":{"type":"loopback","ipam":{},"dns":{}},"Source":"{\"type\":\"loopback\"}"}],"Source":"{\n\"cniVersion\": \"0.3.1\",\n\"name\": \"cni-loopback\",\n\"plugins\": [{\n  \"type\": \"loopback\"\n}]\n}"},"IFName":"lo"},{"Config":{"Name":"containerd-net","CNIVersion":"0.4.0","Plugins":[{"Network":{"type":"bridge","ipam":{"type":"host-local"},"dns":{}},"Source":"{\"bridge\":\"cni0\",\"ipMasq\":true,\"ipam\":{\"ranges\":[[{\"subnet\":\"10.88.0.0/16\"}],[{\"subnet\":\"2001:4860:4860::/64\"}]],\"routes\":[{\"dst\":\"0.0.0.0/0\"},{\"dst\":\"::/0\"}],\"type\":\"host-local\"},\"isGateway\":true,\"promiscMode\":true,\"type\":\"bridge\"}"},{"Network":{"type":"portmap","capabilities":{"portMappings":true},"ipam":{},"dns":{}},"Source":"{\"capabilities\":{\"portMappings\":true},\"type\":\"portmap\"}"}],"Source":"{\n  \"cniVersion\": \"0.4.0\",\n  \"name\": \"containerd-net\",\n  \"plugins\": [\n    {\n      \"type\": \"bridge\",\n      \"bridge\": \"cni0\",\n      \"isGateway\": true,\n      \"ipMasq\": true,\n      \"promiscMode\": true,\n      \"ipam\": {\n        \"type\": \"host-local\",\n        \"ranges\": [\n          [{\n            \"subnet\": \"10.88.0.0/16\"\n          }],\n          [{\n            \"subnet\": \"2001:4860:4860::/64\"\n          }]\n        ],\n        \"routes\": [\n          { \"dst\": \"0.0.0.0/0\" },\n          { \"dst\": \"::/0\" }\n        ]\n      }\n    },\n    {\n      \"type\": \"portmap\",\n      \"capabilities\": {\"portMappings\": true}\n    }\n  ]\n}\n"},"IFName":"eth0"}]},config: {"containerd":{"snapshotter":"overlayfs","defaultRuntimeName":"runc","defaultRuntime":{"runtimeType":"","runtimeEngine":"","PodAnnotations":[],"ContainerAnnotations":[],"runtimeRoot":"","options":{},"privileged_without_host_devices":false,"baseRuntimeSpec":""},"untrustedWorkloadRuntime":{"runtimeType":"","runtimeEngine":"","PodAnnotations":[],"ContainerAnnotations":[],"runtimeRoot":"","options":{},"privileged_without_host_devices":false,"baseRuntimeSpec":""},"runtimes":{"runc":{"runtimeType":"io.containerd.runc.v2","runtimeEngine":"","PodAnnotations":[],"ContainerAnnotations":[],"runtimeRoot":"","options":{"BinaryName":"","CriuImagePath":"","CriuPath":"","CriuWorkPath":"","IoGid":0,"IoUid":0,"NoNewKeyring":false,"NoPivotRoot":false,"Root":"","ShimCgroup":"","SystemdCgroup":true},"privileged_without_host_devices":false,"baseRuntimeSpec":""}},"noPivot":false,"disableSnapshotAnnotations":true,"discardUnpackedLayers":false},"cni":{"binDir":"/opt/cni/bin","confDir":"/etc/cni/net.d","maxConfNum":1,"confTemplate":""},"registry":{"configPath":"","mirrors":{},"configs":{},"auths":{},"headers":{"User-Agent":["containerd/v1.5.8"]}},"imageDecryption":{"keyModel":"node"},"disableTCPService":true,"streamServerAddress":"127.0.0.1","streamServerPort":"0","streamIdleTimeout":"4h0m0s","enableSelinux":false,"selinuxCategoryRange":1024,"sandboxImage":"k8s.gcr.io/pause:3.5","statsCollectPeriod":10,"systemdCgroup":false,"enableTLSStreaming":false,"x509KeyPairStreaming":{"tlsCertFile":"","tlsKeyFile":""},"maxContainerLogSize":16384,"disableCgroup":false,"disableApparmor":false,"restrictOOMScoreAdj":false,"maxConcurrentDownloads":3,"disableProcMount":false,"unsetSeccompProfile":"","tolerateMissingHugetlbController":true,"disableHugetlbController":true,"ignoreImageDefinedVolumes":false,"netnsMountsUnderStateDir":false,"containerdRootDir":"/var/lib/containerd","containerdEndpoint":"/run/containerd/containerd.sock","rootDir":"/var/lib/containerd/io.containerd.grpc.v1.cri","stateDir":"/run/containerd/io.containerd.grpc.v1.cri"},golang: "go1.16.10",lastCNILoadStatus: OK,},} 
{
  "status": {
    "conditions": [
      {
        "type": "RuntimeReady",
        "status": true,
        "reason": "",
        "message": ""
      },
      {
        "type": "NetworkReady",
        "status": true,
        "reason": "",
        "message": ""
      }
    ]
  },
  "cniconfig": {
    "PluginDirs": [
      "/opt/cni/bin"
    ],
    "PluginConfDir": "/etc/cni/net.d",
    "PluginMaxConfNum": 1,
    "Prefix": "eth",
    "Networks": [
      {
        "Config": {
          "Name": "cni-loopback",
          "CNIVersion": "0.3.1",
          "Plugins": [
            {
              "Network": {
                "type": "loopback",
                "ipam": {},
                "dns": {}
              },
              "Source": "{\"type\":\"loopback\"}"
            }
          ],
          "Source": "{\n\"cniVersion\": \"0.3.1\",\n\"name\": \"cni-loopback\",\n\"plugins\": [{\n  \"type\": \"loopback\"\n}]\n}"
        },
        "IFName": "lo"
      },
      {
        "Config": {
          "Name": "containerd-net",
          "CNIVersion": "0.4.0",
          "Plugins": [
            {
              "Network": {
                "type": "bridge",
                "ipam": {
                  "type": "host-local"
                },
                "dns": {}
              },
              "Source": "{\"bridge\":\"cni0\",\"ipMasq\":true,\"ipam\":{\"ranges\":[[{\"subnet\":\"10.88.0.0/16\"}],[{\"subnet\":\"2001:4860:4860::/64\"}]],\"routes\":[{\"dst\":\"0.0.0.0/0\"},{\"dst\":\"::/0\"}],\"type\":\"host-local\"},\"isGateway\":true,\"promiscMode\":true,\"type\":\"bridge\"}"
            },
            {
              "Network": {
                "type": "portmap",
                "capabilities": {
                  "portMappings": true
                },
                "ipam": {},
                "dns": {}
              },
              "Source": "{\"capabilities\":{\"portMappings\":true},\"type\":\"portmap\"}"
            }
          ],
          "Source": "{\n  \"cniVersion\": \"0.4.0\",\n  \"name\": \"containerd-net\",\n  \"plugins\": [\n    {\n      \"type\": \"bridge\",\n      \"bridge\": \"cni0\",\n      \"isGateway\": true,\n      \"ipMasq\": true,\n      \"promiscMode\": true,\n      \"ipam\": {\n        \"type\": \"host-local\",\n        \"ranges\": [\n          [{\n            \"subnet\": \"10.88.0.0/16\"\n          }],\n          [{\n            \"subnet\": \"2001:4860:4860::/64\"\n          }]\n        ],\n        \"routes\": [\n          { \"dst\": \"0.0.0.0/0\" },\n          { \"dst\": \"::/0\" }\n        ]\n      }\n    },\n    {\n      \"type\": \"portmap\",\n      \"capabilities\": {\"portMappings\": true}\n    }\n  ]\n}\n"
        },
        "IFName": "eth0"
      }
    ]
  },
  "config": {
    "containerd": {
      "snapshotter": "overlayfs",
      "defaultRuntimeName": "runc",
      "defaultRuntime": {
        "runtimeType": "",
        "runtimeEngine": "",
        "PodAnnotations": [],
        "ContainerAnnotations": [],
        "runtimeRoot": "",
        "options": {},
        "privileged_without_host_devices": false,
        "baseRuntimeSpec": ""
      },
      "untrustedWorkloadRuntime": {
        "runtimeType": "",
        "runtimeEngine": "",
        "PodAnnotations": [],
        "ContainerAnnotations": [],
        "runtimeRoot": "",
        "options": {},
        "privileged_without_host_devices": false,
        "baseRuntimeSpec": ""
      },
      "runtimes": {
        "runc": {
          "runtimeType": "io.containerd.runc.v2",
          "runtimeEngine": "",
          "PodAnnotations": [],
          "ContainerAnnotations": [],
          "runtimeRoot": "",
          "options": {
            "BinaryName": "",
            "CriuImagePath": "",
            "CriuPath": "",
            "CriuWorkPath": "",
            "IoGid": 0,
            "IoUid": 0,
            "NoNewKeyring": false,
            "NoPivotRoot": false,
            "Root": "",
            "ShimCgroup": "",
            "SystemdCgroup": true
          },
          "privileged_without_host_devices": false,
          "baseRuntimeSpec": ""
        }
      },
      "noPivot": false,
      "disableSnapshotAnnotations": true,
      "discardUnpackedLayers": false
    },
    "cni": {
      "binDir": "/opt/cni/bin",
      "confDir": "/etc/cni/net.d",
      "maxConfNum": 1,
      "confTemplate": ""
    },
    "registry": {
      "configPath": "",
      "mirrors": {},
      "configs": {},
      "auths": {},
      "headers": {
        "User-Agent": [
          "containerd/v1.5.8"
        ]
      }
    },
    "imageDecryption": {
      "keyModel": "node"
    },
    "disableTCPService": true,
    "streamServerAddress": "127.0.0.1",
    "streamServerPort": "0",
    "streamIdleTimeout": "4h0m0s",
    "enableSelinux": false,
    "selinuxCategoryRange": 1024,
    "sandboxImage": "k8s.gcr.io/pause:3.5",
    "statsCollectPeriod": 10,
    "systemdCgroup": false,
    "enableTLSStreaming": false,
    "x509KeyPairStreaming": {
      "tlsCertFile": "",
      "tlsKeyFile": ""
    },
    "maxContainerLogSize": 16384,
    "disableCgroup": false,
    "disableApparmor": false,
    "restrictOOMScoreAdj": false,
    "maxConcurrentDownloads": 3,
    "disableProcMount": false,
    "unsetSeccompProfile": "",
    "tolerateMissingHugetlbController": true,
    "disableHugetlbController": true,
    "ignoreImageDefinedVolumes": false,
    "netnsMountsUnderStateDir": false,
    "containerdRootDir": "/var/lib/containerd",
    "containerdEndpoint": "/run/containerd/containerd.sock",
    "rootDir": "/var/lib/containerd/io.containerd.grpc.v1.cri",
    "stateDir": "/run/containerd/io.containerd.grpc.v1.cri"
  },
  "golang": "go1.16.10",
  "lastCNILoadStatus": "OK"
}
```
<!-- markdownlint-restore -->
这这一步已经要注意的是 `lastCNILoadStatus` 要正常。如果 CNI 不正常，后面的 kubernetes 也会无法正常启动。

### 安装 kubeadm、kubelet 和 kubectl

根据 [安装 kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) 的描述，可以有多种安装方法。但在这个过程中使用的是 google 的仓库安装。在国内环境，需要科学上网才可以。当使用系统工具(apt / dnf / yum)安装时可以使用阿里云镜像加速。

Debian/Ubuntu 的国内镜像配置入选：

```bash
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

其他系统请参考： [kubernetes 镜像](https://developer.aliyun.com/mirror/kubernetes)

> 注意：这个过程中可能无法安装最新版本的 kubernetes 。所以安装完成后需要检查。如果需要最新版本，
> 可以通过文档 [安装 kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl) 的手动安装方式，
> 从 Github 中下载最新发布版本安装。但下载时请准备代理。

## 创建集群

> 注意：具体初始化操作在后面，这里是作为说明。如果你需要立即初始化集群，请跳过此节。
>
> 提示：安装过程中，遇到问题请跳转到后面的 问题记录 和重置集群。不要重装系统。

使用 `kubeadm init` 初始化集群主节点。初始化时，需要调整集群参数，有两种方式：

- 通过命令行传递自定义参数：不便于调整和记录
- 通过配置文件传递自定义参数：方便记录和调整

有关 `kubeadm` 参数的更多信息，请参见 [kubeadm 参考指南](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)。

### 初始化配置

kubernetes 配置手册文档：[kubeadm Configuration (v1beta2)](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/) 。

文档中描述了 kubeadm 的配置内容。

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration

apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration

apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration

apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
```

#### 默认配置

> 如果你需要立即初始化集群，请跳过此节。

```bash
kubeadm config print init-defaults
```

命令输出如下：

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.22.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

该配置为集群默认配置，里面很多内容并不符合实际环境。

#### 解读 kubeadm Configuration (v1beta2)

> 如果你需要立即初始化集群，请跳过此节。

[kubeadm Configuration (v1beta2)](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/) 是使用 kubeadm 初始化集群节点时，传递
配置文件的说明文档。无论是使用  `init` 初始化一个集群主节点，还是使用 `join` 将一个新的节点加入到现有集群中个，都可以使用 `--config kubeadm-config.yaml`
传递配置项，而不是使用冗长的命令行参数。具体相信内容，请阅读文档，这里仅做一些简单介绍。

配置支持如下类型：

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration

apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration

apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration

apiVersion: kubeadm.k8s.io/v1beta2
kind: JoinConfiguration
```

##### InitConfiguration

```yaml
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: node
  taints: null
```

[`InitConfiguration`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/#kubeadm-k8s-io-v1beta2-InitConfiguration) 用来做初始化运行时环境的。其中：

- [`NodeRegistration`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/#kubeadm-k8s-io-v1beta2-NodeRegistrationOptions) ：将新节点注册到集群中的相关字段。可以定义节点名称，要使用的 CRI 套接字和任何仅用于该节点的设置。
- [`LocalAPIEndpoint`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/#kubeadm-k8s-io-v1beta2-APIEndpoint) : 表示要部署在节点的 API 服务器实例的端点。

在 API 服务器和控制平面节点的注意事项请参考： [Considerations about apiserver-advertise-address and ControlPlaneEndpoint](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#considerations-about-apiserver-advertise-address-and-controlplaneendpoint)

##### ClusterConfiguration

```yaml
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta3
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns: {}
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.22.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

[`ClusterConfiguration`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/#kubeadm-k8s-io-v1beta2-ClusterConfiguration) 是用来做集群配置的。

这个配置主要注意如下配置内容：

- [`Networking`](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/#kubeadm-k8s-io-v1beta2-Networking) ：用来这只集群网络的。
- `podSubnet` : 在 [安装 Pod 网络附加组件](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)
中提到了在执行 `kubeadm init` 时使用 `--pod-network-cidr` 参数修改网络插件的 CIDR 。
后面会使用 [flannel](https://github.com/flannel-io/flannel) 作为网络服务，所以这里需要修改为 flannel 默认的 CIDR 。

##### KubeletConfiguration

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
failSwapOn: false
```

这个配置在 [kubeadm Configuration (v1beta2)](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/) 中并没有单独列出来，而是简单的提及。
配置项的具体参数在： [https://godoc.org/k8s.io/kubelet/config/v1beta1#KubeletConfiguration] ，命令行参数在 [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/) 。

KubeletConfiguration 配置唯一需要注意的是 `cgroupDriver` ，这是指定 kubelet 使用哪种 `cgroup driver` 。

根据 [Configuring the kubelet cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/#configuring-the-kubelet-cgroup-driver) 中的说明，在 `v1.22` 版本，如果没有配置 `cgroupDriver` 则默认使用 `systemd` 。

##### KubeProxyConfiguration

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs
```

这个配置在 [kubeadm Configuration (v1beta2)](https://kubernetes.io/docs/reference/config-api/kubeadm-config.v1beta2/) 中并没有单独列出来，而是简单的提及。
配置项的具体参数在： [https://godoc.org/k8s.io/kube-proxy/config/v1alpha1#KubeProxyConfiguration] ，命令行参数在 [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) 。

对于这个配置需要注意的就是调整 `mode` 为 `ipvs` 。

`mode` 的默认值为 `iptables` 。可选的有 `userspace` 、 `iptables` 、 `ipvs` 、 `kernelspace` 。

##### 一个完整的示例配置

这是一个完整参数的示例配置，包含执行 `kubeadm init` 时使用的多个配置项。

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: InitConfiguration
bootstrapTokens:
  - token: "9a08jv.c0izixklcxtmnze7"
    description: "kubeadm bootstrap token"
    ttl: "24h"
  - token: "783bde.3f89s0fje9f38fhf"
    description: "another bootstrap token"
    usages:
      - authentication
      - signing
    groups:
      - system:bootstrappers:kubeadm:default-node-token
nodeRegistration:
  name: "ec2-10-100-0-1"
  criSocket: "/var/run/dockershim.sock"
  taints:
    - key: "kubeadmNode"
      value: "master"
      effect: "NoSchedule"
  kubeletExtraArgs:
    cgroup-driver: "cgroupfs"
  ignorePreflightErrors:
    - IsPrivilegedUser
localAPIEndpoint:
  advertiseAddress: "10.100.0.1"
  bindPort: 6443
certificateKey: "e6a2eb8581237ab72a4f494f30285ec12a9694d750b9785706a83bfcbbbd2204"
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
etcd:
  # one of local or external
  local:
    imageRepository: "k8s.gcr.io"
    imageTag: "3.2.24"
    dataDir: "/var/lib/etcd"
    extraArgs:
      listen-client-urls: "http://10.100.0.1:2379"
    serverCertSANs:
      - "ec2-10-100-0-1.compute-1.amazonaws.com"
    peerCertSANs:
      - "10.100.0.1"
  # external:
    # endpoints:
    # - "10.100.0.1:2379"
    # - "10.100.0.2:2379"
    # caFile: "/etcd/kubernetes/pki/etcd/etcd-ca.crt"
    # certFile: "/etcd/kubernetes/pki/etcd/etcd.crt"
    # keyFile: "/etcd/kubernetes/pki/etcd/etcd.key"
 networking:
   serviceSubnet: "10.96.0.0/12"
   podSubnet: "10.100.0.1/24"
   dnsDomain: "cluster.local"
 kubernetesVersion: "v1.12.0"
 controlPlaneEndpoint: "10.100.0.1:6443"
 apiServer:
   extraArgs:
     authorization-mode: "Node,RBAC"
   extraVolumes:
     - name: "some-volume"
       hostPath: "/etc/some-path"
       mountPath: "/etc/some-pod-path"
       readOnly: false
       pathType: File
   certSANs:
     - "10.100.1.1"
     - "ec2-10-100-0-1.compute-1.amazonaws.com"
   timeoutForControlPlane: 4m0s
 controllerManager:
   extraArgs:
     "node-cidr-mask-size": "20"
   extraVolumes:
     - name: "some-volume"
       hostPath: "/etc/some-path"
       mountPath: "/etc/some-pod-path"
       readOnly: false
       pathType: File
 scheduler:
   extraArgs:
     address: "10.100.0.1"
   extraVolumes:
     - name: "some-volume"
       hostPath: "/etc/some-path"
       mountPath: "/etc/some-pod-path"
       readOnly: false
       pathType: File
certificatesDir: "/etc/kubernetes/pki"
imageRepository: "k8s.gcr.io"
useHyperKubeImage: false
clusterName: "example-cluster"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
# kubelet specific options here
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
# kube-proxy specific options here
```

#### 自定义配置

这里是一个本次需要使用的配置项。

```bash
mkdir ~/k8s
cat << "EOF" > ~/k8s/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.2.178 # 节点主机地址
  bindPort: 6443  # kubelet 服务端口
nodeRegistration:
  criSocket: /run/containerd/containerd.sock  # 使用的容器运行时的 socket
  taints:
    - key: "kubeadmNode"
      value: "master"
      effect: "NoSchedule"
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: 1.22.4
networking:
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16  # 网络插件的网络地址。flannel 默认 CIDR 是 `10.244.0.0/16`
  dnsDomain: cluster.local
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs  # 指定使用 ipvs 。

EOF
```

### 开始安装

#### 初始化集群

操作分为两部，先是初始化操作，完成后，然后配置用户读取集群配置。

```bash
# 使用配置文件初始化 kubeadm
kubeadm init --config=kubeadm-config.yaml  --upload-certs
```

命令输出：

```text
root@kevin:~/k8s# kubeadm init --config=kubeadm-config.yaml  --upload-certs
[init] Using Kubernetes version: v1.22.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kevin kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.2.178]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kevin localhost] and IPs [192.168.2.178 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kevin localhost] and IPs [192.168.2.178 127.0.0.1 ::1]
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
[apiclient] All control plane components are healthy after 10.501446 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
99c0a6631fdbc9d3651d31724719ec92b45c0c07d9b0e942ad38be2e8023cef6
[mark-control-plane] Marking the node kevin as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node kevin as control-plane by adding the taints [kubeadmNode=master:NoSchedule]
[bootstrap-token] Using token: hlohmm.qcqnpt0pz8b7hvju
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

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.2.178:6443 --token hlohmm.qcqnpt0pz8b7hvju \
        --discovery-token-ca-cert-hash sha256:7f894a01b200026e630b1e218847b17733c5d0887082cbab6fc6ff21296c370a
```

当出现上述输出内容，则说明集群已经初始化，但是集群并没完全初始化。因为还没有配置网络，所以集群并没有处于完全可用状态。

在输出的最后几行中，提到了如何使用集群。如果是一个普通用户，则需要为普通用户增加集群配置。具体操作如下：

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

如果是 root 用户，则可用直接使用环境变量。当然也可以使用上面的操作，为 root 用户增加自己的配置。

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

**注意：**不要让普通用户使用 `/etc/kubernetes/admin.conf` 配置。

当集群正常运行后，可以在其他节点通过 `kubeadm join` 加入当前节点：

```bash

kubeadm join 192.168.2.178:6443 --token ykz46g.whwm8guooqxve540 \
        --discovery-token-ca-cert-hash sha256:de290fa76a14f7b6059d15f155191e70bcd9ae36a317df98bcad90e3e3213478
```

##### 查看节点信息

使用 `kubectl get nodes` 获取节点信息

```text
root@kevin:~/k8s# kubectl get nodes
NAME    STATUS     ROLES                  AGE   VERSION
kevin   NotReady   control-plane,master   7s    v1.22.4
root@kevin:~/k8s# kubectl get nodes
NAME    STATUS   ROLES                  AGE   VERSION
kevin   Ready    control-plane,master   9s    v1.22.4
```

这里可以看到节点 `kevin` 的状态为 `NotReady` 。这说明集群还没完全启动。

##### 查看节点描述

使用 `kubectl describe nodes` 获取节点描述信息。如果需要获取具体节点可以使用 `kubectl describe nodes kevin` 。

```text
root@kevin:~/k8s# kubectl describe nodes
Name:               kevin
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kevin
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 26 Nov 2021 15:53:50 +0800
Taints:             kubeadmNode=master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  kevin
  AcquireTime:     <unset>
  RenewTime:       Fri, 26 Nov 2021 15:56:00 +0800
Conditions:
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Fri, 26 Nov 2021 15:54:06 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Fri, 26 Nov 2021 15:54:06 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Fri, 26 Nov 2021 15:54:06 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            True    Fri, 26 Nov 2021 15:54:06 +0800   Fri, 26 Nov 2021 15:54:06 +0800   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  192.168.2.178
  Hostname:    kevin
Capacity:
  cpu:                4
  ephemeral-storage:  102709252Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             32768344Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  94656846487
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             32665944Ki
  pods:               110
System Info:
  Machine ID:                 61a427a0a4c64788be4749626b521f2d
  System UUID:                031b021c-040d-05cc-5906-500700080009
  Boot ID:                    78a72cbd-606e-4d2e-8341-761c3595c959
  Kernel Version:             5.15.1-amd64-desktop
  OS Image:                   Deepin 20.3
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.5.8
  Kubelet Version:            v1.22.4
  Kube-Proxy Version:         v1.22.4
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (5 in total)
  Namespace                   Name                             CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                             ------------  ----------  ---------------  -------------  ---
  kube-system                 etcd-kevin                       100m (2%)     0 (0%)      100Mi (0%)       0 (0%)         2m12s
  kube-system                 kube-apiserver-kevin             250m (6%)     0 (0%)      0 (0%)           0 (0%)         2m12s
  kube-system                 kube-controller-manager-kevin    200m (5%)     0 (0%)      0 (0%)           0 (0%)         2m12s
  kube-system                 kube-proxy-zz7ch                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m3s
  kube-system                 kube-scheduler-kevin             100m (2%)     0 (0%)      0 (0%)           0 (0%)         2m18s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                650m (16%)  0 (0%)
  memory             100Mi (0%)  0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age                    From        Message
  ----     ------                   ----                   ----        -------
  Normal   Starting                 2m2s                   kube-proxy  
  Normal   NodeHasSufficientMemory  2m24s (x5 over 2m24s)  kubelet     Node kevin status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    2m24s (x5 over 2m24s)  kubelet     Node kevin status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     2m24s (x4 over 2m24s)  kubelet     Node kevin status is now: NodeHasSufficientPID
  Normal   Starting                 2m12s                  kubelet     Starting kubelet.
  Normal   NodeHasSufficientMemory  2m12s                  kubelet     Node kevin status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    2m12s                  kubelet     Node kevin status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     2m12s                  kubelet     Node kevin status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  2m12s                  kubelet     Updated Node Allocatable limit across pods
  Warning  InvalidDiskCapacity      2m12s                  kubelet     invalid capacity 0 on image filesystem
  Normal   NodeReady                2m4s                   kubelet     Node kevin status is now: NodeReady
```

从状态信息可以看到集群处于无法调度状态。

##### 获取所有 pod

使用 `kubectl get pods --all-namespaces` 获取所有名称空间的 pod 。

```text
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-4wx58        0/1     Pending   0          31s
kube-system   coredns-78fcd69978-kh2h4        0/1     Pending   0          31s
kube-system   etcd-kevin                      1/1     Running   3          41s
kube-system   kube-apiserver-kevin            1/1     Running   3          41s
kube-system   kube-controller-manager-kevin   1/1     Running   3          40s
kube-system   kube-proxy-gm7gw                1/1     Running   0          31s
kube-system   kube-scheduler-kevin            1/1     Running   3          41s
```

可以看到现在启动的 pod 都在 `kube-system` 的名称空间中。从输出信息中可以看得到， coredns 的状态为 READY 。这是因为节点的污点标记为 NoSchedule ，
所以 coredns 不能被调度。

查看 coredns pod 的描述信息：

```text
root@kevin:~/k8s# kubectl -n kube-system describe pod coredns
Name:                 coredns-78fcd69978-9zj7b
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 <none>
Labels:               k8s-app=kube-dns
                      pod-template-hash=78fcd69978
Annotations:          <none>
Status:               Pending
IP:                   
IPs:                  <none>
Controlled By:        ReplicaSet/coredns-78fcd69978
Containers:
  coredns:
    Image:       k8s.gcr.io/coredns/coredns:v1.8.4
    Ports:       53/UDP, 53/TCP, 9153/TCP
    Host Ports:  0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-m6bvz (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-m6bvz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  31s (x3 over 2m48s)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {kubeadmNode: master}, that the pod didn't tolerate.

Name:                 coredns-78fcd69978-wn4l7
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 <none>
Labels:               k8s-app=kube-dns
                      pod-template-hash=78fcd69978
Annotations:          <none>
Status:               Pending
IP:                   
IPs:                  <none>
Controlled By:        ReplicaSet/coredns-78fcd69978
Containers:
  coredns:
    Image:       k8s.gcr.io/coredns/coredns:v1.8.4
    Ports:       53/UDP, 53/TCP, 9153/TCP
    Host Ports:  0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7cf5h (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-7cf5h:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  31s (x3 over 2m48s)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {kubeadmNode: master}, that the pod didn't tolerate.
```

可以看到最后的输出中描述为 `0/1 nodes are available: 1 node(s) had taint {kubeadmNode: master}, that the pod didn't tolerate.` 。因为没有节点可用，仅有的
一个节点存在污点，所以调度失败。

#### 安装 pod 网络组件

使用 [flannel](https://github.com/flannel-io/flannel) 初始化网络组件。

首先下载 [flannel 配置](https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml) 。由于这个地址访问需要
代理，所以建议先下载保存后再执行。

```bash
export https_proxy=http://127.0.0.1:1081
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml ~/k8s/kube-flannel.yml
unset https_proxy
kubectl apply -f ~/k8s/kube-flannel.yml 
```

执行输入结果如下：

```text
oot@kevin:~/k8s# kubectl apply -f ~/k8s/kube-flannel.yml 
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

##### 检查网络插件 flannel 状态

列出所有 pod

```text
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS     RESTARTS   AGE
kube-system   coredns-78fcd69978-9zj7b        0/1     Pending    0          5m7s
kube-system   coredns-78fcd69978-wn4l7        0/1     Pending    0          5m7s
kube-system   etcd-kevin                      1/1     Running    4          5m16s
kube-system   kube-apiserver-kevin            1/1     Running    4          5m16s
kube-system   kube-controller-manager-kevin   1/1     Running    4          5m16s
kube-system   kube-flannel-ds-l4bdz           0/1     Init:1/2   0          2s
kube-system   kube-proxy-zz7ch                1/1     Running    0          5m7s
kube-system   kube-scheduler-kevin            1/1     Running    4          5m22s
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS            RESTARTS   AGE
kube-system   coredns-78fcd69978-9zj7b        0/1     Pending           0          5m8s
kube-system   coredns-78fcd69978-wn4l7        0/1     Pending           0          5m8s
kube-system   etcd-kevin                      1/1     Running           4          5m17s
kube-system   kube-apiserver-kevin            1/1     Running           4          5m17s
kube-system   kube-controller-manager-kevin   1/1     Running           4          5m17s
kube-system   kube-flannel-ds-l4bdz           0/1     PodInitializing   0          3s
kube-system   kube-proxy-zz7ch                1/1     Running           0          5m8s
kube-system   kube-scheduler-kevin            1/1     Running           4          5m23s
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-9zj7b        0/1     Pending   0          5m9s
kube-system   coredns-78fcd69978-wn4l7        0/1     Pending   0          5m9s
kube-system   etcd-kevin                      1/1     Running   4          5m18s
kube-system   kube-apiserver-kevin            1/1     Running   4          5m18s
kube-system   kube-controller-manager-kevin   1/1     Running   4          5m18s
kube-system   kube-flannel-ds-l4bdz           1/1     Running   0          4s
kube-system   kube-proxy-zz7ch                1/1     Running   0          5m9s
kube-system   kube-scheduler-kevin            1/1     Running   4          5m24s
```

查看 flannel pod 描述

```text
root@kevin:~/k8s# kubectl -n kube-system describe pod kube-flannel-ds
Name:                 kube-flannel-ds-l4bdz
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 kevin/192.168.2.178
Start Time:           Fri, 26 Nov 2021 15:59:12 +0800
Labels:               app=flannel
                      controller-revision-hash=cdfcf6866
                      pod-template-generation=1
                      tier=node
Annotations:          <none>
Status:               Running
IP:                   192.168.2.178
IPs:
  IP:           192.168.2.178
Controlled By:  DaemonSet/kube-flannel-ds
Init Containers:
  install-cni-plugin:
    Container ID:  containerd://ae2fa326701cce618e1f360cb8a52798b1c24f72615db1a7bc0b6ea095006fbb
    Image:         rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.0
    Image ID:      docker.io/rancher/mirrored-flannelcni-flannel-cni-plugin@sha256:bfe8f30c74bc6f31eba0cc6659e396dbdd5ab171314ed542cc238ae046660ede
    Port:          <none>
    Host Port:     <none>
    Command:
      cp
    Args:
      -f
      /flannel
      /opt/cni/bin/flannel
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 26 Nov 2021 15:59:13 +0800
      Finished:     Fri, 26 Nov 2021 15:59:13 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /opt/cni/bin from cni-plugin (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9kzvz (ro)
  install-cni:
    Container ID:  containerd://9b28d86fe319ad34eaee3e935883c36fc05794c5e06d8bc9d9e13fbfdfda6477
    Image:         quay.io/coreos/flannel:v0.15.1
    Image ID:      quay.io/coreos/flannel@sha256:9a296fbb67790659adc3701e287adde3c59803b7fcefe354f1fc482840cdb3d9
    Port:          <none>
    Host Port:     <none>
    Command:
      cp
    Args:
      -f
      /etc/kube-flannel/cni-conf.json
      /etc/cni/net.d/10-flannel.conflist
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 26 Nov 2021 15:59:13 +0800
      Finished:     Fri, 26 Nov 2021 15:59:13 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /etc/cni/net.d from cni (rw)
      /etc/kube-flannel/ from flannel-cfg (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9kzvz (ro)
Containers:
  kube-flannel:
    Container ID:  containerd://3564ff7b81ba7ed8be23c25852c61f750dab4d49a8324d201c11ad331d980a89
    Image:         quay.io/coreos/flannel:v0.15.1
    Image ID:      quay.io/coreos/flannel@sha256:9a296fbb67790659adc3701e287adde3c59803b7fcefe354f1fc482840cdb3d9
    Port:          <none>
    Host Port:     <none>
    Command:
      /opt/bin/flanneld
    Args:
      --ip-masq
      --kube-subnet-mgr
    State:          Running
      Started:      Fri, 26 Nov 2021 15:59:14 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     100m
      memory:  50Mi
    Requests:
      cpu:     100m
      memory:  50Mi
    Environment:
      POD_NAME:       kube-flannel-ds-l4bdz (v1:metadata.name)
      POD_NAMESPACE:  kube-system (v1:metadata.namespace)
    Mounts:
      /etc/kube-flannel/ from flannel-cfg (rw)
      /run/flannel from run (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9kzvz (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  run:
    Type:          HostPath (bare host directory volume)
    Path:          /run/flannel
    HostPathType:  
  cni-plugin:
    Type:          HostPath (bare host directory volume)
    Path:          /opt/cni/bin
    HostPathType:  
  cni:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/cni/net.d
    HostPathType:  
  flannel-cfg:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      kube-flannel-cfg
    Optional:  false
  kube-api-access-9kzvz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 :NoSchedule op=Exists
                             node.kubernetes.io/disk-pressure:NoSchedule op=Exists
                             node.kubernetes.io/memory-pressure:NoSchedule op=Exists
                             node.kubernetes.io/network-unavailable:NoSchedule op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists
                             node.kubernetes.io/pid-pressure:NoSchedule op=Exists
                             node.kubernetes.io/unreachable:NoExecute op=Exists
                             node.kubernetes.io/unschedulable:NoSchedule op=Exists
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  34s   default-scheduler  Successfully assigned kube-system/kube-flannel-ds-l4bdz to kevin
  Normal  Pulled     33s   kubelet            Container image "rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.0" already present on machine
  Normal  Created    33s   kubelet            Created container install-cni-plugin
  Normal  Started    33s   kubelet            Started container install-cni-plugin
  Normal  Pulled     33s   kubelet            Container image "quay.io/coreos/flannel:v0.15.1" already present on machine
  Normal  Created    33s   kubelet            Created container install-cni
  Normal  Started    33s   kubelet            Started container install-cni
  Normal  Pulled     32s   kubelet            Container image "quay.io/coreos/flannel:v0.15.1" already present on machine
  Normal  Created    32s   kubelet            Created container kube-flannel
  Normal  Started    32s   kubelet            Started container kube-flannel
```

可以看到在最后出现了 `Started container kube-flannel` ，则说明 kube-flannel 网络插件正常启动。

##### 查看 coredns 状态

安装网络插件后，检查 coredns 的状态

```text
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-9zj7b        0/1     Pending   0          7m44s
kube-system   coredns-78fcd69978-wn4l7        0/1     Pending   0          7m44s
kube-system   etcd-kevin                      1/1     Running   4          7m53s
kube-system   kube-apiserver-kevin            1/1     Running   4          7m53s
kube-system   kube-controller-manager-kevin   1/1     Running   4          7m53s
kube-system   kube-flannel-ds-l4bdz           1/1     Running   0          2m39s
kube-system   kube-proxy-zz7ch                1/1     Running   0          7m44s
kube-system   kube-scheduler-kevin            1/1     Running   4          7m59s
```

可用发现 coredns 仍然没有被动调度，下面将当前节点的污点信息删除。

#### 去除污点

[污点和容忍度](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)是用来设置集群调度策略的一种方式。设置了污点信息则节点不会在节点上运行，容忍度则
允许将 pod 调度到污点节点上。

污点和容忍度一起工作，以确保 pod 不会被调度到不适当的节点上。一个或多个污点被应用到一个节点；这标志着节点不应接受任何不容忍污点的 pod。

```text
root@kevin:~/k8s# kubectl describe nodes kevin
Name:               kevin
Roles:              control-plane,master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=kevin
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/control-plane=
                    node-role.kubernetes.io/master=
                    node.kubernetes.io/exclude-from-external-load-balancers=
Annotations:        flannel.alpha.coreos.com/backend-data: {"VNI":1,"VtepMAC":"ce:73:f3:5f:28:f3"}
                    flannel.alpha.coreos.com/backend-type: vxlan
                    flannel.alpha.coreos.com/kube-subnet-manager: true
                    flannel.alpha.coreos.com/public-ip: 192.168.2.178
                    kubeadm.alpha.kubernetes.io/cri-socket: /run/containerd/containerd.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Fri, 26 Nov 2021 15:53:50 +0800
Taints:             kubeadmNode=master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  kevin
  AcquireTime:     <unset>
  RenewTime:       Fri, 26 Nov 2021 16:02:38 +0800
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Fri, 26 Nov 2021 15:59:16 +0800   Fri, 26 Nov 2021 15:59:16 +0800   FlannelIsUp                  Flannel is running on this node
  MemoryPressure       False   Fri, 26 Nov 2021 15:59:08 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Fri, 26 Nov 2021 15:59:08 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Fri, 26 Nov 2021 15:59:08 +0800   Fri, 26 Nov 2021 15:53:48 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Fri, 26 Nov 2021 15:59:08 +0800   Fri, 26 Nov 2021 15:54:06 +0800   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  192.168.2.178
  Hostname:    kevin
Capacity:
  cpu:                4
  ephemeral-storage:  102709252Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             32768344Ki
  pods:               110
Allocatable:
  cpu:                4
  ephemeral-storage:  94656846487
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             32665944Ki
  pods:               110
System Info:
  Machine ID:                 61a427a0a4c64788be4749626b521f2d
  System UUID:                031b021c-040d-05cc-5906-500700080009
  Boot ID:                    78a72cbd-606e-4d2e-8341-761c3595c959
  Kernel Version:             5.15.1-amd64-desktop
  OS Image:                   Deepin 20.3
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  containerd://1.5.8
  Kubelet Version:            v1.22.4
  Kube-Proxy Version:         v1.22.4
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (6 in total)
  Namespace                   Name                             CPU Requests  CPU Limits  Memory Requests  Memory Limits  Age
  ---------                   ----                             ------------  ----------  ---------------  -------------  ---
  kube-system                 etcd-kevin                       100m (2%)     0 (0%)      100Mi (0%)       0 (0%)         8m42s
  kube-system                 kube-apiserver-kevin             250m (6%)     0 (0%)      0 (0%)           0 (0%)         8m42s
  kube-system                 kube-controller-manager-kevin    200m (5%)     0 (0%)      0 (0%)           0 (0%)         8m42s
  kube-system                 kube-flannel-ds-l4bdz            100m (2%)     100m (2%)   50Mi (0%)        50Mi (0%)      3m28s
  kube-system                 kube-proxy-zz7ch                 0 (0%)        0 (0%)      0 (0%)           0 (0%)         8m33s
  kube-system                 kube-scheduler-kevin             100m (2%)     0 (0%)      0 (0%)           0 (0%)         8m48s
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                750m (18%)  100m (2%)
  memory             150Mi (0%)  50Mi (0%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
Events:
  Type     Reason                   Age                    From        Message
  ----     ------                   ----                   ----        -------
  Normal   Starting                 8m32s                  kube-proxy  
  Normal   NodeHasSufficientMemory  8m54s (x5 over 8m54s)  kubelet     Node kevin status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    8m54s (x5 over 8m54s)  kubelet     Node kevin status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     8m54s (x4 over 8m54s)  kubelet     Node kevin status is now: NodeHasSufficientPID
  Normal   Starting                 8m42s                  kubelet     Starting kubelet.
  Normal   NodeHasSufficientMemory  8m42s                  kubelet     Node kevin status is now: NodeHasSufficientMemory
  Normal   NodeHasNoDiskPressure    8m42s                  kubelet     Node kevin status is now: NodeHasNoDiskPressure
  Normal   NodeHasSufficientPID     8m42s                  kubelet     Node kevin status is now: NodeHasSufficientPID
  Normal   NodeAllocatableEnforced  8m42s                  kubelet     Updated Node Allocatable limit across pods
  Warning  InvalidDiskCapacity      8m42s                  kubelet     invalid capacity 0 on image filesystem
  Normal   NodeReady                8m34s                  kubelet     Node kevin status is now: NodeReady
```

正如前面在初始化集群的时候提到的，当前被标记为污点：

```text
Taints:             kubeadmNode=master:NoSchedule
```

所以现在需要将污点标记删除：

```bash
kubectl taint nodes --all kubeadmNode=master:NoSchedule-
```

输入如下：

```text
root@kevin:~/k8s# kubectl taint nodes --all node-role.kubernetes.io/master-
node/kevin untainted
```

再次查看 node 的描述，可以发现污点信息为 `Taints:             <none>` 。

再次查看 pod ：

```text
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-9zj7b        1/1     Running   0          9m56s
kube-system   coredns-78fcd69978-wn4l7        1/1     Running   0          9m56s
kube-system   etcd-kevin                      1/1     Running   4          10m
kube-system   kube-apiserver-kevin            1/1     Running   4          10m
kube-system   kube-controller-manager-kevin   1/1     Running   4          10m
kube-system   kube-flannel-ds-l4bdz           1/1     Running   0          4m51s
kube-system   kube-proxy-zz7ch                1/1     Running   0          9m56s
kube-system   kube-scheduler-kevin            1/1     Running   4          10m
```

继续查看 coredns pod 的描述：

```text
root@kevin:~/k8s# kubectl -n kube-system describe pod coredns
Name:                 coredns-78fcd69978-9zj7b
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 kevin/192.168.2.178
Start Time:           Fri, 26 Nov 2021 16:03:53 +0800
Labels:               k8s-app=kube-dns
                      pod-template-hash=78fcd69978
Annotations:          <none>
Status:               Running
IP:                   10.88.0.2
IPs:
  IP:           10.88.0.2
  IP:           2001:4860:4860::2
Controlled By:  ReplicaSet/coredns-78fcd69978
Containers:
  coredns:
    Container ID:  containerd://2f6b47e60e24b6a468fa70c7dfda4bd55cf68f86702474ccb62d3256becab2c1
    Image:         k8s.gcr.io/coredns/coredns:v1.8.4
    Image ID:      k8s.gcr.io/coredns/coredns@sha256:6e5a02c21641597998b4be7cb5eb1e7b02c0d8d23cce4dd09f4682d463798890
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Running
      Started:      Fri, 26 Nov 2021 16:03:55 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-m6bvz (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-m6bvz:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  81s (x10 over 10m)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {kubeadmNode: master}, that the pod didn't tolerate.
  Normal   Scheduled         52s                 default-scheduler  Successfully assigned kube-system/coredns-78fcd69978-9zj7b to kevin
  Normal   Pulled            50s                 kubelet            Container image "k8s.gcr.io/coredns/coredns:v1.8.4" already present on machine
  Normal   Created           50s                 kubelet            Created container coredns
  Normal   Started           50s                 kubelet            Started container coredns

Name:                 coredns-78fcd69978-wn4l7
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 kevin/192.168.2.178
Start Time:           Fri, 26 Nov 2021 16:03:53 +0800
Labels:               k8s-app=kube-dns
                      pod-template-hash=78fcd69978
Annotations:          <none>
Status:               Running
IP:                   10.88.0.3
IPs:
  IP:           10.88.0.3
  IP:           2001:4860:4860::3
Controlled By:  ReplicaSet/coredns-78fcd69978
Containers:
  coredns:
    Container ID:  containerd://f3a4901776fbe0fad1aa19f612f58bfcbc44c5523eba5c89415e46a558386bf8
    Image:         k8s.gcr.io/coredns/coredns:v1.8.4
    Image ID:      k8s.gcr.io/coredns/coredns@sha256:6e5a02c21641597998b4be7cb5eb1e7b02c0d8d23cce4dd09f4682d463798890
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
    State:          Running
      Started:      Fri, 26 Nov 2021 16:03:55 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      memory:  170Mi
    Requests:
      cpu:        100m
      memory:     70Mi
    Liveness:     http-get http://:8080/health delay=60s timeout=5s period=10s #success=1 #failure=5
    Readiness:    http-get http://:8181/ready delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-7cf5h (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
  kube-api-access-7cf5h:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              kubernetes.io/os=linux
Tolerations:                 CriticalAddonsOnly op=Exists
                             node-role.kubernetes.io/control-plane:NoSchedule
                             node-role.kubernetes.io/master:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  81s (x10 over 10m)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {kubeadmNode: master}, that the pod didn't tolerate.
  Normal   Scheduled         52s                 default-scheduler  Successfully assigned kube-system/coredns-78fcd69978-wn4l7 to kevin
  Normal   Pulled            50s                 kubelet            Container image "k8s.gcr.io/coredns/coredns:v1.8.4" already present on machine
  Normal   Created           50s                 kubelet            Created container coredns
  Normal   Started           50s                 kubelet            Started container coredns
```

可用看到 coredns 的调度从 `FailedScheduling` 转换到了 `Started` 。

至此 kubernetes 集群单节点创建完成。

## 检查集群

### 手动检测

请参考 [公开外部 IP 地址以访问集群中应用程序](https://kubernetes.io/zh/docs/tutorials/stateless-application/expose-external-ip-address/) 内容一个可以使用外部 IP 访问集群内部服务的示例。
如果一切正常，则说明集群环境没问题。

### 使用 sonobuoy 检查

使用官方文档中提到的工具 [sonobuoy](https://github.com/vmware-tanzu/sonobuoy) 检测集群。

Sonobuoy 是一种诊断工具，通过以可访问且非破坏性的方式运行一组插件（包括Kubernetes一致性测试），可以更轻松地了解 Kubernetes 集群的状态。它是一种可定制、可扩展且与集群无关的方式，用于生成有关集群的清晰、信息丰富的报告。

它对 Kubernetes 资源对象和集群节点的选择性数据转储允许以下用例：

- 集成的端到端 (e2e)一致性测试
- 工作负载调试
- 通过可扩展插件自定义数据收集

下载最新的 [sonobuoy](https://github.com/vmware-tanzu/sonobuoy/releases/tag/v0.55.1) 并将可执行文件放到 `/usr/local/bin` 中

```bash
export https_proxy=http://127.0.0.1:1081
wget https://github.com/vmware-tanzu/sonobuoy/releases/download/v0.55.1/sonobuoy_0.55.1_linux_amd64.tar.gz /tmp/
tar -zxf /tmp/sonobuoy_0.55.1_linux_amd64.tar.gz -C /tmp
mv /tmp/sonobuoy /usr/local/bin
```

在运行之前，记得设置 kubernetes 的访问配置。 root 用户可以使用 `export KUBECONFIG=/etc/kubernetes/admin.conf` 设置。普通用户，请参考安装文档中的步骤，在用户家目录增加配置文件。

然后执行：

```bash
sonobuoy run --wait
```

**注意：**此过程耗时非常的长。暂时不知道强制结束会造成什么负面影响。检查是会运行 `344` 个任务。

## 使用集群

集群使用技巧可以参考 [Kubernetes 教程](https://kubernetes.io/zh/docs/tutorials/)中的内容。教程中包含：

### 配置

- [使用一个 ConfigMap 配置 Redis](https://kubernetes.io/zh/docs/tutorials/configuration/configure-redis-using-configmap/)

### 无状态应用程序

- [公开外部 IP 地址以访问集群中应用程序](https://kubernetes.io/zh/docs/tutorials/stateless-application/expose-external-ip-address/)
- [示例：使用 Redis 部署 PHP 留言板应用程序](https://kubernetes.io/zh/docs/tutorials/stateless-application/guestbook/)

### 有状态应用程序

- [StatefulSet 基础](https://kubernetes.io/zh/docs/tutorials/stateful-application/basic-stateful-set/)
- [示例：使用 Persistent Volumes 部署 WordPress 和 MySQL](https://kubernetes.io/zh/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)

其他内容请阅读文档 [Kubernetes 教程](https://kubernetes.io/zh/docs/tutorials/) 。

## 重置集群

### 清理集群

```bash
# 重置 kubeadm
kubeadm reset -f
# 删除 flannel 的 cni 配置。因为重置的过程中不会自动删除
rm -rf /etc/cni/net.d/flannel*

rm -rf ~/.kube

# 如果使用的是 iptables ，清理 iptables
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X

# 清理 ipvs
ipvsadm -C
```

输出如下：
<!-- markdownlint-disable MD013 MD033-->
```text
root@kevin:~/k8s# kubeadm reset
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
The 'update-cluster-status' phase is deprecated and will be removed in a future release. Currently it performs no operation
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
W1124 15:05:02.062770    3957 cleanupnode.go:84] [reset] Failed to remove containers: failed to stop running pod f37ef9145140dc372055e7b5ecc3969e1bd2168b983e267c58e84ae05bb4c4df: output: time="2021-11-24T15:05:00+08:00" level=fatal msg="stopping the pod sandbox \"f37ef9145140dc372055e7b5ecc3969e1bd2168b983e267c58e84ae05bb4c4df\": rpc error: code = Unknown desc = failed to destroy network for sandbox \"f37ef9145140dc372055e7b5ecc3969e1bd2168b983e267c58e84ae05bb4c4df\": running [/usr/sbin/ip6tables -t nat -D POSTROUTING -s 2001:4860:4860::62/64 -j CNI-538cb4c66bc34505d6b0701e -m comment --comment name: \"containerd-net\" id: \"f37ef9145140dc372055e7b5ecc3969e1bd2168b983e267c58e84ae05bb4c4df\" --wait]: exit status 1: iptables: Bad rule (does a matching rule exist in that chain?).\n"
, error: exit status 1
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/etcd /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.
```
<!-- markdownlint-restore -->
以上输出信息为之前搭建过程出错后重置时，做的记录。每个人的输出结果会不一样。

### 检查是否清理完成

检查 kubelet 服务状态是否已经停止：

```bash
systemctl status kubelet
```

输出如下：
<!-- markdownlint-disable MD013 MD033-->
```text
root@kevin:~/k8s# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: inactive (dead) since Fri 2021-11-26 16:06:54 CST; 1min 53s ago
     Docs: https://kubernetes.io/docs/home/
  Process: 19673 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=0/SUCCESS)
 Main PID: 19673 (code=exited, status=0/SUCCESS)

11月 26 15:59:16 kevin kubelet[19673]: W1126 15:59:16.345211   19673 manager.go:1176] Failed to process watch event {EventType:0 Name:/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod01d4063e_7b71_47ff_bb20_3efb3b897939.slice/cri-containerd-9b28d86fe319ad34eaee3e935883c36fc05794c5e06d8bc9d9e13fbfdfda6477.sc
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.356878   19673 topology_manager.go:200] "Topology Admit Handler"
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.362612   19673 topology_manager.go:200] "Topology Admit Handler"
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.412714   19673 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"config-volume\" (UniqueName: \"kubernetes.io/configmap/d9e6a190-eda0-4f60-8ac3-4d2e783a8681-config-volume\") pod \"coredns-78fcd69978-wn4l7\" (UID: \"d9e6a190-eda0
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.412757   19673 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"config-volume\" (UniqueName: \"kubernetes.io/configmap/d8df7209-6998-48d7-8b9f-f0c9eb355752-config-volume\") pod \"coredns-78fcd69978-9zj7b\" (UID: \"d8df7209-6998
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.412787   19673 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-7cf5h\" (UniqueName: \"kubernetes.io/projected/d9e6a190-eda0-4f60-8ac3-4d2e783a8681-kube-api-access-7cf5h\") pod \"coredns-78fcd69978-wn4l7\" (UID:
11月 26 16:03:53 kevin kubelet[19673]: I1126 16:03:53.412824   19673 reconciler.go:224] "operationExecutor.VerifyControllerAttachedVolume started for volume \"kube-api-access-m6bvz\" (UniqueName: \"kubernetes.io/projected/d8df7209-6998-48d7-8b9f-f0c9eb355752-kube-api-access-m6bvz\") pod \"coredns-78fcd69978-9zj7b\" (UID:
11月 26 16:06:54 kevin systemd[1]: Stopping kubelet: The Kubernetes Node Agent...
11月 26 16:06:54 kevin systemd[1]: kubelet.service: Succeeded.
11月 26 16:06:54 kevin systemd[1]: Stopped kubelet: The Kubernetes Node Agent.
```
<!-- markdownlint-disable MD013 MD033-->
由于 kubernetes 使用了 containerd 作为底层容器运行时，所以在正常清理完成后的 containerd 中是不会在有 kubernetes 的容器运行的。

使用 containerd 内置的工具 `ctr` 检查

```bash
# 查看帮助命令
ctr --help

# 列出所有名称空间，会看到有个 `k8s.io` 的名称空间。
ctr ns ls

# 查看 `k8s.io` 名称空间下的容器。
ctr -n k8s.io c ls
# 注意： containers 可以缩写为 c 。
# 如果没有容器则说明正常操作。
# 如果有容器，请将重启逐一删除。
```

## 集群管理

## 问题记录

以下列出的问题，由于在本次安装时，没有再出现，所以无法具体排错过程。列出标题，带以后遇到了再补充。

### 常见问题

#### 加速镜像

### 初始化集群-

#### 无法拉去 kubernetes 镜像

这是由于 kubernetes 的镜像在 `k8s.io` 名称空间下，而不是 docker.io 的镜像，在访问是需要访问 google ，考虑
到国内环境，是无法访问的，所以要正确使用镜像，要么为 containerd 的服务配置代理，要么通过其他方式下载镜像到
本地后导入到 containerd 的镜像库中。

#### 初始化时卡住

在初始化时，会等待 kubelet 启动，等待默认时间为 4 分钟，所以当等待时间较长，则说明启动时有错误。

```text
root@kevin:~/k8s# kubeadm init --config=kubeadm-config.yaml  --upload-certs
[init] Using Kubernetes version: v1.22.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kevin kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.2.178]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kevin localhost] and IPs [192.168.2.178 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kevin localhost] and IPs [192.168.2.178 127.0.0.1 ::1]
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
[kubelet-check] Initial timeout of 40s passed.
^C
```

使用 `journalctl -f -u kubelet` 查看 `kubelet` 服务的日志，通过日志排查。

将 containerd ，和 runc 升级到最新版本，可以避免大部分安装过程中的问题。

#### cni plugin not initialized"

出现该问题时，请确保 containerd 是最新版本， kubernetes 是最新版本。

确保 containerd 服务正常，确保 `ctl plugin ls` 输出结果中没有错误， 确保 `crictl info` 输出正常。这三个是保证 kubernetes 能正确启动的前提。

在安装网络插件后去除污点，集群状态仍为 NotReady

此时查看 kubelet 服务，日志显示为 `cni plugin not initialized`

```text
root@kevin:~/k8s# systemctl status kubelet
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Thu 2021-11-25 14:21:26 CST; 28min ago
     Docs: https://kubernetes.io/docs/home/
 Main PID: 3798 (kubelet)
    Tasks: 15 (limit: 4915)
   Memory: 36.9M
   CGroup: /system.slice/kubelet.service
           └─3798 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=/run/containerd/containerd.sock --pod-infr

11月 25 14:49:31 kevin kubelet[3798]: E1125 14:49:31.158114    3798 kuberuntime_gc.go:176] "Failed to stop sandbox before removing" err="rpc error: code = Unknown desc = failed to destroy network for sandbox \"f37ef9145140dc372055e7b5ecc3969e1bd2168b983e267c58e84ae05bb4c4df\
11月 25 14:49:31 kevin kubelet[3798]: E1125 14:49:31.532011    3798 kubelet.go:2337] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
11月 25 14:49:36 kevin kubelet[3798]: E1125 14:49:36.532663    3798 kubelet.go:2337] "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:Network plugin returns error: cni plugin not initialized"
```

检查 kube-flannel 的日志为：

```text
root@kevin:~/k8s# kubectl get pods --all-namespaces
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-78fcd69978-8cm6s        0/1     Pending   0          24s
kube-system   coredns-78fcd69978-blpwv        0/1     Pending   0          24s
kube-system   etcd-kevin                      1/1     Running   26         32s
kube-system   kube-apiserver-kevin            1/1     Running   7          32s
kube-system   kube-controller-manager-kevin   1/1     Running   5          32s
kube-system   kube-flannel-ds-c7d9k           1/1     Running   0          9s
kube-system   kube-proxy-s2pm6                1/1     Running   0          25s
kube-system   kube-scheduler-kevin            1/1     Running   7          32s
root@kevin:~/k8s# kubectl -n kube-system logs kube-flannel-ds-c7d9k
I1125 10:00:30.174435       1 main.go:217] CLI flags config: {etcdEndpoints:http://127.0.0.1:4001,http://127.0.0.1:2379 etcdPrefix:/coreos.com/network etcdKeyfile: etcdCertfile: etcdCAFile: etcdUsername: etcdPassword: help:false version:false autoDetectIPv4:false autoDetectIPv6:false kubeSubnetMgr:true kubeApiUrl: kubeAnnotationPrefix:flannel.alpha.coreos.com kubeConfigFile: iface:[] ifaceRegex:[] ipMasq:true subnetFile:/run/flannel/subnet.env subnetDir: publicIP: publicIPv6: subnetLeaseRenewMargin:60 healthzIP:0.0.0.0 healthzPort:0 charonExecutablePath: charonViciUri: iptablesResyncSeconds:5 iptablesForwardRules:true netConfPath:/etc/kube-flannel/net-conf.json setNodeNetworkUnavailable:true}
W1125 10:00:30.174487       1 client_config.go:608] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I1125 10:00:30.276530       1 kube.go:378] Starting kube subnet manager
I1125 10:00:30.276826       1 kube.go:120] Waiting 10m0s for node controller to sync
I1125 10:00:31.276958       1 kube.go:127] Node controller sync successful
I1125 10:00:31.276975       1 main.go:237] Created subnet manager: Kubernetes Subnet Manager - kevin
I1125 10:00:31.276978       1 main.go:240] Installing signal handlers
I1125 10:00:31.277024       1 main.go:459] Found network config - Backend type: vxlan
I1125 10:00:31.277034       1 main.go:651] Determining IP address of default interface
I1125 10:00:31.277393       1 main.go:698] Using interface with name enp0s31f6 and address 192.168.2.178
I1125 10:00:31.277407       1 main.go:720] Defaulting external address to interface address (192.168.2.178)
I1125 10:00:31.277410       1 main.go:733] Defaulting external v6 address to interface address (<nil>)
I1125 10:00:31.277444       1 vxlan.go:137] VXLAN config: VNI=1 Port=0 GBP=false Learning=false DirectRouting=false
I1125 10:00:31.283411       1 kube.go:339] Setting NodeNetworkUnavailable
I1125 10:00:31.289379       1 main.go:340] Setting up masking rules
I1125 10:00:31.373916       1 main.go:361] Changing default FORWARD chain policy to ACCEPT
I1125 10:00:31.374009       1 main.go:374] Wrote subnet file to /run/flannel/subnet.env
I1125 10:00:31.374065       1 main.go:378] Running backend.
I1125 10:00:31.374074       1 main.go:396] Waiting for all goroutines to exit
I1125 10:00:31.374088       1 vxlan_network.go:60] watching for new subnet leases
I1125 10:00:31.469702       1 iptables.go:216] Some iptables rules are missing; deleting and recreating rules
I1125 10:00:31.469720       1 iptables.go:240] Deleting iptables rule: -s 10.244.0.0/16 -d 10.244.0.0/16 -j RETURN
I1125 10:00:31.569022       1 iptables.go:240] Deleting iptables rule: -s 10.244.0.0/16 ! -d 224.0.0.0/4 -j MASQUERADE --random-fully
I1125 10:00:31.569705       1 iptables.go:216] Some iptables rules are missing; deleting and recreating rules
I1125 10:00:31.569722       1 iptables.go:240] Deleting iptables rule: -s 10.244.0.0/16 -j ACCEPT
I1125 10:00:31.570486       1 iptables.go:240] Deleting iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/24 -j RETURN
I1125 10:00:31.570486       1 iptables.go:240] Deleting iptables rule: -d 10.244.0.0/16 -j ACCEPT
I1125 10:00:31.571425       1 iptables.go:228] Adding iptables rule: -s 10.244.0.0/16 -j ACCEPT
I1125 10:00:31.571525       1 iptables.go:240] Deleting iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/16 -j MASQUERADE --random-fully
I1125 10:00:31.665275       1 iptables.go:228] Adding iptables rule: -d 10.244.0.0/16 -j ACCEPT
I1125 10:00:31.665328       1 iptables.go:228] Adding iptables rule: -s 10.244.0.0/16 -d 10.244.0.0/16 -j RETURN
I1125 10:00:31.667802       1 iptables.go:228] Adding iptables rule: -s 10.244.0.0/16 ! -d 224.0.0.0/4 -j MASQUERADE --random-fully
I1125 10:00:31.669964       1 iptables.go:228] Adding iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/24 -j RETURN
I1125 10:00:31.672175       1 iptables.go:228] Adding iptables rule: ! -s 10.244.0.0/16 -d 10.244.0.0/16 -j MASQUERADE --random-fully
```

##### 排查

###### 排查 containerd

使用 `ctr plugin ls` 查看 containerd 的插件是否都正常

```text
root@kevin:~# ctr plugin ls
TYPE                            ID                       PLATFORMS      STATUS    
io.containerd.content.v1        content                  -              ok        
io.containerd.snapshotter.v1    aufs                     linux/amd64    skip      
io.containerd.snapshotter.v1    btrfs                    linux/amd64    skip      
io.containerd.snapshotter.v1    devmapper                linux/amd64    error     
io.containerd.snapshotter.v1    native                   linux/amd64    ok        
io.containerd.snapshotter.v1    overlayfs                linux/amd64    ok        
io.containerd.snapshotter.v1    zfs                      linux/amd64    skip      
io.containerd.metadata.v1       bolt                     -              ok        
io.containerd.differ.v1         walking                  linux/amd64    ok        
io.containerd.gc.v1             scheduler                -              ok        
io.containerd.service.v1        introspection-service    -              ok        
io.containerd.service.v1        containers-service       -              ok        
io.containerd.service.v1        content-service          -              ok        
io.containerd.service.v1        diff-service             -              ok        
io.containerd.service.v1        images-service           -              ok        
io.containerd.service.v1        leases-service           -              ok        
io.containerd.service.v1        namespaces-service       -              ok        
io.containerd.service.v1        snapshots-service        -              ok        
io.containerd.runtime.v1        linux                    linux/amd64    ok        
io.containerd.runtime.v2        task                     linux/amd64    ok        
io.containerd.monitor.v1        cgroups                  linux/amd64    ok        
io.containerd.service.v1        tasks-service            -              ok        
io.containerd.internal.v1       restart                  -              ok        
io.containerd.grpc.v1           containers               -              ok        
io.containerd.grpc.v1           content                  -              ok        
io.containerd.grpc.v1           diff                     -              ok        
io.containerd.grpc.v1           events                   -              ok        
io.containerd.grpc.v1           healthcheck              -              ok        
io.containerd.grpc.v1           images                   -              ok        
io.containerd.grpc.v1           leases                   -              ok        
io.containerd.grpc.v1           namespaces               -              ok        
io.containerd.internal.v1       opt                      -              ok        
io.containerd.grpc.v1           snapshots                -              ok        
io.containerd.grpc.v1           tasks                    -              ok        
io.containerd.grpc.v1           version                  -              ok        
io.containerd.grpc.v1           cri                      linux/amd64    error
```

可以看到 containerd 的 cri 插件状态为 `errror` 。

如果 cri 插件出现问题，那么 crictl 也会出现问题。为了更好的拍错，现在修改 crictl 的配置文件，将日志调整到 debug。这个配置为文件是在
前面安装 containerd 时，解压后自动生成的。

修改 `/etc/crictl.yaml` 文件

```yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: true
pull-image-on-create: false
```

然后运行 crictl info :

```text
root@kevin:~/k8s# crictl info
DEBU[0000] get runtime connection                       
DEBU[0000] connect using endpoint 'unix:///run/containerd/containerd.sock' with '2s' timeout 
DEBU[0000] connected successfully using endpoint: unix:///run/containerd/containerd.sock 
DEBU[0000] StatusRequest: &StatusRequest{Verbose:true,} 
DEBU[0000] StatusResponse: nil                          
FATA[0000] getting status of runtime: rpc error: code = Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService
```

可以看到 crictl 也有问题。

因为我是升级新的 containerd ，所以猜测可能是老的 containerd 配置文件袋导致了服务问题。

```text
containerd config default > /etc/containerd/config.toml
```

配置 runc 使用 cgroup

修改 `/etc/containerd/config.toml` ，将 `SystemdCgroup` 调整为 `true`

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

然后重启 containerd

```bash
systemctl restart containerd
systemctl status containerd
```

最后再次检查 cri

```bash
ctr pulgin ls
crictl info
```

如果看到 `crictl info` 的最后输出有 `"lastCNILoadStatus": "cni config load failed: no network config found in /etc/cni/net.d: cni plugin not initialized: failed to load cni config"` 的字样。则需要在 `/etc/cni/net.d` 中增加 containerd 的默认配置，因为在重置 kubernetes 集群的时候已经将 `/etc/cni/net.d` 完全删除了。

增加如下内容到 `/etc/cni/net.d/10-containerd-net.conflist`

```json
{
  "cniVersion": "0.4.0",
  "name": "containerd-net",
  "plugins": [
    {
      "type": "bridge",
      "bridge": "cni0",
      "isGateway": true,
      "ipMasq": true,
      "promiscMode": true,
      "ipam": {
        "type": "host-local",
        "ranges": [
          [{
            "subnet": "10.88.0.0/16"
          }],
          [{
            "subnet": "2001:4860:4860::/64"
          }]
        ],
        "routes": [
          { "dst": "0.0.0.0/0" },
          { "dst": "::/0" }
        ]
      }
    },
    {
      "type": "portmap",
      "capabilities": {"portMappings": true}
    }
  ]
}
```

再次重启 containerd 后检查：

```bash
ctr pulgin ls
crictl info
```

此时 cri 正常 。同时可以看到输出结果的最后显示 cni 状态 `"lastCNILoadStatus": "OK"` 。

### 使用集群-

#### 创建工作负载时拉取镜像出错

在创建工作负载时，由于某些负载使用的镜像并不在 docker.io 中，在拉取时可能因为网络访问缓慢或者无法访问而拉取失败，
此时最佳的方法是配置 containerd 的服务使用代理访问。

#### 无法完全删除工作负载

具体请参考 [Using Finalizers to Control Deletion](https://kubernetes.io/blog/2021/05/14/using-finalizers-to-control-deletion/)

### 集群配置

#### 动态存储分配

#### 存储类配置
