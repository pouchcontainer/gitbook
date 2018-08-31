# 关于 Kubernetes

该文档简单介绍 Kubernetes。

想要安装Kubernetes和PouchContainer来向用户提供编排服务，首先要满足一些环境要求。

- 软件版本要求
- 节点资源要求

### 软件版本要求

运行Kubernetes + PouchContainer会对软件版本有一些限制。为了避免集群环境不稳定，安装软件时请严格按照以下软件版本来。

|软件名称|版本|
|:----:|:---:|
|Kubernetes|1.10.0+|
|PouchContainer|1.0.0-rc1+|
|CNI|0.6.0+|
|操作系统发行版|CentOS 7.0+ 或 Ubuntu 16.04 +|
|Linux内核|3.10+(CentOS 7.0+) 或 4.10+(Ubuntu 16.04+)|

### 节点资源要求

To make Kubernetes run stably, we should provide sufficient resource for node(s) in Kubernetes. Currently we list two kinds of deploying mode:
为了确保Kubernetes正常运行，我们应向Kubernetes集群中的节点提供充足的资源。目前，我们列出了两种部署模式：

- 单节点模式（k8s master和kublet运行在相同节点上）
- 集群模式（k8s master和kublet分布在不同节点上）

节点资源要求如下所述：

|部署模式| CPU内核要求| 内核要求|
|:-:|:-:|:-:|
|单节点模式| 1+ | 2GB +|
|集群模式|1+ (任一节点)| 2GB + (任一节点)|

## 安装和配置PouchContainer

PouchContainer作为容器引擎，在集群中的任何节点上都必须安装。只有执行了以下步骤后，PouchContainer才能就绪：

- 安装PouchContainer
- 部署PouchContainer
- 重启PouchContainer

> 注意：文档中的所有命令在普通用户模式下就可完成。但在超级用户root模式下，安装工作也能顺利执行。

### 安装PouchContainer

PouchContainer的安装步骤可参考 [INSTALLATION.md](../../INSTALLATION.md).

### 配置PouchContainer

在安装好后，PouchContainer会自动开始运行。但为了让PouchContainer支持Kubernetes，我们需要更新PouchContainer的配置。详细来说，只需要修改`--enbale-cri`参数，PouchContainer就可以为Kubernetes中的组件kubelet提供CRI服务了。因此，我们可以用sed命令来更新PouchContainer的配置。

对于Ubuntu 16.04+, 相关命令如下:

``` shell
sudo sed -i 's/ExecStart=\/usr\/local\/bin\/pouchd/ExecStart=\/usr\/local\/bin\/pouchd --enable-cri=true --cri-version=v1alpha2/g' /lib/systemd/system/pouch.service
```

对于CentOS 7.0+, 相关命令如下：

``` shell
sudo sed -i 's/ExecStart=\/usr\/local\/bin\/pouchd/ExecStart=\/usr\/local\/bin\/pouchd --enable-cri=true --cri-version=v1alpha2/g' /lib/systemd/system/pouch.service
```

### 重启PouchContainer

一旦PouchContainer的配置被更新，需要重启来让更新后的配置生效。不管是在Ubuntu 16.04+或者CentOS 7.0+中，我们都可以用`systemctl`命令来完成PouchContainer的重启，如下所示：

```shell
sudo systemctl daemon-reload
sudo systemctl restart pouch
```

> 注意：在PouchContainer中开启CRI接口时，长进程会默认监听10011端口。请确保10011端口没有被iptables或者其他防火墙工具阻挡。

### 验证PouchContainer的正确性

在安装和配置好PouchContainer后，我们需要进行验证工作以确保pouchd满足要求。

用户可以通过在节点上执行`pouch info`命令，来观察结果中`IsCRIEnabled`字段的值是否为`true`。如果`IsCRIEnabled`字段的值为`true`，这意味着PouchContainer已经就绪了。

## 安装和配置Kubernetes

无论Kubernetes是以单节点模式运行，还是以集群模式运行，PouchContainer作为底层的容器引擎都要先就绪。在PouchContainer就绪后，再在相应的节点上安装和配置Kubernetes组件。

Kubernetes部分同样包括三个步骤:


- 下载Kubernetes安装包
- 配置Kubernetes组件;
- 初始化Kubernetes主节点;

### 下载Kubernetes安装包

在安装Kubernetes时，我们用[kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)来创建Kubernetes集群。首先，从软件仓库里下载关键组件。因此，对于Ubuntu系统，我们需要将kubernetes.io添加到debian安装包源里，CentOS系统也是类似的步骤。在装配好源仓库后，三个Kubernetes组件就能下载了。

- [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/): 运行在Kubernetes集群的所有节点上的关键性节点代理。
- [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/): clusters为方便构建Kubernetes集群提供的构建工具。
- [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/): 为在Kubernetes集群中运行命令提供的命令行接口.

对于Ubuntu 16.04+，执行以下命令：

``` shell
$ sudo apt-get update && sudo apt-get install -y apt-transport-https
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
$ sudo apt-get update
$ export RELEASE="1.10.2-00"
$ apt-get -y install kubelet=${RELEASE} kubeadm=${RELEASE} kubectl=${RELEASE}
```

对于CentOS 7.0+，执行以下命令：

``` shell
$ sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://yum.kubernetes.io/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
$ export RELEASE="1.10.2-0.x86_64"
$ sudo yum install -y kubelet-${RELEASE} kubeadm-${RELEASE} kubectl-${RELEASE}
```