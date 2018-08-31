## 创建CNI网络

在部署Kubernetes时，网络部分是一个独立的模块。在Kubernetes社区里，在pod间创建网络的常用方式是CNI插件。目前Kubernetes社区里比较稳定的CNI插件有：

- [flannel](https://github.com/coreos/flannel)
- [calico](https://github.com/projectcalico/calico)

在创建CNI网络前，我们建议先阅读[Install a pod network add-on](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)。

### Flannel

通过运行`sysctl net.bridge.bridge-nf-call-iptables=1`将`/proc/sys/net/bridge/bridge-nf-call-iptables`的值置为1，从而将桥接模式的IPv4流量传输给iptable链。对于用户来说，用flannel来创建CNI网络非常简单。只需要在主节点上执行一条简单的命令：

``` shell
kubectl create -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml
```

如果你无法连接到网络，请确保在本地的文件夹下存储了以上文件。然后执行`kubectl create -f kube-flannel.yml`命令，就能创建flannel网络。

### Calico

目前，Calico只支持`amd64`架构的系统。对于用户来说，用Calico来创建CNI网络非常简单。只需要在主节点上执行一条简单的命令：

``` shell
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### 验证网络的正确性

``` shell
$ cat pouch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pouch
spec:
  replicas: 1
  selector:
    matchLabels:
      pouch: nginx
  template:
    metadata:
      labels:
        pouch: nginx
    spec:
      containers:
      - name: pouch
        image: docker.io/library/nginx:latest
        ports:
        - containerPort: 80

$ kubectl create -f pouch.yaml
deployment "pouch" created
$
$ kubectl get pods -o wide
NAME                     READY     STATUS    RESTARTS   AGE       IP           NODE
pouch-7dcd875d69-gq5r9   1/1       Running   0          44m       10.244.1.4   master
$
$ curl 10.244.1.4
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

当看到nginx的输出时，祝贺你成功运行了开启CNI网络的Kubernetes集群。
