# 借助 PouchContainer 部署强隔离 Pod

### 建立Kubernetes集群

在建立好kubelet的配置后，下一步我们就可以安装Kubernetes主节点了。用kubeadm初始化主节点时，必须提供一个CIDR地址（CIDR地址是为pod的网络分配的一个ip范围）。务必不要让这个ip地址和内部网络环境冲突或者重叠。

> 注意：如果你后续想用flannel或者calico配置CNI网络，必须设置为`--pod-network-cidr 10.244.0.0/16`。

``` shell
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --ignore-preflight-errors=all
```

在执行以上命令后，Kubernetes master和kubelet都会运行在这个节点上。最终，**一个部署在单节点上的Kubernetes集群就搭建好了。**

在进一步体验Kubernetes的服务之前，用户必须**在主节点上**执行三个命令：

``` shell
mkdir -p ~/.kube
sudo cp -i /etc/kubernetes/admin.conf  ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
```

另外，在上一步执行命令的输出结果中，能获取到以下信息（注意：在你的环境中，$token ${master_ip:port} $ca-cert将是详细的内容，只需要复制这个命令就好）：

```
    kubeadm join --token $token ${master_ip:port} --discovery-token-ca-cert-hash $ca-cert
```

以上命令`kubeadm join`供除master节点的任何节点使用，通过这个命令可以加入Kubernete集群成为slave。只需要在集群中的slave节点上执行这个命令就好。

> 注意：Kubernetes安装部分并不包括CNI网络部分。建立CNI网络是一个独立的部分，请参考[Setup CNI network]()。

### 验证Kubernetes正确性

在安装好PouchContainer和Kubernetes后，用户可以创建一个运行的pod来验证集群的正确性。以下的例子创建了一个仅包含一个busybox容器的简单pod。如果我们观察到这个pod是运行状态的，这意味着你已经成功地部署了
Kubernetes+PouchContainer。

``` shell
$ cat busybox.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example
spec:
  replicas: 1
  selector:
    matchLabels:
      pouch: busybox
  template:
    metadata:
      labels:
        pouch: busybox
    spec:
      containers:
      - name: busyboxx
        image: docker.io/library/busybox:latest
        command:
           - top
      hostNetwork: true

$ sudo kubectl apply -f busybox.yaml
deployment.apps "example" created

$ sudo kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP           NODE
example-96b47ff48-45j2f   1/1       Running   0          5m        10.140.0.2   master
```