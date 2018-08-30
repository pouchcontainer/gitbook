# 如何定制 CRI

### 配置Kubernetes组件

在下载好所有必须的安装包后，我们需要更新一些配置。对于kubelet，我们要改变它的配置，让它的容器运行时变成PouchContainer。因为PouchContainer是用UNIX socket `unix:///var/run/pouchcri.sock`通信的，这个socket必须被传送给kubelet。使用以下命令更新配置：

``` shell
sudo sed -i '2 i\Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=unix:///var/run/pouchcri.sock --image-service-endpoint=unix:///var/run/pouchcri.sock"' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
sudo systemctl daemon-reload
```