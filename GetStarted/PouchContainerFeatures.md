# PouchContainer 特点

- **富容器**：除了常用的运行容器的方法，PouchContainer 包含一个 `rich container` 的模式，它集成了更多的服务，hook和其他很多容器内部组件，以保证容器的正常运行。
- **强大的隔离**：PouchContainer 默认设置为安全，它包含非常多安全功能，比如基于管理程序的容器技术，LXCFS，Diskquota，Linux内核补丁等等。
- **P2P分发**：PouchContainer 利用基于 P2P 的分布式系统 [Dragonfly](https://github.com/alibaba/dragonfly) 实现企业大规模级的快速容器图像分发。
- **内核兼容性**：使得兼容 OCI 的运行同样能够在像 Linux 2.6.32+ 等旧内核版本上运行。
- **标准兼容性**：PouchContainer 不断拥抱容器生态系统来支持行业标准，比如CNI，CSI等等。
- **Kubernetes兼容性**：PouchContainer 本身实现了 Kubernetes 容器运行时接口（CRI），从其他 Kubernetes 容器运行时迁移到 PouchContainer 会很方便。